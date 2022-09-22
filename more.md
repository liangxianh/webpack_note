# webpack 性能优化

开发环境性能优化和生产环境性能优化

> 开发环境性能优化

* 优化打包构建速度 HMR
* 优化代码调试 source-map

> 生产环境性能优化

* 优化打包构建速度
  * oneOf
  * 缓存（babel缓存）
  * 多进程打包
  * externals（cdn引入）
  * dll （所有库被单独打包成一个文件）

* 优化代码运行性能 
  * 缓存(hash-chunkhash-contenthash
  * tree-shaking 
  * code split
  * 懒加载和预加载
  * pwa



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
  
  [inline-|hidden-|eval- 三个里面选择一个
  [nosources- 要么有要么没有
  [cheap-[module-] 要么cheap 要么cheap-module
  这些进行任意组合
  
  inline和eval代表内联 其他事外部的，内联是指source-map文件和js打包到一起，外部是map文件独立的
  生产环境是使用外部的，这样文件不会过大
  hidden（有构件后的代码）和nosources（完全隐藏） 代表隐藏源代码信息，一般用与生产环境
  cheap 只记录行（source-map默认是行和列）打包速度更快
  module loader的source map加入 打包速度会慢一点
  
  
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
  cheap-module-source-map：外部（最完整）
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

    [hash] is a "unique hash generated for every build"
    [chunkhash] is "based on each chunks' content"
    [contenthash] is "generated for extracted content"
    * hash：每次webpack构建时会生成一个唯一的hash值
    
        问题：因为js和css同时使用一个hash值，如果重新打包，会导致所有缓存失效（可我却只改动了一个文件）
        
    * chunkhash：根据chunk生成的hash值；如果打包来源于同一个chunk，那么hash值就一样
    
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
    前提： 1 必须使用es6模块化； 2 开启production环境(内部有个压缩代码会自动去除无用代码)；
    作用：减少代码体积
    
    不同版本会有差异，有的会将css/@babel/polyfill（副作用）文件干掉
    在package.json中配置
    "sideEffects": false 所有代码都没有副作用（都可以进行tree shaking）
    问题： 可能会把css/@babel/polyfill（副作用）文件干掉
    "sideEffects": ["*.css"， ] 
    
不同的设置[参考文章](https://juejin.cn/post/7002410645316436004)
在 Webpack 中，启动 Tree Shaking 功能必须同时满足三个条件：

1 使用 ESM 规范编写模块代码
2 配置 optimization.usedExports 为 true，启动标记功能
3 启动代码优化功能，可以通过如下方式实现：
 配置 mode = production
 配置 optimization.minimize = true
 提供 optimization.minimizer 数组
*/

```
7. code-split 

一般采用单入口，加spilt将文件打包成多个；（node-modules里面的文件，非node_modules里面的文件可以利用动态引入的方式这样也会进行单独的打包）

```
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

module.exports = {
  // 单入口----》单页面 单入口
  entry: './src/js/main.js',
  // 多入口：有一个入口，最终输出就有一个bundle
  // entry: {
  //   main: './src/js/main.js',
  //   test: './src/js/test.js'
  // },
  output: {
    // filename: 'index.js',
    filename: 'js/[name].[contenthash:10].js',
    // __dirname nodejs的变量，代表当前文件的目录绝对路径
    path: resolve(__dirname, 'dist')
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true
      }
    }),
    new CleanWebpackPlugin(),
  ],
  /* 
    可以将node—modules中的代码单独打包一个chunk，最终输出；
    自动分析多入口chunk中，有没有公共文件；如果有就会打包成单独的一个chunk;不会加载打包重复的东西
  */
  optimization: {
    splitChunks: {
      chunks: 'all',
    }
  },
  // 生产环境会自动压缩js代码
  mode: 'production'
  // mode: 'development'
}
```
```
// 通过js代码，让某个文件被单独打包成一个chunk
// 这样的话每次打包名字都会变话，为了不变话可以引入时加入注释
// import('./test')
import(/* webpackChunkName: 'test' */'./test').then(({mu1, count}) => {
  console.log(mu1(4,3))
}).catch(() => {
  console.log('文件加载失败～')
})
/* 
打包的结果
asset js/main.99e3b96198.js 2.58 KiB [emitted] [immutable] [minimized] (name: main)
asset index.html 332 bytes [emitted]
asset js/411.8bc86be1b1.js 179 bytes [emitted] [immutable] [minimized]

asset js/main.9bc38fe31a.js 2.58 KiB [emitted] [immutable] [minimized] (name: main)
asset index.html 332 bytes [emitted]
asset js/test.fef1ad6003.js 178 bytes [emitted] [immutable] [minimized] (name: test)
*/
```

8. 懒加载与预加载

js文件的懒加载，而不是图片的懒加载
```
document.getElementById('btn').onclick = function() {
  // 懒加载～ 需要代码codesplit
  import(/* webpackChunkName: 'test' */'./test').then(({ mu1 }) => {
    console.log(mu1(4, 2))
  })
  // 预加载prefetch：会在使用之前，提前加载js文件（有很多兼容性问题使用时慎之又慎）
  // 正常加载可以认为是并行加载（同一时间加载多个文件），预加载prefetch等其他资源加载完毕，等浏览器空闲了，在偷偷加载资源
  import(/* webpackChunkName: 'test', webpackPrefetch: true */'./test').then(({ mu1 }) => {
    console.log(mu1(4, 2))
  })
}
```

9. PWA：利用serviceworker+cache 实现的渐进式网络开发应用程序（离线可访问）；比如掘金无pwa 淘宝使用了pwa调整为无网模式淘宝可以继续访问
通过workbox库来实现 -- workbox-webpack-plugin

```
配置文件webpack.config.js 的plugin内
const WorkboxWebpackPlugin = require('workbox-webpack-plugin')
module.exports = {
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true
      }
    }),
    new CleanWebpackPlugin(),
    // pwa
    new WorkboxWebpackPlugin.GenerateSW({
      /* 
        1 帮助serviceworker快速启动
        2 删除旧的serviceworker

        生成一个serviceworker配置文件～
      */
      clientsClaim: true,
      skipWaiting: true
    })
  ],
 }

在入口文件中引入注册
/* 
pwa过程中会遇到两个问题：
1 eslint不认识 window navigator等全局变量
  解决：需要修改eslint配置
  "env": {
    "browser": true  // 支持浏览器全局变量
  }
2 sw代码必须运行在服务器上
  ---> nodejs
  ---> 
     npm i serve -g
     serve -s dist 启动服务器，将dist目录下所有资源作为静态资源暴露出去
*/
// 在入口文件中注册serviceworker
// 处理兼容性问题
if('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('./service-worker.js').then(() => {
      console.log('sw注册成功了～')
    }).catch(() => {
      console.log('sw注册失败了～')
    })
  })
}

```

10 多进程打包thread-loader与babel-loader配合使用

多进程打包，npm install -D thread-loader  ，通常js打包都是单进程的，

```
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modlues/,
        use: [
          /* 
            开启多进程打包thread-loader 是给babel-loader用的
            进程启动大概为600ms，进程通信也有开销，只有工作消耗时间比较长（js代码特别多），才需要多进程打包
          */
          // 'thread-loader', 默认是cpu核数-1的进程，也可以通过如下设置调整
          {
            loader: 'thread-loader',
            options: {
              workers: 2, // 进程2个
            }
          },
          {
            loader: 'babel-loader',
            options: {
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
            }
          }
        ]
      }
    ],
  },
```

11 externals 防止将某些包打包到最终输出的banel中,但是需要手动引入包到html中
```
  mode: 'production',
  // mode: 'development'
  /* 
    <script src="https://cdn.bootcss.com/jquery/1.12.4/jquery.min.js"></script>
    用处：比如开发时有些包需要通过cdn引入，就需要在此处设置拒绝打包，然后在html中通过script标签的形式将其引入
  */
  externals: {
    // 拒绝jQuery被打包进来
    jquery: 'jQuery'
  }
```

12 dll 动态链接库，类似externals 会指定哪些库时不参与打包的，
    dll 会对某些库（如jquery，elementui，vue，react）进行单独的打包，将多个库打包成一个chunk
    意义：将库可以拆开来打包成不同的chunk
流程：
* 1 定义webpack.dll.js对某些包单独的进行打包并生成映射文件，作用将来不需要重复打包（在manifest里面会有对应）
* 2 在主congfig中告知哪些包不需要重新打包了（查看manifest.json文件）（虽然主文件里引入了jquery）
* 3 利用add-asset-html-webpack-plugin插件自动引入（当然也可以手动引入）

只要jqury不变 webpack.dll.js只运行一次即可，反复运行congfig文件即可；这样不会将第三方库进行重复打包了；

```
分两个文件 webpack.dll.js
const { resolve } = require('path')
const webpack = require('webpack')
/* 
    dll 动态链接库，类似externals 会指定哪些库时不参与打包的
    dll 会对某些库（如jquery，elementui，vue，react）进行单独的打包，将多个库打包成一个chunk
    意义：将库可以拆开来打包成不同的chunk
    当运行webpack时，默认查找webpack.config.js文件，
    需要运行webpack.dll.js   webpack --config webpack.dll.js
*/
module.exports = {
  entry: {
    // 最终打包生成的【name】--> jquery
    // ['jquery'] --> 要打包的库时jquery
    jquery: ['jquery']
  },
  output: {
    filename: '[name].js', // 打包的库名称
    path: resolve(__dirname, 'dll'),  //路径
    library: '[name]_[hash]', // 打包的库里面向外暴露出去的内容叫什么名字
  },
  plugins: [
    // 建立依赖关系，告诉webpack将来在打包时不要在打包jquery了
    // 打包生成一个manifest.json ---》 提供和jquery映射
    new webpack.DllPlugin({
      name: '[name]_[hash]', // 映射库的暴露的内容名称
      path: resolve(__dirname, 'dll/manifest.json') // 输出文件路径
    }),
  ],
  // mode: 'production',
  mode: 'development'
}


而在webpack.config.js内也需要告诉那些不用打包了并引入
const { resolve } = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const webpack = require('webpack')
const AddAssetHtmlWebpackPlugin= require('add-asset-html-webpack-plugin')

module.exports = {
  entry: './src/js/main.js',
  output: {
    filename: 'js/[name].[contenthash:10].js',
    path: resolve(__dirname, 'dist')
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html',
      minify: {
        collapseWhitespace: true,
        removeComments: true
      }
    }),
    // 告诉webpack哪些库不参与打包，同时使用时的名称也得变
    new webpack.DllReferencePlugin({
      manifest: resolve(__dirname, 'dll/manifest.json')
    }),
    // 将某个文件打包输出出去，并在html中自动引入改资源 当内容改动时不需要重新运行dll.js 不用重复的进行打包了
    new AddAssetHtmlWebpackPlugin({
      filepath: resolve(__dirname, 'dll/jquery.js')
    })
  ],
  mode: 'production',
  // mode: 'development'
}

```













      
