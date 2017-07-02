---
title: AspectJ学习
date: 2017-06-24 23:31:12
tags: AOP
categories: Programming
---

# Introduction to AspectJ
1.pointcuts 切入点
2.advice
3.inter-type declarations 类型间声明
4.aspects 方面

pointcuts和advice动态的影响程序流，inter-type declarations静态的影响程序的类层级，aspects封装了这些构造。
(pointcuts and advice dynamically affect program flow, inter-type decalrations statically affects program's class hierarchy, and aspects encapsulate these new constructs.)

join point是程序流中定义明确的点，pointcut会挑选处join point和这些点处的值，advice是程序运行至join point时执行的代码。这些就是AspectJ的动态部分。
(A *join point* is a well-defined point in the program flow. A *pointcut* picks out certain join points and values at those points. A piece of *advice* is code that is excuted when a join point is reached. These are the dynamic part of AspectJ.)

AspectJ 还有不同种类的inter-type declarations, 这些，允许程序员修改程序的结果，也就是类的数量和类之间的关系。
(AspectJ alse has defferent kinds of *inter-type declarations* that allow the programmer to modify a program's sturcture, namely, the members of classes and the relationship between classes.)

AspectJ的aspect是横切问题的模块化单元。
(AspectJ's *aspect* are the unit of modularity for crosscutting concerns. They behave somewhat like Java classes, but may alse include pointcuts, advice and inter-type declarations.)

## The Dynamic Join Point Model
The join point model provides the common frame of reference that make it possible to define the dynamic structure of crosscutting concerns.
AspectJ provides for many kinds of join points. One of them is method call join point.

method call join points 包含接收防范调用的对象的行为，它包括构成方法调用的所有行为，从所有参数被评估之后开始，还包括返回（正常返回或抛出异常）。
(A method call join points encompasses the actions of an object receiving a method call. It includes all the actions that comprise a method call, starting after all arguments are evaluated up to and including return (either normally or by throwing an exception).)

运行时的每次方法调用都是不同的join point，尽管它来自程序中相同的调用表达式。许多其他的join point可以在一个method call join point执行的同时运行——当执行方法体时发生的所有连接点，以及在这些方法中调用的方法。我们说这些join point在原始join point的动态上下文中执行。
(Each method call at runtime is a different join point, even if it comes from the same call expression in the program. Many other join point may run while a method call join point is executing -- all the join points that happen while executing the method body, and in those methods called from the body. we say that these join points execute in the dynamic context of the original call join point.)

## Pointcuts

基于显式的方法签名的列举，即name-based crosscutting。
```
//1
call(void Point.setX(int))

//2
call(void Point.setX(int)) ||
call(void Point.setY(int))

//3
pointcut move():
    call(void FigureElement.setXY(int,int)) ||
    call(void Point.setX(int))              ||
    call(void Point.setY(int))              ||
    call(void Line.setP1(Point))            ||
    call(void Line.setP2(Point));
```
基于方法的属性而不是准确的名字来指定pointcut，即property-based crosscutting。
```
//wildcards
call(void Figure.make*(..))
call(public * Figure.* (..))

//cflow
cflow(move())
```

## Advice
pointcuts选出join point，但是它并没有做什么事情。为了实现横切行为，我们使用advice。

Advice brings together a pointcut(to pick out join points) and a body of code(to run at each of those join points).

AspectJ有几个不同的advice。
*before advice*
*after advice*，包括after returning, after throwing, after

### Exposing context in Pointcuts

Pointcuts not only pick out join points, they can alse expose part of execution context at their join points.Values exposed by a pointcut can be used in the body of advice declarations.

An advice declaration has a parameter list(like a method)that gives names to all the pieces of context that it uses. For example:
```
after(FigureElement fe, int x, int y) returning:
        ...SomePointcut... {
    ...SomeBody...
}
```

The body of the advice uses the names just like method parameters, so
```
after(FigureElement fe, int x, int y) returning:
        ...SomePointcut... {
    System.out.println(fe + " moved to (" + x + ", " + y + ")");
}
```

