---
title: "javascript中的继承"
date: 2013-07-11 15:35:00
cover: false
---

在过去的很多时候，javascript 都仅仅被用来当作一个“小工具”，比如用来处理一下异步请求、输入项校验、交互动画等等。这两年，随着 html5 的普及程度日趋提高和各种浏览器技术的不断推广，javascript 在 web 开发中占据越来越重要的位置。

本文尝试向大家展示几种在 javascript 中常见的“类继承”手法并理清他们各自的优缺点及之间的区别，希望对大家使用 javascript 有所帮助。另外，本文内容均基于本人对 javascript 的个人理解，难免有些理解上的偏差和错误，还请各位不吝赐教。同时，阅读此文前，你除了要对 javascript 的简单使用有所了解，最好还会一两门面向对象的编程语言，比如 java、C++、C#...

---

个人认为，要弄清楚 javascript 中的继承方式，首先要弄清楚这三个属性：**prototype**、**constructor**、**\_\_proto\_\_**

**prototype:** 存在于“构造函数”对象中。他本身也是一个对象，用来存放所有由该构造函数所产生的对象实例的公共属性和方法。是不是有点绕？呵呵。

**constructor:** 存在所有的对象中。他指向该对象的构造函数。刚刚说到，prototype 属性本身也是一个对象，所以，prototype 也是有 construstor 属性的。prototype 的 construstor 属性指向了该构造函数本身。

**\_\_proto\_\_:** 存在于实例对象中。他指向该对象的构造函数的 prototype 属性。

---

说完了上面三个属性，接下来就说说目前常见的几种 js 继承的实现方式。

**1、原型链**

```javascript
function SuperType() {
  this.property = true;
}

SuperType.prototype.getSuperValue = function () {
  return this.property;
};

function SubType() {
  this.subproperty = false;
}

SubType.prototype = new SuperType();

SubType.prototype.getSubValue = function () {
  return this.subproperty;
};
```

在上面这段代码中，SuperType 和 SubType 这两个函数就是我们所说的构造函数。第 13 行，我们将 SubType 的 prototype 属性指向了一个 SuperType 的实例。所以，正如刚刚我们对 prototype 属性所解释的那样，prototype 属性“本身也是一个对象，用来存放所有由该构造函数所产生的对象实例的公共属性和方法”。所以，接下来由 SubType 构造函数创建的对象都共享 13 行中那个 SuperType 实例所拥有的属性和方法。当然，他们也可以再自行添加一些自己私有的属性和方法。

引用关系如下：

![](10170313-abb011881316449ca8b549a46641d6ad.png)

这种方法实现的继承虽然很简单，但是问题也很明显。刚刚我们说过，prototype 这个属性指向的对象的所有方法和属性都将被这个构造函数所产生的实例引用。所以，这就会使父类中的属性呈现出“静态”特性，类似在 C#类中声明的 static 属性。

考虑下面的代码：

```javascript
function SuperType() {
  this.nums = [0, 1];
}

function SubType() {}

SubType.prototype = new SuperType();

var ins1 = new SubType();
console.log(ins1.nums); //[0, 1]
ins1.nums.push(2);

var ins2 = new SubType();
console.log(ins2.nums); //[0, 1, 2]
```

可以看到，ins1 实例对 property 属性的修改直接在 ins2 实例中得到了体现，这显然不是我们所希望看到的。

但这里要注意的是，只有当 nums 属性为引用类型值的时候，才会出现这个现象，如果 nums 是一个 Boolean 值，ins1 对象对他的修改是不会影响 ins2 对象中该属性的值的。

这种方式实现的继承还有一个局限，就是无法给父类的构造函数传递参数。

**2、借用构造函数**

为了解决上面说的两个问题，借用构造函数方式应运而生。

```javascript
function SuperType(n) {
  this.nums = [0, n];
}

function SubType(n) {
  SuperType.call(this, n);
}

var ins1 = new SubType(1);
console.log(ins1.nums); //[0, 1]
ins1.nums.push(2);

var ins2 = new SubType(3);
console.log(ins2.nums); //[0, 3]
console.log(ins1.nums); //[0, 1, 2]
```

由于 SubType 对 SuperType 的继承关系由第 6 行实现，所以，此时站在 ins1 和 ins2 的角度来看，他们其实都不知道自己是 SuperType 的子类的实例。

```javascript
ins1 instanceof SuperType; //false
SuperType.prototype.isPrototypeOf(ins1); //false
```

同时，这种实现方式还会带来另一个问题，由于方法和属性的定义都必须放在构造函数中定义，所以子类也就无法实现对父类的方法的复用了。

```javascript
function SuperType() {
  this.play = function () {};
}

function SubType() {
  SuperType.call(this);
}

var ins1 = new SubType();
var ins2 = new SubType();

console.log(ins2.play === ins1.play); //false
```

**3、组合继承**

总结上面的两种方法，一种（原型链）共用属性也共用方法，另一种（借用构造函数）不共用属性也不共用方法。两者都不是我们所希望的，我们想要的是共用方法但不共用属性。于是，组合继承方法解决了这个问题。

```javascript
function SuperType(name) {
  this.name = name;
  this.foods = ["grass"];
}

SuperType.prototype.sayName = function () {
  console.log(this.name);
};

function SubType(name, lang) {
  SuperType.call(this, name);

  this.lang = lang;
}

SubType.prototype = new SuperType();

SubType.prototype.sayLang = function () {
  console.log(this.lang);
};

var ins1 = new SubType("tiger", "wow");
var ins2 = new SubType("leo", "hoh");
```

```javascript
ins1.sayName(); //tiger
ins2.sayName(); //leo
ins1.sayLang(); //wow
ins2.sayLang(); //hoh
ins1.foods.push("pig");
ins1.foods; //["grass", "pig"]
ins2.foods; //["grass"]

ins1.sayName === ins2.sayName; //true
ins1.sayLang === ins2.sayLang; //true
```

由上面的测试代码可以看出，ins1 和 ins2 对于 name、lang、foods 这三个属性都有自己独立保存的值，第 5 行中 ins1 对 foods 属性的修改并不会影响到 ins2 中 foods 属性的值。

于是，通过这种方法，我们近乎完美的解决了前两个方案的问题。为什么说是“近乎”完美呢？稍后会给大家说明。

**4、原型式继承**

有些时候，我们只是希望创建一个和某个对象类似的对象，我们希望创建的过程非常简单，不需要写一堆构造函数、原型引用之类的东西。这时，原型式继承就派上用场了。

```javascript
function object(o) {
  function F() {}
  F.prototype = o;
  return new F();
}

var person = {
  name: "name",
  foods: ["rice", "vegetables"],
  sayHi: function () {
    console.log(this.name);
  },
};

var ins1 = object(person);
ins1.name = "tiger";
ins1.foods.push("fish");

var ins2 = object(person);
ins2.name = "leo";

console.log(ins1.sayHi()); //tiger
console.log(ins2.sayHi()); //leo
console.log(ins2.foods); //["rice", "vegetables", "fish"]

console.log(person.isPrototypeOf(ins2)); //true
console.log(ins1.sayHi === ins2.sayHi); //true
```

这种方式创建的实例和我们所讲的第一种方式很类似，也会有共用引用类型属性的问题和无法向父类构造函数传参的问题。

讲到这里，估计有人就有疑问了，到底什么情况会导致共用父类属性什么情况不共用呢？看看下面这个例子。

```javascript
function SuperType() {
  this.foods = ["rice"];
}

function SubType1() {
  SuperType.call(this);
}
SubType1.prototype = new SuperType();

function SubType2() {}
SubType2.prototype = new SuperType();

ins11 = new SubType1();
ins12 = new SubType1();
console.log(ins11.foods === ins12.foods); //false

ins21 = new SubType2();
ins22 = new SubType2();
console.log(ins21.foods === ins22.foods); //true
```

SubType1 与 SubType2 的区别在于他在自己的构造函数中显式地调用了 SuperType 的构造函数，也就是第 6 行的代码。这样的结果就是 SubType1 的实例，即 ins11 和 ins12，其实拥有两个 foods 属性！对，就是两个，一个在他们自己身上，另一个在他们的\_\_proto\_\_属性所指向的那个对象身上。要证明，很简单：

```javascript
ins11.foods.push("fish");
console.log(ins11.foods); //["rice", "fish"]
console.log(ins11.hasOwnProperty("foods")); //true
delete ins11.foods;
console.log(ins11.foods); //["rice"]
console.log(ins11.hasOwnProperty("foods")); //false
```

而 SubType2 由于没有显式调用 SuperType 的构造函数，所以 SubType2 的实例中的 foods 属性其实只存在于他们的\_\_proto\_\_属性所指向的那个对象身上。证明也很简单：

```javascript
1 console.log(ins21.hasOwnProperty('foods')); //false
```

现在你已经弄清楚了上面的几种方案的原理和他们之间的区别，那么，是时候看看我们的终极解决方案了。

前面我们说到过，组合继承的方案已经是“近乎”完美的了，那么为什么是“近乎”而不是绝对呢？因为他也有一个问题。组合继承方案的问题在于，无论什么情况下，都会调用两次父类的构造函数：一次是在为之类指定 prototype 的时候，另一次是在之类构造函数内的显式调用。这导致了 SubType 的 prototype 上出现多余的、不必要的属性。

**5、寄生组合式继承**

```javascript
function inheritPrototype(subType, superType) {
  function F() {}
  F.prototype = superType.prototype;
  var prototype = new F();
  prototype.constructor = subType;
  subType.prototype = prototype;
}

function SuperType(name) {
  this.name = name;
  this.foods = ["rice"];
}
SuperType.prototype.sayName = function () {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}

inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function () {
  console.log(this.age);
};

var ins1 = new SubType("tiger", 27);
var ins2 = new SubType("leo", 28);
ins1.sayName(); //tiger
ins1.sayAge(); //27
ins2.sayName(); //leo
ins2.sayAge(); //28

ins1.foods.push("fish");
console.log(ins1.foods); //["rice", "fish"]
console.log(ins2.foods); //["rice"]

delete ins1.name;
console.log(ins1.name); //undefined
```

如上所示，寄生组合式继承完美解决了之前几种方案的所有问题。他是如何实现的？看这幅图：

![](11142528-a159764352944520a39630ff42f94547.png)

从这幅图可以看出，ins1 和 ins2 的 constructor 均指向了 SubType 函数，而 SubType 函数中显式调用了 SuperType 的构造函数，所以，ins1 和 ins2 两个对象本身就会有了 SuperType 中定义的 name 和 foods 属性，而且他们的这两个属性是定义在对象自身上的，相互不会干扰。

同时，ins1 和 ins2 的\_\_proto\_\_属性均指向了 inheritPrototype 函数中定义的 prototype 这个对象实例，而 prototype 这个对象实例的\_\_proto\_\_属性指向了 SuperType 的 prototype 对象。所以 ins1 和 ins2 两个对象实例均从 SuperType 的 prototype 属性继承来了 sayName 方法。

---

至此，javascript 中常见的几种继承机制就讲完了，不知道大家看到这里的时候有没有真正弄明白这里面的原理。其实我个人觉得这些原理还是比较绕的（我花了超过两天时间才完全理清这些关系这种事情我会随便告诉你们么，哼～～），但我认为如果你在看得有点糊涂的时候回头去想想我文章最前面说的那三个属性的含义，也许思路就又能变清晰一些了。在学习继承这个知识点的时候我也走了不少弯路，希望这篇文章能给大家一点点的帮助或启发。

参考资料：

[《JavaScript 高级程序设计:第 2 版》](http://book.douban.com/subject/4886879/ "JavaScript高级程序设计:第2版")

[《JavaScript 语言精粹》](http://book.douban.com/subject/11874748/ "JavaScript语言精粹")

[Web 程序员应该知道的 Javascript prototype 原理](http://www.leonzhang.com/2011/12/20/javascript-prototype/ "Web程序员应该知道的Javascript prototype原理")

[Javascript 继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html "Javascript继承机制的设计思想")
