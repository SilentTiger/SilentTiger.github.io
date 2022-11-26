---
title: "javascript典型bug——错误的闭包"
date: 2013-11-26 15:20:00
cover: false
---

昨天 QT 给我的一个功能提了一个 bug。大概意思就是说，一段在不同位置都会被调用的代码，在 A 处被调用的时候，似乎会对其他调用的地方产生影响。

我仔细 debug 了半天，终于找到了原因。简化过的代码如下：

```javascript
function C(name, id) {
  this.name = name;
  var privateId = id;
  if (typeof this.showName != "function") {
    C.prototype.showName = function () {
      console.log(this.name);
    };
    C.prototype.showId = function () {
      console.log(privateId);
    };
  }
}

var c1 = new C("name1", "id1");
var c2 = new C("name2", "id2");

c1.showName(); //name1
c1.showId(); //id1
c2.showName(); //name2
c2.showId(); //id1 !!!!
```

问题出在最后一行，c2 的 showId 方法打印出了 id1。

苦思冥想良久，终于让我想到了问题的原因&mdash;&mdash;c2 对象在调用构造函数的时候，不会进入 if 分支里面！

为什么呢？因为 c1 在实例化的时候，this.showName = undefined，于是进入 if 分支，给自己的 prototype 加上了一个 showName 方法一个 showId 方法。

等到 c2 对象实例化的时候，this.showName 已经不再是 undefined 了，于是不会进入 if 分支。

这样，c2 的 showId 方法和 c1 的 showId 方法是同一个方法，而且这个方法里面打印的 privateId 变量则都是 c1 在实例化的时候创建的那个变量，也就是 id1。

所以效果就是，本来想把 privateId 变量申明成一个私有变量，但这样写了之后，它变成了一个 static 变量了，真是缘木求鱼，南辕北辙啊。

问题原因找到了，那么如何求解呢？

我的思路是，要使用闭包实现私有变量，那么这个闭包的函数就要与需要隐藏的变量绑定起来。而私有变量又是和类的实例绑定的，也就是 c1 和 c2 分别有自己的私有变量，所以闭包函数也必须和类的实例一一绑定。于是就改成了这样：

```javascript
function C(name, id) {
  this.name = name;
  var privateId = id;
  if (typeof this.showName != "function") {
    C.prototype.showName = function () {
      console.log(this.name);
    };
  }

  this.showId = function () {
    console.log(privateId);
  };
}
```

经测试，结果是正确的。

不过我依然在怀疑，我上面说的思路中“闭包的函数就要与需要隐藏的变量绑定起来”这一句，是否是正确的？如果是否，那么还有没有其他的更好的方式实现这个需求呢？

