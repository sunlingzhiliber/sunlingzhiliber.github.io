---
title: "Markdown" #当你不定义title的时候,title会根据文件名生成
sub_title: "The common elements"
excerpt: "Markdown的'剽窃'"    #自定义简介
categories:
  - Other
tags: 
  - Markdown
image: /assets/images/eder-oliveira-180877.jpg
---

Markdown语法的学习,注定是一场艰难的道路啊


# 目录
<!-- TOC -->

- [目录](#目录)
    - [引用](#引用)
    - [换行 换段落](#换行-换段落)
    - [链接和图片](#链接和图片)
    - [文本](#文本)
        - [字体](#字体)
        - [Default](#default)
        - [Left Aligned](#left-aligned)
        - [Center Aligned](#center-aligned)
        - [Right Aligned](#right-aligned)
        - [Justify Aligned 居中对齐](#justify-aligned-居中对齐)
        - [No Wrap 不换行](#no-wrap-不换行)
        - [小写 大写 首字母大写 小字体 文本截断](#小写-大写-首字母大写-小字体-文本截断)
    - [代码](#代码)
    - [分割线](#分割线)
    - [Table](#table)
    - [解释](#解释)
    - [列表](#列表)
    - [地址](#地址)
    - [视频](#视频)
    - [公式](#公式)
        - [直接编辑公式](#直接编辑公式)
        - [利用在线网站,嵌入图片](#利用在线网站嵌入图片)

<!-- /TOC -->

## 引用
> Syntax highlighting is a feature that displays source code, in different colors and fonts according to the category of terms. This feature facilitates writing in a structured language such as a programming language or a markup language as both structures and syntax errors are visually distinct. Highlighting does not affect the meaning of the text itself; it is intended only for human readers.[^1]

[^1]: <http://en.wikipedia.org/wiki/Syntax_highlighting>

>Stay huge,Stay hard
<cite>Steve Jobs</cite> --- Apple Worldwide Developers' Conference, 1997
{: .small}

<cite>Steve Jobs</cite> --- Apple Worldwide Developers' Conference, 1997


## 换行 换段落
换行是两个空格  
换段落是空一行



## 链接和图片
点击[百度](http://www.baidu.com)

![Alt](/assets/images/homePage.jpg){: .align-left}

<ins>align-left图</ins>
{: .text-center}
![Alt](/assets/images/homePage.jpg){: .align-center}

align-center图
{: .text-center}
![Alt](/assets/images/homePage.jpg){: .align-right}

align-right图
{: .text-center}

[跳转到锚点](#引用)


## 文本

### 字体
**加粗**
*斜体*
***斜体加粗***
~~删除线~~
`底纹`|
_强调_
<strike>删除线</strike>
<ins>下划线</ins>
<kbd>keyboard text</kbd>
<pre>代码块</pre>
<b>粗体</b>
H<sub>2</sub>O  
符号转义 \_强调\_

### Default

This is a paragraph. It should not have any alignment of any kind. It should just flow like you would normally expect. Nothing fancy. Just straight up text, free flowing, with love. Completely neutral and not picking a side or sitting on the fence. It just is. It just freaking is. It likes where it is. It does not feel compelled to pick a side. Leave him be. It will just be better that way. Trust me.



### Left Aligned

This is a paragraph. It is left aligned. Because of this, it is a bit more liberal in it's views. It's favorite color is green. Left align tends to be more eco-friendly, but it provides no concrete evidence that it really is. Even though it likes share the wealth evenly, it leaves the equal distribution up to justified alignment.
{: .text-left}

### Center Aligned

This is a paragraph. It is center aligned. Center is, but nature, a fence sitter. A flip flopper. It has a difficult time making up its mind. It wants to pick a side. Really, it does. It has the best intentions, but it tends to complicate matters more than help. The best you can do is try to win it over and hope for the best. I hear center align does take bribes.
{: .text-center}

### Right Aligned

This is a paragraph. It is right aligned. It is a bit more conservative in it's views. It's prefers to not be told what to do or how to do it. Right align totally owns a slew of guns and loves to head to the range for some practice. Which is cool and all. I mean, it's a pretty good shot from at least four or five football fields away. Dead on. So boss.
{: .text-right}

### Justify Aligned 居中对齐

This is a paragraph. It is justify aligned. It gets really mad when people associate it with Justin Timberlake. Typically, justified is pretty straight laced. It likes everything to be in it's place and not all cattywampus like the rest of the aligns. I am not saying that makes it better than the rest of the aligns, but it does tend to put off more of an elitist attitude.
{: .text-justify}

### No Wrap 不换行

This is a paragraph. It has a no wrap. Beef ribs pig tenderloin filet mignon. Spare ribs leberkas ribeye, burgdoggen fatback tenderloin biltong jowl flank sirloin hamburger bacon brisket. Shoulder bresaola drumstick capicola. Beef ribs prosciutto porchetta beef.
{: .text-nowrap}

### 小写 大写 首字母大写 小字体 文本截断
Lowercased text
{: .text-lowercase}

Uppercased text
{: .text-uppercase}

Capitalized text hehe
{: .text-capitalize}

Small text
{: .small}

This is a truncated paragraph of text. Bacon ipsum dolor amet shoulder jowl tail andouille fatback tongue. Ham porchetta kielbasa pork pork chop, tenderloin hamburger meatball. Picanha porchetta swine, brisket salami pork belly burgdoggen. Cupim swine pastrami, chuck tri-tip pork belly jowl shankle alcatra brisket capicola ball tip prosciutto beef ribs doner. Tri-tip bacon ground round pork chop burgdoggen leberkas pork strip steak beef corned beef salami.
{: .text-truncate}


## 代码
1. 一行代码
    `printf("helloworld")`

2. 代码块
    支持的语言包括:c, abnf, accesslog, actionscript, ada, apache, applescript, arduino, armasm, asciidoc, aspectj, autohotkey, autoit, avrasm, awk, axapta, bash, basic, bnf, brainfuck, cal, capnproto, ceylon, clean, clojure, clojure-repl, cmake, coffeescript, coq, cos, cpp, crmsh, crystal, cs, csp, css, d, dart, delphi, diff, django, dns, dockerfile, dos, dsconfig, dts, dust, ebnf, elixir, elm, erb, erlang, erlang-repl, excel, fix, flix, fortran, fsharp, gams, gauss, gcode, gherkin, glsl, go, golo, gradle, groovy, haml, handlebars, haskell, haxe, hsp, htmlbars, http, hy, inform7, ini, irpf90, java, javascript, json, julia, kotlin, lasso, ldif, leaf, less, lisp, livecodeserver, livescript, llvm, lsl, lua, makefile, markdown, mathematica, matlab, maxima, mel, mercury, mipsasm, mizar, mojolicious, monkey, moonscript, n1ql, nginx, nimrod, nix, nsis, objectivec, ocaml, openscad, oxygene, parser3, perl, pf, php, pony, powershell, processing, profile, prolog, protobuf, puppet, purebasic, python, q, qml, r, rib, roboconf, rsl, ruby, ruleslanguage, rust, scala, scheme, scilab, scss, smali, smalltalk, sml, sqf, sql, stan, stata, step21, stylus, subunit, swift, taggerscript, tap, tcl, tex, thrift, tp, twig, typescript, vala, vbnet, vbscript, vbscript-html, verilog, vhdl, vim, x86asm, xl, xml, xquery, yaml, zephir

    ```css
    #container {
    float: left;
    margin: 0 -240px 0 0;
    width: 100%;
    }
    ```
    
    ```python
    var1 = 'Hello World!'
    print "var1[0]: ", var1[0]
    ```

    ```html
    <p>hehe</p>
    <h>haha</h>
    ```

    ```java
    String s="sunlingzhi";
    ```

    ```javascript
    var c="sunlingzhi";
    ```


3. 通过大括号进行高亮 highlight 后面可以加 scss,html(特别注意当是html是要添加raw避免其作为HTML要素插入文档)  以及linenos进行行数统计.
{% highlight scss %}
.highlight {
  margin: 0;
  padding: 1em;
  font-family: $monospace;
  font-size: $type-size-7;
  line-height: 1.8;
}
{% endhighlight %}


{% highlight html linenos %}
{% raw %}
<nav class="pagination" role="navigation">
    <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
</nav><!-- /.pagination -->
{% endraw %}
{% endhighlight %}

## 分割线
---
***



## Table
mardown的表格是画出来的,推荐利用[Table Generator](http://www.tablesgenerator.com/markdown_tables)生成符合markdown语法的表格,在粘贴到markdown中.

```
默认的标题居中对齐,内容居左对齐
第二行为对齐方式
:- 左对齐
:-: 居中对齐
-: 右对齐
```

| Tables   |      Are      |  Cool |
|----------|:-------------:|------:|
| col 1 is |  left-aligned | $1600 |
| col 2 is |    centered   |   $12 |
| col 3 is | right-aligned |    $1 |

## 解释
a
: what is a 

b
: what is b


## 列表

* List item one 
  * List item one 
    * List item one
    * List item two
    * List item three
    * List item four
  * List item two
  * List item three
  * List item four

  1. List item one 
      1. List item one 
          1. List item one
          2. List item two
          3. List item three
          4. List item four
      2. List item two
      3. List item three
      4. List item four
  2. List item two
  3. List item three
  4. List item four

## 地址
<address>
  United States,NewYork City
</address>



## 视频
一般使用iframe对视频源进行嵌入.  
```html
<!-- 16:9 aspect ratio -->
<div class="embed-responsive embed-responsive-16by9">
  <iframe class="embed-responsive-item" src="..."></iframe>
</div>

<!-- 4:3 aspect ratio -->
<div class="embed-responsive embed-responsive-4by3">
  <iframe class="embed-responsive-item" src="..."></iframe>
</div>
```


## 公式

### 直接编辑公式
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML' async></script>
直接在markdown中添加[MathJax](https://www.mathjax.org/),就可以在代码中书写公式.

``` html
<script src='https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.4/MathJax.js?config=TeX-MML-AM_CHTML' async></script>

<!--
  \\(行内内容\\)
  $$块内容$$
-->
```
When \\(a \ne 0\\), there are two solutions to \\((ax^2 + bx + c = 0)\\) and they are

$$x = {-b \pm \sqrt{b^2-4ac} \over 2a}.$$

### 利用在线网站,嵌入图片
在[latex](http://latex.codecogs.com/eqneditor/editor.php)中书写公式,将公式导为图片,在添加到markdown中.

![公式1](https://latex.codecogs.com/gif.latex?ax^{2}&space;&plus;&space;by^{2}&space;&plus;&space;c&space;=&space;0)


