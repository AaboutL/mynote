---
title: 【pyreverse + Graphviz】导出python类图
updated: 2024-01-24 07:17:52Z
created: 2024-01-03 04:04:05Z
latitude: 39.90421100
longitude: 116.40739500
altitude: 0.0000
---

# 1\. 安装

pyreverse包含于pylint中，直接安装pylint就可以  
graphviz：在macbook上使用pip安装，无法被pyreverse识别。需要用brewhome安装

# 2\. 使用

pyreverse参数  
Create UML diagrams for classes and modules in <packages>.</packages>

options:  
\-h, --help show this help message and exit

Pyreverse:  
Base class providing common behaviour for pyreverse commands.

| 选项  | 简写  | 功能  |
| --- | --- | --- |
| \--filter-mode | \-f | 根据<mode>过滤掉属性和函数，默认使用 "PUB_ONLY"过滤掉所有非public属性  <br>equivalent to PRIVATE+SPECIAL_A 'ALL' no filter 'SPECIAL' filter Python special functions except constructor 'OTHER' filter protected and private attributes(default: PUB_ONLY)</mode> |
| \--class | \-c | 创建与类<class>相关的所有类的类图， this uses by default the options -ASmy (default: \[\])</class> |
| \--show-ancestors | \-a | 显示不在类<projects>中的祖先类 (default: None)</projects> |
| \--all-ancestors | \-A | 显示<projects>中所有类的所有祖先 (default: None)</projects> |
| \--show-associated &lt;association_level&gt; | \-s &lt;association_level&gt; | 显示不在类<projects>中的关联类的关联级别 (default: None)</projects> |
| \--all-associated | \-S | 迭代显示所有关联类 (default: None) |
| \--show-builtin | \-b | 在classes的表示中包含内建对象(default: False) |
| \--module-names | \-m | 在类的表示中显示模块名 (default: None) |
| \--only-classnames | \-k | 不在类框中显示属性和方法，会屏蔽-f的值。(default: False) |
| \--output | \-o | 创建<format>后缀的文件，(default: dot)</format> |
| \--colorized |     | 使用带颜色的输出。同一个包的类/模块的颜色相同。(default: False) |
| \--max-color-depth |     | Use separate colors up to package depth of <depth>(default: 2)</depth> |
| \--ignore &lt;file\[,file...\]&gt; |     | 需要被跳过的文件或目录，必须是base names，不能是paths， (default: ('CVS',)) |
| \--project | \-p | 设置project名，(default: ) |
| \--output-directory &lt;output_directory&gt; | \-d &lt;output_directory&gt; | 设置输出目录的路径(default: ) |

\--filter-mode <mode>, -f<mode>  
filter attributes and functions according to <mode>. Correct modes are : 'PUB_ONLY' filter all non public attributes \[DEFAULT\], equivalent to  
PRIVATE+SPECIAL_A 'ALL' no filter 'SPECIAL' filter Python special functions except constructor 'OTHER' filter protected and private attributes  
(default: PUB_ONLY)  
\--class <class>, -c<class>  
create a class diagram with all classes related to <class>; this uses by default the options -ASmy (default: \[\])  
\--show-ancestors <ancestor>, -a<ancestor>  
show <ancestor>generations of ancestor classes not in <projects>(default: None)  
\--all-ancestors, -A show all ancestors off all classes in <projects>(default: None)  
\--show-associated &lt;association_level&gt;, -s &lt;association_level&gt;  
show &lt;association_level&gt; levels of associated classes not in <projects>(default: None)  
\--all-associated, -S show recursively all associated off all associated classes (default: None)  
\--show-builtin, -b include builtin objects in representation of classes (default: False)  
\--module-names <y or="" n="">, -m<y or="" n="">  
include module name in representation of classes (default: None)  
\--only-classnames, -k  
don't show attributes and methods in the class boxes; this disables -f values (default: False)  
\--output <format>, -o<format>  
create a \*. <format>output file if format is available. Available formats are: dot, vcg, puml, plantuml, mmd, html. Any other format will be tried to  
create by means of the 'dot' command line tool, which requires a graphviz installation. (default: dot)  
\--colorized Use colored output. Classes/modules of the same package get the same color. (default: False)  
\--max-color-depth<depth>  
Use separate colors up to package depth of <depth>(default: 2)  
\--ignore &lt;file\[,file...\]&gt;  
Files or directories to be skipped. They should be base names, not paths. (default: ('CVS',))  
\--project <project name="">, -p<project name="">  
set the project name. (default: )  
\--output-directory &lt;output_directory&gt;, -d &lt;output_directory&gt;  
set the output directory path. (default: )</project></project></depth></depth></format></format></format></y></y></projects></projects></projects></ancestor></ancestor></ancestor></class></class></class></mode></mode></mode>