###加载器(Loaders)
	js资源以外的资源加载器
	
 ##示例
 	使用loader 告诉 webpack加载CSS文件，或者将TypeScript转为javaScript，首先要安装对应的loader:
 	npm install --save-dev css-loader
 	npm install --save-dev ts-loader
 	
 	module.exports = {
 		module:{
 			rules:[
 				{test:/\.css$/,use:['css-loader'](/loaders/css-loader)},
 				{test:/\.ts$/,use:['ts-loader'](http://github.com/TypeStrong/ts-loader)}				
 			]
 		}
 	}
 	根据配置选项，如下规范定义了同等的loader用法：
 		01.{test:/\.css$/,[loader](/configuration/module#rule-loader):'css-loader'}
 		02.{test:/\.css$/,[use](/configuration/module#rule-use):'css-loader'}
 		03.{test:/\.css$/,[use](/configuration/module#rule-use):{
 			loader:'css-loader',
 			options:{}
 		}}

 ##配置
 	三种方式：
 	1.通过配置
 	2.在require语句中显示使用
 	3.通过CLI
 	
	1.简单明了，
		module:{
			rules:[
				{
					test:/\.css$/,
					use:[
						{loader:'style-loader'},
						{
						 loader:'css-loader',
						 options:{
						 	modules:true
						 }
						}
					]
				}
			]
		}

	2.通过require
		可以在require语句(或define,require.ensure,等语句)中指定loader ；
		使用!将资源中的loader分开，分开的每个部分都相对于当前目录解析；
			require('style-loader!css-loader?modules!./styles.css');
		
	
	3.通过CLI
	   可选项，
	   webpack-module-bind jade --module-bind'css=style!css'
	 对.jade文件使用jade-loader,对.css文件使用style-loader和css-loader。
	 
 ##Loader特性
	1.loader 支持链式传递。能够对资源使用流水线(pipeline)。一组链式的 loader 将按照相反的顺序执行。loader 链中的第一个 loader 返回值给下一个 loader。在最后一个 loader，返回 webpack 所预期的 JavaScript。
	2.loader 可以是同步的，也可以是异步的。
	3.loader 运行在 Node.js 中，并且能够执行任何可能的操作。
	4.loader 接收查询参数。用于对 loader 传递配置。
	5.loader 也能够使用 options 对象进行配置。
	6.除了使用 package.json 常见的 main 属性，还可以将普通的 npm 模块导出为 loader，做法是在 package.json 里定义一个 loader 字段。
	7.插件(plugin)可以为 loader 带来更多特性。
	8.loader 能够产生额外的任意文件。

 ##解析Loader
 loader 遵循标准的模块解析。多数情况下，loader 将从模块路径（通常将模块路径认为是 npm install, node_modules）解析。

loader 模块需要导出为一个函数，并且使用 Node.js 兼容的 JavaScript 编写。通常使用 npm 进行管理，但是也可以将自定义 loader 作为应用程序中的文件。按照约定，loader 通常被命名为 xxx-loader（例如 json-loader）	















