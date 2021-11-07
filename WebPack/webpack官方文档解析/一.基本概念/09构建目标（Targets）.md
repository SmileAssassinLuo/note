###Welcome to use MarkDown
  	因为服务器和浏览器代码都可以用js编写，所以webpack提供了多种构建目标（target）,你可以在webpack中设置
  	
 ##用法
 	module.exports = {
 		target:'node'
 	};
 	上面例子使用node webpack 会编译为用于类Node.js环境（使用Node.js的require，而不是使用任意内置模块（如fs或path）来加载chunk）;
 	
 	
 ##多个Target
 	webpack不支持target传入多个字符串，你可以通过打包两份分离的配置来创建同构的库
 	
 	var path = require('path');
 	var serverConfig = {
 			target:'node',
 			output:{
 				paht:path.resolve(__dirname,'dist'),
 				filename:'lib.js'
 			}
 	};
 	
 	var clientConfig = {
 		target:'web',
 		output:{
 			path:path.resolve(__dirname,'dist'),
 			filename:'lib.js'
 		}
 	};
 	module.exports = [serverConfig,clientConfig];
 	
 	
 ##打包输出比较








 	
