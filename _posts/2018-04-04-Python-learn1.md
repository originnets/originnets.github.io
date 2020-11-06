---
layout: post
title: python 
categories: python
description: Python
keywords: python
---


# Python之路,Day1 - Python基础1 #

## 本节内容 ##

1. Python介绍
2. 发展史
3. Python 2 or 3?
4. 安装
5. Hello World程序
6. 变量
7. 用户输入
8. 模块初识
9. .pyc是个什么鬼？
10. 数据类型初识
11. 数据运算
12. 表达式if ...else语句
13. 表达式for 循环
14. break and continue 
15. 表达式while 循环
16. 作业需求
 

## 一、 Python介绍 ##
python的创始人为吉多·范罗苏姆（Guido van Rossum）。1989年的圣诞节期间，吉多·范罗苏姆为了在阿姆斯特丹打发时间，决心开发一个新的脚本解释程序，作为ABC语言的一种继承。  

最新的TIOBE排行榜，Python赶超PHP占据第五， Python崇尚优美、清晰、简单，是一个优秀并广泛使用的语言。
![](https://images2015.cnblogs.com/blog/720333/201605/720333-20160506115944794-1155531832.png)

由上图可见，Python整体呈上升趋势，反映出Python应用越来越广泛并且也逐渐得到业内的认可！！！

Python可以应用于众多领域，如：数据分析、组件集成、网络服务、图像处理、数值计算和科学计算等众多领域。目前业内几乎所有大中型互联网企业都在使用Python，如：Youtube、Dropbox、BT、Quora（中国知乎）、豆瓣、知乎、Google、Yahoo!、Facebook、NASA、百度、腾讯、汽车之家、美团等。

目前Python主要应用领域：

- 云计算: 云计算最火的语言， 典型应用OpenStack
- WEB开发: 众多优秀的WEB框架，众多大型网站均为Python开发，Youtube, Dropbox, 豆瓣。。。， 典型WEB框架有Django
- 科学运算、人工智能: 典型库NumPy, SciPy, Matplotlib, Enthought librarys,pandas
- 系统运维: 运维人员必备语言
- 金融：量化交易，金融分析，在金融工程领域，Python不但在用，且用的最多，而且重要性逐年提高。原因：作为动态语言的Python，语言结构清晰简单，库丰富，成熟稳定，科学计算和统计分析都很牛逼，生产效率远远高于c,c++,java,尤其擅长策略回测
- 图形GUI: PyQT, WxPython,TkInter
 
Python在一些公司的应用： 
 
- 谷歌：Google App Engine 、code.google.com 、Google earth 、谷歌爬虫、Google广告等项目都在大量使用Python开发
- CIA: 美国中情局网站就是用Python开发的
- NASA: 美国航天局(NASA)大量使用Python进行数据分析和运算
- YouTube:世界上最大的视频网站YouTube就是用Python开发的
- Dropbox:美国最大的在线云存储网站，全部用Python实现，每天网站处理10亿个文件的上传和下载
- Instagram:美国最大的图片分享社交网站，每天超过3千万张照片被分享，全部用python开发
- Facebook:大量的基础库均通过Python实现的
- Redhat: 世界上最流行的Linux发行版本中的yum包管理工具就是用python开发的
- 豆瓣: 公司几乎所有的业务均是通过Python开发的
- 知乎: 国内最大的问答社区，通过Python开发(国外Quora)
- 春雨医生：国内知名的在线医疗网站是用Python开发的
- 除上面之外，还有搜狐、金山、腾讯、盛大、网易、百度、阿里、淘宝 、土豆、新浪、果壳等公司都在使用Python完成各种各样的任务。
 
### Python 是一门什么样的语言？ ###

----------

编程语言主要从以下几个角度为进行分类，编译型和解释型、静态语言和动态语言、强类型定义语言和弱类型定义语言，每个分类代表什么意思呢，我们一起来看一下。

编译和解释的区别是什么？
编译器是把源程序的每一条语句都编译成机器语言,并保存成二进制文件,这样运行时计算机可以直接以机器语言来运行此程序,速度很快; 

而解释器则是只在执行程序时,才一条一条的解释成机器语言给计算机来执行,所以运行速度是不如编译后的程序运行的快的. 

这是因为计算机不能直接认识并执行我们写的语句,它只能认识机器语言(是二进制的形式)

![](https://images2015.cnblogs.com/blog/720333/201703/720333-20170320124101861-719256772.png)
![](https://images2015.cnblogs.com/blog/720333/201703/720333-20170320140758940-1485075006.png)

### 编译型vs解释型 ###
- 编译型
优点：编译器一般会有预编译的过程对代码进行优化。因为编译只做一次，运行时不需要编译，所以编译型语言的程序执行效率高。可以脱离语言环境独立运行。
缺点：编译之后如果需要修改就需要整个模块重新编译。编译的时候根据对应的运行环境生成机器码，不同的操作系统之间移植就会有问题，需要根据运行的操作系统环境编译不同的可执行文件。

- 解释型
优点：有良好的平台兼容性，在任何环境中都可以运行，前提是安装了解释器（虚拟机）。灵活，修改代码的时候直接修改就可以，可以快速部署，不用停机维护。

- 缺点：每次运行的时候都要解释一遍，性能上不如编译型语言。

 

 

#### 一、低级语言与高级语言 ####

最初的计算机程序都是用0和1的序列表示的，程序员直接使用的是机器指令，无需翻译，从纸带打孔输入即可执行得到结果。后来为了方便记忆，就将用0、1序列表示的机器指令都用符号助记，这些与机器指令一一对应的助记符就成了汇编指令，从而诞生了汇编语言。无论是机器指令还是汇编指令都是面向机器的，统称为低级语言。因为是针对特定机器的机器指令的助记符，所以汇编语言是无法独立于机器(特定的CPU体系结构)的。但汇编语言也是要经过翻译成机器指令才能执行的，所以也有将运行在一种机器上的汇编语言翻译成运行在另一种机器上的机器指令的方法，那就是交叉汇编技术。

高级语言是从人类的逻辑思维角度出发的计算机语言，抽象程度大大提高，需要经过编译成特定机器上的目标代码才能执行，一条高级语言的语句往往需要若干条机器指令来完成。高级语言独立于机器的特性是靠编译器为不同机器生成不同的目标代码(或机器指令)来实现的。那具体的说，要将高级语言编译到什么程度呢，这又跟编译的技术有关了，既可以编译成直接可执行的目标代码，也可以编译成一种中间表示，然后拿到不同的机器和系统上去执行，这种情况通常又需要支撑环境，比如解释器或虚拟机的支持，Java程序编译成bytecode，再由不同平台上的虚拟机执行就是很好的例子。所以，说高级语言不依赖于机器，是指在不同的机器或平台上高级语言的程序本身不变，而通过编译器编译得到的目标代码去适应不同的机器。从这个意义上来说，通过交叉汇编，一些汇编程序也可以获得不同机器之间的可移植性，但这种途径获得的移植性远远不如高级语言来的方便和实用性大。


#### 二、编译与解释 ####

编译是将源程序翻译成可执行的目标代码，翻译与执行是分开的；而解释是对源程序的翻译与执行一次性完成，不生成可存储的目标代码。这只是表象，二者背后的最大区别是：对解释执行而言，程序运行时的控制权在解释器而不在用户程序；对编译执行而言，运行时的控制权在用户程序。

解释具有良好的动态特性和可移植性，比如在解释执行时可以动态改变变量的类型、对程序进行修改以及在程序中插入良好的调试诊断信息等，而将解释器移植到不同的系统上，则程序不用改动就可以在移植了解释器的系统上运行。同时解释器也有很大的缺点，比如执行效率低，占用空间大，因为不仅要给用户程序分配空间，解释器本身也占用了宝贵的系统资源。

编译器是把源程序的每一条语句都编译成机器语言,并保存成二进制文件,这样运行时计算机可以直接以机器语言来运行此程序,速度很快;
而解释器则是只在执行程序时,才一条一条的解释成机器语言给计算机来执行,所以运行速度是不如编译后的程序运行的快的.

 

#### 编译型和解释型 ####
我们先看看编译型，其实它和汇编语言是一样的：也是有一个负责翻译的程序来对我们的源代码进行转换，生成相对应的可执行代码。这个过程说得专业一点，就称为编译（Compile），而负责编译的程序自然就称为编译器（Compiler）。如果我们写的程序代码都包含在一个源文件中，那么通常编译之后就会直接生成一个可执行文件，我们就可以直接运行了。但对于一个比较复杂的项目，为了方便管理，我们通常把代码分散在各个源文件中，作为不同的模块来组织。这时编译各个文件时就会生成目标文件（Object   file）而不是前面说的可执行文件。一般一个源文件的编译都会对应一个目标文件。这些目标文件里的内容基本上已经是可执行代码了，但由于只是整个项目的一部分，所以我们还不能直接运行。待所有的源文件的编译都大功告成，我们就可以最后把这些半成品的目标文件“打包”成一个可执行文件了，这个工作由另一个程序负责完成，由于此过程好像是把包含可执行代码的目标文件连接装配起来，所以又称为链接（Link），而负责链接的程序就叫……就叫链接程序（Linker）。链接程序除了链接目标文件外，可能还有各种资源，像图标文件啊、声音文件啊什么的，还要负责去除目标文件之间的冗余重复代码，等等，所以……也是挺累的。链接完成之后，一般就可以得到我们想要的可执行文件了。 

上面我们大概地介绍了编译型语言的特点，现在再看看解释型。噢，从字面上看，“编译”和“解释”的确都有“翻译”的意思，它们的区别则在于翻译的时机安排不大一样。打个比方：假如你打算阅读一本外文书，而你不知道这门外语，那么你可以找一名翻译，给他足够的时间让他从头到尾把整本书翻译好，然后把书的母语版交给你阅读；或者，你也立刻让这名翻译辅助你阅读，让他一句一句给你翻译，如果你想往回看某个章节，他也得重新给你翻译。 

两种方式，前者就相当于我们刚才所说的编译型：一次把所有的代码转换成机器语言，然后写成可执行文件；而后者就相当于我们要说的解释型：在程序运行的前一刻，还只有源程序而没有可执行程序；而程序每执行到源程序的某一条指令，则会有一个称之为解释程序的外壳程序将源代码转换成二进制代码以供执行，总言之，就是不断地解释、执行、解释、执行……所以，解释型程序是离不开解释程序的。像早期的BASIC就是一门经典的解释型语言，要执行BASIC程序，就得进入BASIC环境，然后才能加载程序源文件、运行。解释型程序中，由于程序总是以源代码的形式出现，因此只要有相应的解释器，移植几乎不成问题。编译型程序虽然源代码也可以移植，但前提是必须针对不同的系统分别进行编译，对于复杂的工程来说，的确是一件不小的时间消耗，况且很可能一些细节的地方还是要修改源代码。而且，解释型程序省却了编译的步骤，修改调试也非常方便，编辑完毕之后即可立即运行，不必像编译型程序一样每次进行小小改动都要耐心等待漫长的Compiling…Linking…这样的编译链接过程。不过凡事有利有弊，由于解释型程序是将编译的过程放到执行过程中，这就决定了解释型程序注定要比编译型慢上一大截，像几百倍的速度差距也是不足为奇的。 

编译型与解释型，两者各有利弊。前者由于程序执行速度快，同等条件下对系统要求较低，因此像开发操作系统、大型应用程序、数据库系统等时都采用它，像C/C++、Pascal/Object   Pascal（Delphi）、VB等基本都可视为编译语言，而一些网页脚本、服务器脚本及辅助开发接口这样的对速度要求不高、对不同系统平台间的兼容性有一定要求的程序则通常使用解释性语言，如Java、JavaScript、VBScript、Perl、Python等等。 

但既然编译型与解释型各有优缺点又相互对立，所以一批新兴的语言都有把两者折衷起来的趋势，例如Java语言虽然比较接近解释型语言的特征，但在执行之前已经预先进行一次预编译，生成的代码是介于机器码和Java源代码之间的中介代码，运行的时候则由JVM（Java的虚拟机平台，可视为解释器）解释执行。它既保留了源代码的高抽象、可移植的特点，又已经完成了对源代码的大部分预编译工作，所以执行起来比“纯解释型”程序要快许多。而像VB6（或者以前版本）、C#这样的语言，虽然表面上看生成的是.exe可执行程序文件，但VB6编译之后实际生成的也是一种中介码，只不过编译器在前面安插了一段自动调用某个外部解释器的代码（该解释程序独立于用户编写的程序，存放于系统的某个DLL文件中，所有以VB6编译生成的可执行程序都要用到它），以解释执行实际的程序体。C#（以及其它.net的语言编译器）则是生成.net目标代码，实际执行时则由.net解释系统（就像JVM一样，也是一个虚拟机平台）进行执行。当然.net目标代码已经相当“低级”，比较接近机器语言了，所以仍将其视为编译语言，而且其可移植程度也没有Java号称的这么强大，Java号称是“一次编译，到处执行”，而.net则是“一次编码，到处编译”。呵呵，当然这些都是题外话了。总之，随着设计技术与硬件的不断发展，编译型与解释型两种方式的界限正在不断变得模糊。

动态语言和静态语言
通常我们所说的动态语言、静态语言是指动态类型语言和静态类型语言。


1. 动态类型语言：动态类型语言是指在运行期间才去做数据类型检查的语言，也就是说，在用动态类型的语言编程时，永远也不用给任何变量指定数据类型，该语言会在你第一次赋值给变量时，在内部将数据类型记录下来。Python和Ruby就是一种典型的动态类型语言，其他的各种脚本语言如VBScript也多少属于动态类型语言。

2. 静态类型语言：静态类型语言与动态类型语言刚好相反，它的数据类型是在编译其间检查的，也就是说在写程序时要声明所有变量的数据类型，C/C++是静态类型语言的典型代表，其他的静态类型语言还有C#、JAVA等。

 

强类型定义语言和弱类型定义语言

1. 强类型定义语言：强制数据类型定义的语言。也就是说，一旦一个变量被指定了某个数据类型，如果不经过强制转换，那么它就永远是这个数据类型了。举个例子：如果你定义了一个整型变量a,那么程序根本不可能将a当作字符串类型处理。强类型定义语言是类型安全的语言。

2. 弱类型定义语言：数据类型可以被忽略的语言。它与强类型定义语言相反, 一个变量可以赋不同数据类型的值。

强类型定义语言在速度上可能略逊色于弱类型定义语言，但是强类型定义语言带来的严谨性能够有效的避免许多错误。另外，“这门语言是不是动态语言”与“这门语言是否类型安全”之间是完全没有联系的！
例如：Python是动态语言，是强类型定义语言（类型安全的语言）; VBScript是动态语言，是弱类型定义语言（类型不安全的语言）; JAVA是静态语言，是强类型定义语言（类型安全的语言）。

 

通过上面这些介绍，我们可以得出，python是一门动态解释性的强类型定义语言。那这些基因使成就了Python的哪些优缺点呢？我们继续往下看。

 
### Python的优缺点 ###

----------

先看优点

1. Python的定位是“优雅”、“明确”、“简单”，所以Python程序看上去总是简单易懂，初学者学Python，不但入门容易，而且将来深入下去，可以编写那些非常非常复杂的程序。
2. 开发效率非常高，Python有非常强大的第三方库，基本上你想通过计算机实现任何功能，Python官方库里都有相应的模块进行支持，直接下载调用后，在基础库的基础上再进行开发，大大降低开发周期，避免重复造轮子。
3. 高级语言————当你用Python语言编写程序的时候，你无需考虑诸如如何管理你的程序使用的内存一类的底层细节
4. 可移植性————由于它的开源本质，Python已经被移植在许多平台上（经过改动使它能够工 作在不同平台上）。如果你小心地避免使用依赖于系统的特性，那么你的所有Python程序无需修改就几乎可以在市场上所有的系统平台上运行
5. 可扩展性————如果你需要你的一段关键代码运行得更快或者希望某些算法不公开，你可以把你的部分程序用C或C++编写，然后在你的Python程序中使用它们。
6. 可嵌入性————你可以把Python嵌入你的C/C++程序，从而向你的程序用户提供脚本功能。
再看缺点：

1. 速度慢，Python 的运行速度相比C语言确实慢很多，跟JAVA相比也要慢一些，因此这也是很多所谓的大牛不屑于使用Python的主要原因，但其实这里所指的运行速度慢在大多数情况下用户是无法直接感知到的，必须借助测试工具才能体现出来，比如你用C运一个程序花了0.01s,用Python是0.1s,这样C语言直接比Python快了10倍,算是非常夸张了，但是你是无法直接通过肉眼感知的，因为一个正常人所能感知的时间最小单位是0.15-0.4s左右，哈哈。其实在大多数情况下Python已经完全可以满足你对程序速度的要求，除非你要写对速度要求极高的搜索引擎等，这种情况下，当然还是建议你用C去实现的。
2. 代码不能加密，因为PYTHON是解释性语言，它的源码都是以名文形式存放的，不过我不认为这算是一个缺点，如果你的项目要求源代码必须是加密的，那你一开始就不应该用Python来去实现。
3. 线程不能利用多CPU问题，这是Python被人诟病最多的一个缺点，GIL即全局解释器锁（Global Interpreter Lock），是计算机程序设计语言解释器用于同步线程的工具，使得任何时刻仅有一个线程在执行，Python的线程是操作系统的原生线程。在Linux上为pthread，在Windows上为Win thread，完全由操作系统调度线程的执行。一个python解释器进程内有一条主线程，以及多条用户程序的执行线程。即使在多核CPU平台上，由于GIL的存在，所以禁止多线程的并行执行。关于这个问题的折衷解决方法，我们在以后线程和进程章节里再进行详细探讨。
 

当然，Python还有一些其它的小缺点，在这就不一一列举了，我想说的是，任何一门语言都不是完美的，都有擅长和不擅长做的事情，建议各位不要拿一个语言的劣势去跟另一个语言的优势来去比较，语言只是一个工具，是实现程序设计师思想的工具，就像我们之前中学学几何时，有的时候需要要圆规，有的时候需要用三角尺一样，拿相应的工具去做它最擅长的事才是正确的选择。之前很多人问我Shell和Python到底哪个好？我回答说Shell是个脚本语言，但Python不只是个脚本语言，能做的事情更多，然后又有钻牛角尖的人说完全没必要学Python, Python能做的事情Shell都可以做，只要你足够牛B,然后又举了用Shell可以写俄罗斯方块这样的游戏，对此我能说表达只能是，不要跟SB理论，SB会把你拉到跟他一样的高度，然后用充分的经验把你打倒。

 

 

### Python解释器 ###
当我们编写Python代码时，我们得到的是一个包含Python代码的以.py为扩展名的文本文件。要运行代码，就需要Python解释器去执行.py文件。

由于整个Python语言从规范到解释器都是开源的，所以理论上，只要水平够高，任何人都可以编写Python解释器来执行Python代码（当然难度很大）。事实上，确实存在多种Python解释器。

CPython

当我们从Python官方网站下载并安装好Python 2.7后，我们就直接获得了一个官方版本的解释器：CPython。这个解释器是用C语言开发的，所以叫CPython。在命令行下运行python就是启动CPython解释器。

CPython是使用最广的Python解释器。教程的所有代码也都在CPython下执行。

IPython

IPython是基于CPython之上的一个交互式解释器，也就是说，IPython只是在交互方式上有所增强，但是执行Python代码的功能和CPython是完全一样的。好比很多国产浏览器虽然外观不同，但内核其实都是调用了IE。

CPython用>>>作为提示符，而IPython用In [序号]:作为提示符。

PyPy

PyPy是另一个Python解释器，它的目标是执行速度。PyPy采用JIT技术，对Python代码进行动态编译（注意不是解释），所以可以显著提高Python代码的执行速度。

绝大部分Python代码都可以在PyPy下运行，但是PyPy和CPython有一些是不同的，这就导致相同的Python代码在两种解释器下执行可能会有不同的结果。如果你的代码要放到PyPy下执行，就需要了解PyPy和CPython的不同点。

Jython

Jython是运行在Java平台上的Python解释器，可以直接把Python代码编译成Java字节码执行。

IronPython

IronPython和Jython类似，只不过IronPython是运行在微软.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码。

小结

----------

Python的解释器很多，但使用最广泛的还是CPython。如果要和Java或.Net平台交互，最好的办法不是用Jython或IronPython，而是通过网络调用来交互，确保各程序之间的独立性。

 

## 二、Python发展史 ## 


- 1989年，为了打发圣诞节假期，Guido开始写Python语言的编译器。Python这个名字，来自Guido所挚爱的电视剧Monty Python’s Flying Circus。他希望这个新的叫做Python的语言，能符合他的理想：创造一种C和shell之间，功能全面，易学易用，可拓展的语言。
- 1991年，第一个Python编译器诞生。它是用C语言实现的，并能够调用C语言的库文件。从一出生，Python已经具有了：类，函数，异常处理，包含表和词典在内的核心数据类型，以及模块为基础的拓展系统。
- Granddaddy of Python web frameworks, Zope 1 was released in 1999
- Python 1.0 January 1994 增加了 lambda, map, filter and reduce.
- Python 2.0 October 16, 2000，加入了内存回收机制，构成了现在Python语言框架的基础
- Python 2.4 November 30, 2004, 同年目前最流行的WEB框架Django 诞生
- Python 2.5 September 19, 2006
- Python 2.6 October 1, 2008
- Python 2.7 July 3, 2010
- In November 2014, it was announced that Python 2.7 would be supported until 2020, and reaffirmed that there would be no 2.8 release as users were expected to move to Python 3.4+ as soon as possible
- Python 3.0 December 3, 2008
- Python 3.1 June 27, 2009
- Python 3.2 February 20, 2011
- Python 3.3 September 29, 2012
- Python 3.4 March 16, 2014
- Python 3.5 September 13, 2015
## 三、Python 2 or 3? ##

In summary : Python 2.x is legacy, Python 3.x is the present and future of the language

Python 3.0 was released in 2008. The final 2.x version 2.7 release came out in mid-2010, with a statement of

extended support for this end-of-life release. The 2.x branch will see no new major releases after that. 3.x is

under active development and has already seen over five years of stable releases, including version 3.3 in 2012,

3.4 in 2014, and 3.5 in 2015. This means that all recent standard library improvements, for example, are only

available by default in Python 3.x.

Guido van Rossum (the original creator of the Python language) decided to clean up Python 2.x properly, with less regard for backwards compatibility than is the case for new releases in the 2.x range. The most drastic improvement is the better Unicode support (with all text strings being Unicode by default) as well as saner bytes/Unicode separation.

Besides, several aspects of the core language (such as print and exec being statements, integers using floor division) have been adjusted to be easier for newcomers to learn and to be more consistent with the rest of the language, and old cruft has been removed (for example, all classes are now new-style, "range()" returns a memory efficient iterable, not a list as in 2.x). 

### py2与3的详细区别 ###
PRINT IS A FUNCTION

The statement has been replaced with a print() function, with keyword arguments to replace most of the special syntax of the old statement (PEP 3105). Examples: 


    Old: print "The answer is", 2*2 New: print("The answer is", 2*2)
    Old: print x, # Trailing comma suppresses newline New: print(x, end=" ") # Appends a space instead of a newline
    Old: print # Prints a newline
    New: print() # You must call the function!
    Old: print >>sys.stderr, "fatal error" New: print("fatal error", file=sys.stderr)
    Old: print (x, y) # prints repr((x, y))
    New: print((x, y)) # Not the same as print(x, y)!

You can also customize the separator between items, e.g.: 


    print("There are <", 2**32, "> possibilities!", sep="")

ALL IS UNICODE NOW

从此不再为讨厌的字符编码而烦恼

 

还可以这样玩： (A,*REST,B)=RANGE(5)


    <strong>>>> a,*rest,b = range(5)
    >>> a,rest,b
    (0, [1, 2, 3], 4)
    </strong>
　　

某些库改名了

 

|	Old Name|	New Name |
|----------:|:----------:|
|	_winreg	|	winreg	|
|	ConfigParser|configparser	|
|	copy_reg|	copyreg	|
|	Queue	|	queue |
|	SocketServer	|socketserver	|
|	markupbase	|	_markupbase	|
|	repr	|	reprlib	|
|	test.test_support	|	test.support	|

　　

#### 还有谁不支持PYTHON3? ####

One popular module that don't yet support Python 3 is Twisted (for networking and other applications). Most

actively maintained libraries have people working on 3.x support. For some libraries, it's more of a priority than

others: Twisted, for example, is mostly focused on production servers, where supporting older versions of

Python is important, let alone supporting a new version that includes major changes to the language. (Twisted is

a prime example of a major package where porting to 3.x is far from trivial 

 

## 四、Python安装 ##
windows

    1、下载安装包
    https://www.python.org/downloads/
    2、安装
    默认安装路径：C:\python27
    3、配置环境变量
    【右键计算机】--》【属性】--》【高级系统设置】--》【高级】--》【环境变量】--》【在第二个内容框中找到 变量名为Path 的一行，双击】 --> 【Python安装目录追加到变值值中，用 ； 分割】
    如：原来的值;C:\python27，切记前面有分号
#### linux、Mac ####

    
    无需安装，原装Python环境
      
    ps：如果自带2.6，请更新至2.7
　　

## 五、Hello World程序 ##
在linux 下创建一个文件叫hello.py,并输入

    print("Hello World!")

然后执行命令:python hello.py ,输出

    localhost:~ jieli$ vim hello.py
    localhost:~ jieli$ python hello.py
    Hello World!

#### 指定解释器 ####

上一步中执行 python hello.py 时，明确的指出 hello.py 脚本由 python 解释器来执行。

如果想要类似于执行shell脚本一样执行python脚本，例： ./hello.py ，那么就需要在 hello.py 文件的头部指定解释器，如下：


    #!/usr/bin/env python
      
    print "hello,world"

如此一来，执行： ./hello.py 即可。

ps：执行前需给予 hello.py 执行权限，chmod 755 hello.py

### 在交互器中执行　 ###

除了把程序写在文件里，还可以直接调用python自带的交互器运行代码，　


    localhost:~ jieli$ python
    Python 2.7.10 (default, Oct 23 2015, 18:05:06)
    [GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> print("Hello World!")
    Hello World!
对比下其它语言的hello world

 C++

     #include <iostream>
     int main(void)
     {
     std::cout<<"Hello world";
     }
 C

     #include <stdio.h>
     int main(void)
     {
     printf("\nhello world!");
     return 0;
     }

 JAVA

     public class HelloWorld{
       // 程序的入口
       public static void main(String args[]){
     // 向控制台输出信息
     System.out.println("Hello World!");
       }
     }

 PHP

     <?php  
     	echo "hello world!";  
     ?>  

 RUBY

    puts "Hello world."

 Go

     package main
     
     import "fmt"
     
     func main(){
     
     	fmt.Printf("Hello World!\n God Bless You!");
     
     }
     

## 六、变量\字符编码　 ##
Variables are used to store information to be referenced and manipulated in a computer program. They also provide a way of labeling data with a descriptive name, so our programs can be understood more clearly by the reader and ourselves. It is helpful to think of variables as containers that hold information. Their sole purpose is to label and store data in memory. This data can then be used throughout your program.

声明变量
    
    #_*_coding:utf-8_*_
     
    name = "Alex Li"
上述代码声明了一个变量，变量名为： name，变量name的值为："Alex Li"　

### 变量定义的规则： ###

- 变量名只能是 字母、数字或下划线的任意组合
- 变量名的第一个字符不能是数字
- 以下关键字不能声明为变量名
 ['and', 'as', 'assert', 'break', 'class', 'continue', 'def', 'del', 'elif', 'else', 'except', 'exec', 'finally', 'for', 'from', 'global', 'if', 'import', 'in', 'is', 'lambda', 'not', 'or', 'pass', 'print', 'raise', 'return', 'try', 'while', 'with', 'yield']

变量的赋值

    name = "Alex Li"
     
    name2 = name
    print(name,name2)
     
    name = "Jack"
     
    print("What is the value of name2 now?")
　

## 七、字符编码 ##
python解释器在加载 .py 文件中的代码时，会对内容进行编码（默认ascill）

ASCII（American Standard Code for Information Interchange，美国标准信息交换代码）是基于拉丁字母的一套电脑编码系统，主要用于显示现代英语和其他西欧语言，其最多只能用 8 位来表示（一个字节），即：2**8 = 256-1，所以，ASCII码最多只能表示 255 个符号。

 ![](https://images2015.cnblogs.com/blog/720333/201607/720333-20160722131729872-496732718.jpg)



关于中文

为了处理汉字，程序员设计了用于简体中文的GB2312和用于繁体中文的big5。

GB2312(1980年)一共收录了7445个字符，包括6763个汉字和682个其它符号。汉字区的内码范围高字节从B0-F7，低字节从A1-FE，占用的码位是72*94=6768。其中有5个空位是D7FA-D7FE。

GB2312 支持的汉字太少。1995年的汉字扩展规范GBK1.0收录了21886个符号，它分为汉字区和图形符号区。汉字区包括21003个字符。2000年的 GB18030是取代GBK1.0的正式国家标准。该标准收录了27484个汉字，同时还收录了藏文、蒙文、维吾尔文等主要的少数民族文字。现在的PC平台必须支持GB18030，对嵌入式产品暂不作要求。所以手机、MP3一般只支持GB2312。

从ASCII、GB2312、GBK 到GB18030，这些编码方法是向下兼容的，即同一个字符在这些方案中总是有相同的编码，后面的标准支持更多的字符。在这些编码中，英文和中文可以统一地处理。区分中文编码的方法是高字节的最高位不为0。按照程序员的称呼，GB2312、GBK到GB18030都属于双字节字符集 (DBCS)。

有的中文Windows的缺省内码还是GBK，可以通过GB18030升级包升级到GB18030。不过GB18030相对GBK增加的字符，普通人是很难用到的，通常我们还是用GBK指代中文Windows内码。

 

 

**显然ASCII码无法将世界上的各种文字和符号全部表示，所以，就需要新出一种可以代表所有字符和符号的编码，即：Unicode**

Unicode（统一码、万国码、单一码）是一种在计算机上使用的字符编码。Unicode 是为了解决传统的字符编码方案的局限而产生的，它为每种语言中的每个字符设定了统一并且唯一的二进制编码，规定虽有的字符和符号最少由 16 位来表示（2个字节），即：2 **16 = 65536，
注：此处说的的是最少2个字节，可能更多

UTF-8，是对Unicode编码的压缩和优化，他不再使用最少使用2个字节，而是将所有的字符和符号进行分类：ascii码中的内容用1个字节保存、欧洲的字符用2个字节保存，东亚的字符用3个字节保存...

所以，python解释器在加载 .py 文件中的代码时，会对内容进行编码（默认ascill），如果是如下代码的话：

报错：ascii码无法表示中文


    #!/usr/bin/env python
      
    print "你好，世界"
改正：应该显示的告诉python解释器，用什么编码来执行源代码，即：


    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
  
    print "你好，世界"
## 注释 ##
　　当行注视：# 被注释内容

　　多行注释：""" 被注释内容 """

　　

　　

　

## 八、用户输入　 ##

    #!/usr/bin/env python
    #_*_coding:utf-8_*_
     
     
    #name = raw_input("What is your name?") #only on python 2.x
    name = input("What is your name?")
    print("Hello " + name )
输入密码时，如果想要不可见，需要利用getpass 模块中的 getpass方法，即：


    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
      
    import getpass
      
    # 将用户输入的内容赋值给 name 变量
    pwd = getpass.getpass("请输入密码：")
      
    # 打印输入的内容
    print(pwd)
     

## 九、模块初识 ##
Python的强大之处在于他有非常丰富和强大的标准库和第三方库，几乎你想实现的任何功能都有相应的Python库支持，以后的课程中会深入讲解常用到的各种库，现在，我们先来象征性的学2个简单的。

**sys**

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
     
    import sys
     
    print(sys.argv)
     
     
    #输出
    $ python test.py helo world
    ['test.py', 'helo', 'world']  #把执行脚本时传递的参数获取到了
　　

**os**


    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
     
    import os
     
    os.system("df -h") #调用系统命令
**完全结合一下**　　


    import os,sys
     
    os.system(''.join(sys.argv[1:])) #把用户的输入的参数当作一条命令交给os.system来执行
自己写个模块

python tab补全模块

 **for mac**

     import sys
     import readline
     import rlcompleter
     
     if sys.platform == 'darwin' and sys.version_info[0] == 2:
     	readline.parse_and_bind("bind ^I rl_complete")
     else:
     	readline.parse_and_bind("tab: complete")  # linux and python3 on mac

**for Linux**

      #!/usr/bin/env python 
      # python startup file 
      import sys
      import readline
      import rlcompleter
      import atexit
      import os
      # tab completion 
      readline.parse_and_bind('tab: complete')
      # history file 
      histfile = os.path.join(os.environ['HOME'], '.pythonhistory')
      try:
      	readline.read_history_file(histfile)
      except IOError:
      	pass
      atexit.register(readline.write_history_file, histfile)
      del os, histfile, readline, rlcompleter
    

写完保存后就可以使用了


    localhost:~ jieli$ python
    Python 2.7.10 (default, Oct 23 2015, 18:05:06)
    [GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.0.59.5)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import tab
你会发现，上面自己写的tab.py模块只能在当前目录下导入，如果想在系统的何何一个地方都使用怎么办呢？ 此时你就要把这个tab.py放到python全局环境变量目录里啦，基本一般都放在一个叫 Python/2.7/site-packages 目录下，这个目录在不同的OS里放的位置不一样，用 print(sys.path) 可以查看python环境变量列表。

　　

## 十、.pyc是个什么鬼？ ##
### 1. Python是一门解释型语言？ ###

我初学Python时，听到的关于Python的第一句话就是，Python是一门解释性语言，我就这样一直相信下去，直到发现了*.pyc文件的存在。如果是解释型语言，那么生成的*.pyc文件是什么呢？c应该是compiled的缩写才对啊！

为了防止其他学习Python的人也被这句话误解，那么我们就在文中来澄清下这个问题，并且把一些基础概念给理清。

  

### 2. 解释型语言和编译型语言 ### 

计算机是不能够识别高级语言的，所以当我们运行一个高级语言程序的时候，就需要一个“翻译机”来从事把高级语言转变成计算机能读懂的机器语言的过程。这个过程分成两类，第一种是编译，第二种是解释。

编译型语言在程序执行之前，先会通过编译器对程序执行一个编译的过程，把程序转变成机器语言。运行时就不需要翻译，而直接执行就可以了。最典型的例子就是C语言。

解释型语言就没有这个编译的过程，而是在程序运行的时候，通过解释器对程序逐行作出解释，然后直接运行，最典型的例子是Ruby。

通过以上的例子，我们可以来总结一下解释型语言和编译型语言的优缺点，因为编译型语言在程序运行之前就已经对程序做出了“翻译”，所以在运行时就少掉了“翻译”的过程，所以效率比较高。但是我们也不能一概而论，一些解释型语言也可以通过解释器的优化来在对程序做出翻译时对整个程序做出优化，从而在效率上超过编译型语言。

此外，随着Java等基于虚拟机的语言的兴起，我们又不能把语言纯粹地分成解释型和编译型这两种。

用Java来举例，Java首先是通过编译器编译成字节码文件，然后在运行时通过解释器给解释成机器文件。所以我们说Java是一种先编译后解释的语言。

 

### 3. Python到底是什么  ###

其实Python和Java/C#一样，也是一门基于虚拟机的语言，我们先来从表面上简单地了解一下Python程序的运行过程吧。

当我们在命令行中输入python hello.py时，其实是激活了Python的“解释器”，告诉“解释器”：你要开始工作了。可是在“解释”之前，其实执行的第一项工作和Java一样，是编译。

熟悉Java的同学可以想一下我们在命令行中如何执行一个Java的程序：

javac hello.java

java hello

 

只是我们在用Eclipse之类的IDE时，将这两部给融合成了一部而已。其实Python也一样，当我们执行python hello.py时，他也一样执行了这么一个过程，所以我们应该这样来描述Python，Python是一门先编译后解释的语言。

### 4. 简述Python的运行过程 ###

在说这个问题之前，我们先来说两个概念，PyCodeObject和pyc文件。

我们在硬盘上看到的pyc自然不必多说，而其实PyCodeObject则是Python编译器真正编译成的结果。我们先简单知道就可以了，继续向下看。

当python程序运行时，编译的结果则是保存在位于内存中的PyCodeObject中，当Python程序运行结束时，Python解释器则将PyCodeObject写回到pyc文件中。

当python程序第二次运行时，首先程序会在硬盘中寻找pyc文件，如果找到，则直接载入，否则就重复上面的过程。

所以我们应该这样来定位PyCodeObject和pyc文件，我们说pyc文件其实是PyCodeObject的一种持久化保存方式。

 

## 十一、数据类型初识 ##

### 1、数字 ###
 2 是一个整数的例子。

 长整数 不过是大一些的整数。

 3.23和52.3E-4是浮点数的例子。E标记表示10的幂。在这里，52.3E-4表示52.3 * 10-4。

 (-5+4j)和(2.3-4.6j)是复数的例子，其中-5,4为实数，j为虚数，数学中表示复数是什么？。

**int（整型）**

　　在32位机器上，整数的位数为32位，取值范围为-2**31～2**31-1，即-2147483648～2147483647

　　在64位系统上，整数的位数为64位，取值范围为-2**63～2**63-1，即-9223372036854775808～9223372036854775807

**long（长整型）**

　　跟C语言不同，Python的长整数没有指定位宽，即：Python没有限制长整数数值的大小，但实际上由于机器内存有限，我们使用的长整数数值不可能无限大。

　　注意，自从Python2.2起，如果整数发生溢出，Python会自动将整数数据转换为长整数，所以如今在长整数数据后面不加字母L也不会导致严重后果了。

**float（浮点型）**

  先扫盲 http://www.cnblogs.com/alex3714/articles/5895848.html 

　　浮点数用来处理实数，即带有小数的数字。类似于C语言中的double类型，占8个字节（64位），其中52位表示底，11位表示指数，剩下的一位表示符号。
**complex（复数）**
　　复数由实数部分和虚数部分组成，一般形式为x＋yj，其中的x是复数的实数部分，y是复数的虚数部分，这里的x和y都是实数。
注：Python中存在小数字池：-5 ～ 257
 
### 2、布尔值 ###
　　真或假

　　1 或 0
### 3、字符串 ###
    "hello world"
万恶的字符串拼接：

　　python中的字符串在C语言中体现为是一个字符数组，每次创建字符串时候需要在内存中开辟一块连续的空，并且一旦需要修改字符串的话，就需要再次开辟空间，万恶的+号每出现一次就会在内从中重新开辟一块空间。

**字符串格式化输出**

    name = "alex"
    print "i am %s " % name
      
    #输出: i am alex
    PS: 字符串是 %s;整数 %d;浮点数%f

字符串常用功能：

- 移除空白
- 分割
- 长度
- 索引
- 切片
- 
### 4、列表 ###

创建列表：

    name_list = ['alex', 'seven', 'eric']
    或
    name_list ＝ list(['alex', 'seven', 'eric'])

基本操作：

- 索引
- 切片
- 追加
- 删除
- 长度
- 切片
- 循环
- 包含

### 5、元组(不可变列表) ###

创建元组：
    
    ages = (11, 22, 33, 44, 55)
    或
    ages = tuple((11, 22, 33, 44, 55))
 
### 6、字典（无序） ###

创建字典：
    
    person = {"name": "mr.wu", 'age': 18}
    或
    person = dict({"name": "mr.wu", 'age': 18})
常用操作：

- 索引
- 新增
- 删除
- 键、值、键值对
- 循环
- 长度

## 十二、数据运算 ##　
　
算数运算：
![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025004451692-544714036.png)

比较运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025004535395-2018056438.png)

赋值运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025004625020-515390534.png)


逻辑运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025004829864-1676917934.png)

成员运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025004934286-1134210223.png)


身份运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025005015567-214206984.png)

位运算：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025005136817-1018821016.png)

    #!/usr/bin/python
      
    a = 60# 60 = 0011 1100
    b = 13# 13 = 0000 1101
    c = 0
      
    c = a & b;# 12 = 0000 1100
    print "Line 1 - Value of c is ", c
      
    c = a | b;# 61 = 0011 1101
    print "Line 2 - Value of c is ", c
      
    c = a ^ b;# 49 = 0011 0001 #相同为0，不同为1
    print "Line 3 - Value of c is ", c
      
    c = ~a;   # -61 = 1100 0011
    print "Line 4 - Value of c is ", c
      
    c = a << 2;   # 240 = 1111 0000
    print "Line 5 - Value of c is ", c
      
    c = a >> 2;   # 15 = 0000 1111
    print "Line 6 - Value of c is ", c
    
*按位取反运算规则(按位取反再加1)   详解[http://blog.csdn.net/wenxinwukui234/article/details/42119265](http://blog.csdn.net/wenxinwukui234/article/details/42119265)

 

运算符优先级：

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151025005448536-1963442899.png)

更多内容：[猛击这里](http://www.runoob.com/python/python-operators.html)

　　

## 十三、表达式if ... else ##

**场景一、用户登陆验证**


    # 提示输入用户名和密码
      
    # 验证用户名和密码
    # 如果错误，则输出用户名或密码错误
    # 如果成功，则输出 欢迎，XXX!
     
     
    #!/usr/bin/env python
    # -*- coding: encoding -*-
      
    import getpass
      
      
    name = raw_input('请输入用户名：')
    pwd = getpass.getpass('请输入密码：')
      
    if name == "alex" and pwd == "cmd":
    	print("欢迎，alex！")
    else:
    	print("用户名和密码错误")

**场景二、猜年龄游戏**

在程序里设定好你的年龄，然后启动程序让用户猜测，用户输入后，根据他的输入提示用户输入的是否正确，如果错误，提示是猜大了还是小了

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
     
     
    my_age = 28
     
    user_input = int(input("input your guess num:"))
     
    if user_input == my_age:
    	print("Congratulations, you got it !")
    elif user_input < my_age:
    	print("Oops,think bigger!")
    else:
    	print("think smaller!")
　　

外层变量，可以被内层代码使用
内层变量，不应被外层代码使用
 
## 十四、表达式for loop ##

**最简单的循环10次**


    #_*_coding:utf-8_*_
    __author__ = 'Alex Li'
     
     
    for i in range(10):
    	print("loop:", i )
输出：

    loop: 0
    loop: 1
    loop: 2
    loop: 3
    loop: 4
    loop: 5
    loop: 6
    loop: 7
    loop: 8
    loop: 9

**需求一：还是上面的程序，但是遇到小于5的循环次数就不走了，直接跳入下一次循环**


    for i in range(10):
    if i<5:
    	continue #不往下走了,直接进入下一次loop
    print("loop:", i )

**需求二：还是上面的程序，但是遇到大于5的循环次数就不走了，直接退出**


    for i in range(10):
    if i>5:
    	break #不往下走了,直接跳出整个loop
    print("loop:", i )
     
## 十五、while loop　 ##

** 有一种循环叫死循环，一经触发，就运行个天荒地老、海枯石烂。**

海枯石烂代码

    
    count = 0
    while True:
    	print("你是风儿我是沙,缠缠绵绵到天涯...",count)
    	count +=1
    
 

其实除了时间，没有什么是永恒的，死loop还是少写为好　

上面的代码循环100次就退出吧

 

    count = 0
    while True:
    	print("你是风儿我是沙,缠缠绵绵到天涯...",count)
    	count +=1
    	if count == 100:
    		print("去你妈的风和沙,你们这些脱了裤子是人,穿上裤子是鬼的臭男人..")
    		break
 

 

**回到上面for 循环的例子，如何实现让用户不断的猜年龄，但只给最多3次机会，再猜不对就退出程序**。

    #!/usr/bin/env python
    # -*- coding: utf-8 -*-
     
     
    my_age = 28
     
    count = 0
    while count < 3:
    	user_input = int(input("input your guess num:"))
     
    	if user_input == my_age:
    		print("Congratulations, you got it !")
    		break
    	elif user_input < my_age:
    		print("Oops,think bigger!")
    	else:
    		print("think smaller!")
    	count += 1 #每次loop 计数器+1
    else:
    print("猜这么多次都不对,你个笨蛋.")
　　

　　

## 十六、作业  ##

### 作业一：博客 ###

### 作业二：编写登陆接口 ###

- 输入用户名密码
- 认证成功后显示欢迎信息
- 输错三次后锁定
 

### 作业三：多级菜单 ###
- 三级菜单
- 可依次选择进入各子菜单
- 所需新知识点：列表、字典
　　

　　

　　
 
## 入门知识拾遗 ##

### 一、bytes类型 ###

 

### 二、三元运算 ###

    result = 值1 if 条件 else 值2

如果条件为真：result = 值1

如果条件为假：result = 值2

### 三、进制 ###

- 二进制，01
- 八进制，01234567
- 十进制，0123456789
- 十六进制，0123456789ABCDEF  二进制到16进制转换http://jingyan.baidu.com/album/47a29f24292608c0142399cb.html?picindex=1

**计算机内存地址和为什么用16进制？**

**为什么用16进制**

1. 计算机硬件是0101二进制的，16进制刚好是2的倍数，更容易表达一个命令或者数据。十六进制更简短，因为换算的时候一位16进制数可以顶4位2进制数，也就是一个字节（8位进制可以用两个16进制表示）
2. 最早规定ASCII字符集采用的就是8bit(后期扩展了,但是基础单位还是8bit)，8bit用2个16进制直接就能表达出来，不管阅读还是存储都比其他进制要方便
3. 计算机中CPU运算也是遵照ASCII字符集，以16、32、64的这样的方式在发展，因此数据交换的时候16进制也显得更好
4. 为了统一规范，CPU、内存、硬盘我们看到都是采用的16进制计算

**16进制用在哪里**

1. 网络编程，数据交换的时候需要对字节进行解析都是一个byte一个byte的处理，1个byte可以用0xFF两个16进制来表达。通过网络抓包，可以看到数据是通过16进制传输的。
2. 数据存储，存储到硬件中是0101的方式，存储到系统中的表达方式都是byte方式

3. 一些常用值的定义，比如：我们经常用到的html中color表达，就是用的16进制方式，4个16进制位可以表达好几百万的颜色信息。

 
### 四、 一切皆对象 ###

对于Python，一切事物都是对象，对象基于类创建

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151031133946575-1667336241.png)

所以，以下这些值都是对象： "wupeiqi"、38、['北京', '上海', '深圳']，并且是根据不同的类生成的对象。

![](https://images2015.cnblogs.com/blog/425762/201510/425762-20151031134348685-1255331039.png)



**注：该文章由[alex](https://www.cnblogs.com/alex3714)的blog搬运而来**
