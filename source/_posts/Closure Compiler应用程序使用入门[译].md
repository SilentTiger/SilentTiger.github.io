---
title: "Closure Compiler应用程序使用入门[译]"
date: 2013-11-25 14:50:00
cover: false
---

## Hello World 示例

Closure Compiler 应用程序是一个 Java 命令行工具，用来对 JavaScript 代码进行压缩、优化和排错。按照下面的步骤，用一个简单的 JavaScript 程序尝试 Closure Compiler 应用程序。

要让程序成功运行，你需要 Java Runtime Environment version 6。

**1、下载** **Closure Compiler**

创建一个叫 closure-compiler 的工作目录。

下载 Closure Compiler [compiler.jar](http://dl.google.com/closure-compiler/compiler-latest.zip) 文件并保存到 closure-compiler 目录。

**2、创建一个** **JavaScript** **文件**

创建一个名为 hello.js 的 JavaScript 文件，并输入下面的内容：

```javascript
// A simple function.
function hello(longName) {
  alert("Hello, " + longName);
}
hello("New User");
```

将这个文件保存到 closure-compiler 目录。

**3、编译** **JavaScript** **文件**

在 closure-compiler 目录运行下面的命令：

```shell
java -jar compiler.jar --js hello.js --js_output_file hello-compiled.js
```

这个命令会创建一个名叫 hello-compiled.js 的 js 文件，它包含以下内容：

```javascript
1 function hello(a){alert("Hello, "+a)}hello("New User");
```

你会注意到编译器已经去掉了代码中的注释、空格和不需要的分号。编译器还把参数名称 longName 变成了一个短名称 a。结果就是，我们得到了一个比原来小得多的 JavaScript 文件。

要确认编译后的 JavaScript 依然能够正确运行，只需把编译后的 hello-compiled.js 文件包含到一个 HTML 文件中，就像这样：

```html
<html>
  <head>
    <title>Hello World</title>
  </head>
  <body>
    <script src="hello-compiled.js"></script>
  </body>
</html>
```

在浏览器中加载这个 HTML 文件，你就会看到一句友好的欢迎词！

## **下一步**

这个例子仅仅展示了 Closure Compiler 所能完成的最简单的优化工作。想全面了解 Closure Compiler 的功能，阅读[Advanced Compilation and Externs](https://developers.google.com/closure/compiler/docs/api-tutorial3).

想了解更多关于 Closure Compiler 的选项，只需在执行 jar 文件的时候加上 --help 标记。

```shell
java -jar compiler.jar --help
```

_Except as otherwise noted, the content of this page is licensed under the [Creative Commons Attribution 3.0 License](http://creativecommons.org/licenses/by/3.0/), and code samples are licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0). For details, see our [Site Policies](https://developers.google.com/site-policies)._

_Last updated July 29, 2013._

原文见：[https://developers.google.com/closure/compiler/docs/gettingstarted_app](https://developers.google.com/closure/compiler/docs/gettingstarted_app)

