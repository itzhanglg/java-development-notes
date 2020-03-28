推荐资源:

- [菜鸟教程](https://www.runoob.com/)
- [W3Cschool文档](https://www.w3school.com.cn/index.html)
- [W3Cschool官网](https://www.w3cschool.cn/)

### 一.jQuery概述

jQuery是一个优秀,兼容多浏览器的javascript库，核心理念是write less,do more;jQuery使用户能更方便地处理HTML、events、实现动画效果，并且方便地为网站提供AJAX交互.

**1.基本功能**

- a.访问和操作DOM元素:获取元素,修改其样式和内容,删除元素,复制元素...
- b.对页面事件的处理:不需要指定事件中的函数名,直接在事件中绑定响应函数(匿名函数)
- c.插件的运用:验证插件,UI插件...
- d.Ajax技术的结合:$.ajax({"json格式"});  Ajax异步读取服务器数据

**2.代码特点**

- a.$符号特点
- b.隐式循环
- c.链式书写

**3.js与jq区别**

传统的方式页面加载会存在覆盖问题，加载比JQ慢(整个页面加载完毕<包括里面的其它内容，比如图片>). JQ不存在覆盖问题，加载的时候是顺序执行,加载比JS要快！(当整个dom树结构绘制完毕就会加载).

加载效率:

- jq:页面框架下载完(页面元素信息)就触发事件  -- 效率高
- js:页面中所应用到的所有资源(img...)全部加载完才触发事件  -- 效率低

覆盖问题:

- jq:不存在覆盖,加载时顺序执行
- js:存在覆盖,加载时执行最后一个

### 二.jQuery选择器

`$`符号通常是jQuery的标识符（别名），一般不改动；`$ `定义为 **选取**, 英文是selector的意思。

`$(selector).action();`  -- 选取元素,获取jQuery对象,再执行方法. 选择器参数是一个字符串,当使用变量时,需使用加号将变量与其他字符串联在一起.

#### 1.基本选择器

```js
#id  -- id属性值
.class  -- class属性值
element  -- 标签名
selector1,selectorn  -- 多个选择器所匹配的元素
```

#### 2.层级选择器

```js
selector1 selector2  -- 后代
selector1>selector2  -- 子类
selector1+selector2(next())  -- 下一个相邻兄弟
selector1~selector2(nextAll())  -- 后面所有兄弟
siblings()  -- 所有兄弟
```

#### 3.过滤选择器

```js
a.基本过滤
    :first或first()	-- 第一个
    :last或last()  -- 最后一个
    :eq(index)或eq(2/even/odd/2n/2n+1)  -- 索引值等于index(从0开始)  :gt()大于    :lt()小于
    :nth-child(index)  -- 子元素过滤(索引从1开始,也可以写成2n)
	:first-child  --  选取父元素的第一个子元素
    :not(selector)  -- 不包含或去除所有与给定选择器匹配的元素
b.内容过滤
	-- 选取包含给定文本的元素(也包含后代元素出现了text内容,text中英文有大小写区分)
    :contains(text)  
    :has(selector)  -- 选取含有选择器所匹配的元素的元素
    :empty  --  选择空元素
    单标签(input,img,br,hr)都属于空元素
c.可见性过滤
    :hidden  -- 不可见(display:none,input type=hidden,宽高=0)
    :visible  -- 可见
d.属性过滤
    [attr]	-- 拥有该属性;选取包含给定属性的元素
    [attr=val]  -- 包含属性且等于val
    [attr!=val]  --  不等于或没有该属性
    [attr^=val]  --  以val开始
    [attr$=val]  --  以val结尾
    [attr*=val]  --  包含val
    [selector1][selectorn]  -- 同时满足属性过滤的多个条件
```

#### 4.表单选择器(属性过滤的简化)

```js
:input  -- input,select,button,textarea元素
:text
:radio
:checkbox
:button  -- input type=button,button元素

/* 表单过滤 */
:checked  -- input  --  :radio:checked,:checkbox:checked
:selected  -- option  --  option:selected
:disabled  -- 不可用     --  text:disabled,text:enabled
```

#### 5.元素筛选方法

```js
过滤:
	first(),last(),eq(index),is(expr/obj/ele)判断集合是否有匹配的元素
查找:
    children([expr])子元素,find(expr/obj/ele)后代元素
    parent([expr])父元素,parents([expr])祖先元素
    next([expr])下一个相邻兄弟,nextAll([expr])后面所有兄弟,
    prev([expr])上一个相邻兄弟,prevAll([expr])前面所有兄弟,
    siblings([expr])所有兄弟
```

### 三.jQuery中dom操作

可以对元素属性,内容,值和CSS样式的操作,如何创建节点,插入节点,复制节点,删除节点和遍历节点.

**jQuery与Dom对象的区别**:

- dom对象:js方法获取元素,将dom对象存储在变量中; jq对象:jq方法获取元素的jq对象,将jq对象存储在变量中. 
- jQuery对象就是使用`jQuery()`或`$()`包装了dom对象后的对象；其中符号`$`是jQuery的一个别名。
- jQuery对象不能使用Dom对象中的任何方法。同样地；Dom对象也不能使用jQuery对象中的任何方法.

**jQuery与Dom对象转换**:

- jQuery对象转为Dom对象:
    - jQuery是一个数组对象，可使用下标,直接通过索引
        ```js
        var $div = $("#divID");
        var divElement = $div[0];
        ```
    - 使用jQuery对象自带的get方法: 不带参数时,返回一个dom对象的数组;带参数时,返回第(index+1)个元素的dom对象(索引从0开始)
        ```js
        var #div = $("#divID");
        var divElement = $div.get(0);
        ```
- Dom对象转为jQuery对象: 只需用$()将dom对象包装起来.
    
    - `var $div = $(divElement);`

#### 1.元素修改

```js
a.元素样式
    css()	-- 增加style属性值 eg: css({name:value,name:value}),css(name,value)
    addClass() -- 增加css类(class属性值),多个类用空格分隔(保留了原有的类别)
    removeClass() -- 删除类别,没参数时,删除所有类样式
    toggleClass() -- 类样式切换(增加/删除类别),检测是否有该类别
b.元素内容及value值
    html() -- 获取第一个匹配元素
    html(content) -- 设置所有匹配的元素
    text() -- 获取所有匹配的元素的文本内容
    text(content) -- 设置所有匹配的元素的文本内容
    val([val]) -- 获取或设置元素的值
c.元素属性
	//attr({name:value,name:value}),attr(name,value),attr(name),attr(name,function(){})
    attr() 
    prop() -- 当属性值为布尔型时,如checked,selected
	removeAttr(name) -- 删除属性
//如果遇上具有 true 和 false 两个属性的属性,如checked, selected 或者disabled则使用prop()
```

#### 2.元素节点

```js
a.元素创建
	$(html) -- 动态创建页面元素
b.元素插入
    A.append(B) -- A内部末尾附加B
    A.appendTo(B) -- A附加到B内部末尾;B内容末尾追加A内容
    prepend() -- A内部前置附加B
    after() -- A之后插入B
    before() -- A之前插入B
c.元素替换
    replaceWith(content) -- 用括号中内容替换jq对象
    replaceAll(selector) -- jq对象去替换括号中的元素
d.元素复制
	clone([true]) -- 带true参数时,元素全部行为也会复制
e.元素删除
    remove() -- 删除节点
    empty() -- 清空节点的html内容,不删除自身节点;删除匹配的元素集合中所有的子节点
```

#### 3.元素遍历

```js
each(callback) -- 先获取匹配元素的集合,再遍历,以每一个匹配的元素作为上下文来执行一个函数
$.each(obj,callback) -- 全局的,obj为遍历对象;通用遍历方法，可用于遍历对象和数组
```

#### 4.获取元素的宽高

```js
$("body").css("width");		//获取页面内容css样式中的宽度属性值
$("body").height();			//获取页面内容的高度
$(window).width();			//获取浏览器窗口的宽度
$(window).css("height");	//获取浏览器窗口css样式中的高度属性值
css("width/height"):值得形式是包含了单位"px"的字符串 -- "160px"
height()/width():值是数字型的,更方便进行数学运算 -- 160
```

### 四.jQuery中事件

#### 1.常用事件

##### 1.1 页面载入事件

```js
ready()方法
格式:
    $(document).ready(function(){});
    $(function(){});
```

##### 1.2 绑定事件

`click(),dblclick(),focus(),blur(),mouseover(),mouseout(),change(),select(),keydown(),keyup()` 

js事件模型:

- 第一种:在html标签上增加事件属性(eg: `onclick="addUser(user)"`),让属性值等于处理该事件的函数名或程序代码
- 第二种:在js代码中设置元素的事件属性(eg: `jq.click(function(){})`),让属性值等于处理该事件的函数名或程序代码

jq事件一种统一的事件模型: 

```js
//在页面加载完毕后,为每个选取元素的事件绑定响应函数
$(function(){
    $("#btn").click(function(){//执行代码});	//统一的事件模型
    $("#btn").bind(type,function(){//执行代码});	//type表示事件类型
    //为所选对象绑定多个事件处理函数
    $("#btn").bind({type:function(){//执行代码},type:function(){//执行代码}});
});
```

##### 1.3 切换事件

```js
hover():使元素在鼠标移入与移出的事件中进行切换
	hover(over,out);	//over:移入时处理的函数,out:移出时处理的函数
toggle():依次调用N个指定的函数,直到最后一个,然后重复对函数进行调用
    toggle(
        function(){},
        function(){},
        function(){},...
    );
        
toggleClass():样式添加/删除开关
hover():鼠标移入/移出开关
toggle():显示/隐藏开关(没参数时)
```

##### 1.4 其它事件

```js
one():为所选的元素绑定一个仅触发一次的处理函数
	one(type,function(){});		//type表示事件类型
trigger():在所选的元素上触发指定类型的事件
	trigger(type);	//type表示触发事件的类型
```

#### 2.事件机制

**事件在触发后分为两个阶段**: 一个是捕获(capture),另一个是冒泡(bubbling). 往往事件触发后,直接执行冒泡过程,冒泡实质就是事件执行中的顺序.(大部分浏览器不支持捕获,jq也不支持).

如: 单击按钮,按钮的父标签div的单击事件也被触发,同时div的父标签body的单击事件也随之触发.

**冒泡过程**: 整个事件波及的过程就行水泡一样向外冒,故称为冒泡过程.

**停止事件的冒泡过程**: 可以通过`return false;`语句实现.(单击按钮就执行单击事件,不触发其它父元素的单击事件).

### 五.jQuery动画

#### 1.显示/隐藏动画效果

```js
//动态的改变当前元素的宽,高和不透明度
show([duration],[fn]);		//显示当前元素
hide([duration],[fn]);		//隐藏当前元素
toggle([duration],[fn]);	//切换当前元素的可见状态
//参数涵义
	duration:动画效果运行的时间,默认为0,立即显示元素
        关键字:"slow","normal","fast" - 0.6,0.4,0.2秒
        数字:600,400,200,... 单位是毫秒
	fn:在动画完成时执行的函数
```

#### 2.淡入/淡出动画效果

```js
//动态的改变当前元素的透明度(其他不变)
fadeIn([duration],[fn])		//显示当前元素	- 淡入效果
fadeOut([duration],[fn])	//隐藏当前元素   - 淡出效果
fadeToggle([duration],[fn])	//切换当前元素的可见状态	-- duration默认为normal,0.4秒
fadeTo([duration],opacity,[fn])	//在指定时间内,从当前透明度淡到指定透明度
//参数涵义
	opacity:指定不透明值,取值范围是0-1,0代表完全透明,1代表完全不透明
```

#### 3.滑入/滑出动画效果

```js
//动态的改变当前元素的高度(其他不变)
slideDown([duration],[fn])		//显示当前元素	- 由上到下滑入
slideUp([duration],[fn])		//隐藏当前元素	- 由下到上滑出
slideToggle([duration],[fn])	//切换当前元素的可见状态
```

#### 4.自定义动画

```js
//动态的改变当前元素的各种CSS属性
animate(properties,[duration],[fn])
//参数涵义
	properties:使用一个"name:value"形式的对象,{name:value,name:value,...},用来设置改变的css属性
    animate():只能改变可以取数字值的css属性,如大小,边框,定位,字体,背景,...
    移动元素是需要显示元素的position属性为absolute/relative.
    "队列"动画:元素执行多个动画效果,即元素执行多个animate()方法,按照方法的顺序进行动画效果的展示.
```

#### 5.动画停止

```js
stop()	//结束当前动画,立即进入到下一个动画
stop([clearQueue],[gotoEnd])
//参数涵义
	clearQueue:是否清空未执行完的动画队列.设置true,可以立即结束动画
	gotoEnd:是否立即完成正在执行的动画.设置true,并且重设show和hide的原始样式，调用回调函数等
```

### 六.常见问题

#### 1.多选框的全选与全不选

```js
//1.遍历:使用each();
$("#checkallbox").click(function(){
    var isChecked = this.checked;
    //使用对象访问的方式进行遍历，语法：$().each(function(){})
    $("input[name='hobby']").each(function(){
        this.checked = isChecked;
    });
});

//2.遍历:使用$.each()
$("#checkallbox").click(function(){
    var isChecked = this.checked;
    //使用工具类遍历方式，语法：$.each(array，function(i,j){})  
    //其中array代表被遍历的对象，i代表角标，j代表遍历后的dom对象。
    $.each($("input[name='hobby']"), function(i,j) {
        j.checked = isChecked;
    });
});

//3.添加属性:prop()
$("#checkallbox").click(function(){
    //获取下面所有的 复选框并将其选中状态设置跟编码的前端 复选框保持一致。
    $("input[name='hobby']").prop("checked",this.checked);
});
```

#### 2.二级联动问题

```html
<select id="province">
    <option>--请选择--</option>
    <option value="0">湖北</option>
    <option value="1">湖南</option>
    <option value="2">河北</option>
    <option value="3">河南</option>
</select>
<select id="city"></select>

<script>
$(function(){
    //2.创建二维数组用于存储省份和城市
    var cities = new Array();
    cities[0] = new Array("武汉市","黄冈市","襄阳市","荆州市");
    cities[1] = new Array("长沙市","郴州市","株洲市","岳阳市");
    cities[2] = new Array("石家庄市","邯郸市","廊坊市","保定市");
    cities[3] = new Array("郑州市","洛阳市","开封市","安阳市");

    $("#province").change(function(){
        //10.清除第二个下拉列表的内容
        $("#city").empty();
        //$("#city option").remove();
        //1.获取用户选择的省份
        var val = this.value;
        //3.遍历二维数组中省份
        $.each(cities,function(i,obj){
            //4.判断用户选择的省份和遍历的省份
            if(val==i){
                //5.遍历该省份下的所有城市
                $.each(cities[i],function(j,obj2){
                    //6.创建城市文本节点
                    //var textNode = document.createTextNode(obj2);
                    //7.创建option元素节点
                    var op = document.createElement("option");
                    //8.将城市文本节点添加到option元素节点中
                    //$(op).append(textNode);
                    //$(op).append(obj2);	向option元素末尾追加内容
                    $(op).html(obj2);	设置option元素内部的html文本内容
                    $(op).val(obj2);	//设置option元素的value值
                    //9.将option元素节点追加到第二个下拉列表中取
                    $(op).appendTo($("#city"));

                    //原生js写法  ------------------------------------ 
                    //创建节点
                    var opt = document.createElement("option");
                    //为节点设置HTML内容
                    opt.innerHTML = pcities[i];
                    //在sel2中末尾追加指定的节点
                    sel2.appendChild(opt);
                });
            }
        });
    });
});
</script>
```

#### 3.下拉列表左右添加

```html
<select multiple="multiple" id="left">
    <option>IPhone6s</option>
    <option>小米4</option>
    <option>锤子T2</option>
</select>
<a href="#" id="selectOneToRight">&gt;&gt;</a> <a href="#" id="selectAllToRight">&gt;&gt;&gt;</a>

<select multiple="multiple" id="right">
    <option>三星Note3</option>
    <option>华为6s</option>
</select>
<a href="#" id="selectOneToLeft">&lt;&lt;</a> <a href="#" id="selectAllToLeft">&lt;&lt;&lt;</a>

<script>
$(function(){ 
	/*1.选中单击去右边*/
    $("#selectOneToRight").click(function(){
        $("#left option:selected").appendTo($("#right"));
    });
    /*2.单击全部去右边*/
    $("#selectAllToRight").click(function(){
        $("#left option").appendTo($("#right"));
    });
    /*3.选中双击去右边*/
    $("#left").dblclick(function(){//这里是下拉选被双击时触发,不是后代元素#left option被双击时触发
        $("#left option:selected").appendTo($("#right"));
    });
    /*1.选中单击去左边*/
    $("#selectOneToLeft").click(function(){
        $("#right option:selected").appendTo($("#left"));
    });
    /*2.单击全部去左边*/
    $("#selectAllToLeft").click(function(){
        $("#right option").appendTo($("#left"));
    });
    /*3.选中双击去左边*/		
    $("#right").dblclick(function(){//这里是下拉选被双击时触发,不是后代元素#left option被双击时触发
        $("#right option:selected").appendTo($("#left"));
    });
}    
</script>
```

