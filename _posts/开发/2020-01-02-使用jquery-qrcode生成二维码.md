---
title: 使用 jquery-qrcode 生成二维码
categories: 开发
toc: false
permalink: /jquery-qrcode-demo.html
---

一个使用[jquery-qrcode](http://jeromeetienne.github.io/jquery-qrcode/)在前端生成二维码的[示例](/pages/qrcode.html)

```html
<!DOCTYPE html>
<html lang="zh-CN">

<head>
    <meta charset="UTF-8">
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
    <script src="https://cdn.bootcdn.net/ajax/libs/jquery.qrcode/1.0/jquery.qrcode.min.js"></script>
    <title>JS生成二维码示例</title>
</head>

<script>
    function qrcode() {
        var width =  $("#qrwidth").val();
        var height = $("#qrheight").val();
        var text = $("#qrtext").val();
        $("#qrcode").qrcode({ width: width, height: height, text: text });
    }
</script>

<body>
    <div>
        内容：<input id="qrtext" type="text" value="http://hiwzc.com"/>
        宽度：<input id="qrwidth" type="text" value="200" />
        高度：<input id="qrheight" type="text"  value="200" />
        <button onclick="qrcode()">生成</button>
    </div>
    <p></p>
    <div id="qrcode"></div>
</body>

</html>
```

