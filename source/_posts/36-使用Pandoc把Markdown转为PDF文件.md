---
title: 使用Pandoc把Markdown转为PDF文件
author: hypo
img: medias/featureimages/54.jpeg
top: false
cover: false
toc: true
mathjax: false
date: 2022-07-06 14:28:00
coverImg:
password:
summary: Pandoc可以很方便地对不同 Markup 语言的文件进行格式转换，因此被誉为格式转换的「瑞士军刀」。使用 Pandoc 把 Markdown 文件转为 PDF 文件，实际上包含两个步骤：
categories: 笔记
tags:
- 笔记
- pandoc
- markdown
- pdf
---
# 使用 Pandoc 把 Markdown 转为 PDF 文件



## 先决条件

* 首先当然是 Pandoc，下载[最新的安装包](https://github.com/jgm/pandoc/releases)，安装完以后，记得把 Pandoc 安装目录加入系统 `PATH` 变量。
* TeX 发行版。请确保你的系统已经安装了 TeX 软件，你可以使用 [TeX Live](https://www.tug.org/texlive/) 或者 [MiKTeX](https://miktex.org/)，安装完成之后可能需要设置 `PATH`，推荐安装 TeX Live。
* 一个强大的文本编辑器

## 背景介绍

[Pandoc](https://pandoc.org/) 可以很方便地对不同 Markup 语言的文件进行格式转换，因此被誉为格式转换的「瑞士军刀」。使用 Pandoc 把 Markdown 文件转为 PDF 文件，实际上包含两个步骤：

## 方法一
- 第一步， Markdown 文件被转为 LaTeX 源文件。
- 第二步，调用系统的 `pdflatex`, `xelatex` 或者其他 TeX 命令，将 `.tex` 文件转换为最终的 PDF 文件 (见上图)。

由于我的文档是中文，并且包含引用，表格等比较复杂的格式，在文件转换过程中遇到了一些问题，下面介绍具体解决办法。

### 如何处理中文

Pandoc 默认使用的 pdflatex 命令无法处理 Unicode 字符，如果要把包含中文的Markdown 文件转为 PDF，在生成 PDF 的过程中会报错。我们需要使用 xelatex 来处理中文，并且需要使用 CJKmainfont 选项指定支持中文的字体。 在 Windows 系统中，对于 Pandoc 2.0 版本以上，可以使用以下的命令生成 PDF 文件：

```
pandoc --pdf-engine=xelatex -V CJKmainfont="Noto Sans Mono CJK SC:style=Regular" test.md -o test.pdf
```

CJKmainfont 后面跟的是支持中文的字体名称。那么如何找到支持中文的字体呢？ 首先，你需要知道你所使用的语言的 language code, 例如，中文(即Chinese)的 language code 是 zh。 然后使用 fc-list 命令查看支持中文的字体(对于 Windows 系统，fc-list 命令在安装 TeX Live 完整版以后可以使用, Unix 系统一般会预装这个程序)：

```
fc-list :lang=zh # zh 是中文的 「language code」
```

在 Linux 系统上，找出支持中文的字体的方式与 Windows 系统是一样的。


### 添加文档标题，作者，更新时间信息
Pandoc 支持 YAML 格式的 header，通过 header 可以指定文章的标题，作者，更新时间等信息，一个示例 header 如下：

---

data:"2022.7.7"
title: "My demo title"
author: "sfsf"

---

### 给 block code 加上 highlight

Pandoc 支持给 block code 里面的代码加上背景高亮，并提供了不同的高亮主题，支持非常多的语言。要列出 Pandoc 提供的高亮方案，使用下面命令，

```
pandoc --list-highlight-styles
```

要列出所有支持的语言，使用下面命令，

```
pandoc --list-highlight-languages
```

要使用语法高亮，Markdown 文件中的 block code 必须指定语言，同时在命令行使用--highlight-style 选项，例如：

```
pandoc --pdf-engine=xelatex --highlight-style zenburn test.md -o test.pdf
```

以上命令，使用了 zenburn 主题, 另外，也推荐使用 tango, zenburn 或者breezedark 高亮主题。


### 给链接加上颜色

根据 Pandoc user guide 的说明，我们可以通过 colorlinks 选项给各种链接加上颜色，便于和普通文本区分开来：

```
colorlinks
    add color to link text; automatically enabled if any of linkcolor, filecolor, citecolor, urlcolor, or toccolor are set
```

同时，为了精确控制不同类型链接颜色，Pandoc 还提供了对不同链接颜色的个性化设置选项：

```
linkcolor, filecolor, citecolor, urlcolor, toccolor
    color for internal links, external links, citation links, linked URLs, and
    links in table of contents, respectively: uses options allowed by xcolor,
    including the dvipsnames, svgnames, and x11names lists
```

例如，如果我们想给 URL 链接加上颜色，并且 urlcolor 要设为 NavyBlue, 可以使用下面的命令：

```
pandoc --pdf-engine=xelatex -V colorlinks -V urlcolor=NavyBlue test.md -o test.pdf
```

其他链接的颜色可以按照上述方式设置。

值得注意的是，urlcolor 选项并不能给文中直接展示的 raw URL link 加上颜色。 要给直接展示的 URL link 加上颜色，可以用 <> 包围要展示的链接，例如<www.google.com>。

另外，也可以使用选项 -f gfm，参考这里。完整命令如下，

```
pandoc --pdf-engine=xelatex -f gfm -Vurlcolor=cyan -o test.pdf test.md
```

使用 -f gfm 的一个缺点是 gfm 不支持公式，因此如果在 Markdown 中包含公式，将不能正确渲染。解决办法是去掉 -f gfm flag，或者使用 Pandoc 自带的 markdown格式。


### 更改 PDF 的 margin

使用默认设置生成的 PDF margin 太大，根据 Pandoc 官方FAQ，可以使用下面的选项更改 margin：

```
-V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm"
```

完整命令为：

```
pandoc --pdf-engine=xelatex -V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm" -o test.pdf test.md
```

### Markdown 文件中使用斜杠 backslash 会出错

原始的 Markdown 格式，支持在文件中使用 backslash，但是 Pandoc 把 backslash 以及后面的内容理解成 LaTeX 命令，因此在编译包含 backslash 的文件时，可能会遇到undefined command 错误或者更加奇怪的错误。参考这里以及这里，解决办法是让 Pandoc 把 Markdown文件当成正常的 Markdown，不要解读 LaTeX 命令，使用下面的 flag:

```
pandoc -f markdown-raw_tex
```

### 直接上命令

```
pandoc --pdf-engine=xelatex -V geometry:"top=2cm, bottom=1.5cm, left=2cm, right=2cm" -V CJKmainfont="Noto Sans Mono CJK SC:style=Regular" -V colorlinks -V urlcolor=NavyBlue -f markdown-raw_tex test.md -o test.pdf
```

## 方法二
>以上的方法一是直接使用pandoc进行转化，下面介绍一下两步转化，即先转为tex文件再转成pdf文件

```sh
#使用pandoc先将test.md文件转为无格式的output.tex文件
pandoc -r markdown-auto_identifiers --no-highlight -w latex -o output.tex test.md
```
 其中一些字段需要修改的可以使用linux自带的sed命令
 使用 sed 替换字符串的语法。

 ```sh
 sed -i 's/Search_String/Replacement_String/g' Input_File 
 ```

 首先我们需要了解 sed 语法来做到这一点。请参阅有关的细节。

sed：这是一个 Linux 命令。
-i：这是 sed 命令的一个选项，它有什么作用？默认情况下，sed 打印结果到标准输出。当你使用 sed 添加这个选项时，那么它会在适当的位置修改文件。当你添加一个后缀（比如，-i.bak）时，就会创建原始文件的备份。
s：字母 s 是一个替换命令。
Search_String：搜索一个给定的字符串或正则表达式。
Replacement_String：替换的字符串。
g：全局替换标志。默认情况下，sed 命令替换每一行第一次出现的模式，它不会替换行中的其他的匹配结果。但是，提供了该替换标志时，所有匹配都将被替换。
/：分界符。
Input_File：要执行操作的文件名。

然后将修改后的output.tex文件插入到latex的模板文件template.tex中，这里推荐一个模板
```latex
% !TEX encoding = UTF-8 Unicode
\documentclass[a4paper,11pt,twoside,fontset = none,UTF8]{ctexart} % 页面A4纸大小，11 磅大小的字体，式样为双面，字体集为Fandol，编码为UTF8，文档类型为cTex的art（支持中文）
\usepackage[a4paper,scale=0.8,hcentering]{geometry} % A4纸大小，缩放80%，设置页面居中
\usepackage{hyperref}      % 超链接
\usepackage{listings}      % 代码块
\usepackage{courier}       % 字体
\usepackage{fontspec}      % 字体
\usepackage{fancyhdr}      % 页眉页脚相关宏包
\usepackage{lastpage}      % 引用最后一页
\usepackage{amsmath,amsthm,amsfonts,amssymb,bm} %数学
\usepackage{graphicx}      % 图片
\usepackage{subcaption}    % 图片描述
\usepackage{longtable,booktabs} % 表格
\setCJKsansfont{Noto Sans SC}  %设置字体为Noto Sans SC
\usepackage{xcolor}
\lstset{
    language=Markdown,  %代码语言使用的是Markdown
    frame=single, %把代码用框圈起来
    numbers=left, % 显示行号
    numberstyle=\tiny,    % 行号字体
    stringstyle=\ttfamily, % 代码字符串的特殊格式
    breaklines=true, %对过长的代码自动换行
    extendedchars=false,  %解决代码跨页时，章节标题，页眉等汉字不显示的问题
    escapebegin=\begin{CJK*}
                    ,escapeend=
    \end{CJK*},      % 代码中出现中文必须加上，否则报错
    texcl=true}
\pagestyle{fancy}         % 页眉页脚风格
\fancyhf{}                % 清空当前设置
\fancyfoot[C]{\thepage\ }%页脚中间显示 当前页 / 总页数，把\label{LastPage}放在最后
\begin{document}
%    insert_begin/
%replace%
%    insert_end/
\end{document}
\label{LastPage}
```

之后将完整的latex文件使用以下命令，即可打印出pdf文件
```sh
latexmk -pdflatex=xelatex -synctex=1 -interaction=nonstopmode -file-line-error -pdf -outdir=test.pdf template.tex
```
祝各位成功！