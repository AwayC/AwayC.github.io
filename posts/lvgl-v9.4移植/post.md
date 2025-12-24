# LVGL-v9.4 移植到Clion项目

配置环境: 

- IDE: Clion
- 构建工具：CMake
- 硬件库： HAL
- 平台：stm32
- 显示设备：tft-LCD ST7789

> [!WARNING]
>
> 注意屏幕型号是否匹配，否则驱动不同

## Stm32CubeMx引脚配置

1. 开启SPI： 开启DMA， `Peripheral`和`Memory`设置为`Half Word`(否则GRAM会出错)

    <img src="posts\lvgl-v9.4移植\img\DMA.png">

2. 配置LCD引脚： LCD_RESET（重置, 可选）, LCD_BL（背光灯）,LCD_DCX, LCD_CS 

    

## 下载lvgl-v9.4

在写这篇博客时是最新版本

下载地址[github]([lvgl/lvgl at release/v9.4](https://github.com/lvgl/lvgl/tree/release/v9.4))

解压到本地，在clion项目中创建一个文件夹`lvgl`, 把解压后的所有文件拷贝到新文件夹中。

<img src="posts\lvgl-v9.4移植\img\tree.png">

## 配置CMake

修改include目录

```cmake
# 原本
include_directories(Core/Inc Drivers/STM32F4xx_HAL_Driver/Inc ....)
# 修改
include_directories(
        lvgl/
        lvgl/src
        Core/Inc Drivers/STM32F4xx_HAL_Driver/Inc ...)
```

加入到源文件中

```cmake
#原本
file(GLOB_RECURSE SOURCES "Core/*.*" "Drivers/*.*")
#修改
file(GLOB_RECURSE SOURCES "Core/*.*" "Drivers/*.*" "lvgl/src/*.*")
```

重新加载CMake

## 修改config和配置LCD

进入`lvgl`文件夹

找到`lv_conf.h`

```c
/* clang-format off */
// 这里修改成 #if 1
#if 0 /* Set this to "1" to enable content */

#ifndef LV_CONF_H
#define LV_CONF_H

...

/* Drivers for LCD devices connected via SPI/parallel port */
#define LV_USE_ST7735        0
#define LV_USE_ST7789        1 //开启对应lcd芯片的使能，这里用的是st7798
#define LV_USE_ST7796        0
#define LV_USE_ILI9341       0
#define LV_USE_FT81X         0
#define LV_USE_NV3007        0
```



新版lvgl提供了LCD的驱动，不需要我们去提供了，只需要配置一下接口，甚至有stm32的模板代码

在`lvgl/expamles/porting`，分别复制`lv_port_lcd_stm32_template.c`, `lv_port_lcd_stm32_template.h` 到`Core/Inc`和`Core/Src`中

，并把名称和文件中的template去掉

`.h`中

```c
/*Copy this file as "lv_port_disp.h" and set this value to "1" to enable content*/
#if 1 //开启使能

...
    
/*********************
 *      INCLUDES
 *********************/
#if defined(LV_LVGL_H_INCLUDE_SIMPLE)
#include "lvgl.h"
#else
#include "lvgl.h" //修改路径
#endif
```

`.c`中

```c
// 修改对应屏幕分辨率
#ifndef MY_DISP_HOR_RES
    #define MY_DISP_HOR_RES    240
#endif

#ifndef MY_DISP_VER_RES
    #define MY_DISP_VER_RES    240
#endif

//导入头文件
#include "main.h"
#include "stm32f4xx_hal.h"
#include "spi.h"

/*
 修改对应spi，删改GPIO
 好像源码的lv_port_display_init 函数写错了，修改为lv_port_disp_init
 */

static int32_t lcd_io_init(void)
{
    /* Register SPI Tx Complete Callback */
    // 这是新版的申请回调的函数，没更新可能没有，要注释
    HAL_SPI_RegisterCallback(&hspi3, HAL_SPI_TX_COMPLETE_CB_ID, lcd_color_transfer_ready_cb);
	
    /* reset LCD */
    // 没有RESET，就注释了
    // HAL_GPIO_WritePin(LCD_RESET_GPIO_Port, LCD_RESET_Pin, GPIO_PIN_RESET);
    // HAL_Delay(100);
    // HAL_GPIO_WritePin(LCD_RESET_GPIO_Port, LCD_RESET_Pin, GPIO_PIN_SET);
    // HAL_Delay(100);

    HAL_GPIO_WritePin(LCD_CS_GPIO_Port, LCD_CS_Pin, GPIO_PIN_SET);
    HAL_GPIO_WritePin(LCD_DC_GPIO_Port, LCD_DC_Pin, GPIO_PIN_SET);

    return HAL_OK;
}

/*
 没有上面的申请函数，加入下方回调
 */
void HAL_SPI_TxCpltCallback(SPI_HandleTypeDef *hspi) {
    if (hspi == &hspi3) {
        lcd_color_transfer_ready_cb(&hspi3);
    }
}

```

吐槽： 老板HAL库的回调设计的是依托答辩，新版才是正常设计



## main.c 代码

```c
...
    
/* USER CODE BEGIN Includes */
#include "lvgl.h"
#include <stdint.h>
#include <stdbool.h>
#include "display/lv_display.h"
#include "lv_port_lcd_stm32.h"
/* USER CODE END Includes */

...
    
    
/* USER CODE BEGIN PFP */
static void hal_init();
static void delay_ms(uint32_t ms);
static void tick_init();

/* USER CODE END PFP */

...
int main(void) { 
	...
        
    /* USER CODE BEGIN 2 */
    lv_init();
    hal_init();

    lv_obj_t *label = lv_label_create(lv_screen_active());
    lv_label_set_text(label, "Hello World!");
    lv_obj_center(label);
    /* USER CODE END 2 */
    
    while(1) { 
    	uint32_t time_till_next = lv_timer_handler(); 
        delay_ms(time_till_next); 
    }
}

...
/* USER CODE BEGIN 4 */
static void hal_init() {
  tick_init();
  lv_port_disp_init();
    
  //开启背光灯
  HAL_GPIO_WritePin(LCD_BL_GPIO_Port, LCD_BL_Pin, GPIO_PIN_SET);
}


static void delay_ms(uint32_t ms) {
  HAL_Delay(ms);
}

static void tick_init() {
  // 设置系统1ms时钟
  HAL_SYSTICK_Config(SystemCoreClock / 1000);
}
/* USER CODE END 4 */
    
```

## 最后添加心跳
在`stm32fxxx_it.c`中添加心跳函数, 在f4中就是`stm32f4xx_it.c`

```c
void SysTick_Handler(void)
{
  /* USER CODE BEGIN SysTick_IRQn 0 */

  /* USER CODE END SysTick_IRQn 0 */
  HAL_IncTick();
  /* USER CODE BEGIN SysTick_IRQn 1 */
  // 调用lvgl的心跳函数
  lv_tick_inc(1);
  /* USER CODE END SysTick_IRQn 1 */
}
```