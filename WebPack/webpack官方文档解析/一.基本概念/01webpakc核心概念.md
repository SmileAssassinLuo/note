### Welcome to use MarkDown

### 概念
​	webpack 是一个js应用程序的模块打包器（module bundle）;当webpack处理应用程序时，它会递归的构建一个依赖关系图，其中包含应用程序需要 的每个模块，然后将所有的模块打包成一个或者多个bundle;
​	

### 四个核心概念

+ 入口 (entry)

* 出口(output)
* loader
* 插件(plugins)

 ## 入口（entry）
 	入口起点(entry point) 提示webpack应使用那个模块，来作为其内部依赖图的开始。进入入口起点后，webpak会找出那些模块和库是入口起点（直接和间接）以来的；
 	1.简单举例
		webpack.congif.js:
		module.exports = {
			entry:'./path/to/my/entry/file.js'
		} 	
 	

 ## 出口(output)
 	output属性告诉webpack在哪里输出它所创建的bundles,以及如何命名这些文件，你可以通过在配置中指定一个output字段，来配置这些处理过程；
 	举例
 		webpack.config.js:
 		const path = require('path');
 		
 		module.exports = {
 			entry:'./path/to/my/entry/file.js',
 			output:{
 				path:path.resolve(__dirname,dist),
 				filename:'my-first-webpack.bundle.js'
 			}
 				
 		}
 	

## loader

 	 loader让webpack能够去处理那些非js文件（webpack自身只理解JavaScript）；	
 	

	在 webpack 的配置中 loader 有两个目标:
			1.识别出应该被对应的 loader 进行转换的那些文件。(使用 test 属性)
			2.转换这些文件，从而使其能够被添加到依赖图中（并且最终添加到 bundle 中）(use 属性)
		
		const path = require('path');
		const config = {
		  entry: './path/to/my/entry/file.js',
		  output: {
		    path: path.resolve(__dirname, 'dist'),
		    filename: 'my-first-webpack.bundle.js'
		  },
		  module: {
		    rules: [
		      { test: /\.txt$/, use: 'raw-loader' }
		    ]
		  }
		};
		
		module.exports = config;
		
		“嘿，webpack 编译器，当你碰到「在 require()/import 语句中被解析为 '.txt' 的路径」时，在你对它打包之前，先使用 raw-loader 转换一下。”
		
		注意:重要的是要记得，在 webpack 配置中定义 loader 时，要定义在 module.rules 中，而不是 rules。然而，在定义错误时 webpack 会给出严重的警告。为了使你受益于此，如果没有按照正确方式去做，webpack 会“给出严重的警告”

## 插件 plugins

	
		插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量;
		想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例;
		
	const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
	const webpack = require('webpack'); // 用于访问内置插件
	const path = require('path');
	
	const config = {
	  entry: './path/to/my/entry/file.js',
	  output: {
	    path: path.resolve(__dirname, 'dist'),
	    filename: 'my-first-webpack.bundle.js'
	  },
	  module: {
	    rules: [
	      { test: /\.txt$/, use: 'raw-loader' }
	    ]
	  },
	  plugins: [
	    new webpack.optimize.UglifyJsPlugin(),
	    new HtmlWebpackPlugin({template: './src/index.html'})
	  ]
	};
	
	module.exports = config;
