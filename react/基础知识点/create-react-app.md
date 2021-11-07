### create-react-app

一.Create React App 是一个官方支持的创建 React 单页应用程序的方法。

​		全局安装 npm install -g create-react-app  / cnpm install -g create-react-app

二.它提供了一个零配置的现代构建设置,以下三种方式可以创建一个官方的新应用;

1. npx create-react-app my-app  *([npx](https://medium.com/@maybekatz/introducing-npx-an-npm-package-runner-55f7d4bd282b) 来自 npm 5.2+*)

   ``` 
   npx介绍：
   	npm v5.2.0引入的一条命令（npx），引入这个命令的目的是为了提升开发者使用包内提供的命令行工具的体验。
   详情：
   	http://www.phonegap100.com/thread-4910-1-1.html
   
   	npx create-react-app reactdemo这条命令会临时安装 create-react-app 包，命令完成后create-react-app 会删掉，不会出现在 global 中。下次再执行，还是会重新临时安装。
   	npx 会帮你执行依赖包里的二进制文件。
   
   	再比如 npx http-server 可以一句话帮你开启一个静态服务器
   ```

2. npm init react-app my-app 

3. yarn create react-app my-app

   ``` 
   
   ```

   

三.生成如下结构的项目

    ``` javascript
my-app
├── README.md
├── node_modules
├── package.json
├── .gitignore
├── public
│   ├── favicon.ico
│   ├── index.html
│   └── manifest.json
└── src
    ├── App.css
    ├── App.js
    ├── App.test.js
    ├── index.css
    ├── index.js
    ├── logo.svg
    └── serviceWorker.js
    ```

