---
title: 'html元素双击事件触发机制猜想及疑惑'
date: 2014-03-03 16:01:00
cover: false
---
今天有个同事遇到一个奇怪的问题，我照着他的代码做了一些简化写了这个demo

![](031453113621138.png)

```html
<!DOCTYPE html>
<html>
<head>
    <style type="text/css">
        div{position: absolute;top: 0;left: 0;height: 50px}
        #back{background-color: blue;left: 20px;width: 200px;z-index: 0}
        #front{background-color: green;top:10px;width: 100px;z-index: 1}
    </style>
</head>
<body>
    <div id="back"></div>
    <div id="front"></div>
    <script type="text/javascript">
        document.getElementById("back").addEventListener("click", function () {
            console.log("back clicked");
        })
        document.getElementById("front").addEventListener("click", function () {
            console.log("front clicked");
            this.setAttribute("style", "z-index:0");
            document.getElementById("back").setAttribute("style", "z-index:1");
        })
        document.getElementById("back").addEventListener("dblclick", function () {
            console.log("back double clicked");
        })
        document.getElementById("front").addEventListener("dblclick", function () {
            console.log("front double clicked");
        })
    </script>
</body>
</html>
```

代码的逻辑大致是这样的：

首先，页面中绿色方块为front，蓝色方块为back。系统的需求是，在绿色方块上单击时，切换两个方块覆盖方式（也就是点击front后back会跑到front前面）。同时，还需要在双击蓝色方块时实现另一个功能逻辑。

于是这哥们很自然了写了类似上面的代码就提交了。没多久，测试MM提了一个bug：“双击绿色方块时，不应触发双击蓝色方块的逻辑”。

后来我自己测了一下，果然如测试MM所说，当双击绿色和蓝色方块重叠的区域时，控制台会打印出这样的log：

![](031504494313099.png)

浏览器会先触发front click，然后是back click，再然后居然是back double click。按理说，在back上面只点击了一次，应该不触发double click才对，毕竟第一次click是在front上触发的。

所以，我大胆猜想，浏览器本身并不会去判定鼠标是否触发双击事件，双击事件是由操作系统直接分发给浏览器的。当浏览器收到操作系统发来的双击消息时，直接根据该双击事件中的坐标去页面中找命中的元素，并在这个元素上触发js中的双击事件。

操作系统在判断是否发生双击时，并不知道第一次click和第二次click的目标不是同一个元素，所以只要两次click的间隔足够短就认为构成双击事件。而浏览器在收到操作系统发来的双击事件时，并没有去检测这次双击是由那两次单击所产生，而是直接根据双击事件的坐标信息将这个事件分发到相应的html元素上了。

为验证这个猜想，我又写了一个demo

```html
<!DOCTYPE html>
<html>
<head>
    <style type="text/css">
        body{transition:padding-left 8s linear;padding-left: 0}
        div{width: 10px;height: 200px;background-color: blue;display: inline-block;float: left;}
    </style>
</head>
<body>
    <div id="div_00"></div>
    <div id="div_01"></div>
    <div id="div_02"></div>
    <div id="div_03"></div>
    <div id="div_04"></div>
    <div id="div_05"></div>
    <div id="div_06"></div>
    <div id="div_07"></div>
    <div id="div_08"></div>
    <div id="div_09"></div>
    <div id="div_10"></div>
    <div id="div_11"></div>
    <div id="div_12"></div>
    <div id="div_13"></div>
    <div id="div_14"></div>
    <div id="div_15"></div>
    <div id="div_16"></div>
    <div id="div_17"></div>
    <div id="div_18"></div>
    <div id="div_19"></div>
    <div id="div_20"></div>
    <div id="div_21"></div>
    <div id="div_22"></div>
    <div id="div_23"></div>
    <div id="div_24"></div>
    <div id="div_25"></div>
    <div id="div_26"></div>
    <div id="div_27"></div>
    <div id="div_28"></div>
    <div id="div_29"></div>
    <script type="text/javascript">
        var divs = document.getElementsByTagName("div");
        for(var i = 0, length = divs.length; i < length; i++){
            divs[i].addEventListener("click", function () {
                console.log("clicked", this.id);
            });
            divs[i].addEventListener("dblclick", function () {
                console.log("double clicked", this.id);
            });
        }

        setTimeout(function(){
            document.body.setAttribute("style", "padding-left:300px");
        }, 1000);
    </script>
</body>
</html>
```

当页面中的蓝色方块开始移动后，在蓝色方块上双击鼠标，可以多试几次，得到下面的结果：

![](031548482238932.png)

可以看到，div\_26、div\_22两次双击事件分别是由div27、div26和div23、div22的两次单击触发的。

从这两个结果来看，确实是符合我的猜想的。

不过，这个demo也带来了一些疑惑：

1、clicked div\_19 这个log打印的时候，我非常确定我点击了两次鼠标，但是只打出了这一个log，为什么？

2、double clickeddiv\_15和double clicked div\_11这两次双击事件触发前，都只触发了一次click事件，这又是为什么？

以上所有代码测试结果都是基于 Chrome 33 和 IE 11 运行环境。

希望有高人指导，也欢迎大家各抒己见。
