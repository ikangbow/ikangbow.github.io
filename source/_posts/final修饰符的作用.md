---
title: final和static的作用
date: 2019-01-06 12:59:46
tags: java
category: java
---
# final关键字的作用
1. 被final修饰的类不可以被继承
2. 被final修饰的方法不可以被重写
3. 被final修饰的变量不可被改变

# 重点
1. 被fianl修饰不可变的是变量的引用，而不是引用指向的内容，引用指向的内容是可以改变的。
2. 被fina修饰的常量，在编译阶段会存入调用类的常量池中。

# static关键字的作用
1. 被static修饰的变量属于类变量，可以通过 类名.变量名 直接引用，而不需要new出一个类。
2. 被static修饰的方法属于类方法，可以通过 类名.变量名 直接引用，而不需要new出一个类。

被static修饰的变量和方法属于类的静态资源，是类实例之间共享的。JDK把不同的静态资源放在了不同的类中而不把所有的静态资源放在一个类里面是因为：
1. 不同的类有自己的静态资源，这可以实现静态资源分类。比如和数学相关的静态资源放在java.lang.Math中，和日历相关的静态资源放在java.util.Calendar中，这样比较清晰
2. 避免重名。不同的类之间有重名的静态变量名，静态方法名也是很正常。
3. 避免静态资源类无限膨胀。

**问题**
静态方法能不能引用非静态资源？静态方法里面能不能引用静态资源？非静态方法里面能不能引用静态资源？比如就以这段代码为例，是否有错？

    public class A{
     	private int i = 1;
    	public static void main(String[] args){
		i = 1;//错误
		}
    }

静态资源属于类，但是是独立于类存在的。从JVM类的加载机制的角度讲，静态资源是类初始化的时候加载的，而非静态资源是类new的时候加载的。类的初始化早于类的new,比如Class.forName("xxx"),就是初始化了一个类，但是并没有new它，只是加载这个类的静态资源。所以对于静态资源来说，它是不可能知道一个类中有哪些非静态资源的；但是对于非静态资源来说就不一样了，它是new出来之后产生的，因此属于类的它都认识。所以结论是：
1. 静态方法不能引用非静态资源。非静态资源在new对象的时候才会产生，晚于一初始化就存在的静态资源。
2. 静态方法里面可以引用静态资源。都是类初始化的时候加载的。
3. 非静态方法可以引用静态资源。非静态方法是new之后产生的，静态资源是类一初始化就存在的。

# 静态块

静态块是static的重要应用之一。主要用于初始化一个类的时候做操作用的，和静态变量，静态方法一样，静态块里面的代码只执行一次，且只在类初始化的时候执行。

    public class A{
    	private static int a = B();
    	static{
    		System.out.println("Enter A.static block");
		}
	    public static void main(String[] args){
	    	new A();
	    }

	    public static int B(){
		    System.out.println("Enter A.B()");
		    return 1;
	    }
    }

打印结果：

    Enter A.B()
    Enter A.static block

结论：

静态资源的加载顺序是严格安装静态资源的定义顺序加载的。

    public class A{
	    static
	    {
	        c = 3;
	        System.out.println(c);//Cannot reference a field before it is defined
	    }
	    
	     private static int c;
	}

结论：

静态代码块对于定义在它之后的静态变量，可以赋值，但是不能访问。

    public class A{
	    static
	    {
	        System.out.println("A.static block");
	    }
	    
	    public A()
	    {
	        System.out.println("A.constructor()");
	    }
	}

    public class B extends A{
	    static 
	    {
	        System.out.println("B.static block");
	    }
	    
	    public B()
	    {
	        System.out.println("B.constructor()");
	    }
	    
	    public static void main(String[] args)
	    {
	        new B();
	        new B();
	    }
	}

打印结果：

    A.static block
    B.static block
    A.constructor()
    B.constructor()
    A.constructor()
    B.constructor()

结论：

静态代码块是严格按照父类静态代码块》子类静态代码块的顺序加载的，且只加载一次。

# 项目中用到的
项目中的角色类型通常用static final修饰

    public static final String TYPE_A = "a";
    public static final String TYPE_B = "b";

public:使接口的实现类可以使用这个常量

static:static修饰的表示是属于类的，随着类的加载而存在。如果是非static的，就表示属于对象的，只有建立对象时才有它，而接口是不能建立对象的，所以接口的常量必须定义为satic

final:fina修饰保证接口定义的常量不能被实现类去修改，如果没有final的话，任由子类随意去修改的话，接口建立这个常量就没有意义了。