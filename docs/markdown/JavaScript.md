# JavaScript基础

> JavaScript（简称“JS”） 是一种具有函数优先的轻量级，解释型或即时编译型的编程语言。虽然它是作为开发Web页面的脚本语言而出名，但是它也被用到了很多非浏览器环境中，JavaScript 基于原型编程、多范式的动态脚本语言，并且支持面向对象、命令式和声明式（如函数式编程）风格。
>
> JavaScript是用于Web前端的动态脚本语言，主要作用是通过操作HTML元素动态的修改页面。JavaScript的运行环境非常简单，一般的浏览器都支持运行。编写的代码可以直接在浏览器中运行，无需安装专用编译器。
>
> JavaScript是基于对象（object-based）的语言，已经广泛用于网页应用开发，常用来为网页添加各种各样的动态功能，为用户提供更流畅更美观的浏览效果。

## JavaScript的历史

> JavaScript在1995年由Netscape公司的Brendan Eich，在网景导航者浏览器上首次设计实现而成。因为Netscape与Sun合作，Netscape管理层希望它外观看起来像Java，因此取名为JavaScript。但实际上它的语法风格与Self及Scheme较为接近。
>
> JavaScript的标准是ECMAScript 。截至 2012 年，所有浏览器都完整的支持ECMAScript 5.1，旧版本的浏览器至少支持ECMAScript 3 标准。2015年6月17日，ECMA国际组织发布了ECMAScript的第六版，该版本正式名称为 ECMAScript 2015，但通常被称为ECMAScript 6 或者ES6。
>
> 它最初由Netscape的Brendan Eich设计。JavaScript是甲骨文公司的注册商标。Ecma国际以JavaScript为基础制定了ECMAScript标准。JavaScript也可以用于其他场合，如服务器端编程。完整的JavaScript实现包含三个部分：ECMAScript，文档对象模型，浏览器对象模型。
>
> Netscape在最初将其脚本语言命名为LiveScript，后来Netscape在与Sun合作之后将其改名为JavaScript。JavaScript最初受Java启发而开始设计的，目的之一就是“看上去像Java”，因此语法上有类似之处，一些名称和命名规范也借自Java。但JavaScript的主要设计原则源自Self和Scheme。JavaScript与Java名称上的近似，是当时Netscape为了营销考虑与Sun微系统达成协议的结果。为了取得技术优势，微软推出了JScript来迎战JavaScript的脚本语言。为了互用性，Ecma国际（前身为欧洲计算机制造商协会）创建了ECMA-262标准（ECMAScript）。两者都属于ECMAScript的实现。尽管JavaScript作为给非程序人员的脚本语言，而非作为给程序人员的脚本语言来推广和宣传，但是JavaScript具有非常丰富的特性。
>
> 发展初期，JavaScript的标准并未确定，同期有Netscape的JavaScript，微软的JScript和CEnvi的ScriptEase三足鼎立。1997年，在ECMA（欧洲计算机制造商协会）的协调下，由Netscape、Sun、微软、Borland组成的工作组确定统一标准：ECMA-262。

## JavaScript的引用

JavaScript的代码引用方式有两种，第一种是直接通过`<script>`标签将代码嵌入到HTML文件中直接引用

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>JavaScript演示</title>

    <script type="text/JavaScript">
        alert("你好！");
    </script>
    
</head>
</html>
```

上述代码在文件类型声明中使用`type="text/JavaScript"`语句告诉浏览器该段落是一个JavaScript代码片段。通过`alert()`函数弹出提示语句对话框，该函数功能主要用来进行在浏览器上弹出的对话框中显示的纯文本。

第二种引用方式需要将JavaScript代码单独保存为一个扩展名为**.js**的文件，然后在HTML中引入，具体引用形式如下。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JavaScript演示</title>    
    <script type="text/JavaScript" src="newfile.js"></script>
</head>
</html>
```

## `script`的属性

| 属性      | 值          | 描述                                               |
| :-------- | :---------- | :------------------------------------------------- |
| type      | *MIME-type* | 指示脚本的 MIME 类型。                             |
| src       | *URL*       | 规定外部脚本文件的 URL。                           |
| defer     | defer       | 规定是否对脚本执行进行延迟，直到页面加载为止。     |
| language  | *script*    | 不赞成使用。规定脚本语言。请使用 type 属性代替它。 |
| charset   | *charset*   | 规定在外部脚本文件中使用的字符编码。               |
| xml:space | preserve    | 规定是否保留代码中的空白。                         |
| async     | async       | 规定异步执行脚本（仅适用于外部脚本）。             |









