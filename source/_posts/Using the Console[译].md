---
title: 'Using the Console[译]'
date: 2013-06-25 14:26:00
cover: false
---
由于最近的项目需要大量用到浏览器端的js编码和调试，所以仔细阅读了一下Chrome对于开发者工具中js部分的说明。虽然原来也用这个工具，但读后仍然觉得受益匪浅。于是抽空翻译一下，与大家分享。

本人英文水平较渣，如有不妥之处，还请不吝赐教。

JavaScript Console为开发人员测试网页和应用提供了两种主要的功能：

* 提供了一个通过Console API，比如console.log()和[console.profile()](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)，来显示诊断信息的地方。
* 提供了一个让你能够通过输入命令与html文档和Chrome开发者工具交互的shell。你能在Console中直接对表达式求值，还能使用[Command Line API](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)提供的各种方法，比如用[$()](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)命令选择元素,或者用[profile()](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)命令开始对CPU资源进行监控分析。

这篇文章将向您展示这两类API的概况和一些基本用法.您也可以浏览[Console API](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)和[Command Line API](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)的使用手册。

[**Basic operation**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Opening the Console**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Clearing the console history**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Console settings**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Using the Console API**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Writing to the console**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Errors and warnings**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Assertions**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Filtering console output**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Grouping output**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**String substitution and formatting**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Formatting DOM elements as JavaScript objects**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Styling console output with CSS**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Measuring how long something takes**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Marking the Timeline**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Setting breakpoints in JavaScript**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Using the Command Line API**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Evaluating expressions**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Selecting elements**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Inspecting DOM elements and JavaScript heap objects**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Accessing recently selected elements and objects**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Monitoring events**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

[**Controlling the CPU profiler**](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)

基本操作

**打开Console**

通过Chrome DevTools，我们有两种方式打开JavaScript Console：独立的Console标签页,或者在其他标签页（比如Element标签页或Source标签页）中以分离视图的方式显示。

我们可以通过下面这些方法中的一种打开Console标签页：

* 通过键盘快捷键打开，**Command - Option - J** (Mac)或者**Control -Shift -J** (Windows/Linux)。
* 通过菜单打开，**View > Developer > JavaScript Console**。

![](25141845-f20bec4d82c647b99845c80a37ac5a83.png)

如果需要在其他标签页上切换Console的开启状态,可以在键盘上按**Esc**键,或者点击Chrome DevTools窗口左下角的**Show/Hide Console**按钮。下面的这幅屏幕截图中Console被以分离视图的形式显示在Element标签页面板中。

![](25141847-4e12eff72a534b7a8183b965d28a4b5c.png)

**清除Console的历史信息**

可以通过以下办法中的一个清除Console的历史信息：

* 在Console窗口的任何位置点击鼠标右键，并在打开的右键菜单中选择Clear Console。
* 在shell prompt中输入并执行[**clear()**](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)命令行API。
* 通过JavaScript调用[**console.clear()**](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN) Console API。
* 使用键盘快捷键**⌘****K**或**⌃****L** (Mac)，**Control - L** (Windows and Linux)。

在默认状态下，Console历史信息会在你跳转到其他页面时自动清除。你可以通过在设置对话框的Console区域启用**Preserve log upon navigation**来改变这一行为(参考Console preferences)。

**Console设置**

在DevTools设置对话框的General标签页中有两个关于Console的全局设置项：

* **Log XMLHTTPRequests**&mdash;&mdash;它决定了是否将每个XMLHTTPRequest请求都显示在Console面板中。
* **Preserve log upon navigation**&mdash;&mdash;它决定了当你跳转到其他页面时是否保留当前页面的console历史信息。

默认状态下，这两个设置项都是关闭状态的。

你还可以通过在Console的任何区域点击鼠标右键所显示的右键菜单中更改这两个设置项。

![](25141848-1c734f5b7f9f4121a2d7e801d1cd7f16.png)

使用Console API

Console API是一组由全局对象console所提供的方法的集合，这个全局对象是有DevTools定义的。这些API的一个主要功能就是在你的程序运行时往console区域打印信息(比如一个属性值，一个完整的对象或者一个DOM元素)。你还能对console区域的信息进行可视化的分组以避免混乱的信息显示。

**向console区域输出信息**

[console.log()](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)可以支持传入一个或多个表达式作为参数，它会把传入的参数输出到console区域。比如：

vara \=document.createElement('p');a.appendChild(document.createTextNode('foo'));a.appendChild(document.createTextNode('bar'));console.log("Node count："+a.childNodes.length);

![](25141848-a678973058be462da93fe11f5e535919.png)

除了像上面那样用+将表达式链接起来,你还可以把这些表达式分别作为一个独立的参数传给此方法，他们会被合并为一行，并用空格分隔开。

console.log("Node count：",a.childNodes.length,"and the current time is：",Date.now());

![](25141849-e055cfc395bb4af7b717f4048776c8fe.png)

**错误和警告**

[console.error()](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)方法会在显示红色信息的同时显示一个红色的图标。

functionconnectToServer(){console.error("Error：%s (%i)","Server is not responding",500);}connectToServer();

![](25141849-54b90155a266433392bc01e472bb9b63.png)

[console.warn()](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)方法会在现实信息的同时显示一个黄色的图标。

if(a.childNodes.length <3){console.warn('Warning! Too few nodes (%d)',a.childNodes.length);}

![](25141850-d67e1b379410426e81532fa1bd156b9a.png)

**断言**

[console.assert()](https://developers.google.com/chrome-developer-tools/docs/console-api?hl=zh-CN)方法会在他的第一个参数表达式的值为false时显示一条错误信息（就是他的第二个参数）。举个栗子，在下面的示例中，当list元素中的子元素数目大于500时就会向console区域输出一条错误信息。

console.assert(list.childNodes.length <500,"Node count is > 500");

![](25141851-ac61b22fa0274cfc980ddd747f4f18b0.png)

**过滤console输出信息**

你可以通过选择Console窗口下方的过滤选项来快速地通过输出信息的严重等级来过滤输出的信息。如下图所示：

![](25141852-d53aab32794f47f486c81b8f22d86805.png)

过滤选项：

* **All**&mdash;&mdash;显示所有的输出信息。
* **Errors**&mdash;&mdash;只显示通过console.error()方法输出的信息。
* **Warnings**&mdash;&mdash;只显示通过console.warn()方法输出的信息。
* **Logs**&mdash;&mdash;只显示通过console.log()，console.info()和console.debug()方法输出的信息。
* **Debug**&mdash;&mdash;只显示通过console.timeEnd()方法输出的信息。

**对输出信息分组**

你可以通过console.group()和groupEnd()命令对相关的console输出信息进行分组显示。

varuser \="jsmith",authenticated \=false;console.group("Authentication phase");console.log("Authenticating user '%s'",user);// authentication code here...if(!authenticated){console.log("User '%s' not authenticated.",user)}console.groupEnd();

![](25141852-83c355469bfd4bcf9fcae94d64f43932.png)

你还可以嵌套分组。比如在下面的例子，在登入过程中创建了一个认证信息分组。如果用户认证通过，就会在认证信息分组中再创建一个信息分组。

varuser \="jsmith",authenticated \=true,authorized \=true;// Top-level groupconsole.group("Authenticating user '%s'",user);if(authenticated){console.log("User '%s' was authenticated",user);// Start nested groupconsole.group("Authorizing user '%s'",user);if(authorized){console.log("User '%s' was authorized.",user);}// End nested groupconsole.groupEnd();}// End top-level groupconsole.groupEnd();console.log("A group-less log trace.");

![](25141853-f3f4c7d11f6e48bf9f9d6ddc8a8c9d2a.png)

如果要创建一个在初始时处于折叠状态的信息分组，就用console.groupCollapsed()方法，而不是console.group()方法,如下所示：

console.groupCollapsed("Authenticating user '%s'",user);if(authenticated){...}

![](25141854-265537604f20415d9d81d181caef806b.png)

**格式字符串和字符串格式化**

你传给任何console信息输出方法(比如log()或error())的第一个参数都可以包含一个或多个*格式字符串。*一个格式字符串由一个%符号和紧跟其后的一个字母构成，这个字母代表了将要应用到值上面的一种特定格式(比如%s代表字符串)。格式字符串还标识出了在哪里替换成后面的参数值。

下面的例子演示了用%s(字符串)和%d(整数)格式字符串将值插入到输出的字符串中。

console.log("%s has %d points","Sam","100");

这个表达式将会在console区域输出"Sam has 100 points"。

下面的表格列出了所有受支持的格式字符串和他们说代表的含义：

|||
<colgroup><col /><col /></colgroup>|---|---|
|**Format specifier**|**Description**|
|%s|将值格式化为字符串。|
|%dor%i|将值格式化为整数。|
|%f|将值格式化为浮点数。|
|%o|将值格式化为可展开的DOM元素(就像在Elements面板中一样)。|
|%O|将值格式化为可展开的JavaScript对象。|
|%c|将第二个参数说知道的css规则应用到输出信息。|

下面这个例子中%d格式字符串会被document.childNodes.length的值替换，并显示为一个整数;而%f格式字符串则被Date.now()的值替换，并显示为一个浮点数。

console.log("Node count：%d, and the time is %f.",document.childNodes.length,Date.now());

![](25141854-7766712895f64fe1a1befd362b19eef7.png)

**将DOM元素格式化为JavaScript对象**

默认状态下，当你将一个DOM元素输出到console区域时，它会像在Elements面板中一样显示为XML格式：

console.log(document.body.firstElementChild)

![](25141855-2bbefc23ceb947889d12f71cb8fa5bbd.png)

你还可以用console.dir()方法将DOM元素输出为JavaScript样式：

console.dir(document.body.firstElementChild);

![](25141855-3d613dbc358f4c36b786fd576536b096.png)

你还可以用console.log()方法配合%O格式字符串以达到同样的效果：

console.log("%O",document.body.firstElementChild);

**对console输出信息应用CSS样式**

你可以用%c格式字符串将自定义CSS规则应用到任何你用console.log()或类似的方法输出到Console的信息上。

console.log("%cThis will be formatted with large, blue text","color：blue; font-size：x-large");

![](25141856-d9dc095b7855488e9e3eed75f0f38c30.png)

**测量花费的时间**

你可以用console.time()和console.timeEnd()方法测量一个函数或操作完成所需的时间。你可以在开始计时的地方调用console.time()方法，然后调用console.timeEnd()方法停止计时。这两次调用之间所花费的时间将被显示在console区域。

console.time("Array initialize");vararray\=newArray(1000000);for(vari \=array.length \-1;i >=0;i\--){array\[i\]\=newObject();};console.timeEnd("Array initialize");

![](25141856-253e6b074530447b9e929e5e0bc555a0.png)

**注意：**你必须在调用console.time()和timeEnd()时传入同样的字符串以得到你想要的结果。

**创建Timeline**

[Timeline面板](https://developers.google.com/chrome-developer-tools/docs/timeline?hl=zh-CN)为你提供了一个关于页面或应用在加载和使用时哪些地方花费了时间的概览。console.timeStamp()方法会在它被执行的时候在Timeline上做一个标记。这样，你可以很容易地找到你的应用中的事件和浏览器事件，比如layout和paints，之间的联系。

**注意：**console.timeStamp()方法只在Timeline处于记录过程中时才会起作用。

在下面的例子中当程序执行到AddResult()函数内部时，就会在Timeline上做一个标记。

functionAddResult(name,result){console.timeStamp("Adding result");vartext \=name +'：'+result;varresults \=document.getElementById("results");results.innerHTML +=(text +"<br>");}

如下面的屏幕截图所示，timeStamp()命令会在Timeline的这些位置留下标记：

* 在Timeline的summary和detail视图中的垂直黄线。
* 在事件记录列表中的一条记录。

![](25141857-247b3c011f9f4a47bbe46a878e6c6267.png)

**在JavaScript中设置断点**

你可以在JavaScript代码中调用debugger命令来开始一次debug。比如在下面的例子中，当brightness()方法被调用时就会开始一次JavaScript调试：

brightness：function(){debugger;varr \=Math.floor(this.red\*255);varg \=Math.floor(this.green\*255);varb \=Math.floor(this.blue\*255);return(r \*77+g \*150+b \*29)>>8;}

![](25141858-f26a48eff5a647dfa5dbc8f4a0b6542a.png)

Brian Arnold开发了一种非常有趣的条件调试技术，参见Breakpoint Actions in JavaScript.。

使用Command Line API

Console除了用作通过应用程序显示信息的地方，它还是一个让你可以直接对表达式求值或执行由Command Line API提供的命令的地方。该API提供了以下功能：

* 方便的DOM元素选取功能
* 对CPU性能监控分析的方法
* 部分Console API方法的别名
* 监视事件
* 查看对象上注册的事件监听

**表达式求值**

当你按下Return或Enter键时，Console会尝试对任何你输入的JavaScript表达式求值。Console还提供了自动完成提示和tab自动完成输入。比如你输入一个表达式，他的属性名称就会自动显示在提示中。如果有多个属性都拥有相同的前缀，此时按tab键就会循环遍历它们。按下向右箭头按键，就会输入当前的建议项。当只有一个建议项时，按tab键也会输入该建议项。

![](25141859-e71c8eb429f143e399b4d79db4cbf33d.png)

如果要输入一个多行的表达式（比如一个方法的定义内容），可以按Shift+Enter换行。

![](25141859-3a2b6202daee430babf76d444cd6fe35.png)

**选择元素**

Command Line API提供了一些方法以访问应用中的DOM元素。比如，$()方法会像document.querySelector()一样返回与指定CSS选择器相匹配的第一个元素。下面的代码会返回ID为"loginBtn"的元素。

$('#loginBtn');

![](25141900-a6f452a7ecc94583838c431653ceca8a.png)

[$$()](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)命令则像document.querySelectorAll()一样返回所有与指定CSS规则匹配的元素。如下面的代码会显示所有CSS class为"loginBtn"的<button>元素：

$$('button.loginBtn');

![](25141901-a42730823e34403b8b0dec4215abf029.png)

最后，x()方法会返回所有与传入的XPath路径参数相匹配的元素。下面的代码会返回 body 标签下的所有 script 元素：

$x('/html/body/script');

**检查DOM元素和JavaScript对象**

[inspect()](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)将一个指向DOM元素的引用(或指向JavaScript对象的引用)作为参数，并将其显示在相应的面板中&mdash;&mdash;在Elements面板显示DOM元素，或在Profile面板显示JavaScript对象。

比如，在下面的截图中，$()方法会得到一个对<li>元素的引用。然后在最后的表达式中用($\_)传给inspect()方法，以在Elements面板中显示该元素。

![](25141902-3fe24a47dd78425a980fc09b731ce727.png)

**访问当前选中的元素和对象**

通常，你会在测试的时候用放大镜或直接在Element面板中选择DOM元素，同时你也就可以进一步检视这些元素。或者，在你用Profiles面板分析一个内存使用情况快照时，你也可以选择一个JavaScript对象做进一步查看。

Console会记录下你最近选择的5个元素（或者堆对象），并给他们命名为$0,$1,$2,$3和$4.。最近选择的对象为$0，其次为$1，以此类推。

下面的截图显示了在选择了3个元素后这些属性的值：

![](25141902-3fc820dce0e741a1967f58542c78e4fc.png)

**注意：**你也可以在任何元素上用鼠标右键点击并选择**Reveal in Elements Panel**。

**监视事件**

[monitorEvents()](https://developers.google.com/chrome-developer-tools/docs/commandline-api?hl=zh-CN)可以监视一个对象的一个或多个事件。当被监视的对象上有事件被触发时，相应的事件对象就会被显示在Console中。你可以指定要监视的对象和它上的事件。比如，下面的代码会监视window对象上的"resize"事件：

monitorEvents(window,"resize");

![](25141903-7248e8ca8a7d49548f6aea9b819164cf.png)

如果需要监视一组事件，你可以将事件名称数组作为第二个参数传给该方法。下面的代码同时监视了body的"mousedown"事件和"mouseup"事件。

monitorEvents(document.body,\["mousedown","mouseup"\]);

你还可以传一个合法的"事件类型"给这个方法，DevTools会把它对应到一组事件名称。比如，"touch"事件类型会让DevTools监视指定对象上的"touchstart"，"touchend"，"touchmove"，和"touchcancel"事件。

monitorEvents($('#scrollBar'),"touch");

你可以在Console API Reference中查看monitorEvents()章节，以获得合法的事件类型列表。

你可以调用unmonitorEvents()方法停止监视事件，只需将需要停止监视的对象作为参数传入该方法。

unmonitorEvents(window);

**控制CPU分析器**

你可以在命令行用profile()和profileEnd()方法创建JavaScript CPU性能分析。你还可以给你创建的分析指定一个名字。

下面的例子中用默认名称创建了一个新的性能分析：

![](25141904-8727335b852041b0a2a934cc5c7723d4.png)

这个新的性能分析会在Profile面板中的"Profile 1"项目下显示：

![](25141905-759c0705cb50431a8f446d321fa220d1.png)

如果你为新的性能分析指定了一个标签，它会被用作这个性能分析的标题。如果你对多个性能分析用了相同的名称，它们会被作为独立的性能分析并分组显示在同一个名称下：

![](25141906-30e3f880bac049d6b0f477bdf13be7f7.png)

Profiles面板中显示的结果：

![](25141907-a9a47783a1a149f280d1f4ca17eafbf1.png)

CPU性能分析也可以被嵌套使用：

profile("A");profile("B");profileEnd("B")profileEnd("A")

性能分析的开始和结束方法并不一定需要被嵌套调用。比如，下面的用法将得到和前面相同的效果：

profile("A");profile("B");profileEnd("A");profileEnd("B");

*Except as otherwise [noted](https://developers.google.com/readme/policies?hl=zh-CN), the content of this page is licensed under the [Creative Commons Attribution 3.0 License](http://creativecommons.org/licenses/by/3.0/), and code samples are licensed under the[Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).*

*Last updated六月21, 2013.*

原文见：[https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN](https://developers.google.com/chrome-developer-tools/docs/console?hl=zh-CN)
