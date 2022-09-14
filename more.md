# webpack 性能优化

开发环境性能优化和生产环境性能优化

> 开发环境性能优化

* 优化打包构建速度

1 HMR（开发环境，而生产环境利用缓存做相对的优化处理）
``` 
  HMR：hot module replacement 热模块替换
      作用： 一个模块发生变化，只会重新打包这一个模块，不会打包其他模块,在devServer下hot开启即可
      devServer: {
        // 开启hmr功能
        // 当修改了webpack配置。需要重启webpack
        hot: true
      }

      1 样式文件：可以使用hmr功能，因为style-loader内部实现了,在devServer下hot开启即可
      2 js文件：默认不能使用hmr功能--->需要修改js，添加支持hmr的代码
         注意：hmr功能对js的处理，只能飞入口文件
         在devServer下hot开启后还需要在入口文件内
         if(module.hot) {
           module.hot.accept('需要hot监听的文件路径如print.js', function(){
            // 方法会监听print.js，一旦发生变化，其他模块不会被重新打包构建
            // 会执行后面的回掉函数；
           })
         }
      3 html文件：默认不能使用hmr功能，同时回导致问题：html文件不能热更新了（作为仅有的一个主文件，可以不用hmr）
        解决：修改entry入口，将html文件引入
        entry: './src/index.js' ---> entry: ['./src/index.js', './src/index.html'],
        但是变化了内容整体还会重新执行
      4 vue框架在底层完成了2中js文件配置的内容，顾在devServer下hot开启即可；
```
2 source-map
```
source-map：一种提供源代码与构建代码映射的技术
  基础写法：devtool: 'source-map'
  [inline-|hidden-|eval-][nosources-][cheap-[module-]source-map
  source-map：外部
    错误代码的准确信息和源代码的错误位置， 精确到行和列
  inline-source-map：内联 只生成一个内联的source-map
    错误代码的准确信息和源代码的错误位置， 精确到行和列
  hidden-source-map：外部
     错误代码的原因，无错误位置；不能追踪源代码错误，只能提示到构建后代码的错误位置；（只隐藏源代码）
  eval-source-map：内联 每个文件都生成对应的source-map，都在eval
     错误代码的准确信息和源代码的错误位置， 精确到行和列
  nosources-source-map：外部
     错误代码的准确信息，无任何源代码信息（全部隐藏）
  cheap-source-map：外部
     错误代码的准确信息和源代码的错误位置，精确到行
  cheap-module-source-map：外部
     错误代码的准确信息和源代码的错误位置，精确到行
      module会将loader的source map加入
  1 外部生成了文件，内联没有
  2 内联构建速度更快

  开发环境： 速度快，调试更好（eval-source-map ｜｜ ceval-cheap-module-source-map）
      速度快（eval>inline>cheap）
         eval-cheap-souce-map
         eval-source-map
      调试更好
         cheap-module-source-map
         cheap-source-map
         source-map
  生产环境：源代码要不要隐藏？调试要不要更友好？(source-map)
         内联会让代码体积变大，所以生产环境不用内联
         nosources-source-map
         hidden-source-map
         ----> source-map --> cheap-module-source-map
```

* 优化代码调试

> 生产环境性能优化

* 优化打包构建速度
* 优化代码运行性能

3 vue框架在底层完成了2中js文件配置的内容，顾在devServer下hot开启即可；

4 利用oneof优化

```
module: {
  // rules里面有很多loader规则，每个文件都会走下面所有的规则去匹配，可以利用oneof来优化只执行一个loader
  // 缺点：不能同时执行两个loader处理同一种类型的文件；解决方法：可以放到外面一个
  rules: [
    {
      test: /.js$/,
      exclude: /node-module/,
      loader: 'eslint-loader',
      options: {
        // 自动修复语法
        fix: true
      }
    },
    {
      oneof: [
        {
          test: /.js$/,
          exclude: /node-module/,
          loader: "babel-loader",
          options: {
            // 预设：提示babel做怎样的兼容处理
            presets: ['@babel/preset-env']
          }
        },
        {
          test: /\.css$/,
          use: [
            MiniCssExtractPlugin.loader,
            'css-loader'
          ]
        },
      ],
    }
  ]
}
```


      
      
      
      
