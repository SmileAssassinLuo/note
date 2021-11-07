#### 一，CSS 流，元素，基本尺寸

HTML 常见的标签有 <div>、<p>、<li>和<table>以及<span>、<img>、<strong>和<em>等。虽然标签种类繁多，但通常我们就把它们分为两类：块级元素（**block-level element**）和内联元素（**inline element**）。

##### （1）块级元素

###### 1. “块级元素”和“ **display** 为 **block** 的元素”不是一个概念；

举个例子：

​	<li>元素默认的 **display** 值是 **list-item**，<table>元素默认的 **display** 值是 **table**，但是它们均是“块级元素”，因为它们都符合块级元素的基本特征，也就是***一个水平流上只能单独显示一个元素，多个块级元素则换行显示***。

由于“块级元素”具有换行特性，因此理论上它都可以配合**clear** 属性来清除浮动带来的影响。例如：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
        .box{
            padding: 10px;
            background-color: #cd0000;
        }
        .box img {
            float:left;
        }
        /*实际开发时，display 要么使用 block，要么使用 table ，并不会使用 list-item;
        主要原因是 IE 浏览器不支持伪元素的 display 值为 list-item。这是不使用 display:list-item 清除浮动的主因，兼容性不好。对于 IE浏览器（包括 IE11），普通元素 设 置 display:list-item 有 效 ， 但 是 :before/:after 伪元素就不行。
        */
        .clear::after{
            content: "";
            display:table;
            clear: both;
        }
    </style>
</head>
<body>
    <div class="box clear">
        <img src="./images/aaa.png">
    </div>
</body>
</html>
```

***Q:为什么IE浏览器不支持伪元素设置 `display:list-item`?***

***Q:为什么 `list-item` 元素会出现项目符号?***

之所以 `list-item `元素会出现项目符号是因为生成了一个附加的盒子，学名“**标记盒子**”（**marker box**），专门用来放圆点、数字这些项目符号。IE浏览器下伪元素不支持 `list-item` 或许就是无法创建这个“**标记盒子**”导致的。



按照 `display` 的属性值不同，值为 `block` 的元素的盒子实际由外在的“块级盒子”和内在的“块级容器盒子”组成，值为 `inline-block` 的元素则由外在的“内联盒子”和内在的“块级容器盒子”组成，值为 `inline` 的元素则内外均是“内联盒子”。



***Q:`display:inline-table` 的盒子是怎样组成的?***

外面是“内联盒子”，里面是“table 盒子”。得到的就是一个可以和文字在一行中显示的表格。

```html
/*HTML：*/
和文字平起平坐的表格：<div class="inline-table">
    <p>第1列</p>
    <p>第2列</p>
</div>
```

```css
/*CSS：*/
.inline-table {
    display: inline-table;
    width: 128px;
    margin-left: 10px;
    border: 1px solid #cad5eb;
}
.inline-table > p {
    display: table-cell;
}
```

***Q:`width/height` 作用在哪个盒子上?***

内在盒子，也就是“容器盒子”。

###### 2.`width/height` 作用的具体细节

(1),`width` 的默认值是 `auto`,但是至少包含了以下 4 种不同的宽度表现。

* **充分可利用空间**：像 <div>、<p>这些元素的宽度默认是 100%于父级容器的。这种充分利用可用空间的行为，叫作 **fill-available**；

* **收缩与包裹**：典型代表就是浮动、绝对定位、`inline-block` 元素或 `table` 元素，英文称为 ***shrink-to-fit***，直译为“收缩到合适”，有那么点儿意思，但不够形象，我一直把这种现象称为“包裹性”。CSS3 中的 `fit-content `指的就是这种宽度表现。

* **收缩到最小**:这个最容易出现在 `table-layout` 为 `auto` 的表格中,称为 ***min-content***;

* **超出容器限制**:除非有明确的 `width` 相关设置，否则上面 3 种情况尺寸都不会主动超过父级容器宽度的，但是存在一些特殊情况。例如，内容很长的连续的英文和数字，或者内联元素被设置了 `white-space:nowrap`;

  如下例子：子元素既保持了 `inline-block` 元素的收缩特性，又同时让内容宽度最大，直接无视父级容器的宽度限制。这种现象后来有了专门的属性值描述，这个属性值叫作 ***max-content***。

  ```html
  <style>
      *{padding: 0;margin: 0;}
      .father{
          background-color: #cd0000;
          white-space: nowrap;
          width: 150px;
          padding: 10px;
      }
      .child{
          display: inline-block;
          background-color: #f0f3f9;
          padding: 5px;
      }
  </style>
  
  <div class="father">
      <span class="child">恰如一江春水向东流，流到断崖也不回头</span>
  </div>
  ```

(2),外部尺寸与流体特性

* **正常流宽度**：当我们在一个容器里倒入足量的水时，水一定会均匀铺满整个容器；当一个元素本身或者设置为块元素时，不要设置宽度 `100%`，让他保持自身的流动性；所谓流动性，并不是看上去的宽度`100%`显示这么简单，而是一种`margin/border/padding和 content` 内容区域自动分配水平空间的机制.

  ```html
  <style>
  .width {
    width: 100%;
  }
  .nav {
    background-color: #cd0000;
  }
  .nav-a {
    display: block;
    margin: 0 10px;
    padding: 9px 10px;
    border-bottom: 1px solid #b70000;
    border-top: 1px solid #de3636;
    color: #fff;
  }
  .nav-a:first-child { border-top: 0; }
  .nav-a + .nav-a + .nav-a { border-bottom: 0;}
  </style>
  
  
  <h4>无宽度，借助流动性</h4>
  <div class="nav">
    <a href="" class="nav-a">导航1</a>
    <a href="" class="nav-a">导航2</a>
    <a href="" class="nav-a">导航3</a>
  </div>
  <h4>width:100%</h4>
  <div class="nav">
    <a href="" class="nav-a width">导航1</a>
    <a href="" class="nav-a width">导航2</a>
    <a href="" class="nav-a width">导航3</a>
  </div>
  ```

  ![](D:\work\note\CSS\img\块元素流动性.png)

  后者的尺寸超出了外部的容器，完全就不像“水流”那样完全利用容器空间，即所谓的“流动性丢失”。

* **格式化宽度**：格式化宽度仅出现在“绝对定位模型”中，也就是出现在 position属性值为 `absolute` 或 `fixed` 的元素中；和上面的普通流一样，“格式化宽度”具有完全的流体性，也就是 margin、border、padding 和 content 内容区域同样会自动分配水平（和垂直）空间。

(3),内部尺寸和流体特性

所谓“内部尺寸”，简单来讲就是元素的尺寸由内部的元素决定，而非由外部的容器决定。如何快速判断一个元素使用的是否为“内部尺寸”呢？假如这个元素里面没有内容，宽度就是 0，那就是应用的“内部尺寸”。

* **包裹性**：指的是元素尺寸由内部元素决定，但永远小于“包含块”容器的尺寸（除非容器尺寸小于元素的“首选最小宽度”）；

  ```html
  <!-- 按钮就是 CSS 世界中极具代表性的 inline-block 元素，可谓展示“包裹性”最好的例子，具体表现为：按钮文字越多宽度越宽（内部尺寸特性），但如果文字足够多，则会在容器的宽度处自动换行（自适应特性）。 -->
  
  <style>
      .box {
          width: 240px;
          margin: 20px auto;
      }
  </style>
  <div class="box">
    <button>按钮文字越多宽度越宽（内部尺寸特性），但如果文字足够多，则会在容器的
  宽度处自动换行（自适应特性）</button>
  </div>
  ```

  开发示例：

  需求：页面某个模块的文字内容是动态的，可能是几个字，也可能是一句话。然后，希望文字少的时候居中显示，文字超过一行的时候居左显示。该如何实现？

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
      <style>
          *{padding: 0;margin: 0;}
          .box {
              padding: 10px;
              background-color: #cd0000;
              text-align: center;
          }
          .content {
              display: inline-block;
              text-align: left;
          }
      </style>
  </head>
  <body>
      <div class="box">
          <p id="conMore" class="content">文字内容</p>
      </div>
      <!-- 按钮 -->
      <p><button id="btnMore">更多文字</button></p>
      <script>
          var btn = document.getElementById('btnMore'), 
          content = document.getElementById('conMore');
      
          if (btn && content) {
          btn.onclick = function() {
              content.innerHTML += '-新增文字';
          };
          }
      </script>
  </body>
  </html>
  ```

  

* **首选最小宽度**：指的是元素最适合的最小宽度。

  * 东亚文字（如中文）最小宽度为每个汉字的宽度;

  * 西方文字最小宽度由特定的连续的英文字符单元决定。并不是所有的英文字符都会组成连续单元，一般会终止于空格（普通空格）、短横线、问号以及其他非英文字符等。例如，“display:inline-block”这几个字符以连接符“-”作为分隔符，形成了“display:inline”和“block”两个连续单元，由于连接符“-”分隔位置在字符后面，因此，最后的宽度就是“display:inline-”的宽度;

    ![](D:\work\note\CSS\img\最小宽度.png)

    ***“首选最小宽度”对我们实际开发有什么作用呢？***

    可以让我们遇到类似现象的时候知道原因是什么，方便迅速找到问题；

    实际开发很少用到，但利用“首选最小宽度”的行为特点把需要的图形勾勒出来，比如

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Document</title>
        <style>
            *{padding: 0;margin: 0;}
            .ao,
            .tu {
                display: inline-block;
                width: 0;
                font-size: 14px;
                line-height: 18px;
                margin: 35px;
                color: #fff;
            }
            .ao:before,
            .tu:before {
                outline: 2px solid #cd0000;
                font-family: Consolas, Monaco, monospace;
            }
            .ao:before {
               content: "love你love";
            }
            .tu {
               direction: rtl;
            }
            .tu:before {
                content: "我love你";
            }
        </style>
    </head>
    <body>
        <span class="ao"></span>
        <span class="tu"></span>
    </body>
    </html>
    ```

    

* **最大宽度**：最大宽度就是元素可以有的最大宽度，“最大宽度”实际等同于“包裹性”元素设置` white-space:nowrap` 声明后的宽度.

###### 3.width 值作用的细节

设置元素宽度 为100px,

```css
div{width:100px}
```

***Q:100px 的宽度是如何作用到这个<div>元素上的？***

`width:100px` 作用在了 content box上，由于<div>元素默认的 padding、border 和 margin 都是 0，因此，该<div>所呈现的宽度就是 100 像素。

实际的元素宽度 等于 content-box + padding-box + border-box ;似乎符合逻辑，但缺点是什么呢？

*  **流动性缺失**：块元素定宽失去流动性；

* **与现实世界表现不一致的困扰**：当padding，border变化时，不希望整个值发生变化，导致页面计算出现错位问题；

  如何避免？CSS 流体布局下的**宽度分离原则**；

  代码实现：

  ```css
  .box{ width : 100px ; border : 1px , solid , red ;}
  .box{ width : 100px ; padding : 20px ; }
  ```

  尽量使用下述代码替代上面内容

  ```
  .father{ width : 100px;}
  .son{
  	margin : 10px ;
  	padding : 20px ;
  	border : 2px solid red ;
  }
  ```

  为什么要使用**宽度分离**？便于维护，虽然多嵌套了一层html，但是维护方便，无需计算。但是对于开发人员具有挑战性；

###### 4.改变 width/height 作用细节的 box-sizing

“内在盒子”的 4 个盒子？它们分别是 `content box、padding box、border box 和 margin box`。默认情况下，width是作用在 content box 上的，***box-sizing***的作用就是可以把 width 作用的盒子变成其他几个，因此，理论上，box-sizing 可以有下面这些写法：
	`.box1 { box-sizing: content-box; }`
	`.box2 { box-sizing: padding-box; }`
	`.box3 { box-sizing: border-box; }`
	`.box4 { box-sizing: margin-box; }`
理论美好，现实残酷。实际上，支持情况如下：
	`.box1 { box-sizing: content-box; }` /* 默认值 */
	`.box2 { box-sizing: padding-box; } `/* Firefox 曾经支持 */
	`.box3 { box-sizing: border-box; } `/* 全线支持 */
	`.box4 { box-sizing: margin-box; } `/* 从未支持过 */

因此，使用 `box-sizing: border-box`能解决错误和层级嵌套html标签的问题，但是，



在 CSS 世界中，唯一离不开 `box-sizing:border-box` 的就是原生普通文本框<input>和文本域<textarea>的 100%自适应父容器宽度。

在作者(张鑫旭)看来，box-sizing 被发明出来最大的初衷应该是解决替换元素宽度自适应问题。

###### 5.为何 height:100%无效

```html
 /* 先渲染父元素，图片宽度和文字的宽度决定，再渲染子元素，100% ，宽度不够，溢出 ，overflow 属性就是为此而生的*/
.box {
     display: inline-block;
     white-space: nowrap;
     background-color: #cd0000;
 }
 .text {
     display: inline-block;
     width: 100%;
     background-color: #34538b;
     color: #fff;
 }
 
<div class="box">
    <img src="./images/aaa.png">
    <span class="text">红色背景xxxx是父级</span>
</div>
```

###### 6.如何让元素支持 height:100%效果

* 设定显式的高度值

* 使用绝对定位

  绝对定位元素的百分比计算和非绝对定位元素的百分比计算是有区别的，区别在于绝对定位的宽高百分比计算是相对于 padding box 的，也就是说会把 padding 大小值计算在内，但是，非绝对定位元素则是相对于 content box 计算的。

  绝对定位方法剑走偏锋，支持隐式高度计算，给人意外之喜，但本身脱离文档流，使其仅在某些场景有四两拨千斤的效果，比方说“图片左右半区点击分别上一张图下一张图效果”的布局:
  
  ```html
/*原理很简单，就是在图片外面包一层具有“包裹性”同时具有定位特性的标签。例如：
  .box {
  	display: inline-block;
  	position: relative;
  }
  此时，只要在图片上覆盖两个绝对定位，同时设 height:100%，则无论图片多高，我们的左右半区都能自动和图片高度一模一样，无须任何使用 JavaScript 的计算。*/
  
  .box {
  	display: inline-block;
  	position: relative;
  }
  .prev, 
  .next {
  	width: 50%; height: 100%;
  	position: absolute;
  	top: 0;
  	opacity: .5;
  }
  .prev {
      left: 0;
  	background-color: #cd0000;
  }
  .next {
  	right: 0;
  	background-color: #34538b;
  }
  
   <div class="box">
       <a href="javascript:" class="prev" title="上一张">上一张</a>
       <a href="javascript:" class="next" title="下一张">下一张</a>
       <img src="/images/aaa.png">
   </div>
  ```
  



###### 7.为流体而生的 min-width/max-width

在公众号的热门文章中，经常会有图片，这些图片都是用户上传产生的，因此尺寸会有大有小，为了避免图片在移动端展示过大影响体验，常常会有下面的 max-width 限制：
`img {
	max-width: 100%;
	height: auto!important;
}`
`height:auto` 是必需的，否则，如果原始图片有设定 height，max-width 生效的时候，图片就会被水平压缩。强制 height 为 auto 可以确保宽度不超出的同时使图片保持原来的比例.

但这样也会有体验上的问题，那就是在**加载时图片占据高度会从 0 变成计算高度，图文会有明显的瀑布式下落**。



**min-width/max-width** 和 **min-heigh/max-height** 从长相上看明显和 **width/height** 是一个家族的，总以为属性值、模型都是一样的，但是有一个地方就搞了特殊，那就是初始值.

* width/height 的默认值是 auto；
* max-width和max-height的初始值是none；
* min-width和min-height的初始值是 auto；

**超越!important，超越最大**:

***超越!important***  举例：`<img src="1.jpg" style="width:480px!important;">
		img { max-width: 256px; }
答案是 256px。style、!important 通通靠边站！因为 max-width 会覆盖 width。`

***超越最大***指的是min-width覆盖max-width，此规则发生在min-width和max-width冲突的时候。例如：

```css
.container {
	min-width: 1400px;
	max-width: 1200px;
}
```

##### （2）内联元素

###### 什么是内联元素？

* 从定义看，“内联元素”的“内联”特指“外在盒子”，和“display 为 inline的元素”不是一个概念！inline-block 和 inline-table 都是“内联元素”，因为它们的“外在盒子”都是内联盒子。自然 display:inline 的元素也是“内联元素”，那么，<button>按钮元素是内联元素，因为其 display 默认值是 inline-block；<img>图片元素也是内联元素，因为其 display 默认值是 inline 等。
* 从表现看，就行为表现来看，“内联元素”的典型特征就是可以和文字在一行显示。因此，文字是内联元素，图片是内联元素，按钮是内联元素，输入框、下拉框等原生表单控件也是内联元素。

###### 内联世界深入的基础 — 内联盒模型

从下面一段HTML开始:

```html
<p>这是一行普通的文字，这里有个 <em>em</em> 标签。</p>
```

看上去很普通的一段语句，实际包含了很多盒子；

* 内容区域（content area）：
* 内联盒子（inline box）：
* 行框盒子（line box）：
* 包含盒子（containing box）：

