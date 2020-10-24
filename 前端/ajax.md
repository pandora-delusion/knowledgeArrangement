# AJAX

**AJAX 是一种在无需重新加载整个网页的情况下，能够更新部分网页的技术。**

AJAX = 异步 JavaScript 和 XML。

通过在后台与服务器进行少量数据交换，AJAX 可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

传统的网页（不使用 AJAX）如果需要更新内容，必需重载整个网页面。

#### XMLHttpRequest对象

所有现代浏览器均支持 XMLHttpRequest 对象，除了IE5和IE6

XMLHttpRequest 用于在后台与服务器交换数据

##### 创建XMLHttpRequest对象

```javaScript
var xmlhttp;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
```

##### 向服务器发送请求

```javascript
// 两个方法都要调用
xmlhttp.open("GET","test1.txt",true);
xmlhttp.send();
```

1. open(method, url, async)：规定请求的类型、URL以及是否异步处理请求

   method：请求的类型；GET或者POST

   url：文件在服务器上的位置

   async：true(异步)或false(同步)

2. send(string)：将请求发送到服务器

   string：仅用于POST请求

**GET还是POST**

与 POST 相比，GET 更简单也更快，并且在大部分情况下都能用。

然而，在以下情况中，请使用 POST 请求：

- 无法使用缓存文件（更新服务器上的文件或数据库）
- 向服务器发送大量数据（POST 没有数据量限制）
- 发送包含未知字符的用户输入时，POST 比 GET 更稳定也更可靠

```JavaScript
<html>
<head>
<script type="text/javascript">
function loadXMLDoc()
{
var xmlhttp;
if (window.XMLHttpRequest)
  {// code for IE7+, Firefox, Chrome, Opera, Safari
  xmlhttp=new XMLHttpRequest();
  }
else
  {// code for IE6, IE5
  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
  }
xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
    document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
  }
// get请求
xmlhttp.open("GET","/ajax/demo_get.asp",true);
xmlhttp.send();
}

// post请求
xmlhttp.open("POST","/ajax/demo_post.asp",true);
xmlhttp.send();
</script>
</head>
<body>

<h2>AJAX</h2>
<button type="button" onclick="loadXMLDoc()">请求数据</button>
<div id="myDiv"></div>

</body>
</html>
```

如果需要像 HTML 表单那样 POST 数据，请使用 setRequestHeader() 来添加 HTTP 头。然后在 send() 方法中规定您希望发送的数据：

```JavaScript
xmlhttp.open("POST","ajax_test.asp",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("fname=Bill&lname=Gates");
```

setRequestHeader(header, value)：向请求添加HTTP头

对于 web 开发人员来说，发送异步请求是一个巨大的进步。很多在服务器执行的任务都相当费时。AJAX 出现之前，这可能会引起应用程序挂起或停止。

**异步**

通过 AJAX，JavaScript 无需等待服务器的响应，而是：

- 在等待服务器响应时执行其他脚本
- 当响应就绪后对响应进行处理

**Async = true**

当使用 async=true 时，请规定在响应处于 onreadystatechange 事件中的就绪状态时执行的函数:

```JavaScript
xmlhttp.onreadystatechange=function()
  {
  if (xmlhttp.readyState==4 && xmlhttp.status==200)
    {
    document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
    }
  }
xmlhttp.open("GET","test1.txt",true);
xmlhttp.send();
```

**Async = false**

我们不推荐使用 async=false，但是对于一些小型的请求，也是可以的。

请记住，JavaScript 会等到服务器响应就绪才继续执行。如果服务器繁忙或缓慢，应用程序会挂起或停止。

**注释：**当您使用 async=false 时，请不要编写 onreadystatechange 函数 - 把代码放到 send() 语句后面即可

##### 服务器响应

如需获得来自服务器的响应，请使用 XMLHttpRequest 对象的 responseText 或 responseXML 属性。

##### onreadystatechange事件

onreadystatechange：存储函数（或者函数名），每当readyState属性改变时，就会调用该函数

readyState：存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。

- 0： 请求未初始化
- 1：服务器连接已建立
- 2：请求已接受
- 3：请求处理中
- 4：请求已完成，且响应已就绪

status：200：“OK”， 404：未找到页面