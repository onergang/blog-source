title: try catch finally分析
date: 2016-08-22 00:59:59
categories: 技术
tags: 异常处理
---

​	最近在看Java基础，隔了好长一段时间不去接触底层的东西，好多东西就在念念不忘中忘记了，现在要重新拾起来，看的书是《Think in Java》。越看越觉得一些细微的点自己以前没有注意到，幸好这个时候看了书，要不然永远也不会记起来了。

​	写此篇Blog并不是为了分享读书的新的，而是最近在刷脉脉的时候看到了一个很有去的题目，匿名区好多人讨论。

<!--more-->

题目是输出一段代码的运行结果，代码是：

```java
// main方法
public static void main(String[] args) {
  System.out.println(convert("a"));
}	
public static String convert(String str) {
		try {
			return str;
		} finally {
			str.toUpperCase();
		}
	}
```

最后程序运行的结果是:`a`   (笔者自己本地运行测试了，确实是`a`)。
这是很容易错的一个问题，因为一般大家会想到最后输出的是A。这就需要了解try-catch-finallly的运行机制。
**一个简单的例子(例1)：**
```java
public class TryCatchFinally {
    public static final String test() {
        String t = "";
        try {
            t = "try";
            return t;
        } catch (Exception e) {
            t = "catch";
            return t;
        } finally {
            t = "finally";
        }
    }
    public static void main(String[] args) {
        System.out.print(TryCatchFinally.test());
    }
}
```

有了第一个例子，这个例子输出的结果肯定的 "try"。

为什么会这样，我们不妨先看看这段代码编译出来的class对应的字节码，看虚拟机内部是如何执行的。

我们用**javap -verbose TryCatchFinally** 来显示目标文件(.class文件)字节码信息.

```bash
系统运行环境： windows10 64bit
JDK基本信息：
			java version "1.8.0_91"
			Java(TM) SE Runtime Environment (build 1.8.0_91-b14)
			Java HotSpot(TM) 64-Bit Server VM (build 25.91-b14, mixed mode)
```

.class 文件的信息:

```java
public static final java.lang.String test();
    descriptor: ()Ljava/lang/String;
    flags: ACC_PUBLIC, ACC_STATIC, ACC_FINAL
    Code:
      stack=1, locals=4, args_size=0
         0: ldc           #16                 // String
         2: astore_0
         3: ldc           #18                 // String try
         5: astore_0
         6: aload_0
         7: astore_3
         8: ldc           #20                 // String finally
        10: astore_0
        11: aload_3
        12: areturn
        13: astore_1
        14: ldc           #22                 // String catch
        16: astore_0
        17: aload_0
        18: astore_3
        19: ldc           #20                 // String finally
        21: astore_0
        22: aload_3
        23: areturn
        24: astore_2
        25: ldc           #20                 // String finally
        27: astore_0
        28: aload_2
        29: athrow
      Exception table:
         from    to  target type
             3     8    13   Class java/lang/Exception
             3     8    24   any
            13    19    24   any
      LineNumberTable:
        line 5: 0
        line 8: 3
        line 9: 6
        line 15: 8
        line 9: 11
        line 10: 13
        line 12: 14
        line 13: 17
        line 15: 19
        line 13: 22
        line 14: 24
        line 15: 25
        line 16: 28
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            3      27     0     t   Ljava/lang/String;
           14      10     1     e   Ljava/lang/Exception;
      StackMapTable: number_of_entries = 2
        frame_type = 255 /* full_frame */
          offset_delta = 13
          locals = [ class java/lang/String ]
          stack = [ class java/lang/Exception ]
        frame_type = 74 /* same_locals_1_stack_item */
          stack = [ class java/lang/Throwable ]
SourceFile: "TryCatchFinally.java"
```

从以上文件可以看出：

> 首先看**LocalVariableTable**信息，这里面定义了两个变量 一个是t String类型,一个是e Exception 类型
> 接下来看Code部分
> 第[0-2]行，给第0个变量赋值""，也就是String t=""；
> 第[3-6]行，也就是执行try语句块 赋值语句 ，也就是 t = "try";
> 第7行，重点是第7行，把第s对应的值"try"付给第三个变量；
> 第[8-10] 行，对第0个变量进行赋值操作，也就是t="finally"
> 第[11-12]行，把第三个变量对应的值返回。
> 通过字节码，我们发现，在try语句的return块中，return 返回的引用变量（t 是引用类型）并不是try语句外定义的引用变量t，而是系统重新定义了一个局部引用t’，这个引用指向了引用t对应的值，也就是try ，即使在finally语句中把引用t指向了值finally，因为return的返回引用已经不是t ，所以引用t的对应的值和try语句中的返回值无关了。
>
> ​										--引用[java中关于try、catch、finally中的细节分析][1]

假若将上述例子加以修改(**例2**)：

```java
public class TryCatchFinally {
    @SuppressWarnings("finally")
    public static final String test() {
        String t = "";
        try {
            t = "try";
            return t;
        } catch (Exception e) {
            t = "catch";
            return t;
        } finally {
            t = "finally";
            return t;
        }
    }
    public static void main(String[] args) {
        System.out.print(test());  // 在当前类调用可不加类名
    }
}
```

则最后输出的结果为："finallly"。

**例3** try抛出异常,finally无返回：

```java
 public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			t = "catch";
			return t;
		} finally {
			t = "finally";
		}
	}
```

此时，解析异常，会抛出java.lang.NumberFormatException: null的异常，从try语句中的`Integer.parseInt(null);`直接跳转到catch语句中运行，t赋值为catch，

> [在执行return之前，会把返回值保存到一个临时变量里面t '，执行finally的逻辑，t赋值为finally，但是返回值和t'，所以变量t的值和返回值已经没有关系了，返回的是catch][1]

**例4** try抛出异常,finally有返回：

```java
     @SuppressWarnings("finally")
	public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (Exception e) {
			e.printStackTrace();
			t = "catch";
			return t;
		} finally {
			t = "finally";
			return t;
		}
	}
```

由于try语句里面抛出异常，程序转入catch语句块，catch语句在执行return语句之前执行finally，而finally语句有return,则直接执行finally的语句值，返回finally.

**例5**  try catch抛出异常,finally无返回：

```java
	public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (Exception e) {
			t = "catch";
			Integer.parseInt(null);
			return t;
		} finally {
			t = "finally";
		}
	}
```

try异常--->catch--->catch异常 --->执行finally语句块，对t进行赋值.
结果是抛出java.lang.NumberFormatException异常.

**例6**  try catch抛出异常,finally有返回：	
```java
	public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (Exception e) {
			t = "catch";
			Integer.parseInt(null);
			return t;
		} finally {
			t = "finally";
			return t;
		}
	}
```
这个例子里面finally 语句里面有return语句块。try catch中运行的逻辑和上面例子一样，当catch语句块里面抛出异常之后，进入finally语句快，然后返回t。则程序忽略catch语句块里面抛出的异常信息，直接返回t对应的值 finally。

**例7**  捕获NullPointer异常：	

```java
public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (NullPointerException  e) {
			t = "catch";
			return t;
		} finally {
			t = "finally";
		}
	}
```

try语句中抛出异常，进入到catch语句中，catch的是NullPointerException，而抛出的是NumberFormatException，直接进入finally语句块，finally对t赋值之后，由try语句抛出java.lang.NumberFormatException异常。

**例8**  捕获异常：	

```java
	@SuppressWarnings("finally")
	public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (NumberFormatException e) {
			t = "catch";
			return t;
		} finally {
			t = "finally";
			return t;
		}
	}
```
最后结果是 finally

**例9**  多重异常：	

```java
@SuppressWarnings("finally")
	public static final String test() {
		String t = "";
		try {
			t = "try";
			Integer.parseInt(null);
			return t;
		} catch (Exception e) {
			t = "catch";
			return t;
		} finally {
			t = "finally";
			Integer.parseInt(null);
			return t;
		}
	}
```

首先程序执行try语句，异常后进入catch语句执行，在返回执行执行finally语句块，finally语句抛出NumberFormatException异常，整个结果返回NumberFormatException异常。

**总结:**

1. try、catch、finally语句中，若try中有return语句，不跑出异常的情况下当前try中变量此时对应的值，此后对变量做任何的修改，都不影响try中return的返回值；
2. 若finally块中有return 语句，则返回try或catch中的返回语句忽略；
3. 若finally块中抛出异常，则整个try、catch、finally块中抛出异常。


所以使用try、catch、finally语句块中需要注意的是:
1. 尽量在try或者catch中使用return语句。
2. finally块中避免使用return语句，因为finally块中使用return语句会覆盖掉try、catch中的异常信息。
3. finally块中避免再次抛出异常，否则整个包含try语句块的方法回抛出异常，并且会消化掉try、catch块中的异常。

在平常的异常处理中，try-catch负责对异常的捕获处理，而finallly块则负责收尾工作。例如在程序开发中流的关闭，连接的关闭等等。

参考链接：
1.[java中关于try、catch、finally中的细节分析](http://www.cnblogs.com/aigongsi/archive/2012/04/19/2457735.html)



[1]: http://www.cnblogs.com/aigongsi/archive/2012/04/19/2457735.html/	"java中关于try、catch、finally中的细节分析"





