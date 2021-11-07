#### CSS 

CSS 并没有像 HTML 和 JavaScript 那样的一份标准文档,在 W3C 搜索关于 CSS 的部分:  [https://www.w3.org/TR/?title=css](https://www.w3.org/TR/?title=css) ；

CSS 语法的最新标准:[CSS Syntax Module Level 3](https://www.w3.org/TR/css-syntax-3/) ; （英文版本，难以阅读）。



##### 一，CSS 规则

CSS 的顶层样式表由两种规则组成的规则列表构成：

* 一种被称为 **at-rule**，也就是 **at** 规则：**at-rule** 由一个 **@** 关键字和后续的一个区块组成，如果没有区块，则以分号结束；
* 一种是 **qualified rule**，也就是普通规则：由选择器和属性指定构成的规则。

###### （1）at-rule

  * **@charset** ： https://www.w3.org/TR/css-syntax-3/

    @charset 用于提示 CSS 文件使用的字符编码方式，它如果被使用，必须出现在最前面；

    ```css
    @charset "utf-8"
    ```

  * **@import **：https://www.w3.org/TR/css-cascade-4/

    @import 用于引入一个 CSS 文件，除了 @charset 规则不会被引入，@import 可以引入另一个文件的全部内容。

    * **import规则一定要先于除了@charset的其他任何CSS规则**；可以通过相对路径和绝对路径引入。

      ```css
      @import 'custom.css';
      @import url("chrome://communicator/skin/");
      ```

    * @import和link一样，同样可以定义媒体查询(media queries)，**不论是link还是import方式，会下载所有css文件，然后根据媒体去应用css样式**，而不是根据媒体去选择性下载css文件；

      ```css
      @import url("fineprint.css") print;
      @import url("bluish.css") projection, tv;
      @import "common.css" screen, projection;
      @import url('landscape.css') screen and (orientation:landscape);
      
      /* 媒体查询:
       * print : 打印机
       * projection,tv: 投影,电视
       * screen, projection : 屏幕，投影
       * orientation:简单理解横屏还是竖屏。值 : portrait : 定义输出设备中的页面可见区域高度是否大于或等于宽度。默认是 : landscape；
       */
      ```

      使用@import引用的文件只有在引用它的那个css文件被下载、解析之后，浏览器才会知道还有另外一个css需要下载，这时才去下载，然后下载后开始解析、构建render tree等一系列操作。 浏览器在页面所有css下载并解析完成后才会开始渲染页面。

  * **@media** ：https://www.w3.org/TR/css3-conditional/

    @media 就是大名鼎鼎的 media query 使用的规则了，它能够对设备的类型进行一些判断。在 media 的区块内，是普通规则列表。

    ```css
    @media print {
        body{
            font-size:16px;
        }
    }
    ```

  * **@page** ： https://www.w3.org/TR/css-page-3/

    page 用于分页媒体访问网页时的表现设置，页面是一种特殊的盒模型结构，除了页面本身，还可以设置它周围的盒。

    ```css
    @page {
      size: 8.5in 11in;
      margin: 10%;
    
      @top-left {
        content: "Hamlet";
      }
      @top-right {
        content: "Page " counter(page);
      }
    }
    ```

  * **@counter-style** ：https://www.w3.org/TR/css-counter-styles-3

    counter-style 产生一种数据，用于定义列表项的表现。

    ```css
    @counter-style triangle {
      system: cyclic;
      symbols: ‣;
      suffix: " ";
    }
    ```

  * **@keyframes** ：https://www.w3.org/TR/css-animations-1/

    ```css
    @keyframes diagonal-slide {
    
      from {
        left: 0;
        top: 0;
      }
    
      to {
        left: 100px;
        top: 100px;
      }
    }
    ```

    如果@keyframes 改变的属性是与layout相关的话，就会触发重新布局，导致渲染和绘制的时间会更加地长;

    * 触发重新布局的属性有： width, height, margin, padding, border, display, top, right, bottom ,left, position, float, overflow等。应该尽量规避使用。
    * 不会出发重新布局的属性有：transform(其中的translate, rotate, scale), color, background等。应该尽量用这些去取代。

  * **@fontface** ：https://www.w3.org/TR/css-fonts-3/

    fontface 用于定义一种字体，icon font 技术就是利用这个特性来实现的。·············

    ```css
    @font-face {
      font-family: Gentium;
      src: url(http://example.com/fonts/Gentium.woff);
    }
    
    p { font-family: Gentium, serif; }
    ```

  * **@supports** ：https://www.w3.org/TR/css3-conditional/

    support 检查环境的特性，它与 media 比较类似。

  * **@namespace** ：https://www.w3.org/TR/css-namespaces-3/

    用于跟 XML 命名空间配合的一个规则，表示内部的 CSS 选择器全都带上特定命名空间。

* **@ viewport**

  用于设置视口的一些特性，不过兼容性目前不是很好，多数时候被 HTML 的 meta 代替。

###### （2）qualified rule

​		qualified rule 主要是由选择器和声明区块构成。声明区块又由属性和值构成;

* 普通规则
  * 选择器
  * 声明列表
    * 属性
    * 值
      * 值的类型
      * 函数

**选择器**：任何选择器，都是由几个符号结构连接的：空格、大于号、加号、波浪线、双竖线，这里需要注意一下，空格，即为后代选择器的优先级较低。

* 不是伪元素的话，由几个可选的部分组成，标签类型选择器，id、class、属性和伪类，它们中只要出现一个，就构成了选择器。
* 如果它是伪元素，则在这个结构之后追加伪元素。伪类可以出现在伪元素之后。

**声明**：属性 和 值 

* 属性是由**中划线、下划线、字母**等组成的标识符，CSS 还支持使用反斜杠转义。我们需要注意的是：属性不允许使用连续的两个中划线开头，这样的属性会被认为是 CSS 变量。

  ```css
  /*以双中划线开头的属性被当作变量，与之配合的则是 var 函数：
  */
  
  :root {
    --main-color: #06c;
    --accent-color: #006;
  }
  /* The rest of the CSS file */
  #foo h1 {
    color: var(--main-color);
  }
  ```

  

* 值的部分，根据每个 CSS 属性可以取到不同的值，这里的值可能是字符串、标识符。

  CSS 属性值可能是以下类型。

  * CSS 范围的关键字：initial，unset，inherit，任何属性都可以的关键字。

  * 字符串：比如 content 属性。

  * URL：使用 url() 函数的 URL 值。

  * 整数 / 实数：比如 flex 属性。

  * 维度：单位的整数 / 实数，比如 width 属性。

  * 百分比：大部分维度都支持。

  * 颜色：比如 background-color 属性。

  * 图片：比如 background-image 属性。

  * 2D 位置：比如 background-position 属性。

  * 函数：来自函数的值，比如 transform 属性。

    CSS 支持一批特定的计算型函数：

    * calc()

      ```css
      /*calc() 函数是基本的表达式计算，它支持加减乘除四则运算。在针对维度进行计算时，calc() 函数允许不同单位混合运算，这非常的有用。*/
      
      section {
        float: left;
        margin: 1em; border: solid 1px;
        width: calc(100%/3 - 2*1em - 2*1px);
      }
      ```

    * max()

      max()、min() 和 clamp() 则是一些比较大小的函数，max() 表示取两数中较大的一个，min() 表示取两数之中较小的一个，clamp() 则是给一个值限定一个范围，超出范围外则使用范围的最大或者最小值。

    * min()

    * clamp()

    * toggle()

      ```css
      /*toggle() 函数在规则选中多于一个元素时生效，它会在几个值之间来回切换，比如我们要让一个列表项的样式圆点和方点间隔出现:*/
      
      
      ul { list-style-type: toggle(circle, square); }
      ```

    * attr()

      attr() 函数允许 CSS 接受属性值的控制。

  