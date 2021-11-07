###06配置(Configuration)
	很少有webpack配置看起来完全一样，这是因为webpack的配置文件是JavaScript文件导出的一个对象
	此对象，有webpack根据对象定义的属性进行解析

   	因为webpack是标准的Node.js,CommonJS模块，可以如下：
   		1.通过require(...) 导入其他文件
   		2.通过require(...) 使用npm的工具函数
   		3.通过JavaScript控制流表达式，例如 ？:操作符
   		4.对常用值使用常量或者变量
   		5.编写并执行函数来生成部分配置
   		
   		
 ##最简单的配置
 	const path = require('path');
 	
 	module.exports = {
 		entry: './foo.js',
 		output:{
 			path:path.resolove(__dirname,'dist'),
 			filename:'foo.bundle.js'
 		}
 	};
 	
 	
 ##多个Target
   	const path = require("path");
   	const webpack = require("webpack");
   	const wepackMerge = require('webpack-merge');
   	
   	const baseConfig = {
   		target:'async-node',
   		entry:{
   			entry:'./entry.js'
   		},
   		output:{
   			path:path.resolve(__dirname,"dist"),
   			filename:'[name].js'
   		},
   		plugins:[
   			new webpack.optimize.CommonsChunkPlugin({
   				name:'inline',
   				filename:'inline.js',
   				minChunks:Infinity
   			}),
   			new webpack.optimize.AggressiveSplittingPlugin({
   				minSize:5000,
   				maxSize:10000
   			})
   		]
   	};
   	
   	let targets = ['web','webworker','node','async-node','node-webkit','electron-main'].map((target)=>{
   		let base = webpackMerge(baseConfig,{
   			target:target,
   			output:{
   				path:path.resolve(__dirname,'dist/'+target),
   				filename:'[name]'+target+'.js'
   			}
   		});
   		return base;
   	
   	});
	module.exports = targets;
	


 ##使用JSX
  	使用JSX(React JavaScript标记)和Babel创建了一个webpack可以支持的JSON配置；
  	
  	import h from 'jsxobj';
  	//导入插件的示例
  	const CustomPlugin = config =>({
  		...config,
  		name:'custom-plugin'
  	});
  	
  	export default(
  		<webpack target='web' watch>
  			<entry path = 'src/index.js'/>
  			<resolve>
  				<alias {...{
  					react:'preact-compat',
  					'react-dom':'preact-compat'
  				}}/>
  			</resolve>
  			<plugins>
  				<uglify-js opts={{
  					compression:true,
  					mangle:false
  				}}/>
  				<CoustomPlugin foo='bar'/>
  			</plugins>
  		</webpack>
  	)












