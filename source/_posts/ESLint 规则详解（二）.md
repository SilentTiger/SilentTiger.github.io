---
title: 'ESLint 规则详解（二）'
date: 2017-05-15 15:00:00
top_img: 2017/05/15/ESLint%20%E8%A7%84%E5%88%99%E8%AF%A6%E8%A7%A3%EF%BC%88%E4%BA%8C%EF%BC%89/450824-20170210085550791-2142208530.png
cover: 2017/05/15/ESLint%20%E8%A7%84%E5%88%99%E8%AF%A6%E8%A7%A3%EF%BC%88%E4%BA%8C%EF%BC%89/450824-20170210085550791-2142208530.png
---
![](450824-20170210085550791-2142208530.png)

接上篇[ESLint 规则详解（一）](http://www.cnblogs.com/silenttiger/p/6384927.html "ESLint 规则详解（一）")

前端界大神 Nicholas C. Zakas 在 2013 年开发的 ESLint，极大地方便了大家对 Javascript 代码进行代码规范检查。这个工具包含了 200 多条 Javascript 编码规范且运行迅速，是几乎每个前端项目都必备的辅助工具。可是，这么多规则，每个规则的设计出发点是什么，我们该如何选择适合自己项目的规则，又成了新问题。前不久，我所在的项目开始对前端代码进行代码规范的要求，于是我们详细梳理了 eslint 中的 230 个规则。我摘录了其中一些比较重要或特别的规则列在这里，希望能对大家的工作有所帮助。

1. no-sparse-arrays

   使用代码质量检查工具的一个重要目的就是为了提高代码的可读性，或者说是降低其他人阅读并理解代码的难度，这条规则就是这样。当你看到这样一段代码 var userList = \['Tiger', 'Kate', , 'Mike'\]; 你真的很难确定原来写这段代码的人是不是故意要在数组中留下一个 undefined 元素，毕竟这样写并没有语法上的错误。这条规则的目的就是禁止通过这种方式在数组中插入 undefined 元素，因为这种写法太有迷惑性了。

2. no-extra-bind

   如果你对 javascript 中的 this 变量有所了解，你一定也知道 bind 方法的作用，它可以很方便的帮我们修改方法执行时的上下文环境，但事实上有些时候并不需要使用 bind。如果你在一些不需要使用 bind 的地方也用 bind 来保证方法执行时的上下文环境，这会让代码执行的效率变低。所以，启用这条规则，可以帮你避免不必要的性能损失。

3. no-useless-call

   和上一条规则类似，call 和 apply 也是帮助我们修改上下文环境的好工具，但我们应该只在需要修改上下文的时候才去使用这两个方法，如果你的代码检查工具发现你修改后的上下文和函数或方法原始的上下文相同，它就会给出提示。

4. yoda

   yoda 表达式其实是用写争议的。有人觉得 if ('red' == color) 这样的写法可以避免程序员不小心把 == 写成了 =，但如前篇所说，我们用过在代码中禁用 ==，一律换成 ===，同时在代码检查工具的帮助下，把 == 写成 = 的可能性其实不大。而同时这样的写法在阅读时也显得比较别扭，所以我个人觉得还是禁用 yoda 表达式比较好。

5. no-delete-var

   delete 操作符只能删除对象上的属性，并不能删除当前上下文中的某个变量，虽然代码不会报错，但很可能实际运行的结果和开发人员设想的不同，所以，应该明确禁止删除变量的操作。

6. no-undef

   禁用未声明的变量。javascript 异常灵活，以至于你可以在没有声明一个变量的时候直接给他赋值，比如 t = 'test message'，但这样的写法却是非常危险的，因为这种写法虽然会自动生命变量 t，但他的作用域却和用 var 声明的变量作用域不同，t 变量的作用域在全局变量上，所以，不用 var 直接声明并给变量赋值，经常导致意料之外的程序 bug。

7. no-new-require

   当我们使用 CommonJS 的包管理规范时，经常用 require 引入一些依赖，当我们引入的依赖是一个类定义函数时，直接在 require 上进行 new 操作很可能会引起误解。比如 var tiger = new require('User'); 和 var tiger = new (require('User')); 所以，还是禁用这种写法比较好。

最后附上 ESLint 规则列表，详细列出了每条规则的名称，官方是否推荐开启，以及每条规则是否能够用 --fix 参数自动修复。 [点击下载](http://files.cnblogs.com/files/silenttiger/lintrules_final.xlsx.zip "点击下载")

![](450824-20170515145600369-672374454.png)