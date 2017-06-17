title: jQuery基础选择器
date: 2015-11-13 20:59:22
tags: jQuery
categories: note
---
# jQuery基础选择器
<!--more-->
## id选择器-#id

`$(#id)`根据id查找元素 
```
<html>
    <head>
        <title>#id选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    <body>
        <div id="divtest">div的内容</div>
        <div id="default"></div>
        <script type="text/javascript">
        //default显示div的内容
            $("#default").html($("#divtest").html());
        </script>
    </body>
</html>
```

## element选择器
`$("element")`根据元素名查找元素
相当于从铅笔盒里找铅笔
```
<html>
    <head>
        <title>element选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    <body>
        <button id="btntest">点我</button>
        <script type="text/javascript">
        //button变灰
            $("button").attr("disabled","true");
        </script>
    </body>
</html>
```
## .class 选择器
`$(".class")`根据元素的类别属性朝找元素
相当于从铅笔盒中找红色铅笔
```
<html>
    <head>
        <title>.class选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <div class="red">立正，向我这边看齐</div>
        <div class="green">我先歇歇脚</div>
        <script type="text/javascript">
        //先取得红色的字
            var $redHTML = $(".red").html();
            //让绿色和红色相同
            $(".green").html($redHTML);
        </script>
    </body>
</html>
```
## * 选择器
`$("*")`获取页面全部元素
相当于取走全部铅笔
慎用，反应缓慢
```
<html>
    <head>
        <title>*选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    <body>
        <form action="#">
        <input id="Button1" type="button" value="button" />
        <input id="Text1" type="text" />
        <input id="Radio1" type="radio" />
        <input id="Checkbox1" type="checkbox" />
        </form>
        <script type="text/javascript">
        //使form表单中所有元素都不可用
            $("form *").attr("disabled", "true");
        </script>
    </body>
</html>
```
## sele1,sele2,seleN选择器
`$("sele1,sele2,seleN")`选择多个选择器
可以是`$("#id")`,`$(".class")`.`$("selector")`等
相当于从铅笔盒中选择任意多个铅笔
```
<html>
    <head>
        <title>sele1,sele2,seleN选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <div class="red">选我吧！我是red</div>
        <div class="green">选我吧！我是green</div>
        <div class="blue">选我吧！我是blue</div>
        
        <script type="text/javascript">
        //红绿变
            $(".red,.green").html("hi,我们的样子很美哦!");
        </script>
    </body>
</html>
```
## ance desc选择器
`$("ance desc")`通过层次选择器将多个元素选中
ance是父元素，desc是子元素，中间用空格分开
不管是孙子辈还是孩子辈，只要有就选择

```
<html>
    <head>
        <title>ance desc选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>码农家族
            <p>
               <label></label>
            </p>
            <label></label>
        </div>
        
        <script type="text/javascript">
        //只要是div下面的lable都会改变
            $("div label").css("background-color","blue");
        </script>
    </body>
</html>
```
## parent > child选择器
`$("parent > child")`选择孩子辈的子集元素，只有孩子辈，不包括孙子等
```
<html>
    <head>
        <title>parent > child选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>
            码农家族
            <p>
                <label></label>
            </p>
            <label></label>
            <label></label>
        </div>
        <label></label>
        
        <script type="text/javascript">
        //只有div的孩子label才改变
            $("div > label").css("border", "solid 5px red");
        </script>
    </body>
</html>
```
## prev + next选择器
`$("prev + next")`选择与`prev`邻近的下一个元素，且只有一个
```
<html>
    <head>
        <title>prev + next选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>
            码农家族
            <label></label>
            <p></p>
            <label></label>
            <label></label>
        </div>
        <label></label>
        
        <script type="text/javascript">
            $("p + label").css("background-color","red");
        </script>
    </body>
</html>
```
## prev ~ siblings选择器
`$("prev ~ sibling")`获取当前元素的多个相邻元素，但是只是后面的，不包括前面和本身
```
<html>
    <head>
        <title>prev ~ siblings选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>
            码农家族
            <label></label>
            <p></p>
            <label></label>
            <label></label>
        </div>
        <label></label>
        
        <script type="text/javascript">
        //获取当前元素的后面的所有label
            $("p ~ label").css("border", "solid 1px red");
            $("p ~ label").html("我们都是p先生的粉丝");
        </script>
    </body>
</html>
```

