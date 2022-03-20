#### vue-cli

##### 一，安装,升级

安装

```
npm install -g @vue/cli
# OR
yarn global add @vue/cli
```

查看是否安装成功

```
vue --version
```

升级

```
npm update -g @vue/cli

# 或者
yarn global upgrade --latest @vue/cli
```

**项目依赖**

上面列出来的命令是用于升级全局的 Vue CLI。如需升级项目中的 Vue CLI 相关模块（以 `@vue/cli-plugin-` 或 `vue-cli-plugin-` 开头），请在项目目录下运行 `vue upgrade`：

```
用法： upgrade [options] [plugin-name]

（试用）升级 Vue CLI 服务及插件

选项：
  -t, --to <version>    升级 <plugin-name> 到指定的版本
  -f, --from <version>  跳过本地版本检测，默认插件是从此处指定的版本升级上来
  -r, --registry <url>  使用指定的 registry 地址安装依赖
  --all                 升级所有的插件
  --next                检查插件新版本时，包括 alpha/beta/rc 版本在内
  -h, --help            输出帮助内容
```

##### 二，创建项目

使用 **git bash** 通过命令`vue create project-name` 新建项目;

![](D:\work\gitRespository\note\vue\img\vue-cli-new-project.png)

弹出如下交互，使用键盘箭头键选择一个预设,比如选择手动配置选择功能，再按下 **Enter**键；

![](D:\work\gitRespository\note\vue\img\vue-cli-pick-preset.png)

***箭头键无效，则需要如下处理：***在git安装目录找到 `bash.bashrc`文件，文件末尾添加 `alias vue='winpty vue.cmd'`;重启`git bash`;

<img src="D:\work\gitRespository\note\vue\img\vue-cli箭头问题.png" style="zoom:67%;" />



选择预设默认 vue2 和 vue3 ,自动加载脚手架配置；

`Manually select features`：提供可选的配置：

![](D:\work\gitRespository\note\vue\img\vue-cli-manully-select-features.png)

按照提示即可选择所需配置；

安装完毕，在当前目录下 `cd project-name`,执行 `yarn serve`或者` npm run serve`;即可启动项目；

##### 三，解读 vue -cli 目录，文件

查看 `vue-cli` 内嵌的 `webpack`配置：两种方式

* `win + R`打开 命令行 ，在项目根目录 执行 `vue inspect --mode=development > webpack.dev.js`和`vue inspect --mode=production > webpack.prod.js` 分别在根目录生成 开发与生成环境的`webpack`配置;

  ***git bash***执行此命令时会报  `stdout is not a tty` 错误；所以不要使用***git bash***执行此命令；

* 利用 vscode 终端，在项目根目录 执行 上述命令；

  vscode 终端 若是出现禁止脚本运行，是**Windows PowerShell**权限未开，电脑端搜索 **PowerShell**，按如下步骤操作后即可通过vscode运行命令；[ Windows Powershell相关问题](https://docs.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.1)

  1. 以管理员身份运行 Windows Powershell
  2. 执行：`get-ExecutionPolicy`，显示 Restricted，表示状态是禁止的; 
  3. 执行：`set-ExecutionPolicy RemoteSigned`; 交互窗口选择 Y；
  4. 再执行`get-ExecutionPolicy`，就显示 RemoteSigned;

  

  

  

  

  

  

  

  

  

  

  

  

  