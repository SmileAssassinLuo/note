#### Webpack 优化

不同项目的构建，在整个流程的前期初始化阶段与最后的产物生成阶段的构建时间区别不大。真正影响整个构建效率的还是 Compilation 实例的处理过程，这一过程又可分为两个阶段：编译模块和优化处理。

##### 一，编译阶段

###### 1，优化前的准备工作

* 准备基于时间的分析工具：我们需要一类插件，来帮助我们统计项目构建过程中在编译阶段的耗时情况，这类工具可以是尝试手写的，也可以是使用第三方的工具。例如插件  speed-measure-webpack-plugin。
* 准备基于产物内容的分析工具：从产物内容着手分析，因为从中我们可以找到对产物包体积影响最大的包的构成，从而找到那些冗余的、可以被优化的依赖项。例如插件 webpack-bundle-analyzer

###### 2，提升编译阶段的构建效率，大致可以分为四个方向：

* **减少执行编译的模块**

  * **插件 IgnorePlugin**，每次编译如果有很多模块不需要编译，可以通过 IgnorePlugin 忽略编译；比如 ‘’moment.js‘在构建时会自动引入其 locale 目录下的多国语言包,假如只需要中文;

    ![](D:\work\gitRespository\note\WebPack\img\IgnorePlugin.png)

  * **按需引入类库模块**，比如 `lodash`只引入部分方法，在导入声明时只导入依赖包内的特定模块，这样就可以大大减少构建时间，以及产物的体积。

  * **DllPlugin**：将项目依赖的框架等模块单独构建打包，与普通构建流程区分开。

    原先一个依赖 React 与 react-dom 的文件，通过 DllPlugin 和 DllReferencePlugin 分别配置后的构建时间提升。

  * **Externals**：Webpack 配置中的 externals 和 DllPlugin 解决的是同一类问题：将依赖的框架等模块从构建过程中移除。它们的区别在于：

    * 在 Webpack 的配置方面，externals 更简单，而 DllPlugin 需要独立的配置文件。

    * DllPlugin 包含了依赖包的独立构建流程，而 externals 配置中不包含依赖框架的生成方式，通常使用已传入 CDN 的依赖包。

    * externals 配置的依赖包需要单独指定依赖模块的加载方式：全局对象、CommonJS、AMD 等。

    * 在引用依赖包的子模块时，DllPlugin 无须更改，而 externals 则会将子模块打入项目包中。

* **提升单个模块构建的速度**

  * **include/exclude**：include 的用途是只对符合条件的模块使用指定 Loader 进行转换处理。而 exclude 则相反，不对特定条件的模块使用该 Loader（例如不使用 babel-loader 处理 node_modules 中的模块）。

  * **noParse**：使用 noParse 进行忽略的模块文件中不会解析 import、require 等语法。

    ```js
    module: { 
        noParse: /jquery|lodash/,
        rules: [
          {
            test: /\.js$/i,
            include: resolve('src'),
            exclude: /node_modules/,
            use: [
              // ...
            ]
          },
          // ...
        ]
      }
    ```

  * 合适的 **Source Map**：对于生产环境的代码构建而言，会根据项目实际情况判断是否开启 Source Map。在开启 Source Map 的情况下，优先选择与源文件分离的类型，例如 "source-map"。有条件也可以配合错误监控系统，将 Source Map 的构建和使用在线下监控后台中进行，以提升普通构建部署流程的速度

  * **Resolve**：resolve 配置制定的是在构建时指定查找模块文件的规则。（小项目不明显）

    ```js
    resolve: {
         // 利用别名加快查找 使用 noParse 进行忽略的模块文件中不会解析 import、require 等语法
          alias: {
            '@': resolve('src')
          }
        }
    ```

    * resolve.modules：指定查找模块的目录范围。

    * resolve.extensions：指定查找模块的文件类型范围。

    * resolve.mainFields：指定查找模块的 package.json 中主文件的属性名。

    * resolve.symlinks：指定在查找模块时是否处理软连接。

* **并行构建以提升总体效率**

  *  **thread-loader**：作用于模块编译的 Loader 上，用于在特定 Loader 的编译过程中，以开启多进程的方式加速编译。配置在 [thread-loader](https://link.juejin.cn/?target=https%3A%2F%2Fwebpack.docschina.org%2Floaders%2Fthread-loader%2F%23root) 之后的 loader 都会在一个单独的 worker 池（worker pool）中运行。

    > **注意**：实际上在小型项目中，开启多进程打包反而会增加时间成本，因为启动进程和进程间通信都会有一定开销。

  * **parallel-webpack**：多配置构建。Webpack 的配置文件可以是一个包含多个子配置对象的数组，在执行这类多配置构建时，默认串行执行，而通过 parallel-webpack，就能实现相关配置的并行处理。

* **利用缓存**：

  

  ```js
  module: { 
      noParse: /jquery|lodash/,
      rules: [
        {
          test: /\.js$/i,
          include: resolve('src'),
          exclude: /node_modules/,
          use: [
            // ...
            {
              loader: 'babel-loader',
              options: {
                cacheDirectory: true // 启用缓存
              }
            },
          ]
        },
        // ...
      ]
    }
  ```



##### 二，打包阶段

###### 1，优化阶段效率提升的整体分析

###### 2，主要方式

* 针对某些任务，使用效率更高的工具或配置项，从而提升当前任务的工作效率。
  * 压缩`js`
  * 压缩`css`
  * 清除无用的`css`:利用[purgecss-webpack-plugin](https://link.juejin.cn/?target=https%3A%2F%2Fwww.purgecss.cn%2Fplugins%2Fwebpack.html%23%E7%94%A8%E6%B3%95) 插件

* 提升特定任务的优化效果，以减少传递给下一任务的数据量，从而提升后续环节的工作效率。
  * **Split Chunks（分包）**：是指在 Chunk 生成之后，将原先以入口点来划分的 Chunks 根据一定的规则（例如异步引入或分离公共依赖等原则），分离出子 Chunk 的过程。
  * **Tree Shaking（摇树）**：是指在构建打包过程中，移除那些引入但未被使用的无效代码（Dead-code elimination）