### webpack5 入门原理

1 npm包 初始化package.json时一定注意name不要和报名重复
2 npm用来安装包的，而npx是临时的将node_module下面的bin目录作为环境变量
```
npx 会自动查找当前依赖包中的可执行文件，如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装！

使用案例：
1、使用create-react-app创建一个react项目
老方法：
npm install -g create-react-app
create-react-app my-app

新方法：
npx create-react-app my-app
这条命令临时安装create-react-app包，命令完成后create-react-app会删掉，不会出现在global中。下次执行，还会重新临时安装。

2、npx 甚至支持运行远程仓库的可执行文件：
npx github:piuccio/cowsay hello

3、指定node版本来运行npm scripts：
npx -p node@8 npm run build

谈起npx和npm的区别，虽然网上的说法是：npx是npm的升级版本。但是，苏南大叔认为：两者就是拼写比较像。命令的基本功能完全不一样，并不存在着升级不升级的说法，不是替代的关系。
npx侧重于执行命令的，执行某个模块命令。虽然会自动安装模块，但是重在执行某个命令。
npm侧重于安装或者卸载某个模块的。重在安装，并不具备执行某个模块的功能。
npx非常智能的识别模块，如果模块存在，就使用。如果不存在，就临时下载，用完就删除。
使用某个node模块的时候，根本不用关心是否安装过了。npx会给你最满意的答案（没有对应模块就临时下载）。
```
[npx介绍参考](https://newsn.net/say/npx.html)
[尚硅谷 Web 前端之 Webpack5 教程](https://yk2012.github.io/sgg_webpack5/senior/reduceVolume.html#tree-shaking)
[PWA相关介绍](https://blog.csdn.net/qq_41801117/article/details/125842543)
