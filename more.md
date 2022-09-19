# webpack 性能优化

开发环境性能优化和生产环境性能优化

> 开发环境性能优化

* 优化打包构建速度

1 HMR（开发环境，而生产环境利用缓存做相对的优化处理）
``` 
  HMR：hot module replacement 热模块替换是基于devserver的
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
     /* 
        语法检测，eslint-loader eslint
        注意：只检测自己的源代码，第三方的库是不用检查的
        设置检查规则：
        package.json中eslintConfig中设置 继承airbnb-base
        "eslintConfig": {
          "extends": "airbnb-base"
        }
        airbnb --> eslint-config-airbnb-base eslint eslint-plugin-import
        若是在js中某行代码不想要被检测，可以使用eslint-disable-next-line
        表示下一行eslint所有的规则都失效；
      */
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
        /* 
          js兼容性处理：babel-loader @babel-core @babel/preset-env
           1 @babel/preset-env 
           问题：只能转换基本语法，如promise高级语法不能转换
           2 全部js兼容处理 ---> 使用@babel/polyfill 在js文件里面直接引入即可import '@babel/polyfill'
           问题：只要部分兼容处理时，会将所有的兼容性代码全部引入，体积太大了
           3 需要做兼容性处理的就做：按需加载 ===》core-js
             下载，配置
        */
        {
          test: /.js$/,
          exclude: /node-module/,
          loader: "babel-loader",
          options: {
            // 预设：提示babel做怎样的兼容处理
            presets: [
              '@babel/preset-env',
              {
                // 按需加载
                useBuiltIns: usage,
                // 指定corejs版本
                corejs: {
                  version: 3
                },
                // 指定兼容性具体做到哪个版本
                targets: {
                  chrome: '60',
                  firefox: '50'
                }
              }
            ]
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

5. 缓存

  > babel缓存 （针对js兼容性的缓存）


    cacheDirectory: true
    --> 让第二次打包构建速度更快
    node代码可以设置max-age来确定缓存的时间

    为了避免有内容更新，而不能及时更新到远程，可以使用hash处理，hash有以下三种形式

  > 文件资源缓存


    * hash：每次webpack构建时会生成一个唯一的hash值
    
        问题：因为js和css同时使用一个hash值，如果重新打包，会导致所有缓存失效（可我却只改动了一个文件）
        
    * chunkhash：更具chunk生成的hash值；如果打包来源于同一个chunk，那么hash值就一样
    
        问题：js和css的hash值还是一样
          因为css是在js中被引入的所以同属于一个chunk
          
    * contenthash：根据文件的内容生成hash值，不同文件的hash值一定不一样；        
      -->让代码上线运行缓存更好使用
```
       output: {
          // filename: 'index.js',
          // filename: 'js/index.[hash:10].js',
          // filename: 'js/index.[chunkhash:10].js',
          filename: 'js/index.[contenthash:10].js',
          // __dirname nodejs的变量，代表当前文件的目录绝对路径
          path: resolve(__dirname, 'dist')
        },

        ***
        
        {
          test: /.js$/,
          exclude: /node-module/,
          loader: "babel-loader",
          options: {
            // 预设：提示babel做怎样的兼容处理
            presets: [
              '@babel/preset-env',
              {
                // 按需加载
                useBuiltIns: usage,
                // 指定corejs版本
                corejs: {
                  version: 3
                },
                // 指定兼容性具体做到哪个版本
                targets: {
                  chrome: '60',
                  firefox: '50'
                }
              }
            ],
            // 开启babel缓存
            // 第二次构建时，会读取之前的
            cacheDirectory: true
          }
        },

```

6. tree shaking去除无用代码

```
/* 
    tree shaking（树摇）去除无用代码(可能是js/css代码)
    （比如，js定义的常用方法，只引入部分的 ，就只打包部分的）
    前提： 1 必须使用es6模块化； 2 开启production环境；
    作用：减少代码体积
    
    不同版本会有差异，有的会将css/@babel/polyfill（副作用）文件干掉
    在package.json中配置
    "sideEffects": false 所有代码都没有副作用（都可以进行tree shaking）
    问题： 可能会把css/@babel/polyfill（副作用）文件干掉
    "sideEffects": ["*.css"， ] 
*/

```
7. 

      
      
