#### Chrome Dev Tools

##### 一，打开开发者工具

* 打开浏览器右边    `...`  - ->> 更多工具 -->> 开发者工具；
* 快捷键： 键盘 `F12`； 或者 `Ctrl + Shift + i`;

##### 二，打开菜单命令

* `Ctrl + Shift + p`（win）;
* `Command + Shift + p`(mac);

直接快捷更改开发者工具的设置，比如语言，背景色等；

主题背景白黑切换：`theme`

截屏：`screenshot`,可选择多种截图模式，比如全图，node节点截图，区域截图；

dock：将开发者工具页面置于不同的位置，左右下，还是新窗口。

##### 三，常用的Tab

* `Elements`
* `Console`
* `Sources`
* `Network`
* `Performance`
* `Memory`
* `Application`
* `Security`

##### 四，CSS调试

###### (1),选择查找元素

* 检查，在页面选择`dom`节点,鼠标右击选择检查，即可跳转到`Elements`下，当前元素节点标签处。
* 查找**DOM**树：`Ctrl + F `;
  * 文本查询：直接输入文本内容；
  * `CSS`查询：输入`CSS`选择符,比如 `section#section_one`, `id`为`section_one`的`section`元素；
  * `Xpath`: 比如 `//section/p`:全局 `section下的p标签`；
* **Console** 选项下 输入 `inspect(document.getElementById("international-home-app"))`,可跳转至`id`所指的元素处；

######  (2),编辑元素样式

`Elements`左侧是元素，右侧是样式内容：

* `Styles`面板是样式的详细信息，可以对元素添加编辑删除样式，还会显示告知样式来源；还包括一个元素模型的大小，包括边距等；
  * 使用 `atuo-complete`特性给元素添加样式：在`element.style {  color: red;}`,添加`color:red`,或者输入`bold`,会自动提示包含`bold`属性值的属性名称。
  * 让`hover`常驻：右边栏有个`:hov`,点击弹出可勾选多种选中，悬停等选项。
  * 编辑`class`:右边栏 `.cls`,就可以选择修改单个元素的`class`,而不是将修改的`class`应用于全局元素。
  * 复制样式：选中元素右击，选择复制`styles`;

* `Computed`面板显示元素的所有样式，但不包括来源等细节，`show all`可以显示包括继承而来的样式，`Group`则可以展示样式的分类；
* `Layout`面板:布局信息，`Grid`，`Flexbox` 可以自选查看使用了上述布局的页面的布局信息；
* `Event Listeners`:查看绑定的所有事件
* `DOM Breakpoints`:在`Elements`面板选中元素，右击选择`break on`,可以选择3中情况下代码中断`js`执行的方式；某些`js`代码导致以下变化，那么这些`js`代码便会中断，跟断点一致。
  * `subtree modifications`:以此元素为根节点的子元素发生改变;
  * `attribute modifications`:此元素属性发生改变；
  * `node removal`:此元素节点被移除；
* `Properties`:以原型链继承的方式记录节点对象信息；
* `Accessibility`: 构建无障碍页面，待补充；



##### 五，Console 控制台

* 快捷键：`Ctrl + Shift + j` ,或者 `Command + Shift + j`（mac‘）;

* 执行语句：直接在蓝色箭头编写`js`代码；

* `$_`:返回上句代码返回结果；

* `$0`:返回上一个`node`节点,比如在`Elements`面板点击了一个`div`节点，在`Console `面板输入 `$0` ,即可打印`node`节点的信息。`$1,$2`一次返回上上个，上上上个节点。

* `console(log,error,warn)等`

  * `console.groud`:打印一组`log`信息;

  ```javascript
  console.groud("console.groud test");
  console.log(1);
  console.log(2);
  console.groudEnd("console.groud test");
  ```

  * `console.time`:常用于一段代码执行时间

  ```javascript
  onsole.time("test time");
  let sum = 0 ;
  for(let i = 0 ;i < 100000 ;i++){
      sum += i;
  }
  console.timeEnd("test time");
  // test time: 252886.60180664062 ms
  ```

  * `console.table`:将一个数组，展示为表格的形式

  ```javascript
  console.table([{id:'1',name:'isaiah'},{id:"2",name:"jack"}])
  /*
  (index)	id 	 name
  0		'1'	'isaiah'
  1		'2'	'jack'
  */
  ```

* 其他，比如 “一个类似眼睛的”用来监控变量；`default levels`,设置`LOG`筛选；

##### 六，Javascript 调试（主要在source面板）

调试的时候 ，`Ctrl + Shift + p` 输入 `Enable code folding` ,可以将调试代码折叠；

* 在代码处添加 `debugger`
* 在`source`面板，点击某行代码最前行号位置，出现蓝色箭头；

* `DOM Breakpoints`在元素标签选择`break on`,进行断点；
* `Event Listeners Breakpoints`: 可以选择勾选某些事件，触发时打断点；有些时候使用框架开发，点击事件会跳转至框架的js中，但框架是不需要我们调试的，在`Call Stack` 选择 `add script to ignore list` ,就可以忽略这断点；
* 在页面还有其他的断点方式等，可自行摸索；

##### 七，Network 面板

* `preserve log`:勾选上就会把之前的请求留存在请求列表，主要用于跳转页面后，不想前一个页面的请求被清除；
* `Disable cache`: 去掉 浏览器缓存；
* `No throtting`: 节流器，模拟低速网络；
* `more network condition`: 类似网络发射的图标，,请求时 `request headers` 的值,会被更改为模拟设置后的值；
  * 更改 `user agent` 
  * 更改`Accepted Content-Encodings`
* `export har file` 和 `import har file`: 使用说明；比如测试时出现不明bug，而本机又无法模拟，可以通过在测试机上导出此文件，在本机上导入，就可以查看分析此次请求的情况了；

##### 八，Application

`storge`和`cookie`的信息等；



##### 九，Performance











