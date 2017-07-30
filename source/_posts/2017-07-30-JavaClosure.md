---
title:         Java中的闭包之争
date:       2017-07-30 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
tags:
    - Java
    - 2017
---



> 闭包一直都是`Java`社区中争论不断的话题,很多语言例如`JavaScript`,`Ruby`,`Python`等都支持闭包这个语言特性,闭包功能强大且灵活,`Java`并没有显式地支持它,但其实`Java`中也存在着所谓的"闭包".


----------


> 本文作者为: [SylvanasSun][1].转载请务必将下面这段话置于文章开头处(保留超链接).
> 本文转发自[SylvanasSun Blog][2],原文链接: https://sylvanassun.github.io/2017/07/30/2017-07-30-JavaClosure/



### 闭包


----------



定义一个闭包的要点如下: 

 - 一个依赖于外部环境的`自由变量`的函数.


 - 这个函数能够访问外部环境的`自由变量`.


也就是说,**外部环境持有内部函数所依赖的`自由变量`,由此对内部函数形成了闭包.**


#### 自由变量


----------


那么什么是`自由变量`呢?**`自由变量`就是在函数自身作用域之外的变量**,一个函数$f(x) = x + y$,其中`y`就是`自由变量`,它并不是这个函数自身的自变量,而是通过外部环境提供的.



下面以`JavaScript`的一个闭包为例: 

```javascript
function Add(y) {
	return function(x) {
		return x + y;
	}
}
```

对于内部函数`function(x)`来说,`y`就是`自由变量`.而`y`是函数`Add(y)`内的参数,所以`Add(y)`对内部函数`function(x)`形成了一个闭包.

这个闭包将`自由变量y`与内部函数绑定在了一起,也就是说,当`Add(y)`函数执行完毕后,它不会随着函数调用结束后被回收(不能在栈上分配空间).

```javascript
var add_function = Add(5); // 这时y=5,并且与返回的内部函数绑定在了一起
var result = add_function(10); // x=10,返回最终的结果 10 + 5 = 15
```


### Java中的闭包


----------



`Java`与`JavaScript`又或者其他支持闭包的语言不同,它是一个基于类的面向对象语言,也就是说**一个方法所用到的`自由变量`永远都来自于其所在类的实例的.**

```java
class AddUtils {  
    private int y = 5;  

    public int add(int x) {
    	retrun x + y;
    }
}
```

这样一个方法`add(x)`拥有一个参数`x`与一个`自由变量y`,它的返回值也依赖于这个`自由变量y`.`add(x)`想要正常工作的话,就必须依赖于`AddUtils`类的一个实例,不然它无法知道`自由变量y`的值是多少,也就是`自由变量`未与`add(x)`进行绑定.

严格上来说,`add(x)`中的`自由变量`应该为`this`,这是因为`y`也是通过`this`关键字来访问的.


所以说,在`Java`中闭包其实无处不在,只不过我们难以发现而已.但面向对象的语言一般都不把类叫成闭包,这是一种习惯.


`Java`中的内部类就是一种典型的闭包结构.

```java
public class Outer {
	private int y = 5;

	private class Inner {
		private int x = 10;

		public int add() {
			return x + y;
		}
	}

}  
```

内部类通过一个指向外部类的引用来访问外部环境中的`自由变量`,由此形成了一个闭包.


### 匿名内部类


----------



```java
public interface AnonInner() {
	int add();
}

public class Outer {
	
	public AnonInner getAnonInner(final int x) {
		final int y = 5;
		return new AnonInner() {
			public int add() {
				return x + y;
			}
		}
	}

}
```

`getAnonInner(x)`方法返回了一个匿名内部类`AnonInner`,匿名内部类不能显式地声明构造函数,也不能对构造函数传参,且返回的是一个`AnonInner`接口,但它的`add()`方法实现中用到了两个`自由变量`(`x`与`y`),也就是说外部方法`getAnonInner(x)`对这个匿名内部类构成了闭包.

但我们发现`自由变量`都被加上了`final`修饰符,这是因为`Java`对闭包支持的不完整导致的.

对于`自由变量`的捕获策略有以下两种: 

 - capture-by-value: 只需要在创建闭包的地方把捕获的值拷贝一份到对象里即可.`Java`的匿名内部类和`Java 8`新的`lambda`表达式都是这样实现的.


 - capture-by-reference: 把被捕获的局部变量“提升”（hoist）到对象里.`C#`的匿名函数(匿名委托/lambda表达式)就是这样实现的.


`Java`只实现了`capture-by-value`,但又没有对外说明这一点,为了以后能进一步扩展成支持`capture-by-reference`留后路,所以干脆就不允许向被捕获的变量赋值,所以这些`自由变量`需要强制加上`final`修饰符(在`Jdk8`中似乎已经没有这种强制限制了).



### 参考文献


----------



 - [Java theory and practice: The closures debate][3]


 - [关于对象与闭包的关系的一个有趣小故事][4]


 - [JVM的规范中允许编程语言语义中创建闭包(closure)吗？ - 知乎][5]


 - [为什么Java闭包不能通过返回值之外的方式向外传递值？ - 知乎][6]


[1]: https://github.com/SylvanasSun
[2]: https:/sylvanassun.github.io
[3]: https://www.ibm.com/developerworks/java/library/j-jtp04247/index.html
[4]: http://rednaxelafx.iteye.com/blog/245022
[5]: https://www.zhihu.com/question/27416568/answer/36565794
[6]: https://www.zhihu.com/question/28190927/answer/39786939