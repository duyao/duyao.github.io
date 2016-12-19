title: jQuery事件与应用
date: 2015-11-15 16:38:48
tags: jQuery
categories: note
---
# jQuery事件与应用
<!--more-->

## 页面加载时触发ready()事件
`ready()`事件类似于`onLoad()`事件

- `ready()`只要页面的DOM结构加载后便触发,ready()可以写多个，按顺序执行
- `onLoad()`必须在页面全部元素加载成功才触发
下列写法是相等的：
`$(document).ready(function(){})`等价于`$(function(){});`

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>ready()事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>页面载入时触发ready()事件</h3>
        <div id="tip"></div>
        <input id="btntest" type="button" value="点下我" />
        
        <script type="text/javascript">
            $(document).ready(function() {
                $("#btntest").bind("click", function () {
                    $("#tip").html("我被点击了！");
                });
            });
            $(function(){
            	$("#btntest").bind("click", function () {
                    $("#tip").html("我被点击了！");
                });
            });
        </script>
    </body>
</html>
```

## 使用bind()方法绑定元素的事件
`bind()`方法绑定元素的事件非常方便，绑定前，需要知道被绑定的元素名，绑定的事件名称，事件中执行的函数内容就可以
它的绑定格式如下：
```
$(selector).bind(event,[data] function)
```
参数event为事件名称，多个事件名称用空格隔开，function为事件执行的函数。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>bind()方法绑定事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
    </head>
    
    <body>
        <h3>bind()方法绑多个事件</h3>
        <input id="btntest" type="button" value="点击或移出就不可用了" />
        
        <script type="text/javascript">
        //使用bind()方法绑定单击(click)和鼠标移出(mouseout)这两个事件，触发这两个事件中，按钮将变为不可用。
            $(function () {
                $("#btntest").bind("click mouseout", function(){
                    $(this).attr("disabled", "true");   
                })  
            });
        </script>
    </body>
</html>
```

## 使用hover()方法切换事件
`hover()`方法的功能是当鼠标移到所选元素上时，执行方法中的第一个函数，鼠标移出时，执行方法中的第二个函数
实现事件的切实效果，调用格式`$(selector).hover(over，out);`
over参数为移到所选元素上触发的函数，out参数为移出元素时触发的函数。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>hover()方法切换事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>hover()方法切换事件</h3>
        <div>别走！你就是土豪</div>
        
        <script type="text/javascript">
            $(function () {
                $("div").hover(
                function () {
                    $(this).addClass("orange");
                },
                function () {
                    $(this).removeClass("orange")
                })
            });
        </script>
    </body>
</html>
```

## 使用toggle()方法绑定多个函数
`toggle()`方法可以在元素的click事件中绑定两个或两个以上的函数，同时，它还可以实现元素的隐藏与显示的切换
绑定多个函数的调用格式`$(selector).toggle(fun1(),fun2(),funN(),...)`
其中，fun1，fun2就是多个函数的名称
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>toggle()方法绑定多个函数</title>
        <script src="http://libs.baidu.com/jquery/1.8.2/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>toggle()方法绑定多个函数</h3>
        <input id="btntest" type="button" value="点一下我" />
        <div>我是动态显示的</div>
        <script type="text/javascript">
            $(function () {
                $("#btntest").bind("click", function () {
                    $("div").toggle(
                    	//没有下面这段，实现的是显示和消失的功能
                    	//下面的这段，实现按钮显示123
                    	/*
                        function(){
                            $(this).html("1");
                        },
                        function(){
                            $(this).html("2");
                        },
                        function(){
                            $(this).html("3");
                        }*/
                        );
                        
                })
            });
        </script>
    </body>
</html>
```


## 使用one()方法绑定元素的一次性事件
`one()`方法可以绑定元素任何有效的事件，但这种方法绑定的事件只会触发一次
它的调用格式`$(selector).one(event,[data],fun)`
参数event为事件名称，data为触发事件时携带的数据，fun为触发该事件时执行的函数。

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>one()方法执行一次绑定事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>  
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    <body>
        <h3>one()方法执行一次绑定事件</h3>
        <div>请点击我一下</div>
        <script type="text/javascript">
            $(function () {
                var intI = 0;
                //相应一次
                $("div").one("click", function () {
                    intI++;
                    $(this).html(intI + "px");
                })
                //响应多次，每次都会++
                // $("div").bind("click", function () {
                //     intI++;
                //     $(this).html(intI + "px");
                // })

            });
        </script>
    </body>
</html>
```

## 调用trigger()方法手动触发指定的事件
`trigger()`方法可以直接手动触发元素指定的事件，这些事件可以是元素自带事件，也可以是自定义的事件，总之，该事件必须能执行
调用格式为`$(selector).trigger(event)`
其中event参数为需要手动触发的事件名称。

```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>trigger()手动触发事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>trigger()手动触发事件</h3>
        <div>土豪，咱们交个朋友吧</div>
        
        <script type="text/javascript">
            $(function () {
                $("div").bind("change-color", function () {
                    $(this).addClass("color");
                    $(this).html("hello");
                });
                //触发change-color事件
                $("div").trigger ("change-color");
            });
        </script>
    </body>
</html>
```


## 文本框的focus和blur事件

- `focus`事件在元素获取焦点时触发，如点击文本框时，触发该事件
- `blur`事件则在元素丢失焦点时触发，如点击除文本框的任何元素，都会触发该事件。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>表单中文本框的focus和blur事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>表单中文本框的focus和blur事件</h3>
        <input id="txtest" type="text" value="" />
        <div></div>
        
        <script type="text/javascript">
            $(function () {
                $("input")
                .bind("focus", function () {
                    $("div").html("请输入您的姓名！");
                })
                $("input").bind("blur", function () {
                    if ($(this).val().length == 0)
                        $("div").html("你的名称不能为空！");
                })
            });
        </script>
    </body>
</html>
```

## 下拉列表框的change事件
当一个元素的值发生变化时，将会触发change事件，例如在选择下拉列表框中的选项时，就会触change事件。
```
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>下拉列表的change事件</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>下拉列表的change事件</h3>
        <select id="seltest">
            <option value="葡萄">葡萄</option>
            <option value="苹果">苹果</option>
            <option value="荔枝">荔枝</option>
            <option value="香焦">香焦</option>
        </select>
        <div></div>
        <script type="text/javascript">
            $(function () {
                $("#seltest").bind("change", function () {
                    if ($(this).val() == "苹果")
                        $("div").html($(this).val()+"apple");
                    else
                        $("div").html("select"+$(this).val());
                })
            });
        </script>
    </body>
</html>
```
