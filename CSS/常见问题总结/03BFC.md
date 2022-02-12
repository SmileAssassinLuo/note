##### BFC

###### 什么是BFC

`BFC`是一个块级容器，**具有 BFC 特性的元素可以看作是隔离了的独立容器，容器里面的元素不会在布局上影响到外面的元素，并且 BFC 具有普通容器所没有的一些特性。**



######  触发BFC

只要元素满足下面任一条件即可触发 BFC 特性：

- HTML 根元素
- 浮动元素：float 除 none 以外的值
- 绝对定位元素：position (absolute、fixed)
- display 为 inline-block、table-cells、flex
- overflow 除了 visible 以外的值 (hidden、auto、scroll)

###### 使用场景

* **同一个 BFC 下外边距会发生折叠**，**如果想要避免外边距的重叠，可以将其放在不同的 BFC 容器中。**

  同一BFC

```html
<head>
div{
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
}
</head>
<body>
    <div></div>
    <div></div>
</body>
```

不同BFC

```html
<style>
	.container {
    	overflow: hidden;
    }
    p {
        width: 100px;
        height: 100px;
        background: lightblue;
        margin: 100px;
    }
</style>

<div class="container">
    <p></p>
</div>
<div class="container">
    <p></p>
</div>
```

*  **BFC 可以包含浮动的元素（清除浮动）**

  ```html
  <!--
  浮动元素会脱离文档流，但利用 BFC 可以将浮动元素包裹在容器中
  -->
  <div style="border: 1px solid #000;">
      <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
  </div>
  
  <!--
  overflow: hidden 利用 BFC 
  -->
  <div style="border: 1px solid #000;overflow: hidden">
      <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
  </div>
  
  ```

* **BFC 可以阻止元素被浮动元素覆盖**

  ```html
  <!--
  	同级元素，一个元素浮动后，会覆盖另一个元素（文本不会被覆盖），但是通过设置 BFC 可以阻止被覆盖
  	触发第二个元素的 BFC 特性，在第二个元素中加入 overflow: hidden，
  	利用此特性，可以实现两列自适应布局，左边的宽度固定，右边的内容自适应宽度(去掉右边内容的宽度)
  -->
  <div style="height: 100px;width: 100px;float: left;background: lightblue">我是一个左浮动的元素</div>
  <div style="width: 200px; height: 200px;background: #eee;overflow: hidden">我是一个没有设置浮动, 
  也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>
  ```

  