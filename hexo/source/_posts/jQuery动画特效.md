title: jQuery动画特效
date: 2015-11-18 09:17:16
tags: jQuery
categories: note
---
# jQuery动画特效
<!--more-->
## 调用show()和hide()方法显示和隐藏元素
`show()`和`hide()`方法用于显示或隐藏页面中的元素
```
$(selector).hide(speed,[callback])
$(selector).show(speed,[callback])
```
参数speed设置隐藏或显示时的速度值，可为“slow”、“fast”或毫秒数值，可选项参数callback为隐藏或显示动作执行完成后调用的函数名。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>show()和hide()方法动画方式显示和隐藏元素</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>show()和hide()方法动画方式显示和隐藏元素</h3>
        <div>
            <h4>我喜欢吃的水果</h4>
            <ul>
                <li>苹果</li>
                <li>甘桔</li>
                <li>梨</li>
            </ul>
            <input id="hidval" type="hidden" value="0"/>
            <input id="button" type="button" value="show"/>
         </div>
        <script type="text/javascript">
            $(function () {
                $("h4").bind("click", function () {
                    if ($("#hidval").val() == 0) {
                    	//设置时间及函数
                        $("ul").show(3000,function(){
                            $("#hidval").val(1);
                            //无参数
                            $("#button").hide();
                        })
                    } else {
                        $("ul").hide(3000,function(){
                            $("#hidval").val(0);
                            $("#button").show();
                        })
                    }
                })
            });
        </script>
    </body>
</html>
```

## 调用toggle()方法实现动画切换效果
实现元素的显示与隐藏需要使用`hide()`与`show()`，那么有没有更简便的方法来实现同样的动画效果呢？
调用`toggle()`方法就可以很容易做到

- 如果元素处于显示状态，调用该方法则隐藏该元素
- 如果元素处于隐藏状态，调用该方法则显示该元素
它的调用格式是：
`$(selector).toggle(speed,[callback])`
其中speed参数为动画效果时的速度值，可以为数字，单位为毫秒，也可是“fast”、“slow”字符，可选项参数callback为方法执行成功后回调的函数名称。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>toggle()方法的动画切换效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>toggle()方法的动画切换效果</h3>
        <div>
            <h4>
               <span class="fl">我喜欢吃的水果</span>
               <span class="fr" id="spnTip">显示</span>
            </h4>
            <ul>
                <li>苹果</li>
                <li>甘桔</li>
                <li>梨</li>
            </ul>
        </div>
        
        <script type="text/javascript">
            $(function () {
                var $spn = $("#spnTip");
                $("h4").bind("click", function () {
                    $("ul").toggle("slow",function(){
                     $spn.html() == "隐藏" ? $spn.html("显示") : $spn.html("隐藏");
                    })
                });
            });
        </script>
    </body>
</html>
```

## 使用slideUp()和slideDown()方法的滑动效果
可以使用`slideUp()`和`slideDown()`方法在页面中滑动元素

- `slideUp()`用于向上滑动元素
- `slideDown()`用于向下滑动元素
调用方法分别为：
`$(selector).slideUp(speed,[callback])`和`$(selector).slideDown(speed,[callback])`
其中speed参数为滑动时的速度，单位是毫秒，可选项参数callback为滑动成功后执行的回调函数名。
要注意的是：`slideDown()`仅适用于被隐藏的元素；`slideup()`则相反。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用slideUp()和slideDown()方法的滑动效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>使用slideUp()和slideDown()方法的滑动效果</h3>
        <div>
            <h4>我喜欢吃的水果</h4>
            <ul>
                <li>苹果</li>
                <li>甘桔</li>
                <li>梨</li>
            </ul>
            <input id="hidval" type="hidden" value="0"/>
        </div>
        
        <script type="text/javascript">
            $(function () {
                $("h4").bind("click", function () {
                    if ($("#hidval").val() == 0) {
                        $("ul").slideDown("slow",function() {
                            $("#hidval").val(1);
                        })
                    } else {
                        $("ul").slideUp("slow",function(){
                            $("#hidval").val(0);
                        })
                    }
                })
            });
        </script>
    </body>
</html>
```

## 使用slideToggle()方法实现图片“变脸”效果
使用`slideToggle()`方法可以切换`slideUp()`和`slideDown()`
即调用该方法时，如果元素已向上滑动，则元素自动向下滑动，反之，则元素自动向上滑动
格式为`$(selector).slideToggle(speed,[callback])`
其中speed参数为动画效果时的速度值，可以为数字，单位为毫秒，也可是“fast”、“slow”字符，可选项参数callback为方法执行成功后回调的函数名称。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用slideToggle()方法切换滑动效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>使用slideToggle()方法切换滑动效果</h3>
        <div>
            <h4>
               <span class="fl">我喜欢吃的水果</span>
               <span class="fr" id="spnTip">向下滑</span></h4>
            <ul>
                <li>苹果</li>
                <li>甘桔</li>
                <li>梨</li>
            </ul>
            <input id="hidval" type="hidden" value="0"/>
        </div>
        <script type="text/javascript">
            $(function () {
                var $spn = $("#spnTip");
                $("h4").bind("click", function () {
                    $("ul").slideToggle("slow",function() {
               $spn.html() == "向下滑" ? $spn.html("向上滑") : $spn.html("向下滑");
                    })
                })
            });
        </script>
    </body>
</html>
```

## 使用fadeIn()与fadeOut()方法实现淡入淡出效果
`fadeIn()`和`fadeOut()`方法可以实现元素的淡入淡出效果，前者淡入隐藏的元素，后者可以淡出可见的元素
它们的调用格式分别为`$(selector).fadeIn(speed,[callback])`和`$(selector).fadeOut(speed,[callback])`
其中参数speed为淡入淡出的速度，callback参数为完成后执行的回调函数名。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用fadeIn()与fadeOut()方法实现元素淡入淡出的效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>使用fadeIn()与fadeOut()方法实现元素淡入淡出的效果</h3>
        <div>
            <h4>我喜欢吃的水果</h4>
            <ul>
                <li>苹果</li>
                <li>甘桔</li>
                <li>梨</li>
            </ul>
            <input id="hidval" type="hidden" value="0"/>
        </div>
        
        <script type="text/javascript">
            $(function () {
                $("h4").bind("click", function () {
                    if ($("#hidval").val() == 0) {
                        $("ul").fadeIn("slow",function() {
                            $("#hidval").val(1);
                        })
                    } else {
                        $("ul").fadeOut(3000,function() {
                            $("#hidval").val(0);
                        })
                    }
                })
            });
        </script>
    </body>
</html>
```

## 使用fadeTo()方法设置淡入淡出效果的不透明度
调用`fadeTo()`方法，可以将所选择元素的不透明度以淡入淡出的效果调整为指定的值
该方法的调用格式为`$(selector).fadeTo(speed,opacity,[callback])`
其中speed参数为效果的速度，opacity参数为指定的不透明值，它的取值范围是0.0~1.0，可选项参数callback为效果完成后，回调的函数名。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>使用fadeTo()方法设置淡入淡出效果的不透明度</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <style type="text/css">
            span
            {
                width: 70px;
                height: 70px;
                float: left;
                border: solid 1px #ccc;
                margin: 0px 8px;
            }
            .red
            {
                background-color: Red;
            }
            .orange
            {
                background-color:Orange;
            }
            .blue
            {
                background-color: Blue;
            }
        </style>
    </head>
    
    <body>
        <h3>使用fadeTo()方法设置淡入淡出效果的不透明度</h3>
        <span class="red"></span><span class="orange"></span><span class="blue"></span>
        
        <script type="text/javascript">
            $(function () {
                $("span").each(function (index) {
                    switch (index) {
                        case 0:
                            $(this).fadeTo(3000,0.3);
                            break;
                        case 1:
                            $(this).fadeTo(3000,0.6);
                            break;
                        case 2:
                            $(this).fadeTo(3000,0.9);
                            break;
                    }
                });
            });
        </script>
    </body>
</html>
```

## 调用animate()方法制作简单的动画效果
调用`animate()`方法可以创建自定义动画效果
调用格式为`$(selector).animate({params},speed,[callback])`
其中，params参数为制作动画效果的CSS属性名与值，speed参数为动画的效果的速度，单位为毫秒，可选项callback参数为动画完成时执行的回调函数名。

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>制作简单的动画效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <style type="text/css">
            span
            {
                float: left;
                border: solid 1px #ccc;
                margin: 0px 8px;
                background-color: Blue;
                color:White;
                vertical-align:middle
            }
        </style>
    </head>
    
    <body>
        <h3>制作简单的动画效果</h3>
        <span></span>
        <div id="tip"></div>
        
        <script type="text/javascript">
            $(function () {
                $("span").animate({
                    width: "80px",
                    height: "80px"
                },
                3000, function () {
                    $("#tip").html("执行完成!");
                });
            });
        </script>
    </body>
</html>
```

## 调用animate()方法制作移动位置的动画
调用`animate()`方法不仅可以制作简单渐渐变大的动画效果，而且还能制作移动位置的动画
在移动位置之前，必须将被移元素的“position”属性值设为“absolute”或“relative”，否则，该元素移动不了。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>制作移动位置的动画</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <style type="text/css">
			span
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
        <h3>制作移动位置的动画</h3>
        <span></span>
        <div id="tip"></div>
        
        <script type="text/javascript">
            $(function () {
                $("span").animate({
                    left: "+=100px"
                }, 3000, function () {
                    $("span").animate({
                        height: '+=30px',
                        width: '+=30px'
                    }, 3000, function () {
                        $("#tip").html("执行完成!");
                    });
                });
            });
        </script>
    </body>
</html>
```

## 调用stop()方法停止当前所有动画效果
`stop()`方法的功能是在动画完成之前，停止当前正在执行的动画效果，这些效果包括滑动、淡入淡出和自定义的动画，
调用格式为`$(selector).stop([clearQueue],[goToEnd])`
其中，两个可选项参数clearQueue和goToEnd都是布尔类型值，前者表示是否停止正在执行的动画，后者表示是否完成正在执行的动画，默认为false。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>调用stop()方法停止当前所有动画效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <style type="text/css">
            div
            {
                margin: 10px 0px;
            }
            span
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
        <h3>调用stop()方法停止当前所有动画效果</h3>
        <span></span>
        <input id="btnStop" type="button" value="停止" />
        <div id="tip"></div>
        
        <script type="text/javascript">
            $(function () {
                $("span").animate({
                    left: "+=100px"
                }, 3000, function () {
                    $(this).animate({
                        height: '+=60px',
                        width: '+=60px'
                    }, 3000, function () {
                        $("#tip").html("执行完成!");
                    });
                });
                $("#btnStop").bind("click", function () {
                   $("span").stop();
                    $("#tip").html("执行停止!");
                });
            });
        </script>
    </body>
</html>
```
## 调用delay()方法延时执行动画效果
`delay()`方法的功能是设置一个延时值来推迟动画效果的执行
调用格式为`$(selector).delay(duration)`
其中参数duration为延时值，它的单位是毫秒，当超过延时值时，动画继续执行。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>调用delay()方法延时执行动画效果</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
       <style type="text/css">
            div
			{
			    margin: 10px 0px;
			}
			span
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
        <h3>调用delay()方法延时执行动画效果</h3>
        <span></span>
        <input id="btnStop" type="button" value="延时" />
        <div id="tip"></div>
        
        <script type="text/javascript">
            $(function () {
                $("span").animate({
                    left: "+=100px"
                }, 3000, function () {
                    $(this).animate({
                        height: '+=60px',
                        width: '+=60px'
                    }, 3000, function () {
                        $("#tip").html("执行完成!");
                    });
                });
                $("#btnStop").bind("click", function () {
                    $("span").delay(5000);
                    $("#tip").html("正在延时!");
                });
            });
        </script>
    </body>
</html>