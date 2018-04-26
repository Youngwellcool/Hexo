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
然后在终端中可以进行打包了。你会发现图片被很好的打包了。并且路径也完全正确。
## 8、自动处理CSS3属性前缀
### 介绍
PostCSS是一个CSS的处理平台，它可以帮助你的CSS实现更多的功能，但是今天我们就通过其中的一个加前缀的功能，初步了解一下PostCSS。postcss-loader需要和autoprefixer（自动添加前缀的插件）一起使用。
### 安装
npm install  postcss-loader autoprefixer -D  (-D 是--save-dev的简写)
### 配置
postCSS推荐在项目根目录（和webpack.config.js同级），建立一个postcss.config.js文件。
postcss.config.js
```bash
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```
这就是对postCSS一个简单的配置，引入了autoprefixer插件。让postCSS拥有添加前缀的能力，它会根据 can i use 来增加相应的css3属性前缀。
对postcss.config.js配置完后，还需要在rules的配置postcss-loader
```bash
{
                test: /\.(css|less)$/,
                use: extractPlugin.extract({   // css代码分离插件
                    fallback: "style-loader",
                    use:[{
                        loader: "css-loader"
                    }, {
                        loader: "less-loader"
                    },
                        'postcss-loader',  // 对css新特性，自动补全前缀，需和插件autoprefixer（自动添加前缀的插件）一起使用，此外还需在该配置文件的同级目录下新建postcss.config.js文件，默认只会补全-webkit-前缀，如需补全其他前缀需在package.json中添加browserslist参数配置
                ],
                    publicPath:'../'    // TODO 解决分离后的css的图片url路径问题
                })
            },
```
全部配置好了后，打包，你会发现打包后的css中就只自动增加了-webkit-前缀，why?，上网查询后，原来还需要在package.json中配置browserslist参数，才会出现-ms-前缀，但是还是没有-moz-、-o-前缀。
package.json文件中 新增browserslist参数
```bash
"browserslist": [
    "defaults",
    "last 2 versions",
    "> 1%",
    "iOS 7",
    "last 3 iOS versions"
  ]
```
## 9、消除未使用的CSS
### 介绍
像Bootstrap这样的框架往往会带有很多CSS。在项目中通常我们只使用它的一小部分。就算我们自己写CSS，随着项目的进展，CSS也会越来越多，有时候需求更改，带来了DOM结构的更改，这时候我们可能无暇关注CSS样式，造成很多CSS的冗余。这节就学习用webpack消除未使用的CSS----purifycss-webpack插件
### 安装
purifycss-webpack插件要依赖于purify-css，所以这两个都需要安装
cnpm install purifycss-webpack purify-css -D
### 配置
#### 引入glob
因为我们需要同步检查html模板，所以我们需要引入node的glob对象使用。在webpack.config.js文件头部引入glob。
const glob = require('glob')
#### 引入purifycss-webpack插件
const PurifycssPlugin = require('purifycss-webpack');
#### 配置plugins
引入完成后我们需要在webpack.config.js里配置plugins。代码如下
```bash
new PurifycssPlugin({
    paths:glob.sync(path.join(__dirname,'src/*.html'))
})
```
这里配置了一个paths，主要是寻找html模板，purifycss根据这个配置会遍历你的文件，查找哪些css被使用了。
#### 总结
使用这个插件必须配合extract-text-webpack-plugin这个插件，在工作中记得一定要配置这个plugins，因为这决定你代码的质量，非常有用。
## 10、打包后的调试
### 介绍
那我们使用了webpack后，所以代码都打包到了一起，(如果使用了代码压缩的话，调试更麻烦)给调试带来了麻烦，但是webpack已经为我们充分考虑好了这点，它支持生产Source Maps来方便我们的调试。
### 配置
在使用webpack时只要通过简单的devtool配置，webapck就会自动给我们生产source maps 文件，map文件是一种对应编译文件和源文件的方法，让我们调试起来更简单。通常，devtool常用的值：source-map、cheap-module-source-map、 eval-source-map、cheap-module-eval-source-map，这四种打包模式，有左到右打包速度越来越快，不过同时也具有越来越多的负面作用，较快的打包速度的后果就是对执行和调试有一定的影响。
```bash
module.exports = {
    devtool:"#source-map"
            // "#cheap-module-source-map"
            // "#eval-source-map"
            // "#cheap-module-eval-source-map"
    ....
    ....
}
```
注意：调试在开发中也是必不可少的，但是一定要记得在打包上线前一定要修改webpack配置，关闭source map。
## 11、切换开发模式和生产模式
开发环境和生产环境所依赖的包是不同，有时候，请求的资源也是不同的，比如我们在开发的时候异步请求的是我们自己mock的数据，上线后请求的是正式的数据，这时就要能够灵活的切换开发和生产环境了。
具体设置如下：
在package.json中的“scripts”中设置requestUrl的值
```bash
"scripts": {
    "server": "webpack-dev-server",
    "dev": "set requestUrl=mockUrl&webpack",  // 将mockUrl赋值给requestUrl并执行webpack打包命令
    "buid": "set requestUrl=trueUrl&webpack"
  },
```
这样当我们在命令行中执行npm run dev我们就知道这个是开发环境的打包，npm run buid就是生产环境的打包
requestUrl是我们定义的一个node的环境变量，可以通过process.env.requestUrl来读取。
然后我们在webpack.config.js配置文件中通过if-else来判断
```bash
if(process.env.requestUrl == 'mockUrl'){  
    console.log('开发模式')
}else{
    console.log('生产模式')
}
console.log(encodeURIComponent(process.env.requestUrl)); // 读取requestUrl环境变量
```
## 12、打包第三方库
### 介绍
在工作中引用第三方的框架是必不可少的，比如引入JQuery或者Vue，该小节讲两种引入方式import的局部引入方式和webpack配置文件中ProvidePlugin插件的全局引入；以引入vue为例
### 安装vue
cnpm install vue --save
安装时需要注意的时vue最终要在生产环境中使用，所以我们这里要使用--save进行安装。
### import局部引入
在需要vue.js的文件中
import Vue from 'vue'
这里引入是不需要我们写相对路径的，因为vue的包是在node_modules里的，只要写一个包名vue，系统会自动为我们查找的。
然后
```bash
var vm = new Vue({
    el:'#app',
    data:{
        str:'Hello Vue'
    }
})
```
打包，运行，报错
Error：You are using the runtime-only build of Vue where the template compiler is not available. Either pre-compile the templates into render functions, or use the compiler-included build
上网查询，原来默认npm包导出的是 “运行时” 的vue构建(也就是说vue=vue.runtime.js,而我们需要的是vue=vue.js)，运行时构建不包含模板编译器，因此不支持 template 选项，只能用 render 选项，但即使使用运行时构建，在单文件组件中也依然可以写模板，因为单文件组件的模板会在构建时预编译为 render 函数。
上面一段是官方api中的解释。就是说，如果我们想使用template，我们不能直接在客户端使用npm install之后的vue。而应自己在webpack配置文件中手动修改vue,使其不指向运行时的构建(vue.runtime.js)，具体配置如下，在webpack配置中新增resolve参数
```bash
resolve: {
    alias: {  // 设置别名 
        'vue': 'vue/dist/vue.js'   // 设置vue=vue/dist/vue.js
    }
}
```
设置之后import Vue from 'vue'中的vue就是vue/dist/vue.js而不再是vue/dist/vue.runtime.js
局部引入就是，如果其他js中也需要vue，需在需要vue的js中import Vue from 'vue'，你还可以在任何你需要的js中引入，webpack并不会重复打包，它只给我们打包一次。
### ProvidePlugin插件全局引入
ProvidePlugin是一个webpack自带的插件，Provide的意思就是装备、提供。因为ProvidePlugin是webpack自带的插件，所以要先在webpack.config.js中引入webpack。
```bash
const webpack = require('webpack');
```
在webpack.config.js里引入必须使用require，否则会报错的，这点一定要注意。
引入成功后配置plugins模块，代码如下。
```bash
plugins:[
    new webpack.ProvidePlugin({
        Vue:"vue"  // 当然这里的vue也是需要设置alias，使其指向vue/dist/vue.js
    })
],
```
配置好后，就可以在你的入口文件中使用了，而不用再次引入了。这是一种全局的引入，在实际工作中也可以很好的规范项目所使用的第三方库。
