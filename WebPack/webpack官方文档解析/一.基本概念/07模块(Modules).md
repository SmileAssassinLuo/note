###07模块(Modules).md
   	在模块化编程，开发者将程序分解成称为模块的离散功能。
   	
   	模块化是程序设计的未来趋势，精心编写的模块能更好的完成迭代和代码重用。
   	
 ##什么是webpack模块？
	对比Node.js模块，webpack模块能够以各种方式表达他们的依赖关系，几个例子如下：
		1.ES2015 import语句
		2.CommonJS require()语句
		3.AMD define和require语句
		4.css/sass/less文件中的@import语句
		5.样式(url(...))或者HTML文件(<img src=...>)中的图片链接(image url)
		
 ##支持的模块类型
 	webpack通过loader可以支持各种语言和预处理器编写模块，loader描述了webpack如何处理非JavaScript模块，并且在bundle中引入这些依赖。
 	webpack社区已经为各种流行语言和语言处理器构建了loader，包括：
 		1.CoffeeScript
 		2.TypeScript
 		3.ESNext(Babel)
 		4.Sass
 		5.Less
 		6.Stylus
 		
###模块解析(Module Resolution)	
	解析器是一个通过绝对路径来帮助定位模块的库(library),一个模块可以作为另一个模块的依赖，然后被后者引用，例如：
	import foo from 'path/to/module'
	
	//or
	require('path/to/module')
	
 ##webpack中的解析规则
 	使用enhanced-resolve webpack能够解析三种文件路径：
 		1.绝对路径
 		2.相对路径
 		3.模块路径
 		
 		
 ##解析加载器
 
 
 	
 	
 	

 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
 	
