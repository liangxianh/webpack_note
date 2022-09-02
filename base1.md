## 基础学习

### 一 五个基本概念
 
1. 入口
2. 输出
3. loader
4. 插件plugin
5. 模式（开发和生产）

### 二 实践
1. 简单的大包命令
```
  开发环境 webpack % webpack ./src/index.js  -o ./dist/ --mode=development
  生产环境 webpack % webpack ./src/index.js  -o ./dist/ --mode=production
```
2. 结论：
    * webpack能处理js/json资源，但是不能处理css/img等其他资源（若处理需要引入插件）
    * 生产环境比开发环境多一个压缩js代码
    * 生产环境和开发环境将es6模块化编译成浏览器能识别的模块华；

3. 利用webpack打包css less html等资源

```
/* 
  注意安装；
  loader: 1下载 2使用 
  plugins: 1下载 2引入 3使用 
*/
// resolve用来拼接绝对路径的方法
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'main.js',
    // __dirname nodejs的变量，代表当前文件的目录绝对路径
    path: resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      // 处理css文件
      {
        // test 匹配哪些文件
        test: /\.css$/,
        use: [
          // use数组中的loader执行顺序，从右到左，从下到上，依次运行；
          // 创建style标签，将js中的样式资源插入进去，添加到head中生效
          'style-loader',
          // 将css文件变成commonjs模块加载js中，里面内容是样式字符串
          'css-loader'
        ]
      },
      // 处理less文件
      {
        test: /\.less$/,
        use: [
          'style-loader',
          'css-loader',
          // 将less文件编译为css文件
          'less-loader',
        ]
      }
    ]
  },
  plugins: [
    // html-webpack-plugin 默认会创建一个空的html，自动引入打包输出的所有资源（js/css）
    new HtmlWebpackPlugin({
      // 复制./src/index.html文件，自动引入打包输出的所有资源（js/css
      template: './src/index.html'
    })
  ],
  // mode: 'production'
  mode: 'development'
}
```

4. 利用webpack打包图片资源
```
      // 处理img资源
      {
        // 问题：处理不了html中的img 一定注意此格式 /\.(jpg|jpeg|gif|png)|/ 之前写成了这个样子导致图片没有被加载
        test: /\.(jpg|jpeg|gif|png)$/,
        // 下载url-loader 和file-loader 因为url-loader依赖file-loader
        loader: 'url-loader',
        options: {
          /* 
           图片大小小于8k，就会被base64处理
           优点：减少请求数量（减轻服务器压力）
           缺点：图片体积会更大（文件请求速度更慢）
          */
          // limit: 8 * 1024,
          // 问题：因为url-loader默认使用es6的模块化解析，而html-loader引入图片是commonjs
          // （不做处理）解析时会出问题<img src="[object Module]" alt="angular">
          // 解决：关闭url-loader的es6模块化，使用commonjs解析  esModule: false
          // esModule: false,
          // [hash:10]取图片hash的💰10位
          // [ext]取文件原来的扩展名
          name: '[hash:10].[ext]'
        },
        // type: "javascript/auto"
      },
      /* 
        若不设置解析html里面的图片就会报下面的错误，找不到当前文件
        GET file:///Users/name/Desktop/code/mywork/webpack/webpack-test3/dist/b47.jpg net::ERR_FILE_NOT_FOUND
      */
      // 处理html内部图片html内以img引入的内容
      {
        test: /\.html$/,
        // 处理html文件的img图片（负责引入img，从而能被url-loader进行处理）
        loader: 'html-loader'
      }
```
根据webpack的官方文档，可知，webpack5以后，使用资源模块（asset module)，它允许使用资源文件，而无需配置额外的loader。详情如下

> 何为 webpack 模块
 
与 Node.js 模块相比，webpack 模块 能以各种方式表达它们的依赖关系。下面是一些示例：

* ES2015 import 语句
* CommonJS require() 语句
* AMD define 和 require 语句
* css/sass/less 文件中的 @import 语句。
* stylesheet url(...) 或者 HTML <img src=...> 文件中的图片链接。


webpack5 支持jpeg｜webp｜svg格式的图片么 直接引用都可以
```
webpack.config.js
module rules设置如下：
      /* 
        若不设置解析html里面的图片就会报下面的错误，找不到当前文件
        GET file:///Users/name/Desktop/code/mywork/webpack/webpack-test3/dist/b47.jpg net::ERR_FILE_NOT_FOUND
      */
      // 处理html内部图片  
      {
        test: /\.html$/,
        // 处理html文件的img图片（负责引入img，从而能被url-loader进行处理）
        loader: 'html-loader'
      }


html

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>html css less img</title>
</head>
<body>
  <div class="webpack-container">
    <h1 id="title">hello webpack</h1>
    <!-- img -->
    <div class="box1">11111</div>
    <div class="box2">22222</div>
    <div class="box3">33333</div>
    <div class="box4">44444</div>
    <img src="./a6.jpeg"></img>
    <img src="./b47.jpg"></img>
    <img src="./active.svg"></img>
    <img src="./15.webp"></img>
  </div>
</body>
</html>


less 引入图片不用做任何设置即可（但是处理less的配置还是要的）

.box1 {
  width: 100px;
  height: 100px;
  background-image: url('./a6.jpeg');
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
.box2 {
  width: 200px;
  height: 200px;
  background-image: url('./b47.jpg');
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
.box3 {
  width: 300px;
  height: 300px;
  background-image: url('./c77.jpg');
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
.box4 {
  width: 400px;
  height: 400px;
  background-image: url('https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg-blog.csdnimg.cn%2F20210107105646524.png%3Fx-oss-process%3Dimage%2Fwatermark%2Ctype_ZmFuZ3poZW5naGVpdGk%2Cshadow_10%2Ctext_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xsdzY4Njg%3D%2Csize_16%2Ccolor_FFFFFF%2Ct_70&refer=http%3A%2F%2Fimg-blog.csdnimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1664671932&t=4e83c9c241c2817bef0bfba4b7b13210');
  background-repeat: no-repeat;
  background-size: 100% 100%;
}
```
