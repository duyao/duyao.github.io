title: jQuery操作DOM元素
date: 2015-11-15 10:02:26
tags: jQuery
categories: note
---
# jQuery操作DOM元素
<!--more-->
## 使用attr()方法控制元素的属性
`attr()`方法的作用是设置或者返回元素的属性

- `attr(属性名)`格式是获取元素属性名的值
- `attr(属性名，属性值)`格式则是设置元素属性名的值。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>操作元素属性</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>attr()方法设置元素属性</h3>
        <a href="http://127.0.0.1" id="a1">点我就变</a>
        <div>我改变后的地址是：<span id="tip"></span></div>
        
        <script type="text/javascript">
            $("#a1").attr("href" , "www.imooc.com");
            var $url = $("#a1").attr("href");
            $("#tip").html($url);
        </script>
    </body>
</html>
```
## 操作元素的内容
使用`html()`和`text()`方法操作元素的内容，当两个方法的参数为空时，表示获取该元素的内容，
而如果方法中包含参数，则表示将参数值设置为元素内容。

- `html()`获取的html代码，如果该内容是斜体加粗等格式，都会获取到
- `text()`获取文本内容，不会获取到内容的格式等
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>操作元素内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>html()和text()方法设置元素内容</h3>
        <div id="html"></div>
        <div id="text"></div>
        
        <script type="text/javascript">
            var $content = "<b>唉，我又变胖了！</b>";
            $("#html").html($content);
            $("#text").text($content);
        </script>
    </body>
</html>
```
## 操作元素的样式
通过`addClass()`和`css()`方法可以方便地操作元素中的样式

- `addClass()`括号中的参数为增加元素的样式名称，增加多个样式名称时，要用空格隔开
- `css()`直接将样式的属性内容写在括号中。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>操作元素样式</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>css()方法设置元素样式</h3>
        <div id="content">我穿了一件红色外衣</div>
        
        <script type="text/javascript">
            $("#content").css({"background-color":"red","color":"white"});
        </script>
        <style type="text/css">
           div
            {
                padding: 8px;
                width: 180px;
            }
            .back{
                background-color:blue;
            }
            .word{
                color:white;
            }

        </style>
        <h3>addClass()方法设置元素样式</h3>
        <div id="cont">我穿一件蓝色外衣</div>
        <script type="text/javascript">
        //名字与css中相同
            $("#cont").addClass("back word");
        </script>
        
    </body>
</html>
```

## 移除属性和样式
使用`removeAttr(name)`和`removeClass(class)`分别可以实现移除元素的属性和样式的功能，

- `removeAttr(name)`方法中参数表示移除属性名
- `removeClass(class)`方法中参数则表示移除的样式名

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>移除元素样式</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>removeClass()方法移除元素样式</h3>
        <div id="content" class="blue white">我脱下了一件蓝色外衣</div>
        <style type="text/css">
			div
			{
			    padding: 8px;
			    width: 180px;
			}
			.blue
			{
			    background-color: Blue;
			}
			.white
			{
			    color: White;
			}
		</style>
        <script type="text/javascript">
            $("#content").removeClass("blue white");
        </script>
    </body>
</html>
```
## 使用append()方法向元素内追加内容
`append(content)`方法的功能是向指定的元素中追加内容，
被追加的content参数，可以是字符、HTML元素标记，还可以是一个返回字符串内容的函数。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>append()方法追加内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>append()方法追加内容</h3>
        
        <script type="text/javascript">
            function rethtml() {
                var $html = "<div id='test' title='hi'>我是调用函数创建的</div>"
                return $html;
            }
            var $myhtml = "<div id='test' title='hi'>我是调用函数创建的</div>"
            $("body").append(rethtml());
            $("body").append($myhtml);
        </script>
    </body>
</html>
```
## 使用appendTo()方法向被选元素内插入内容
`appendTo()`方法也可以向指定的元素内插入内容，它的使用格式是：
```
$(content).appendTo(selector)
```
参数content表示需要插入的内容，参数selector表示被选的元素，即把content内容插入selector元素内，默认是在尾部
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>appendTo()方法插入内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>appendTo()方法插入内容</h3>
        <div>
            <span class="green">小乌龟</span>
        </div>
        <span class='red'>小青蛙</span>
        
        <script type="text/javascript">
            var $html = "<span class='red'>小青蛙</span>";
            //content是$html,第一个$是调用格式
            $($html).appendTo("div");
            //将class加入到div中
            $(".red").appendTo("div");
        </script>
    </body>
</html>
```

## 使用before()和after()在元素前后插入内容
使用`before()`和`after()`方法可以在元素的前后插入内容，
它们分别表示在整个元素的前面和后面插入指定的元素或内容，调用格式分别为：
```
$(selector).before(content)和$(selector).after(content)
```
其中参数content表示插入的内容，该内容可以是元素或HTML字符串。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>在元素前后插入内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>after()方法在元素之后插入内容</h3>
        <span class="green">我们交个朋友吧！</span>
        
        <script type="text/javascript">
            var $html = "<span class='red'>兄弟。</span>"
            $(".green").after($html);
            $(".green").before($html);
        </script>
    </body>
</html>
```

## 使用clone()方法复制元素
调用`clone()`方法可以生成一个被选元素的副本，
即复制了一个被选元素，包含它的节点、文本和属性，它的调用格式为：
```
$(selector).clone()
```
其中参数selector可以是一个元素或HTML内容。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用clone()方法复制元素</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>使用clone()方法复制元素</h3>
        <span class="red" title="hi">我是美猴王</span>
        
        <script type="text/javascript">
            $("body").append($(".red").clone());
        </script>
    </body>
</html>
```

## 替换内容
`replaceWith()`和`replaceAll()`方法都可以用于替换元素或元素中的内容，但它们调用时，内容和被替换元素所在的位置不同，分别为如下所示：

- `$(selector).replaceWith(content)`
- `$(content).replaceAll(selector)`
参数selector为被替换的元素，content为替换的内容。

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用替换元素和内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>使用replaceAll()方法替换元素内容</h3>
        <span class="green" title="hi">我是屌丝</span>
        <script type="text/javascript">
            var $html = "<span class='red' title='hi'>我是土豪</span>";
            $($html).replaceAll(".green");
            $(".green").replaceWith($html);
        </script>
    </body>
</html>
```

## 使用wrap()和wrapInner()方法包裹元素和内容
`wrap()`和`wrapInner()`方法都可以进行元素的包裹

- `wrap()`用于包裹元素本身
- `wrapInner()`用于包裹元素中的内容
调用格式分别为：
```
$(selector).wrap(wrapper)
$(selector).wrapInner(wrapper)
```
参数selector为被包裹的元素，wrapper参数为包裹元素的格式。

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>包裹元素和内容</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>   
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>使用wrapInner()方法包裹元素</h3>
        <span class="red" title='hi'>我的身体有点歪</span>
        
        <script type="text/javascript">
            $(".red").wrapInner("<i></i>");
        </script>
    </body>
</html>
```

## 使用each()方法遍历元素
使用each()方法可以遍历指定的元素集合，在遍历时，通过回调函数返回遍历元素的序列号，它的调用格式为：
```
$(selector).each(function(index))
```
参数function为遍历时的回调函数，index为遍历元素的序列号，它从0开始。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用each()方法遍历元素</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>使用each()方法遍历元素</h3>
        <span class="green">香蕉</span>
        <span class="green">桃子</span>
        <span class="green">葡萄</span>
        <span class="green">荔枝</span>
        
        <script type="text/javascript">
            $("span").each(function (index) {
                if (index == 1) {
                    $(this).attr("class", "red");
                }
            });
        </script>
    </body>
</html>
```

## 使用remove()和empty()方法删除元素

- `empty()` will remove all the contents of the selection.
- `remove()` will remove the selection and its contents.
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>删除元素</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>    
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        
        <h3>使用empty()方法删除元素</h3>
        <span class="green">香蕉</span>
        <span class="green">桃子</span>
        <span class="red">葡萄</span>
        <span class="green">葡萄</span>
        <span class="red">荔枝</span>
        <span class="green">荔枝</span>
        
        
        <script type="text/javascript">
            //删除.red所在的元素,包括span
            $(".red").remove();
            //只是删除了内容，span还在
            $(".green").empty();
        </script>
        
    </body>
</html>
```
`style.css`
```
span
{
    color: White;
    padding: 8px;
    margin: 5px;
    float: left;
}
.green
{
    background-color: Green;
}

```