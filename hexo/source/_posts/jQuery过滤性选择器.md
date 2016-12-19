title: jQuery过滤性选择器
date: 2015-11-14 10:40:51
tags: jQuery
categories: note
---
# jQuery过滤性选择器
书写时以`:`号开头,通常用于查找集合元素中的某一位置的单个元素
<!--more-->
## :first过滤选择器

`:first`获得相同标签的第1个元素
`:last`获得相同标签的第1个元素

```
<html>
    <head>
        <title>:first过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>改变最后一行"苹果"背景颜色：</div>
        <ol>
            <li>葡萄</li>
            <li>香蕉</li>
            <li>橘子</li>
            <li>西瓜</li>
            <li>苹果</li>
        </ol>
        
        <script type="text/javascript">
        //最后一个li变色
            $("li:last").css("background-color", "red");
        </script>
    </body>
</html>
```

## :eq(index)过滤性选择器

`:eq(index)`可以选择任意的标签元素
`index`是索引号，从0开始

```
<html>
    <head>
    <title>:eq(index)过滤选择器</title>
    <link href="style.css" rel="stylesheet" type="text/css" />
    <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    
    <body>
        <div>改变中间行"葡萄"背景颜色：</div>
        <ol>
        <li>橘子</li>
        <li>香蕉</li>
        <li>葡萄</li>
        <li>苹果</li>
        <li>西瓜</li>
        </ol>
        
        <script type="text/javascript">
            $("li:eq(2)").css("background-color", "#60F");
        </script>
    </body>
</html>
```

## :contains(text)过滤选择器
`:contains(text)`按照文本内容查找元素，包含指定字符串`text`的的全部内容
```
<html>
    <head>
        <title>:contains(text)过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>改变包含"jQuery"字符内容的背景色：</div>
        <ol>
            <li>强大的"jQuery"</li>
            <li>"javascript"也很实用</li>
            <li>"jQuery"前端必学</li>
            <li>"java"是一种开发语言</li>
            <li>前端利器——"jQuery"语言</li>
        </ol>
        
        <script type="text/javascript">

            $("li:contains('jQuery')").css("background", "green");
        </script>
    </body>
</html>
```

**注意**`$("li:contains('jQuery')")`中`'jQuery'`带单引号，是因为jQuery是不是变量，不能用双引号

## :has(selector)过滤选择器
`:has(selector)`通过元素名称来选择元素,不是文本！

```
<html>
    <head>
        <title>:has(selector)过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <div>改变包含"label"元素的背景色：</div>
        <ol>
            <li><p>我是P先生</p></li>
            <li><label>L妹纸就是我</label></li>
            <li><p>我也是P先生</p></li>
            <li><label>我也是L妹纸哦</label></li>
            <li><p>P先生就是我哦</p></li>
        </ol>
        
         <script type="text/javascript">
            $("li:has('label')").css("background-color", "blue");
        </script>
    </body>
</html>
```

## :hidden过滤选择器
`:hidden`过滤选择器的功能是获取全部不可见的元素，这些不可见的元素中包括type属性值为hidden的元素。
```
<!DOCTYPE html>
<html>
    <head>
        <title>:hidden过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    
    <body>
        <h3>显示隐藏元素的内容</h3>
        <input id="hidstr" type="hidden" value="我已隐藏起来"/>
        <div></div>
        
        <script type="text/javascript">
        var $strHTML = $("input:hidden").val();
        $("div").html($strHTML);
    </script>
    </body>
</html>
```
## :visible过滤选择器
与`:hidden`过滤选择器相反，`:visible`过滤选择器获取的是全部可见的元素，也就是说，只要不将元素的display属性值设置为“none”，那么，都可以通过该选择器获取。
```
<!DOCTYPE html>
<html>
    <head>
        <title>:visible过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>修改可见“水果”的背景色</h3>
        <ul>
            <li style="display:none">橘子</li>
            <li style="display:block">香蕉</li>
            <li style="display:">葡萄</li>
            <li>苹果</li>
            <li style="display:none">西瓜</li>
        </ul>
        
        <script type="text/javascript">
            $("li:visible").css("background-color","blue");
        </script>
    </body>
</html>
```
## [attribute=value]属性选择器
`[attribute=value]`属性选择器的功能是获取与属性名和属性值完全相同的全部元素，
其中[]是专用于属性选择器的括号符，参数attribute表示属性名称，value参数表示属性值。
```
<!DOCTYPE html>
<html>
    <head>
        <title>[attribute=value]属性选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>    
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>改变"title"属性值为"蔬菜"的背景色</h3>
        <ul>
            <li title="蔬菜">茄子</li>
            <li title="水果">香蕉</li>
            <li title="蔬菜">芹菜</li>
            <li title="水果">苹果</li>
            <li title="水果">西瓜</li>
        </ul>
        
        <script type="text/javascript">
            $("li[title='蔬菜']").css("background-color", "green");
        </script>
    </body>
</html>
```
## [attribute*=value]属性选择器
`[attribute*=value]`可以获取属性值中包含指定内容的全部元素
```
<!DOCTYPE html>
<html>
    <head>
        <title>[attribute*=value]属性选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" role="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>改变"title"属性值包含"果"的背景色</h3>
        <ul>
            <li title="蔬菜">茄子</li>
            <li title="水果">香蕉</li>
            <li title="蔬菜">芹菜</li>
            <li title="人参果">小西红柿</li>
            <li title="水果">西瓜</li>
        </ul>
        
        <script type="text/javascript">
            $("li[title*='果']").css("background-color", "green");
        </script>
    </body>
</html>
```
## :first-child子元素过滤选择器
`:first`过滤选择器可以获取指定父元素中的首个子元素，但该选择器返回的只有一个元素，并不是一个集合，
而使用`:first-child`子元素过滤选择器则可以获取每个父元素中返回的首个子元素，它是一个集合，常用多个集合数据的选择处理。
**注意**`:`与`first-child`无空格
``` 
<!DOCTYPE html>
<html>
    <head>
        <title>:first-child子元素过滤选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>改变每个"蔬菜水果"中第一行的背景色</h3>
        <ol>
            <li>芹菜</li>
            <li>茄子</li>
            <li>萝卜</li>
            <li>大白菜</li>
            <li>西红柿</li>
        </ol>
        <ol>
            <li>橘子</li>
            <li>香蕉</li>
            <li>葡萄</li>
            <li>苹果</li>
            <li>西瓜</li>
        </ol>
        
        <script type="text/javascript">
        //li中的第一个
            $("li:first-child").css("background-color", "green");
            //li中最后一个
            $("li:last-child").css("background-color", "green");
        </script>
    </body>
</html>
```


