# Makefile中四种赋值运算符 := ?= += = 的区别
  
  在Makefile中我们经常看到 = := ?= +=这几个赋值运算符，那么他们有什么区别呢？  
  通过下面几个例子，就比较好理解了。

###    1. =
  
直接上例子:  
```Makefile
  #equal_test.mk

  x = foo
  y = $(x) bar
  x = xyz
  
  all:
    @echo $(y)

```
  
  在这个例子中，y的值将会是 **xyz bar**，而不是 **foo bar** 。  
  
  相信看懂了上面这个例子，就发现原来这是一个坑。平常程序员的理解，代码都是顺序执行的，在给y赋值时，  
  x的值应该是"foo"，为什么最后变成"xyz"了呢？  
  
  **make解析makefile的过程，是先将整个makefile展开后，最后再决定变量的值，所以变量的值将会是整个makefile中最后被指定的值。**

###    2. :=
  
  为了填=赋值运算符的坑，Makefile同时提供了:=运算符。:=才是大多数人认为的正常赋值，是位置想关的。  
  在:=赋值运算时，如果涉及到变量值的替换展开，只会使用:=赋值语句之前的语句。  
  还是上面的例子，只是把:换成:=运算符:
```Makefile
  #colon_equal_test.mk

  x = foo
  y = $(x) bar
  x = xyz
  
  all:
    @echo $(y)

```
  
  这次y的值将会是 **foo bar**，而不是 **xyz bar**。  

###    3. ?=  
  
  ?=在赋值之前会判断一下变量之前是否已经赋值过了，如果已经有值了，就不再赋值，即使这次赋的值和之前的值不同；  
  如果没有值，就赋值给变量。
  **需要注意的是，?=不是位置想关的，而是看代码行是否真正执行到了。**  
  
  看例子:
```Makefile  
  #question_equal_test.mk
  
  ifeq ($(xflag),on)
    info = "The flag is on!"
  endif
  
  info ?= "The flag is off!"

  all:
    @echo $(info)
```
这个Makefile是根据一个变量xflag的值给变量info赋值。  
我们在命令行中给xflag一个值，如果xflag=on，则执行结果是 **"The flag is on!"**。  
  
  
如果xflag不等于on，或者没有给xflag赋值，则执行结果是 **"The flag is off!"**。  
  
  
从上面的执行结果来看，当xflag=on时，会走入if分支，给info变量赋值一次。然后再执行到?=赋值语句时，就不会再赋值了。

###    4. +=
  
  +=和Java语言中=+类似，就是字符串的追加合并。
  还是用?=用过的例子，稍加修改:
  ```Makefile
  #plus_equal_test.mk
  
  ifeq ($(xflag),on)
    info = "The flag is on!"  
  endif
  
  info += " -- The flag is off!"

  all:
    @echo $(info)

  ```
    
  执行结果是: **"The flag is on! -- The flag is off!"**。 
  把变量多次赋值的结果按先后顺序串起来了。

