推荐资源:

- [菜鸟教程](https://www.runoob.com/)
- [W3Cschool文档](https://www.w3school.com.cn/index.html)
- [W3Cschool官网](https://www.w3cschool.cn/)

### 一.html语言基础

#### 1.基本结构

```html
<html>
    <head>
        <!--元信息：提供额外信息：关键字、作者信息、页面更新时间、设置字符编码-->
        <meta charset="utf-8" />
        <title>窗口标题</title>
        <base target="_black" />	<!--统一设置超链接的打开方式-->
        <sytle type="text/css">
            内嵌样式
            </style>
        <link rel="stylesheet" type="text/css" href="链接地址" />
    </head>

    <body bgcolor="" backgroud="" leftmargin=""  >
        主体：显示内容
    </body>
</html>
```

#### 2.基本标签

格式标签:

```html
<p>段落</p>、<center>居中</center>
<ul>项目</ul>、<li>列表项</li>
<br/>换行、<hr size="宽度" noshade="去除阴影效果" />水平线
<div sytle="border-top=1px dashed black; width=#; height=#;">虚线</div>
```

文本标签:

```html
<hn>标题</hn>、<strong>加重文本：粗体</strong>、<cite>强调文本：斜体</cite>
<font face="字体" size="字号" color="颜色">文本</font>
<b>加粗</b>、<u>下划线</u>、<i>斜体</i>、<sup>上标</sup>、<sub>下标</sub>
```

图像标签:

```html
<img src="图片路径" width="" height="" border="" alt="替换文字" title="工具提示文本" align="对齐方式" />
```

超链接标签:

```html
<a href="跳转路径" target="_black" >链接文本</a>
<a href="自己其他文档地址" target="_black" >链接文本</a>
<a name="标记">文本</a>、<a href="#标记">链接文本</a>	当前文档
<a name="标记">文本</a>、<a href="文件名称#标记">链接文本</a>	其它文档
```

特殊字符:

```html
'<' --> &lt;(less than)
'>' --> &gt;(greater than)
'版权' --> &cope;(copyright)
'商标' --> &reg;(register)
'空格字符' --> &nbsp;
'&' --> &amp;
```

### 二.表格和表单

#### 1.表格

```html
<!-- cellspacing默认为2px -->
<table border="" cellspacing="" cellpadding="" width="" align="" bgcolor="">
    <tr>
        <th>表头：粗体加居中</th>
        <th>表头：粗体加居中</th>
    </tr>
    <tr>
        <td width=""  rowspan="" colspan="" height="" align="" bgcolor="" >
            数据信息
        </td>
        <td width=""  rowspan="" colspan="" height="" align="" bgcolor="" >
            数据信息
        </td>
    </tr>
</table>
```

#### 2.表单

```html
<form action="提交到路径" method="提交方式:get/post" name="" enctype="文件上传">
    <input type="输入元素" name=""  />

    <select name="">
        <option value="" selected="selected">下拉选</option>
    </select>

    <textarea name=“” cols="列" rows="行" readonly>多行文本框</textarea>
</form>
```

input表单元素中type属性值: 

> text:单行文本框, 
> password:密码框, 
> radio:单选按钮, 
> checkbox:复选框
>
> file:文件域, 
> hidden:隐藏域
>
> submit:提交, 
> reset:重置, 
> button:普通按钮, 
> image:用图片按钮提交

注意:

1. `<form>`表单中元素都必须有name属性

    ```html
    <lable for="id">单行</lable>
    <input type="text" name="" id="" size="可见宽度" />
    <lable for="id">密码</lable>
    <input type="password" name="" id="" size="" />
    ```

2. 照片上传

    ```html
    <lable for="id">文件域</lable>
    <input type="file" name="" id="" size="" />
    ```

3. 单选、复选、下拉选必须给value值;单选、复选的name属性必须相同：给一个name赋多个值，不方便服务器访问

    ```html
    <lable for="id">单选</lable>	
    <input type="radio" name="" value="" id="" checked>
    <lable for="id">复选</lable>	
    <input type="checkbox" name="" value="" id="" checked>
    ```

4. 按钮

    ```html
    <input type="submit" name="" value="提交"  />
    <input type="reset" name="" value="重置"  />
    <input type="image" src="图片路径" align=""  />
    <input type="button" value="单击我"  />
    ```

### 三.样式表与选择器

#### 1.样式表

为什么需要样式表?避免标签过多、标签过于重复、难于修改.

CSS：层叠样式表，将内容和表现形式分离.

- 行内样式表：`<p style="color:red; font-size:16px"></p>`

    - 作用于一个标签
    - 样式规则不能重复使用
    - 解决了标签过多的问题

- 内嵌样式表：位于`<style>`标签内

    ```css
    <style type="text/css">
        /*CSS注释*/
        p{
            border:1px dashed black;
            font-size:16px;
            color:red;
        }
    </style>
    ```

    - 可以做到样式规则的重用
    - 不能实现多个HTML文档共用同一套样式规则

- 外部样式表：`<link href="链接地址.css" type="text/css" rel="stylesheet(文档关系)" />` (存在于head部分)

    - 能实现多个HTML文档共用同一套样式规则

#### 2.选择器

CSS选择器：组织样式规则的容器，用于内嵌样式和外部样式规则的定义.

```css
1.标签选择器：同种元素的样式
    标签名{
        属性：属性值；
    }
    同种元素具有统一的外观
    不能重复应用于其他元素
    同种元素定义了优先级高，则先采用优先级高的，没有覆盖的样式将保留

2.类选择器；任意元素的样式
    .类名{
        属性：属性值；
    }
	任意的元素可以应用共同的样式

3.ID选择器：一个元素的样式
    #id名称{
        属性：属性值；
    }
    id元素具有唯一性
    样式不能重复使用

4.扩展选择器：
	a.伪类选择器：特殊的类选择器
        选择器：伪类名{
            属性：属性值；
        }
		定义对象在不同状态下的样式效果：
            hover：悬停
            active：点击
            after：追加内容	content：#;
            只作用于a对象：
            link：未被访问前
            visited：已被访问后

    b.包含选择器："子选择器"
        E1 E2{
            属性：属性值；
        }
        指定元素E1中子元素E2的样式
        解决了子元素选择器的定义(class、id)
        避免了样式表中样式的结构不清
        便于阅读

    c.组合选择器：被分组的选择器应用相同的样式
        选择器1，选择器2{
            属性：属性值；
        }
        避免了元素之间重复定义相同的样式
        解决了代码臃肿赘余
```

#### 3.优先级

选择器优先级: **ID > 类 > 标签**.

样式表优先级:

- 行内样式 > 内嵌样式
- 行内样式 > 外部样式

就近原则：谁距离文本内容近采用谁.

- 内嵌样式--外部样式	--`<head>`内	(文本内容)
- 行内样式--标签		--`<body>`内	(文本内容)

### 四.常用标签和常用样式属性

#### 1.常用标签

```html
1.<div>与<span>
	<div>块级容器：自动换行		用于分割文档内容
	<span>行级容器：自动适应文本	用于一个段落中
2.<ul>与<li>
	<ul>与<li>：通常用于制作板块的内容列表和导航菜单。
3.<p>与<h>
	<p>定义段落，<h>定义标题，用于段落文本排版。
4.<table>与<form>
	<table>创建表格
	<form>创建表单
```

#### 2.常用样式属性

1.字体样式：设置文本字体的样式

```css
font-size:#;字号
color:#;字体颜色
font-weight:#;粗细
line-height:#;行高
font-style:#;字体样式--italic斜体
```

2.边框样式：为元素添加边框

```css
border：border-width border-style border-color	宽 样式 颜色
    border-top;		上	(实线，虚线)
    border-righy; 	右	(实线，虚线)
    border-buttom; 	下	(实线，虚线)
    border-left; 	左	(实线，虚线)
border-radius:水平半径 [垂直半径]	cala(6px);
    border-top-left-radius ：左上边框
    border-top-right-radius ：右上边框
    border-buttom-right-radius ：右下边框
    border-buttom-left-radius：左下边框
```

3.文本样式：为文本内容进行排版

```css
text-align：文本水平对齐方式
vertical-align：文本垂直对齐方式
text-indent：文本的缩进值
text-decoration：文本的装饰效果
text-shadow：h-shadow v-shadow blur color; 文本阴影
text-transform：uppercase/lowercase/capitalize; 文本的大小写
```

4.方框样式：任何元素都是一个方框：内容、边框、内补丁、外补丁

- 内补丁(填充`padding`)：内容与边框距离 --》上、右、下、左	顺时针
- 外补丁(外边距`margin`)：边框与边框距离 --》上、右、下、左  顺时针

5.显示样式：

```html
<!-- 行级标签(inline)：大小根据内容自适应，width/height属性无效 -->
<span>、<a>、<img/>、<lable>、<input/>、<strong>、<em>
    
<!-- 块级标签(block)：显示时独占一行，width/height属性有效 -->
<div>、<ul>与<li>、<hn>、<p>、<form>
    
<!-- display属性：-->
none：隐藏标签元素
inline：同一行显示元素
block：以块状方式独占一行
inline-block：用于块级元素在一行横向排列
```

6.浮动与清除：

```css
float：left；文本流向对象的右边
float：right；文本流向对象的左边
float元素后，父级容器height将变为0(父级容器没有设置高度)
clear：both；清除元素左右两边的浮动元素(即左右两侧不容许出现浮动元素)，元素将在下一行显示
clear后，父级容器height将自适应
```

7.定位：

```css
position：absolute；绝对定位-->将对象从文档中拖出
    left:#;
    top:#;
position：relative；相对定位-->不会将对象从文档中拖出
    left:#;
    top:#;
```

### 五.页面布局

#### 1.表格布局

```html
<link href="table.css" type="text/css" rel="stylesheet" />
<table width="" border="0" cellspacing="0" cellpadding="0">
    <tr >
        <td class="title"></td>
    </tr>
    <tr class="cell">
        <td></td>
    </tr>
</table>
<div>
    <input type="button" name="button" class="button" value="搜索" />
</div>
```

table.css:

```css
body(主体){
    color:#999999;
    font-size:14px;
}
table(表格){
    border:1px solid gray;
}
td(单元格){
    text-align：center;
    vertical-align：middle；
    padding-top:10px;		--> height
    padding-bottom:10px;	--> height
    border-bottom:1px solid gray;
    border-left:1px solid gray;		
    color:gray;
}
.title(标题){
    font-size:18px;
    font-weight:bold;
    color:#084B5C;
}
.cell(单元格):hover td{
    color:#000000;
    cursor:pointer/text/crosshair/wait/help;	//鼠标指针所用的光标形状
    backgroud-color:#99CCCC;
}
.button(按钮){
    backgroud-color:gray;
    width:70px;
    height:25px;
    border:0px;
    vertical-align:middle;
}
```

#### 2.框架布局

将浏览器窗口分成几个独立的窗口，分别显示不同的页面布局方式.

```html
<frameset rows="90,*" frameborder="0px">
    <frame src="top.html" name="top" />
    <frameset cols="182,*" frameborder="0px">
        <frame src="left.html" name="left" />
        <frame src="right.html" name="right" />
    </frameset>		
</frameset>
<noframe>
    <body>浏览器不兼容时显示内容</body>
</noframe>
```

#### 3.DIV+CSS布局

```html
<link href="blogstyle.css" type="text/css" rel="stylesheet"/>
<div id="top">
    <div id="tfun"></div>
    <div id="blogtitle"></div>
    <div id="nav"></div>
</div>
<div id="content">
    <div id="left"></div>
    <div id="righr"></div>
    /*<div class="clear"></div>*/content有固定高度，float后不会回弹
</div>
<div id="footer">
    <div></div>
    <p></p>
</div>
```

blogstyle.css:

```css
------------------------------------------------------
/*页面元素样式初始化：*/
    *{
        margin:0px;
        padding:0px;
    }
    body,a{
        font-size:12px
    }
    a{
    	text-decoration:none;
    {
    .clear{
        clear:both;
    }
------------------------------------------------------
/*上、中、下共享样式规则：*/
    #top，#content,#footer{
        width:960px;
        margin:10px auto;
        border:1px solid gray;
    }
------------------------------------------------------
/*top部分的样式规则；*/
    #top{
        height:#;
    }
    #top #tfun{
        height:#;
    }
    #top #blogtitle{
        height:#;
    }
    #top #nav{
        height:#;
    }
-------------------------------------
/*子菜单中的样式规则：*/
    #top #nav ul{}
    #top #nav ul li{}
------------------------------------------------------
/*content部分的样式规则；*/
    #content{
        height:#;
        border:0px;
    }
    #content #left{
        width:68%;
        border:1px solid gray;
        height:100%;
        float:left;
    }
    #content #right{
        width:30%;
        border:1px solid gray;
        height:100%;
        float:right;
    }
------------------------------------------------------
/*footer部分的样式规则：*/
    #footer{
        heighr:#;
    }
    #footer div{
        width:#;
        margin:0px auto;
        text-align：center;
    }
    #footer div a{
        display:block;
        width:#;
        heighr:#;
        float:left;
    }
    #footer p{
        text-align：center;
    }
```

### 六.方法和技巧

#### 1.ul,li和div,a

```css
/*通配符选择器：所有的元素
	#content *{} 代表content选择器的后代所有元素都共享样式
*/

*{
    margin:0px;
    padding:0px;
}
ul{
    width:#;
    list-style:none;
    margin-left:20px;
}
li{
    width:100px;
    float:left;
}
-----------------------------
li{
    display:inline-block;
    width:#;
    height:#;
}
-----------------------------
li{
    width:#;
    height:#;
    font-size:#;
    line-height:#;
}
--------------------------------------------
div{
    width:600px;
    margin:0px auto;
    text-align：center;
}
div a{
    display:block;
    width:60px;
    height:25px;
    float:left;
}
```

#### 2.制作横向排列

```css
/* ul与li */
li{
    display:block;
    width:#;
    height:#;
    float:left;
}
--------------------------------------------------
ul{
    /*font-size:0px;*/
}
li{
    display；inline-block;
    width:#;
    height:#;
    float:left;
    /*font-size:18px*/
    /*li与li之间注释掉*/
}
--------------------------------------------------

/* div与a */
a{
    display:block;
    width:#;
    height:#;
    float:left;
}
--------------------------------------------------
a{
    dispaly:inline-block;
    width:#;
    height:#;
    float:left;
}
```

#### 3.设置元素之间距离方法

- margin与padding	方框样式
- text-align与vertical-align   文本样式
- align与valign		表格属性
- line-height	字体样式

两盒子都有外边距是取较大者.

设置两个图片之间的距离:

```html
<img>
<div>
<table>
<ul>与<li>
margin与padding调节距离
```

#### 4.元素默认样式

```css
body{
    margin:8px;
}
ul{
    margin-top:16px;
    margin-button:16px;
    paddint-left:40px;
}
p{
    margin-top:(font-size)px;
    margin-button:(font-size)px;
    height:自适应内容；
    width：独占整行；
}
hn{
    margin-top:(n的变化而变化)px;
    margin-top:(n的变化而变化)px;
    height:自适应内容；
    width：独占整行；
}
div{
    width:独占整行；
    height：自适应内容；
}
span{
    width:自适应内容；
    height：自适应内容；
}
font-size:16px;
block元素：独占一行
inline元素：自适应内容
```

### 七.项目常见问题

#### 1.元素居中问题

```html
1.<center>文本</center>	元素
2.align/valign		属性
3.margin:0px auto;		方框样式
4.position: absolute;		绝对定位
    left: 50%;		
    top: 50%;		
    transform: translate(-50%,-50%);
```

#### 2.图片问题

```css
/* 背景图片与插入图片 */
background：url() repeat-x;		//属性
<img src=""/>	  //元素

/* 表单图片提交按钮与图片链接 */
<input type="image" src="" />
<a href="#"><img src="" /></a>

/* 图片透明度问题：opacity属性(0.0~~1.0) */
<img src="" class="image" />
img{opacity:1.0;}
img:hover{opacity:0.5;}
```

#### 3.文字容器伪类问题

```css
td{
    backgroud-color:#99cccc;	(背景色)
    cursor:pointer/text/crosshair/wait/help	 (指针状态)
}
```

#### 4.定位问题

```css
/* 背景图片的定位 */
background：url() 重复 水平移动距离 垂直移动距离(正下移，负上移)
background：url() position定位

/* float定位 */
父级容器的回弹(最好都给定width和height)
float定位时文件的流动性(对象)
clear的使用：只对块级容器有作用

/* position定位 */
/* 绝对定位是“相对于”最近的已定位祖先元素，如果不存在已定位的祖先元素，
那么“相对于”最初的包含块(页面或画布)*/
position:absolute；绝对定位
/* 元素“相对于”它的起点进行移动,元素仍然占据原来的空间
父级容器绝对定位与相对定位对子容器定位的影响 */
position:relative；相对定位
```

#### 5.行级容器与块级容器的区别

行级容器：inline

- weith和height不受影响
- padding与margin不能改变本身位置
- 居中时可直接设与容器等高的line-height

块级容器：block 、inline-block、 flex

- 可设width和height	  
- padding与margin能改变本身位置