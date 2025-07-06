### 我是谁

Away

<hr>

### 我在哪

hdu

<hr>

### 在干啥

瞎折腾

<hr>


#### 下面讲讲关于这个博客的搭建

1. 初衷： 

    - 平时会浏览其他大佬的技术博客，我也希望把我学到的东西或多或少的记录下来，作为成长的见证。
    - 还有就是享受搭建博客的快乐，学习进步。

2. 搭建前： 

    - 首先搭建主要有两个选择，一个是选择网上的博客模板（大多精美，功能完善），二是自己从零开始设计。我选择了后者，主要是没有了博客独我的感觉。
    - 学习技术栈： 三大件（`html, css, js`)， 框架（可以不用，这里我选了`vue`)，原本博客可以在上个寒假完成，但是没有学习框架，导致难度对于我有点大了，就先搁置了。

3. 搭建： 
    - 就是按照自己现在简单的需求搭建了一下，主要post列表，和post页面。让后为了后续方便添加文章，文章选用`markdown`编写，再通过js的`marked`库来转成html渲染到页面中。
    - 主页也是添加了一个交互动画（没什么用，说实话）。

4. 部署：

    - 配置好`vue.config.js`文件，这里是我的示例，这个有点头疼搞了好久，尤其是这个路径。

        ```js
        // vue.config.js
        const { defineConfig } = require('@vue/cli-service')
        
        module.exports = defineConfig({
          transpileDependencies: true,
        
          // 生产环境：如果是 user/org page (AwayC.github.io)，这里应该是 '/'
          // 如果是 project page (AwayC.github.io/your-repo)，这里是 '/your-repo/'
          publicPath: process.env.NODE_ENV === 'production'
            ? '/' 
            : './',
        
          outputDir: 'docs', 
        
          chainWebpack: config => { //引入markdown的依赖（我也不懂，反正就要）
            config.module
              .rule('markdown')
              .test(/\.md$/)
              .use('raw-loader')
              .loader('raw-loader')
              .end();
          }
        });
        ```

        

    - 把vue项目编译压缩成用于生成环境的文件，就是转换成`index.html，js, css`文件

        ```bash
        npm run build
        ```
        
        或者在`vue ui`里build
        
        最后build出来的文件在`dist`文件夹里。
        
        目录示例
        
        ```plainText
        docs/
        ├── css/
        │   ├── app.e9f07865.css
        │   └── chunk-vendors.6039dd2d.css
        ├── img/                # 静态图片资源
        ├── js/
        │   ├── 336.e06cc517.js
        │   ├── 641.d8b0671c.js
        │   ├── app.3b33056c.js
        │   └── chunk-vendors.09369b76.js
        └── index.html          # 入口文件
        
        ```
        
        
        
    - 在github里创建一个仓库， `name`格式是`{$userName}.github.io`
    
    - 把dist里的文件取出来放一个文件夹里，在git push到刚刚创建的仓库，就可以了。
    
    - 最后你就可以访问`https://{$userName}.github.io`去访问自己的网站了。



