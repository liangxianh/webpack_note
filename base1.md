## 基础学习

### 一 五个基本概念
 
1. 入口
2. 输出
3. loader
4. 插件plugin
5. 模式（开发和生产）

### 二 实践
1 简单的大包命令
  开发环境 webpack % webpack ./src/index.js  -o ./dist/ --mode=development
  生产环境 webpack % webpack ./src/index.js  -o ./dist/ --mode=production
2 结论：
    1. webpack能处理js/json资源，但是不能处理css/img等其他资源（若处理需要引入插件）
    2。 生产环境比开发环境多一个压缩js代码
    3。 生产环境和开发环境将es6模块化编译成浏览器能识别的模块华；
