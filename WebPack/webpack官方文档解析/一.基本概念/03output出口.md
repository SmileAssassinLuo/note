###输出Output
	output选项控制webpack如何向硬盘写入编译文件，注意，及时可以存在多个入口起点，但只能指定一个输出配置；
	如果你使用了哈希（[hash]或者[chunkhash]），请确保模块具有一致的顺序，可以使用OccurrenceOrderPlugin 或 recordsPath;
	
   ##用法(Usage)
   	在webpack中配置output属性的最低要求是，将它的值设置为一个对象，包括以下两点：
   	编译文件的文件名(filename),首选推荐：//main.js||bundle.js||index.js;
   	output.path 对应一个绝对路径，此路径是你希望一次性打包的目录
   		const config = {
   			output:{
   				filename:"bundle.js",
   				path:'./home/proj/public/assets'
   			
   			}
   		}
   	 module.exports = config;
   	 
   	 
   ##选项(Options)
     1.output.chunkFilename
  	 非入口的chunk(non-entry chunk)的文件名，路径相对于output.path目录
  	[id] 被 chunk 的id替换;
  	[name]被 chunk的name替换(或者在chunk没有name时使用id替换);
  	[hash]被compilation生命周期的hash替换;
  	[chunkhash]被chunk的hash替换;	
 
 	 2.output.crossOriginLoading
 	 此选项启用跨域加载(cross-origin loading) chunk
 	 可选的值有：
 	 	false --禁用跨域加载
 	 	"anonymous" - 启用跨域加载，当使用anonymous时，发送不带凭据(credential)的请求；
 	 	"use-credential"-启用跨域加载，当使用use-credential时，发送                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                带凭据(credential)的请求；

	3. output.devtoolLineToLine
   	所有/指定模块启用行到行映射(line-to-line mapped)模式. 行到行映射使用一个简单的SourceMap，即生成资源(generated source)的每一行都映射到原始资源(original source)的同一行。
   	 true  在所有模块中启用 （不推荐）；
	 {test,include,exclude}对象，对特定文件启用（类似于module.loaders）；

	4.output.filename
   	指定硬盘每个输出文件的名称;在这里不能指定为绝对路径，output.path选项规定了文件被写入硬盘的位置，filename仅用于命名每个文件；
   	单个入口
   		{
   			entry :"./src/app.js",
   			output:{
   				filename:'bundle.js',
   				path:__dirname+"/build"		
   			}
   		}
   		//写入到硬盘:./build/bundle.js
   		
   	多个入口
   		如果你的配置创建了多个"chunk"(例如使用多个入口起点或使用类似CommonsChunkPlugin的插件),你应该使用以下的替换方式来确保每个文件名都不重复
   	[name]被 chunk的name替换;
  	[hash]被compilation生命周期的hash替换;
  	[chunkhash]被chunk的hash替换;	
   		
   		{
   			entry:{
   				app:'./src/app.js',
   				search:"./src/search.js"
   			},
   			output:{
   				filename:"[name].js",
   				path:__dirname+'/build'
   			}
   		}
	//写入到硬盘：./build/app.js,./build/search.js
	
	
	5.oputput.hotUpdateChunkFilename
 	 热更新chunk(Hot Update Chunk)的文件名，他们在 output.path目录中。
 	 [id] 被 chunk 的id替换;
 	 [hash]被compilation生命周期的hash替换;
 	 
 	6.output.hotUpdateFunction
 	webpack中用于异步加载(async load)热更新(hot update)chunk的JSONP函数
 	 默认值：“webpackHotUpdate”
 	 
 	7.output.HotUpdateMainFilename
 	 热更新主文件(hot update main file)的文件名;
 	 默认值：“[hash].hot-update.json”
 	 
 	8.output.jsonpFunction
 	  webpack中用于异步加载(async load)chunk的JSONP函数
 	  较短的函数可能减少文件大小。当单页有多个webpack实例时，请使用不同的标识符(identifier);
 	  
 	9.output.library
 	如果设置此选项，会将bundle导出为library。 output.library是library的名称。
 	当你正在编写library，并且需要将其发布为单独的文件时，请使用此选项。
 	
 	10.output.lisraryTarget
 	 library的导出格式
 	 （具体查阅相关文档）
 	 
 	11.output.path
 	 导出目录为绝对路径
 	[hash]被compilation生命周期的hash替换;
 	js:config.js
 	output:{
 		path:'./home/proj/public/assets',
 		publicPath:'assets'
 	}
 	
 	html:index.html
 		<head>
 			<link href='assets/spinner.gif' />
 		</head>
 	
 	
 	下一个例子，说明对资源使用CDN和hash
 	config.js
 		opuput:{
 			path:'/home/proj/cdn/assets/[hash]',
 			publicPaht:"http://cdn.example.com/assets/[hash]"
 		}
 	 注意：在编译不知道最终输出文件的publicPath的情况下，publicPath可以留空，并且在入口起点文件运行时动态设置，设置_webpack_public_path_;
 	 _webpack_public_path_ = myRuntimePublicPath
 	 //其他的应用程序入口
 	 
 	12.outut.sourceMapFilename
 	js文件的SourceMap的文件名，它们在output.path目录中
 	[file]被javaScript文件的文件名替换;
 	[id]被chunk的id替换;
  	[hash]被compilation生命周期的hash替换;
  	默认值“[file].map”
  	
 	
 	
 	
 	
 	
 	
 	
 	
 	








