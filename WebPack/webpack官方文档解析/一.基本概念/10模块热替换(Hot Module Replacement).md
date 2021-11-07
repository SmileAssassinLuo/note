###10模块热替换(Hot Module Replacement).md
	模块热替换功能会在应用程序运行过程中替换，添加或者删除模块，而无须重新加载页面，这使得你可以在独立模块变更后，无需刷新整个页面，就可以更新这些模块，提高开发效率
	

 ##如何运行?
 	##APP角度
 		1.app代码要求HMR runtime 检查更新
 		2.HMR runtime(异步)下载更新，然后通过app代码更新可用
 		3.app代码要求HMR runtime应用更新
 		4.HMR runtime(异步)应用更新
 		
 	可以设置HMR ，使此进程自动触发更新，或者你可以选择要求在用户交互后进行更新；
 	
 	
 	##站在编译器(webpack)的角度
 		除了普通资源，编译器(compiler)需要发出"update",以允许更新之前的版本到新的版本；
 		‘update’由两部分组成：
 			1.待更新manifest(JSON);
 			2.一个或多个待更新chunk(JavaScript)
 		manifest包括新的编译hash和所有的待更新chunk目录；
 		
 	##站在模块角度
 		HMR是可选功能，只会影响包含HMR代码的模块
 		
 		
 	##站在HMR Runtime的角度
 	
 	
 	
 	
 ##通过HMR能做什么？
 	
