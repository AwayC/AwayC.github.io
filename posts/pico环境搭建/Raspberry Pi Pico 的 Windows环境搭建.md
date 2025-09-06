# Raspberry Pi Pico 的 Windows环境搭建

windows搭建往往很繁琐，网上有很多不同的搭建过程，这里只描述我的搭建

### 1.下载依赖工具

网上可能让你下这下那很多，但是这里有一个官方的[Windows installer](https://www.raspberrypi.com/news/raspberry-pi-pico-windows-installer/)，**一次安装**所有几乎你肯已经有了或者没有的依赖工具，甚至vscode中的插件都安装了，所以相当巨大，对于完全没接触开发的朋友很友好。

```
//构建工具
ninja
cmake
//编译器
gcc-arm-none-eabi
//调试工具
openocd
//...
git
python //是真的什么都有😂

pico-sdk
pico-expamle
```

### 2.设置环境变量

也可以不用设置，但是你要严格按照后面的程序，以免出错。

按win，输入path回车。。。（省略）

```
PICO_SDK_PAHT = pico-sdk的路径
```

### 3.下载project-generator

其实现在就已经可以开始建立项目了，但是每次都自己构建难免会抱怨，所以下载一个官方的[generator](https://github.com/raspberrypi/pico-project-generator)，是python编写的。

运行在他的readme里，这里直接摘抄出来

- 进入文件夹
- 输入`py pico-project.py --gui`，这样就会进入他的ui页面进行配置项目。

<img src=".\picture\generator.png" alt="example" style="zoom: 33%;" />

### 4.编译代码

按`win`，搜索`Pico - Visual Studio Code`, 务必从这个打开项目，因为这会在启动时，配置好环境，而直接从vscode打开，后面可能会出错。当然你也会发现，列表下面还会有`Pico - Developer PowerShell`, 如果后面构建命令报错的话，可以用这个打开试一下。

进入项目**删除build**（这个build是generator生成的，他本来是给unix生成的，会出错）。

然后进入到`.vscode/c_cpp_properties.json`，修改一下`"compilerPath"`成如下形式

```json
"compilerPath": "E:\\mcu_test\\picoSDK\\gcc-arm-none-eabi\\bin\\arm-none-eabi-gcc.EXE", //分割符变成 '\\'
```

最后点击最下面的 ![build](.\picture\build.png)，齿轮图标（`生成`或`build`)，最上边会选择编译器，选择`gcc-arm-none-eabi`就会重新生成build文件，构建和编译。

### 5.烧录程序

这个烧录是用过最简单的微控制器了，先**一直按住**pico上的`BOOTSEL`键，插上电脑usb，松开按键，就会有识别一个卷宗

在build目录下有一个`.uf2`文件，把他直接拖到卷宗就完成了。
