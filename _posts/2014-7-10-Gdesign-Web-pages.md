---
layout: post
title: Gdesign - Web Pages
---

## 概述

本文主要讲 Gdesign 中的以下内容：

- 前端 与 JavaScript
- 后端 与 数据库

##1、前端页面

在前端与 JavaScript 中，内容可以理解为1-4-1，具体为：

- 1个 CSS
- 4个 jQuery 插件
- 1个 AJAX 操作

**1.1 CSS 用于构建动态菜单栏：**

{% highlight css linenos %}
.wrapper {...}  

#menu {...}  
#menu a {...} 
#menu li:first-child a {...} 
#menu li:last-child a {...} 
#menu a:hover {...}  
#menu a:active {...}  
{% endhighlight %}

**1.2 jQuery 插件的使用**

4个 jQuery 插件为：omniWindows、morris、fotorama 与 zebra_datepicker。

- omniWindows ： 登陆框弹窗插件

在调用 omniWindows 的 HTML 中，需要声明 DIV 的 class 为 `ow-overlay ow-closed` 与 `window ow-closed` 。
并在 `window ow-closed` 中构建需要的弹窗。

在使用中，omniWindows 对应的 CSS 属性写在 `index.css` 中，对应的 HTML 调用代码如下。

{% highlight html linenos %}
<div class="ow-overlay ow-closed"></div>
<div class="window ow-closed">
... //想要调用的内容
</div>
{% endhighlight %}

- Morris ： 曲线绘制插件

在调用 Morris 时，包含 Morris 的 JS 文件，这里可以根据服务器环境选择官网的做法，也可以直接下载 JS 文件。

[Morris官网](http://morrisjs.github.io/morris.js/) 有很不错的入门教程，想要深入了解更多的定制信息，则需要进入顶部菜单栏的分页中。

{% highlight html linenos %}
<link href="css/morris.css" rel="stylesheet">
<script src="js/jquery-1.8.2.min.js"></script>
<script src="js/morris.js"></script>
{% endhighlight %}

调用 Morris 代码如下。

{% highlight js linenos %}
new Morris.Line({
		element: 'desktop',
		data: data_json, 
	  
		xkey: 'time',
		ymax: 'auto',
		
		ykeys: ykey_arr,
		labels: label_arr,

		lineColors: ['#000'],
		pointFillColors: ['#F00'],
	});
{% endhighlight %}

- Fotorama : 图片画廊插件

[Fotarama官网](http://fotorama.io/) 百度关键词 fotoroma 就能找到，较好的插件官网上总有很多调用教程和 demo，而且还有许多深入文档，学习 fotorama 就需要学习很多的定制信息（[Customize](http://fotorama.io/customize/)）。

调用一般的 JS 文件，步骤都是：添加引用、调用。

声明代码如下：

{% highlight html linenos %}
<script src="js/jquery-1.8.2.min.js"></script>
<link href="css/fotorama.css" rel="stylesheet">
<script src="js/fotorama.js"></script>
{% endhighlight %}

调用代码如下：

{% highlight js linenos %}
$(".fotorama").fotorama({
		width: '100%',
		height: '68%',
		fit: 'cover',
		allowfullscreen: true,
		nav: 'thumbs',
	});
{% endhighlight %}

在调用中需要注意的是，由于 fotorama 是首次构建 CSS 时创建的，所以不接受动态创建的 DIV 直接声明使用，而需要使用 innerHTML 来创建，否则无法获取到对应图片信息。

{% highlight js linenos %}
for (var i = 0; i < data_json.length; i ++) {
	img_box.innerHTML += '<img src="' + data_json[i].sdir + '"\
	data-caption="' + data_json[i].time + '">'
	}
{% endhighlight %}

- zebra_datepicker：日期选择器

[zebra_datepicker官网](http://stefangabos.ro/jquery/zebra-datepicker/)，zebre_datepicker 是基于 raphael 在定制的一套日期选择器。

声明代码如下：

{% highlight html linenos %}
<link href="css/fotorama.css" rel="stylesheet">
<script src="js/raphael-2.1.0.min.js"></script>
<script src="js/zebra_datepicker.js"></script>
{% endhighlight %}

调用代码如下：

{% highlight js linenos %}
$('#datepicker').Zebra_DatePicker({
		disabled_dates: ['* * * *'],
		enabled_dates: eval(ret_json),
		onSelect: function(date) {
			dvar = date

			var td = new Date().Format("yyyy-MM-dd")

			if (dvar != td) {
				remove_status()
			}

			if (tbvar == "camer") {
				show_img(tbvar, dvar)
			} else {
				remove_status()
				show_data(tbvar, dvar)
			}
		}
	});
{% endhighlight %}

```
TIPS: zebra_datepicker 的日期格式需为 JSON 格式。
```

**1.3 AJAX 操作**

实现 AJAX ，数据传输的主要流程为： 前端页面操作、后端数据传输 与 数据库数据获取。

- 前端页面操作

在前端页面中，主要通过构建菜单栏操作函数实现。

{% highlight js linenos %}
$("#menu").click( function() { ...
});
{% endhighlight %}

AJAX 使用的 API 主要通过使用 getdb、setdb 实现获取数据、设置数据。

{% highlight js %}
function getdb(table, day) { }
{% endhighlight %}

在 Gdesign.js 中，在 getdb、setdb 外还封装了一层调用函数，实现对不同的菜单的相应，调用三个函数原型如下：

{% highlight js linenos %}
function show_data(tbvar, day) { ... }
function show_img(tbvar, day) { ... }
function controls(tbvar, gdatas) { ... }
{% endhighlight %}

参数 `tbvar` 即为 "table variable" 的意思，对应于不同的菜单 ID。          
参数 `day` 即为当前日期，若使用 zebra_picker 筛选出不同的的可选日期。     
参数 `gdatas` 本质上就是字符串  "getdatas" 。

getdb 的 AJAX 操作如下，在操作中仅使用 GET 操作，通过 tbvar -> table 的不同参数传递，实现对不同 PHP 文件的选择。相关的 AJAX 学习，可以参考 [W3CSchool AJAX 教程](http://www.w3school.com.cn/ajax/index.asp)。

{% highlight js linenos %}
function getdb(table, day) { 
	xmlHttp=GetXmlHttpObject()
	if (xmlHttp==null) {
		alert ("Browser does not support HTTP Request")
		return
	}

	var url = "php/" + table + ".php?req=" + day
	xmlHttp.onreadystatechange=retValues
	xmlHttp.open("GET", url, false)
	xmlHttp.send(null)
}
{% endhighlight %}

- 后端数据传输

在后端数据传输中，主要依靠 PHP 对数据库实现操作。在 PHP 中，其核心在于参数的获取 以及连接数据库、操作数据库。

{% highlight js linenos %}
$req=$_GET["req"];

if (!$con)
{
die('Could not connect: ' . mysql_error());
}

$sql = "";
$result = mysql_query($sql);
...
{% endhighlight %}

获取后的数据以 `echo` 的方式打印在页面上，而 js 文件通过 `GetXmlHttpObject` 来获取页面上的数据。

- 数据库数据获取

详见论文3.5节。

&copy; 2014 plinx
