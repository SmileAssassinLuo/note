### 入口起点
​	在webpack配置中有多种方式定义entry属性：
   ## 方式一：单个入口语法
 	用法：entry:string|Array<string>
 	参数是：string
 		const config = {
 			entry:'./path/to/my/entry/file.js'
 		}
		module.exports = config;
	

	参数是：Array：
	当向entry 传入一个数组时，会发生什么呢？
 	将创建“多个主入口(multi-main entry)”;
 	好处：可以实现在你想要多个依赖文件一起注入，并且将他们的依赖导向(graph)到一个"chunk"时。
 		
   ## 方式二：对象语法
 	用法:entry :{[entryChunkName:string]:string|Array<string>}
 		const config = {
 			entry:{
 				app:'./src/app.js',
 				vendors:'./src/vendors.js'
 			}
 		}
 		备注：这种"可扩展的webpack配置"是指，可重用并且可以与其他配置组合使用，这是一种流行的技术，用于将关注点(concern)从环境(environment),构建目标(build target),运行时(runtime)中分离，然后使用专门的工具(如webpack-merge)将他们合并。
 		
 		
### 常见场景
   	##分离应用程序(app)和公共库(vendor)入口
   		const config = {
 			entry:{
 				app:'./src/app.js',
 				vendors:'./src/vendors.js'
 			}
 		}
 		
 	是什么?
 	
 	为什么？此设置允许你使用CommonsChunkPlugin从应用程序bundle中提取vendor引用(vendor reference) 到 vendor bundle，并把vendor引用的部分替换为 __webpack_require__() 调用；
 	如果应用程序bundle中没有vendor代码，那么你可以在webpack中实现被称为长效缓存的通用模式；
 	
 	
 	#### 多个页面应用程序
 		const config = {
 			entry:{
 				pageOne:'./src/pageOne/index.js',
 				pageTwo:'./src/pageTwo/index.js',
 				pageThree:'./src/pageThree/index.js'
 			}
 		} 
 		
 	是什么？告诉webpack 需要3个独立分离的依赖表；
 	为什么？在多页面应用中，服务器将为你获取一个新的HTML文档，页面重新加载新文档，并且资源被重新下载。这给了特殊的机会去做很多事:
 		1.使用CommonsChunkPlugin 为每个页面间的应用程序共享代码创建bundle。由于入口起增多，多页应用能够在入口起点重用大量代码/模块；
 	

​			Tip：根据经验：每个 HTML 文档只使用一个入口起点。
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 		
 	