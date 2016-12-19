title: jQuery表单选择器
date: 2015-11-14 17:03:02
tags: jQuery
categories: note
---
# jQuery表单选择器

<!--more-->
##:input表单选择器
` :input`表单选择器可以实现，它的功能是返回全部的表单元素
不仅包括所有<input>标记的表单元素，而且还包括<textarea>、<select> 和 <button>标记的表单元素，因此，它选择的表单元素是最广的。
**注意**`:`前面有个空格
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitiona
//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>:input表单选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>修改全部表单元素的背景色</h3>
        <form id="frmTest" action="#">
        <input type="button" value="Input Button" /><br />
        <select>
            <option>Option</option>
        </select><br />
        <textarea rows="3" cols="8"></textarea><br />
        <button>
            Button</button><br />
        </form>
        
        <script type="text/javascript">
            $("#frmTest :input").addClass("bg_blue");
        </script>
    </body>
</html>

## :text表单文本选择器
` :text`表单选择器只获取单行的文本输入框元素，对于`<textarea>`区域文本、按钮等元素无效。
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>:text表单选择器</title>
        <script src="http://libs.baidu.com/jquery/1.9.0/jquery.js" type="text/javascript"></script>
        <link href="style.css" rel="stylesheet" type="text/css" />
    </head>
    
    <body>
        <h3>修改多个单行输入框元素的背景色</h3>
        <form id="frmTest" action="#">
        <input type="text" id="Text1" value="我是小纸条"/><br />
        <textarea rows="3" cols="8"></textarea><br />
        <input type="text" id="Text2" value="我也是小纸条"/><br />
        <button>
            Button</button><br />
        </form>
        
        <script type="text/javascript">
            $("#frmTest :text").addClass("bg_blue");
        </script>
    </body>
</html>
```

