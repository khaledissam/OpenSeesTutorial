TCL语言入门
===============

一条TCL的命令串包含若干条命令，命令使用换行符或分号来隔开；而每一条命令包含若干个域(field)，域使用空白（空格或TAB）来隔开——第一个域是命令的名字，其它的域是该命令的参数。

基本语法
---------

注释
~~~~~~~~

注释在调试的过程中轻常碰到。TCL语言的注释符号是 ``#`` ，加在每一行的最前面。

脚本、命令、单词
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

一个TCL ``脚本`` 可以包含一个或多个 ``命令`` 。 ``命令`` 之间必须用换行符或分号隔开，推荐使用换行符分开。下面就是一个合法的TCL ``脚本`` ，它由两个赋值 ``命令`` 组成。

.. code-block:: TCL

  set a 1
  set b 2

TCL的每一个 ``命令`` 包含一个或几个 ``单词``，第一个单词代表命令名，另外的单词则是这个命令的参数，单词之间必须用 ``空格`` 或 ``TAB键`` 隔开。上面代码中的 ``set`` ， ``a`` ， ``1`` 分别是三个单词。

TCL解释器对一个 ``命令`` 的求值过程分为两部分：分析和执行。在分析阶段，TCL 解释器运用规则把 ``命令`` 分成一个个独立的单词，同时进行必要的 ``置换(substitution)`` ； 在执行阶段，TCL 解释器会把第一个单词当作 ``命令名`` ，并查看这个命令是否有定义，如果有定义就激活这个命令对应的C/C++过程，并把所有的单词作为参数传递给该命令过程，让命令过程进行处理。

置换(substitution)
~~~~~~~~~~~~~~~~~~~~~~~

TCL解释器在分析命令时，把所有的命令参数都当作字符串看待，例如：

.. code-block:: TCL

    OpenSees > set x 10
    10
    OpenSees > set y x+1
    x+1

上例的第二个命令中，x被看作字符串x+1的一部分，此时y的值为 ``x+1`` 如果我们想使用x的值'10' ，就必须告诉TCL解释器：我们在这里期望的是变量x的值，而非字符'x'。怎么告诉TCL解释器呢，这就要用到TCL语言中提供的 ``置换`` 功能。TCL提供三种形式的置换： ``变量置换`` 、 ``命令置换`` 和 ``反斜杠置换`` 。

变量置换
^^^^^^^^^^^

在变量符号之前用 ``$`` 符号标记。这会导致变量的值插入一个单词中。

.. code-block:: TCL

	OpenSees > set y $x+1
	10+1

这里x的值已经被替换成 ``10`` ，但是没有执行我们想要的 ``x+1`` 计算。这时就要用到命令置换。

命令置换
^^^^^^^^^^^
  
命令置换是由 ``[]`` 括起来的TCL命令及其参数，命令置换会导致某一个命令的所有或部分单词被另一个命令的结果所代替。例如：

.. code-block:: TCL

	OpenSees > set y [expr $x+1]
	11

这里的 ``expr`` 是一个TCL命令，表示执行数学表达式。这个命令在OpenSees脚本中经常可以用到。支持如下的一些常用运算：

- ``+`` 、 ``-`` 、 ``*`` 、 ``/`` ：加减乘除
- ``>`` 、 ``<`` 、 ``<=`` 、 ``>=`` 、 ``==`` 、 ``!=`` ：布尔运算
- ``abs()`` 、 ``sin()`` 、 ``pow()`` 、 ``exp()``：常用数学函数

其它命令可以参考 expr命令文档_

.. _expr命令文档: http://www.TCL.tk/man/TCL/TCLCmd/expr.htm

.. warning:: ``expr 5/4`` 会返回 ``1``，因为两个整数中间使用 ``/`` 运算符表示相除取整。这点要时刻注意。如写成 ``expr 5.0/4`` 即可得到正确的结果。


反斜杠置换
^^^^^^^^^^^^^^^^^

TCL语言中的反斜杠置换类似于C语言中反斜杠的用法，主要用于在单词符号中插入诸如换行符、空格、[]、$等被TCL解释器当作特殊符号对待的字符。例如：

.. code-block:: TCL

	OpenSees > set msg multiple\ space
	multiple space

如果没有 ``\`` 的话，TCL会报错，因为解释器会把这里最后两个单词之间的空格认为是分隔符，于是发现set命令有多于两个参数，从而报错。加入了 ``\`` 后，空格不被当作分隔符，``multiple space`` 被认为是一个 ``单词``。

TCL支持以下反斜杠置换：
	===================  ===============================
	Backslash Sequence   Replaced By
	===================  ===============================
	\\a 					 Audible alert (0x7)
	\\b 					 Backspace (0x8)
	\\f 					 Form feed (0xc)
	\\n 					 Newline (0xa)
	\\r 					 Carriage return (0xd)
	\\t 					 Tab (0x9)
	\\v 					 Vertical tab (0xb)
	\\ddd 				 Octal value given by ddd
	\\xhh 				 Hex value given by hh
	\\ newline space 	 A single space character.
	===================  ===============================

双引号和花括号
^^^^^^^^^^^^^^^^^^^^^

除了使用反斜杠外，TCL提供另外两种方法来使得解释器把分隔符和置换符等特殊字符当作普通字符，而不作特殊处理，这就要使用双引号和花括号({})。

TCL解释器对双引号中的各种分隔符将不作处理，但是对换行符 及＄和[]两种置换符会照常处理。例如：

.. code-block:: TCL

	OpenSees > set  x  100
	100
	OpenSees > set  y  "$x   ddd"   
	100   ddd

而在花括号中，所有特殊字符都将成为普通字符，失去其特殊意义，TCL解释器不会对其作特殊处理。

.. code-block:: TCL

	OpenSees > set  y {/n$x   [expr 10+100]}   
	/n$x   [expr 10+100]     


变量
------

TCL的变量有两种，分别是简单变量和数组。

简单变量
~~~~~~~~~~~

一个TCL的简单变量包含两个部分：名字和值。名字和值都可以是任意字符串。变量推荐使用字母，数字与下划线的组合来命名。
TCL解释器在分析一个变量置换时，只把从＄符号往后直到第一个不是字母、数字或下划线的字符之间的单词符号作为要被置换的变量的名字。例如:

.. code-block:: TCL

	OpenSees > set mat_tag 2
	2

TCL中的set命令能生成一个变量、也能读取或改变一个变量的值。如果变量 ``mat_tag`` 还没有定义，这个命令将生成该变量，并将其值置为 ``2`` ，若 ``mat_tag`` 已定义，就简单的把 ``mat_tag`` 的值置为 ``2`` 。

.. code-block:: TCL

    OpenSees > set mat_tag
    2

这个只有一个参数的set命令读取 ``mat_tag`` 的当前值 ``2`` 。

数组
~~~~~~

在TCL中，数组是带有字符串值索引的变量，请注意，是字符串索引，而不是数字索引。由于TCL语言的这个特性，导致其数组的声明和引用都不是很方便。在OpenSees编程时，建议使用 ``列表(List)`` 。

列表
~~~~~~

TCL中列表(list)是由一堆元素组成的 **有序** 集合，list可以嵌套定义，list每个元素可以是任意字符串，也可以是list。下面都是TCL中的合法的list：

.. code-block:: TCL

	{}    //空list
	{a b c d}
	{a {b c} d} //list可以嵌套

list是TCL中比较重要的一种数据结构，对于编写复杂的脚本有很大的帮助，TCL提供了很多基本命令对list进行操作，下面一一介绍：

list命令
^^^^^^^^^^

.. code-block:: TCL

    list ? value value...?

这个命令生成一个list，list的元素就是所有的value。例：

.. code-block:: TCL

	OpenSees > list 1 2 {3 4}
	1 2 {3 4}

concat命令
^^^^^^^^^^^^^

.. code-block:: TCL

    concat list ?list...?

这个命令把多个list合成一个list，每个list变成新list的一个元素。

lindex命令
^^^^^^^^^^^^^

.. code-block:: TCL

    lindex list index

返回list的第index个(0-based)元素。例：

.. code-block:: TCL

	OpenSees > lindex  {1 2 {3 4}} 2
	3 4

lappend命令
^^^^^^^^^^^^^^^

.. code-block:: TCL

    lappend varname value ?value...?

把每个value的值作为一个元素附加到变量varname后面，并返回变量的新值，如果varname不存在，就生成这个变量。例：

.. code-block:: TCL

	OpenSees > lappend  a  1 2 3
	1 2 3
	OpenSees > set a
	1 2 3

.. note:: 更多列表相关命令，可以参考 TCL列表命令文档_

.. _TCL列表命令文档: http://www.TCL.tk/man/TCL/TCLCmd/list.htm


控制流
--------

TCL中的控制流和C语言类似，包括if、while、for、foreach、switch、break、continue等命令。下面分别介绍。

if命令
~~~~~~~~~

单个if命令：

.. code-block:: TCL

    if { $x>0 } {
      ...
    }

if-else组合命令：

.. code-block:: TCL

    if { $x>0 } {
      ...
    } elseif { $x<-2 } {
      ...
    }

.. warning:: 上例中 ``{`` 一定要写在上一行，因为如果不这样，TCL 解释器会认为if命令在换行符处已结束，下一行会被当成新的命令，从而导致错误的结果。在下面的循环命令的书写中也要注意这个问题。书写中还要注意的一个问题是 ``if``  和 ``{`` 之间应该有一个空格，否则TCL解释器会把 ``if{`` 作为一个整体当作一个命令名，从而导致错误。

while命令
^^^^^^^^^^^

示例：

.. code-block:: TCL

    while  { $x>0 }  {
      ...
    }

for命令
^^^^^^^^^^^^^^

示例：

.. code-block:: TCL

    for {set i 0}  { $i<10 }  {incr i 2} {
      ...
    }

for后面加三个花括号。与C语言中的for命令类似，第一个花括号中初始化变量的值，示例中为变量 ``i`` 赋初值 ``0`` ，第二个花括号中为循环进行下去的条件。示例中如果不满足 ``$i<10`` 这一条件就会退出循环。第三个花括号中为每次循环后要执行的语句，示例中对变量 ``i`` 的值加2。

foreach命令
^^^^^^^^^^^^^^^^

示例1：

.. code-block:: TCL

    foreach i  {a b c d} {
      ...
    }

这一语句循环4次，循环体中i的值分别为 ``a`` ， ``b`` ， ``c`` ， ``d`` 。

source命令
------------

source命令读一个文件并把这个文件的内容作为一个脚本进行求值。例如：

.. code-block:: TCL

    source e:/hello.TCL

注意这里的路径采用的是 ``/`` 而不是Windows中的 ``\`` 。

过程(procedure)
-----------------

TCL支持过程的定义和调用，在TCL中,过程可以看作是用TCL脚本实现的命令，效果与TCL的固有命令相似。我们可以在任何时候使用proc命令定义自己的过程，TCL中的过程类似于C中的函数。

在OpenSees脚本中，使用过程可以把一部分语句 ``封装`` 起来，方便多次引用。建议多使用过程。

TCL中过程是由proc命令产生的。示例：

.. code-block:: TCL

	proc add {x y} {
	    expr $x+$y
	}

proc命令的第一个参数是你要定义的过程的名字，第二个参数是过程的参数列表，参数之间用空格隔开，第三个参数是一个TCL脚本，代表过程体。proc生成一个新的命令，可以象固有命令一样调用：

.. code-block:: TCL

	OpenSees > add 1 2
	3

在定义过程时，你可以利用return命令在任何地方返回你想要的值。 return命令迅速中断过程，并把它的参数作为过程的结果。例如：

.. code-block:: TCL

	proc abs {x} {
	    if {$x >= 0} { return $x }
	    return [expr -$x]
	}

过程的返回值是过程体中最后执行的那条命令的返回值。可以用如下方法调用：

.. code-block:: TCL
    
    OpenSees > set a [abs -3]
    3

文件读写
-------------

TCL提供了丰富的文件操作的命令。通过这些命令你可以对文件名进行操作(查找匹配某一模式的文件)、以顺序或随机方式读写文件、检索系统保留的文件信息（如最后访问时间)。


文件名
~~~~~~~~

TCL中文件名和我们熟悉的windows表示文件的方法有一些区别：在表示文件的目录结构时它使用 ``/`` ，而不是 ``\`` ，这和TCL最初是在UNIX下实现有关。比如C盘TCL目录下的文件sample.TCL在TCL中这样表示： ``C:/TCL/sample.TCL`` 。	

写文件示例
~~~~~~~~~~~~~~~~

在OpenSees脚本中，一般只用到写文件。所以本教程中只介绍写文件的方法。如果想要了解读取文件的方法，请参考 TCL文件读写文档_ 。

.. _TCL文件读写文档: http://www.TCL.tk/man/TCL/TCLCmd/open.htm

.. code-block:: TCL
    
    set f [open hello.txt w]
    puts $f "Hello, world!"
    close $f

open命令
~~~~~~~~~

示例：

.. code-block:: TCL
    
    open "hello.txt" "r"

open命令 以"r"方式打开文件"hello.txt"。返回供其他命令(gets,close等)使用的文件标识。

文件的打开方式和我们熟悉的C语言类似，有以下方式：

	======    ======================================
	方式       描述
	======    ======================================
	r		  只读方式打开。文件必须已经存在。这是默认方式。
	r+		  读写方式打开，文件必须已经存在。	
	w		  只写方式打开文件，如果文件存在则清空文件内容，否则创建一新的空文件。
	w+		  读写方式打开文件，如文件存在则清空文件内容，否则创建新的空文件。
	a		  只写方式打开文件，文件必须存在，并把指针指向文件尾。
	a+		  读写方式打开文件，并把指针指向文件尾。如文件不存在，创建新的空文件。
	======    ======================================

open命令返回一个字符串用于表识打开的文件。当调用别的命令（如：gets,puts,close）对打开的文件进行操作时，就可以使用这个文件标识符。


puts命令
~~~~~~~~~~

示例：

.. code-block:: TCL
    
    puts $f "Hello, world!"

puts命令把"Hello, world!"字符串写到 ``$f`` 中，如果命令中不输入 ``$f`` 则输出到控制台。

close命令
~~~~~~~~~~~~

示例：

.. code-block:: TCL

    close $f

关闭标识为 ``$f`` 的文件，命令返回值为一空字符串。
