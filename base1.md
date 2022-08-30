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
