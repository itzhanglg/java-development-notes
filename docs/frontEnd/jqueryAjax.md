### 一.jquery中ajax

#### 1.ajax请求格式

```js
//重点掌握
//指定dataType为json时,不需要用var obj = eval("("+data+")");去再次解析,会报错
$.ajax({
    async: true,	//异步
    type: "post", //请求方式
    url: "user/add", //请求地址
    data: {"account":"jack","pwd":"123"}, //请求数据
    dataType: "json", //响应内容格式
    success: function(data){ //请求成功后的回调函数
        //dowork
    },
    error: function(msg){
        alert(msg);
    }
});

//其它请求: 请求地址,请求参数,回调函数,响应内容格式
$.get(url,[data],[callback],[type]);
$.post(url,[data],[callback],[type]);
$.getJSON(url, [data], [callback]);
```

#### 2.json格式与解析

json格式:

```json
1.json对象
	{"name":"jack","sex":"男","age":"25"}
2.json数组或集合格式
	[{"name":"jack","sex":"男","age":"25"},{"name":"tom","sex":"男","age":"26"}]
3.json对象,数组嵌套
```

阿里的fastjson包解析json:

```java
JSON-jsonString(toJOSNString) 
JSONObject-map集合(put) 
JSONArray-list集合(add)
    
//1.响应jsonString
	String jsonString = JSON.toJSONString(car);
    resp.getWriter().write(jsonString);
//2.响应json对象
    JSONObject data=JSONObject.parseObject(JSON.toJSONString(car));
    resp.getWriter().print(jsonString);
//3.JSONObject对象
    JSONObject jo = new JSONObject();
    jo.put("message", "用户名可注册!");
    resp.getWriter().print(jo);
//4.JSONArray数组
    JSONObject jo1 = new JSONObject();
    JSONObject jo2 = new JSONObject();
    JSONArray ja = new JSONArray();
    ja.add(jo1);
    ja.add(jo2);
```

谷歌的gson包解析json:

```java
Gson gson = new Gson();
String json = gson.toJson(car);
resp.getWriter().write(json);
```

### 二.原生Ajax

ajax实现步骤：

- 创建ajax
- 设置onreadystatechange回调方法
- 调用open(method,url,async)
- 调用send(data)，data使用于post方式并且需要设置requestHeader

#### 1.概念

ajax局部刷新技术.不是一门新技术,是多种技术的组合.是浏览器端的技术.用来实现在当前结果页中显示其他请求的响应内容. 下面ajax的基本流程是原始使用方法.一般会用jquery封装的ajax请求方法.

ajax原理: 请求由ajax引擎对象发送，响应数据，浏览器不会直接进行处理，而是流转给发请求的ajax引擎对象。这样我们可以通过操作ajax引擎对象变相的实现在页面中显示新的响应资源.

ajax本质: js的DOM操作中的数据由程序员自己写死声明，变成从服务器动态的获取.

ajax的基本流程:

```java
//创建ajax引擎对象
//复写onreadystatement函数
	//判断ajax状态码
		//判断响应状态码
			//获取响应内容(响应内容的格式)
				//普通字符串：responseText
				//json(重点)：responseText	--分工:主要做数据传输. 其实就是讲述数据按照json的格式拼接好的字符串，方便使用eval方法. 将接受的字符串数据直接转换为js的对象

				json格式：
					var 对象名={
						属性名:属性值,
						属性名:属性值,
						……
					}

				//XML数据：responseXML.返回document对象 --分工:主要做配置文件. 通过document对象将数据从xml中获取出来
			//处理响应内容(js操作文档结构)
//发送请求
	//get请求: get的请求实体拼接在URL后面，？隔开，键值对
	ajax.open("get","url");
	ajax.send(null);
	//post请求: 有单独的请求实体
	ajax.open("post", "url");
	ajax.setRequestHeader("Content-Type","application/x-www-form-urlencoded");
	ajax.send("name=张三&pwd=123");
```

ajax的状态码:

-   ajax状态码: 
    -   0: 表示XMLHttpRequest已建立，但还未初始化，这时尚未调用open方法
    -   1: 表示open方法已经调用，但未调用send方法（已创建，未发送）
    -   2: 表示send方法已经调用，其他数据未知
    -   3: 表示请求已经成功发送，正在接受数据
    -   4: 表示数据已经成功接收
-   响应状态码:
    -   200: 表示请求成功
    -   404: 资源未找到
    -   500: 内部服务器错误

ajax的异步和同步:

-   `ajax.open(method,url,async)` 
-   async: 设置同步代码执行还是异步代码执行. true代表异步,默认是异步. false代表同步.

#### 2.示例

```javascript
function getData(){
  //创建ajax引擎对象
  var ajax;
  if(window.XMLHttpRequest){//火狐
    ajax=new XMLHttpRequest();
  }else if(window.ActiveXObject){//ie
    ajax=new ActiveXObject("Msxml2.XMLHTTP");
  }

  //复写onreadystatement函数
  ajax.onreadystatechange=function(){
    //判断Ajax状态码
    if(ajax.readyState==4){
      //判断响应状态码
      if(ajax.status==200){
        //获取响应内容
        var result=ajax.responseText;
        alert(result);
        //处理响应内容
        //获取元素对象
        var showdiv=document.getElementById("showdiv");
        showdiv.innerHTML=result;
      }else if(ajax.status==404){
        //获取元素对象
        var showdiv=document.getElementById("showdiv");
        showdiv.innerHTML="请求资源不存在";
      }else if(ajax.status==500){
        //获取元素对象
        var showdiv=document.getElementById("showdiv");
        showdiv.innerHTML="服务器繁忙";
      }
    }else{
      //获取元素对象
      var showdiv=document.getElementById("showdiv");
      showdiv.innerHTML="<img src='img/2.gif' width='200px' height='100px'/>";
    }
  }
  //发送请求
  ajax.open("get","ajax",true);
  ajax.send(null);
  alert("哈哈");
}
```

html代码:

```html
<input type="button" value="测试 " onclick="getData()"/>
<div id="showdiv"></div>
```

