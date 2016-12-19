title: jQuery实现Ajax应用
date: 2015-11-18 11:06:09
tags: jQuery
categories: note
---
# jQuery实现Ajax应用
<!--more-->

## 使用load()方法异步请求数据
使用`load()`方法通过Ajax请求加载服务器中的数据，并把返回的数据放置到指定的元素中
调用格式为`load(url,[data],[callback])`

- 参数url为加载服务器地址
- 可选项data参数为请求时发送的数据
- callback参数为数据请求成功后，执行的回调函数。

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用load()方法异步请求数据</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <style type="text/css">
            #divtest
            {
                width: 282px;
            }
            #divtest .title
            {
                padding: 8px;
                background-color:Blue;
                color:#fff;
                height: 23px;
                line-height: 23px;
                font-size: 15px;
                font-weight: bold;
            }
            ul
            {
                float: left;
                width: 280px;
                padding: 5px 0px;
                margin: 0px;
                font-size: 14px;
                list-style-type: none;
            }
            img
            {
                margin: 8px;
            }
            ul li
            {
                float: left;
                width: 280px;
                height: 23px;
                line-height: 23px;
                padding: 3px 8px;
            }
            .fl
            {
                float: left;
            }
            .fr
            {
                float: right;
            }
        </style>
    </head>
    
    <body>
        <div id="divtest">
            <div class="title">
                <span class="fl">我最爱吃的水果</span> 
                <span class="fr">
                    <input id="btnShow" type="button" value="加载" />
                </span>
            </div>
            <ul></ul>
        </div>
        
        <script type="text/javascript">
            $(function () {
                $("#btnShow").bind("click", function () {
                    var $this = $(this);
                    $("ul").html("loading");
                    $("ul").load("http://imooc.com/data/fruit_part.html",function() {
                        $this.attr("disabled", "true");
                    });
                })
            });
        </script>
    </body>
</html>
```

## 使用getJSON()方法异步加载JSON格式数据
使用`getJSON()`方法可以通过Ajax异步请求的方式，获取服务器中的数组，并对获取的数据进行解析，显示在页面中
调用格式为`jQuery.getJSON(url,[data],[callback])`或`$.getJSON(url,[data],[callback])`
其中，url参数为请求加载json格式文件的服务器地址，可选项data参数为请求时发送的数据，callback参数为数据请求成功后，执行的回调函数。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用getJSON()方法异步加载JSON格式数据</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div id="divtest">
            <div class="title">
                <span class="fl">我最喜欢的一项运动</span> 
                <span class="fr">
                    <input id="btnShow" type="button" value="加载" />
                </span>
            </div>
            <ul></ul>
        </div>
        
        <script type="text/javascript">
            $(function () {
                $("#btnShow").bind("click", function () {
                    var $this = $(this);
                    $.getJSON("http://www.imooc.com/data/sport.json",function(data){
                        $this.attr("disabled", "true");
                        $.each(data, function (index, sport) {
                            //if(index==3)
                            $("ul").append("<li>" + sport["name"] + "</li>");
                        });
    
                    });
                })
            });
        </script>
    </body>
</html>
```
