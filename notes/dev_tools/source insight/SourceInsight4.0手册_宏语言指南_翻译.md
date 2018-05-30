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


####**用户级别的宏 vs 函数**  

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

  所有的宏函数同属于全局命名空间。