---
title: 'requestAnimationFrame优势何在'
date: 2013-06-19 11:18:00
cover: false
---
大概半年前，无意中在网上看到一个新的js函数requestAnimationFrame，据说，此函数可以优化传统的js动画效果，似乎是未来js动画的新方向。

当时我所在的项目正好用到了和js动画有关的技术，于是在网上查阅了一些相关资料。虽然国内外都有人写过一些关于这个js函数的文章，但大多都只是简要说明工作原理，使用方式等等，一直都没有找到验证其优势所在的示例。

今天我就自己写两个testcase验证requestAnimationFrame的优势所在。

关于requestAnimationFrame这个函数在[MDN](https://developer.mozilla.org/zh-CN/docs/DOM/window.requestAnimationFrame "MDM")上的说明是“告诉浏览器,你想要执行一个动画,该请求要求浏览器提前安排一下下一帧动画显示时需要进行的浏览器窗口的重绘”。也就是说，调用这个api就表示告诉浏览器，下次重绘页面时，记得执行我刚刚传给你的那个逻辑。

这样做一个最大的好处就是可以避免不必要的过度重绘，关于过度重绘的产生原因[MSDN](http://msdn.microsoft.com/zh-cn/library/ie/hh920765(v=vs.85).aspx "MSDN")上已经说得很清楚了。

于是，我的测试思路是，用js构造若干个独立的动画，每个动画都有一个定时器去控制动画的执行，动画的效果就是通过修改一个div的位置坐标，使该div围绕一点做圆周运动。

定时器用两种不同的方式实现，一种用传统的setTimeout函数，每个20ms触发一次重绘逻辑。另一种用requestAnimationFrame触发重绘逻辑。（代码见文末）

下面两幅图为同时运行1500个动画，分别使用requestAnimationFrame和setTimeout时的效果：

使用 setTimeout

![](19105323-b100b7b3539a4d9d8761ee6fc8ff2053.png)

使用 requestAnimationFrame

![](19105350-900f89940170468da328db5e4a62fae5.png)

下表为在不同的动画数目情况下，使用requestAnimationFrame和setTimeout时浏览器的渲染帧数对比，单位：FPS

|||||||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|
|Animation count|500|600|700|800|900|1000|1100|1200|1300|1400|1500|
|reqestanimationframe|30|30|28|26|20|20|20|20|19|17|15|
|setTimeout|34|32|29.5|27|25.5|21.8|19.7|16|14.7|13.1|12|

由上表可以看出，当animation count大于1100的时候，使用requestAnimationFrame的性能是要优于使用setTimeout的，可是当animation count小于1000的时候，使用requestAnimationFrame的性能反而要差于用传统的setTimeout

同时，在上面的测试案例中，无论动画数目是多少，我们使用setTimeout时的延迟间隔始终都是20ms，但在一些情况下适当增加这个时间间隔，setTimeout函数还能得到更好的效果。比如当animation count为1100时，如果把这个间隔调整到40ms，浏览器的帧率可以达到25.5 FPS，明显优于使用requestAnimationFrame时的20 FPS。

看到这里估计大家和我一样都很失望吧，这个传说中实现了各种优化的新api怎么表现如此差强人意呢？

结合上次我写的关于统一帧管理的blog《[使用统一帧管理优化前端性能](http://www.cnblogs.com/silenttiger/archive/2013/05/19/3087781.html "使用统一帧管理优化前端性能")》，我又想到另一种测试场景。

仍然是用js构造若干个独立的动画，但所有这些动画都共用同一个定时器去定时更新自己的状态并重绘，详见代码中的“test case 02”部分。

还是贴两幅同时运行1500个动画，分别使用requestAnimationFrame和setTimeout时的效果：

使用 setTimeout

![](19105911-4e48ecd1ea004126abd9f180b5ccfd93.png)

使用 RequestAnimationFrame

![](19105938-430a0e6c4b194e5394bc234b58090037.png)

同时附上不同的动画数目情况下，使用requestAnimationFrame和setTimeout时浏览器的渲染帧数对比，单位：FPS

|||||||||||||
|---|---|---|---|---|---|---|---|---|---|---|---|
|Animation count|500|600|700|800|900|1000|1100|1200|1300|1400|1500|
|reqestanimationframe|30|30|30|30|29.5|28|20|20|20|20|20|
|setTimeout|37.5|35|34.4|32|31.5|29.5|29|26.5|25|24.3|23|

由上表可见，当使用一个定时器控制所有动画的时候，使用setTimeout函数在各种动画数目场景下，其效果均优于使用requestAnimationFrame函数。

通过上面的两个测试案例，我们可以看到requestAnimationFrame函数似乎并不像大家所想的那样能给js动画带来性能上的大幅提升。虽然MDN和MSDN上对这个api的原理说明都让我觉得它确实是有用的，但实际测试的效果却并不能让我满意。也许各大浏览器厂商还会继续优化这个api的实现，使其能真正达到预期的效果吧。

BTW，其实到这里我自己都很怀疑是不是我的测试案例有问题？如果大家对测试案例有任何意见或建议，还请不吝赐教啊！

以上所有测试的运行环境为 Windows 7 Ultimate sp1 x64 + Chrome 25.0.1364.97 + Intel Core i3 530(2.93GHz) + 4G RAM

代码

```html
<!DOCTYPE html>
<html>
<head>
    <style type="text/css">
        html,body{height:100%;width:100%}
        div{width:20px;height:20px;background-color:black;position:absolute;display:inline-block}
    </style>
</head>
<body>
    <div id="content" style="height:100%;width:100%;background-color:#979797;overflow:hidden;position:relative"></div>
    <script type="text/javascript">
        var cArray = new Array();

        for (var i = 0; i < 1500; i++) {
            var x = parseInt(Math.random() * document.body.clientWidth);
            var y = parseInt(Math.random() * document.body.clientHeight);
            var id = newGuid();
            content.innerHTML += '<div id="' + id + '"></div>';
            cArray.push(new Clock(x, y, 100, id));
        }

        function Clock(x, y, r, id) {
            this.start = new Date();
            this.r = r;
            this.x = x;
            this.y = y;
            this.offsetX = r;
            this.offsetY = 0;
            this.id = id;
            this.draw = function(){
                //update data
                var timespan = new Date() - this.start;
                var offsetR = ((timespan % 36000) % 720) / 360 * Math.PI;
                this.offsetX = this.r * Math.cos(offsetR);
                this.offsetY = this.r * Math.sin(offsetR);
                //draw
                var dom = document.getElementById(this.id);
                dom.style.left = this.x + this.offsetX + 'px';
                dom.style.top = this.y + this.offsetY + 'px';

                //var that = this;                                  //test case 01
                //setTimeout(function(){that.draw()}, 40);          //test case 01 - 1
                //requestAnimationFrame(function(){that.draw()});   //test case 01 - 2
            }
            //this.draw();                                          //test case 01
        }

        function renderLoop(){
            for (var i = 0; i < cArray.length; i++) {
                cArray[i].draw();
            }
            //setTimeout(renderLoop, 20);           //test case 02 - 1
            requestAnimationFrame(renderLoop);      //test case 02 - 2
        }
        renderLoop();                               //test case 02

        function newGuid() {
            var guid = "";
            for (var i = 1; i <= 32; i++) {
                var n = Math.floor(Math.random() * 16.0).toString(16);
                guid += n;
                if ((i == 8) || (i == 12) || (i == 16) || (i == 20))
                    guid += "-";
            }
            return guid;
        }
    </script>
</body>
</html>
```
