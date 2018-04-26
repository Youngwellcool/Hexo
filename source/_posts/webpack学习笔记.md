---
title: webpack学习笔记
date: 2018-04-24 20:44:08
tags:
    - 笔记
    - 心得
---
## 1、什么是webpack
WebPack可以看做是模块打包机：它做的事情是，分析你的项目结构，找到JavaScript模块以及其它的一些浏览器不能直接运行的拓展语言（Sass，TypeScript等），并将其转换和打包为合适的格式供浏览器使用。在3.0出现后，Webpack还肩负起了优化项目的责任。

这段话有三个重点：
1.打包：可以把多个Javascript文件打包成一个文件，减少服务器压力和下载带宽。
2.转换：把拓展语言转换成为普通的JavaScript，让浏览器顺利运行。
3.优化：前端变的越来越复杂后，性能也会遇到问题，而WebPack也开始肩负起了优化和提升性能的责任

## 2、安装
使用cnpm i webpack --save-dev来局部安装webpack，官方不建议全局安装webpack，“这会将您项目中，webpack 锁定到指定版本，并且在使用不同的 webpack 版本的项目中，可能会导致构建失败。”
安装完成后，可以使用 webpack -v 命令，来检测是否安装成功了。
<!--more-->
    **--save-dev 这里的参数--save是要保存到package.json中，-dev是在开发时使用这个包(将这个包放在devdependencies中)，而生产环境中不使用**

## 3、开启服务和热更新
使用 cnpm i webpack-dev-server --save-dev 来局部安装webpack-dev-server，安装完成后，需要在webpack.config.js中配置devServer参数(这是关键，不然没有热更新)
webpack.config.js文件中：
```bash
devServer:{
        //设置基本目录结构
        contentBase:path.resolve(__dirname,'dist'),
        //服务器的IP地址，可以使用IP也可以使用localhost
        host:'localhost',
        //服务端gzip压缩是否开启
        compress:true,
        //配置服务端口号
        port:1717,
        //自动打开浏览器
        open:true
    }
```
配置好了之后，命令行中输入webpack-dev-server,回车，就会按照webpack.config.js中的配置预打包，预打包输出的bundle.js会存在电脑内存中，设置open:true，会自动打开默认浏览器，到此，webpack服务就开启了。
命令行中输入 webpack,回车,就是执行正式打包了，会在dist目录中生成bundle.js文件。
(tips：如果嫌每次开启服务都输入webpack-dev-server太麻烦了，可以在package.json文件中配置“scripts”参数，具体设置如下)
package.json文件中：
```bash
"scripts": {
    "dev": "webpack-dev-server",
    "build": "webpack"  
  },
```
tips：如果你不是全局安装webpack和webpack-dev-server的，直接在命令行中输入webpack和webpack-dev-server就会报错，这时如上配置“scripts”可以解决，执行npm run dev和npm run build就相当于执行webpack-dev-server和webpack
## 4、生产环境和开发环境
* 开发环境：在开发时需要的环境，package.json中的devdependencies指在开发时需要依赖的包。
* 生产环境：程序开发完成，开始运行后的环境，package.json中的dependencies这里指要使项目运行，所需要的依赖包。

我们新建的src文件，用来存放我们开发时的JavaScript文件、css文件和html文件，这个就是我们的开发目录！
我们执行webpack命令，打包后生成的dist目录，存放的是打包后生成的JavaScript文件和html文件，这个就是我们正式的生产目录了
## 5、html自动生成插件
html-webpack-plugin插件
### 安装
使用命令cnpm i html-webpack-plugin --save-dev安装该插件，然后在webpack.config.js配置文件中先const htmlPlugin = require('html-webpack-plugin')引入该插件(注意：写在配置文件中plugins参数中的都需要先引入，而loader都不需要引入直接使用)，然后在plugins中配置该插件如下：
### 配置
```bash
new htmlPlugin({  // 打包自动生成html插件
                minify:{  // 是对html文件进行压缩，removeAttrubuteQuotes是去掉属性的双引号。
                    removeAttributeQuotes:true
                },
                hash:true, // 为了开发中js有缓存效果，所以加入hash，这样可以有效避免缓存JS。
                template:'./src/index.html' //是要打包的html模版路径和文件名称。
            }),
```
配置好后，命令行中执行webpack命令就会将src目录下的index.html文件打包到dist目录下，文件名为index.html
(tips：src目录下的index.html文件中不需要引入css和js，打包时，会根据webpack配置文件中的entry入口文件里所依赖的css和js，自动将依赖的js和css打包到dist中的index.html，会按照打包后正确的路径引入index.html中)
### 总结
html文件的打包可以有效的区分开发目录和生产目录，在webpack的配置中也要搞清楚哪些配置用于生产环境，哪些配置用于开发环境，避免两种环境的配置冲突。
## 6、CSS代码分离插件
extract-text-webpack-plugin插件
有时候不想把css代码打包进js中，因为这样会影响页面加载css样式，导致页面错乱，虽然webpack官方的宗旨是将所有请求文件打包成一个文件，以减少请求数，所有官方不建议将css分离，但是为了不影响页面渲染，还是有必要将css分离出来的。
### 安装
使用命令cnpm i extract-text-webpack-plugin@next --save-dev安装该插件，(tips：因为extract-text-webpack-plugin插件的官方正式版还没有支持到webpack4，所以安装时加上@next,[具体详情](https://segmentfault.com/q/1010000013564212)),然后在webpack.config.js配置文件中先const extractPlugin = require('extract-text-webpack-plugin')引入该插件，再开始配置
### 配置
在webpack配置中的plugins参数中：new extractPlugin('css/index.css'),参数css/index.css表示在dist目录中新建css文件夹，将分离出来的css放入index.css中。并且，相应的cssloader要改为以下这样的:
```bash
    {
        test: /\.css$/,
        use: extractPlugin.extract({   // css代码分离插件
            fallback: "style-loader",
            use: "css-loader",
            publicPath:'../'    // 解决分离后的css的图片url路径问题(这个对解决css中图片路径很关键)
        })
    }
```
然后执行webpack打包命令，在dist目录会自动生成css文件夹，index.css是分离后的css文件，因为配置了publicPath:'../'，css中引入的图片路径就正确了
### 总结
因为webpack4不支持extract-text-webpack-plugin最新版正式插件，导致引入该插件一直报错，上网找了很久的资料才找到解决办法，入了很多坑，还有该插件的publicPath配置参数对css中引入图片非常关键，不然打包后图片路径就不对了。
## 7、处理HTML中的图片
问题：我们直接在src中的index.html中用img标签引入图片，打包后，是不会把该引入的图片打包到dist目录中的，因此，浏览器中运行时就找不到该图片了。
在webpack中是不喜欢你使用标签<img>来引入图片的，但是我们作前端的人特别热衷于这种写法，国人也为此开发了一个：html-withimg-loader。这个小loader。解决的问题就是在hmtl文件中引入<img>标签的问题。
### 安装
cnpm install html-withimg-loader --save
### 配置
在webpack配置文件rules中
```bash
{
    test: /\.(htm|html)$/i,
     use:[ 'html-withimg-loader'] 
}
```
然后在终端中可以进行打包了。你会发现图片被很好的打包了。并且路径也完全正确。范德萨发
