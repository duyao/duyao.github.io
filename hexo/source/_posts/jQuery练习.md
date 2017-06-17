title: jQuery练习
date: 2015-11-14 16:09:28
tags: jQuery
categories: note
---
# p1
## 题目
在页面中,添加一个`<ul>`元素,里面放置多个(至少7个以上)的``<li>`元素,此外,再添加一个`<a>`元素.

- 初始时:`<ul>`元素中仅显示5个`<li>`元素,其中包含还包括最后一个`<li>`元素,<a>元素中的显示"更多"字符.

- 当点击"更多"链接时,自身内容变为"简化",同时,`<ul>`元素中显示全部的`<li>`元素.

- 当点击"简化"链接时,自身内容变为"更多",同时,`<ul>`元素中仅显示包含最后一个`<li>`元素在内的5个元素.

## 代码
```
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
          <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
      
        <title>挑战题</title>
    </head>
    
    <body>
    <ul>
        <li>1</li>
        <li>2</li>
        <li>3</li>
        <li>4</li>
        <li class="1" style="display:none">5</li>
        <li class="1" style="display:none">6</li>
        <li >7</li>
        <a id="aa" href="#" onclick="fun()">更多</a>
    </ul>
    
    <script type="text/javascript">
    function fun(){
       var $str=$("#aa").html();

        if($str=="更多"){
            $("#aa").html("简化");
            $("li:hidden").css('display','block');
        }
        else if($str=="简化"){
            $("#aa").html("更多");
            //$(".1").css('display','none');
            $("li[class*='1']").css('display','none');


        }
        
    }
        
    </script>
    </body>
</html>
```
# p2
## 题目
在页面中，创建两个按钮。
点击第一个“左移”按钮后，将页面中的<div>元素在当前的位置上，以动画的效果向左移动50个像素；
点击第二个“右移”按钮后，页面中的<div>元素在当前的位置上，以动画的效果向右移动50个像素。

## 代码
```
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
         <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <title>jQuery动画特效</title>
         <style type="text/css">
            div
            {
                position:absolute;
                width:80px;
                height:80px;
                border: solid 1px #ccc;
                margin: 0px 8px;
                background-color: Red;
                color:White;
                vertical-align:middle
            }
        </style>
    </head>
    <body>
        <input type="button" id="b1" value="左移">
        <input type="button" id="b2" value="右移">
        <div>I'm div</div>
        <script type="text/javascript">
        $(function () {
            $("#b1").bind("click",function () {
                $("div").animate({
                    left: "-=50px"
                }, "fast")
            })

            $("#b2").bind("click",function  () {
                $("div").animate({
                    left: "+=50px"
                },3000)
            })

        })
        
        </script>
    </body>
</html>
```