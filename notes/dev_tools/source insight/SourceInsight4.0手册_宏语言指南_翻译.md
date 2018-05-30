#Souce Insight 4.0 宏语言指南  
  
## **宏函数**
  定义Source Insight宏函数的语法与C语言定义函数类似。空白字符会被自动忽略，并且函数名和变量名不区分大小写字母。宏函数的函数名前必须是macro或者function关键字。下面是一个简单的宏函数例子：  
  
	macro HelloWorld()
	{
       msg("Hello World!")   // message box appears with "Hello World"
    }

  宏函数可以有参数，也可以调用其它的宏函数。也可以通过"return n"返回一个返回值n。下面是一个带参数，并且有返回值的例子：

    function add2(n)
    {
       return n + 2
    }


#### **用户级别的宏 vs 函数**  

  Source Insight支持三种最常用的宏函数类型：
  - 可以关联快捷键和菜单项的用户级别命令。这类宏函数必须用 **macro** 关键字前导声明，并且不能有函数参数。命令的名字就是宏函数的名字。
  - 能被宏函数调用的子函数。这类宏函数必须用 **function** 关键字前导声明。
  - 事件钩子函数。这类宏函数必须必须以 **event** 关键字前导声明。


  下面就是一个可以关联快捷键和菜单项的宏函数例子：  

    macro SaveCurrentFileNow ()
    {
       perform_save()
    }

  你可以通过菜单项 **Options > Key Assignments**找到 **SaveCurrentFileNow** 命令，并且为其指定一个触发的快捷键。  

  但是下面这个函数就只能在其他宏函数中被调用，因为这个函数是以 **function**关键字定义的。

    function perform_save ()
    {
       var hbuf
     
       hbuf = GetCurrentBuf()
       if (hbuf != nil)
       SaveBuf(hbuf)
    }




## **宏作用域和引用**

  所有的宏函数同属于全局命名空间。声明在任何打开的文件中的宏，保存在项目文件中的宏，甚至只要宏的符号在项目中能找到，就算这个宏在作用域中。所以，你可以前置引用宏，即并可直接调用在作用域中的宏，而不需要先声明。

  当用户执行宏，或者调用一个宏命令时，Source Insight使用内置的符号查找引擎解析宏引用。因此符号查找规则也适用于宏名称绑定。

  你也可以用各种符号查找技术找到一个宏函数。例如，里可以在项目符号浏览对话窗口中键入宏函数的名字，然后根据找到的结果列表跳转到宏定义。


## **运行宏**

你可以通过快捷键或者菜单项来执行宏，可以通过** Run Marcro **命令在当前光标位置开始执行宏语句，也可以在**  Options > Key Assignments, or Options > Menu Assign­ments **对话框中选择要执行的宏命令，然后点击** Run **按钮运行。


## **语句**

Source Insight的宏是由很多独立的语句构成的。复合语句则是用**{ }**括起来的两个以上的语句组成。

一条宏语言语句可以是下列情形中的一种：
- 一条语句可以是宏语言语句要素，例如** if **或者** while **等。
- 一条语句可以是对用户定义的宏函数的调用。用户定义的宏函数一般保存在宏文件中。
- 一条语句可以是对内部宏函数的调用。例如调用** GetCurrentBuf **。Source Insight提供了很多内置的函数。
- 一条语句可以是对用户命令的调用。例如** Line_Up **或者** Copy **。所有用户级别的命令对宏语言来说，都是可见的，可以直接调用。调用用户级别的命令，可以通过宏函数的名字直接调用，没有参数，括号也不需要。如果用户命令本身含有空白字符，必须用下划线** _ **代替空白字符。例如，调用**"Select All"**命令，语句必须写成**Select_All**。

可以用** stop **来停止宏语句的执行。

如果你在一个宏命令中打开了对话框，则对话框会出现。

函数和变量名是不区分大小写的。

除了宏语句结束时不需要分号，语法基本上和** C ** 或者 ** Java **相同。语句后加上分号也会被直接忽略。


#### **宏语句样例**

下面是一些宏语句的样例：

    hbuf = OpenBuf("file.c")// call internal function OpenBuf
    SaveBufAs(hbuf, "filenew.c")  // call internal function SaveBufAs
    Select_All// call user-level command "Select All"
    Copy // call user-level command "Copy"
    Line_Up   // call user-level command "Line Up"
    x = add2(n)   // call user-defined macro function add2.


#### **宏语句要素**

下表列出了宏语句中的关键要素。
    Table 5.1: Macro Statements

| 语句  | 描述 |
| ------------- | ------------- |
| break  | 从while循环中退出  |
| if (cond)…… else  | 条件判断  |
| continue  | 继续while下一次循环  |
| global  | 声明一个全局变量  |
| return n 或 return;  | 函数返回一个整数n。如果**;**跟在return后面，则表示返回一个空的字符串  |
| stop  | 停止执行宏  |
| strictvars true/false  | 打开/关闭严格变量声明模式  |
| var  | 声明一个函数本地变量  |
| while (cond)  | 条件为true时执行while循环体  |



## **变量**