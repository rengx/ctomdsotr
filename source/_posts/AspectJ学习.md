---
title: AspectJ学习
date: 2017-06-24 23:31:12
tags: AOP
categories: Programming
---

AspectJ ，Java语言的AOP(Aspect-Oritend Programming)扩展。

# Introduction to AspectJ
*（主要是在官网学习时的笔记）*

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


# 使用
## 安装aspectj

aspect安装下载，
官网地址，速度可能会慢：http://www.eclipse.org/aspectj/downloads.php#install
也可使用：http://download.csdn.net/detail/smaiiboy/9477346

笔者下载的是最新稳定版（Latest Stable Release）：aspectj-1.8.10.jar（下载时间2017/7/7），这是一个可执行jar，有别于我们通常使用的静态jar包。下载后使用jar命令安装：
```
java -jar {jar包位置}/aspectj-1.8.10.jar
```
运行后会看到安装界面，安装后如下：
![安装成功图](index_files/d88b0746-1755-488e-b822-addca3002f92.png)

## 开发
AspectJ需要专门的JDK来编译，可以使用IDE插件比如Eclipse AJDT开发，也可以使用注解方式（AspectJ1.5版本或更新）。
### 使用注解开发
笔者使用Maven来构建的，使用的Mojo的aspectj-maven-plugin [http://mojo.codehaus.org/aspectj-maven-plugin/](http://mojo.codehaus.org/aspectj-maven-plugin/) 。
（从http://blog.csdn.net/zl3450341/article/details/7673942 获悉的Maven编译插件，感谢博主！）

pom.xml配置
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ctomdsotr.study</groupId>
    <artifactId>aspect</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>aspect Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <properties>
        <aspectjrt.version>1.8.10</aspectjrt.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>${aspectjrt.version}</version>
        </dependency>

        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectjrt.version}</version>
        </dependency>
    </dependencies>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.codehaus.mojo</groupId>
                    <artifactId>aspectj-maven-plugin</artifactId>
                    <version>1.8</version>
                    <configuration>
                        <!-- 选择大于1.5的版本，因为使用的是aspectj的注解方式 -->
                        <complianceLevel>1.8</complianceLevel>
                        <source>1.8</source>
                        <target>1.8</target>
                        <sources>
                            <source>
                                <basedir>src/main/java</basedir>
                                <includes>
                                    <include>**/UserAspect.java</include>
                                    <include>**/UserServiceImpl.java</include>  <!-- 不仅是aspect,还有其他的，不然只会编译aspect，没有办法测试，运行时会报错误: 找不到或无法加载主类 -->
                                </includes>
                            </source>
                        </sources>
                    </configuration>
                    <executions>
                        <execution>
                            <goals>
                                <goal>compile</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```
源码会放在github上。

具体的步骤笔者不在赘述，从官网或搜索引擎上都能找到资料，足以帮助开发者学习。笔者列举一下自己遇到的问题，如果有其他人正好遇到，希望能有所帮助。
#### 如何编译
配置好上述pom，然后使用mvn aspectj:complile 编译，在eclipse中，就是在项目或pom.xml上右键"Run As"，选择"Build build..."，填入"aspectj:complile" 。

#### 编译时报错（没有加@aspect）

#### 编译时提示http.servet.httpServert

#### 编译成功，但是没有执行切面的advices

#### 编译成功，但是只执行切面的advice,原来的方法被覆盖了，没有执行

### 使用AJDT开发
官网下载地址：http://www.eclipse.org/aspectj/downloads.php
（AJDT for Eclipse：http://www.eclipse.org/ajdt/ ，其他IDEs: http://www.eclipse.org/aspectj/downloads.php#ides ）
*（笔者只是写了一个例子，先贴一下代码）*
```
/**先写一个java class*/
package com.ctomdsotr.study.aspectj.user;
public class User {
    private String userName;
    private String userId;

    public void addUser(String userId, String userName) {
        this.userId = userId;
        this.userName = userName;
        System.out.println("added "+userId);
    }
    public String getUser(String userId) {
        return userName;
    }
    public static void main(String[] args) {
        User user = new User();
        user.addUser("1", "2");
    }
}

/**创建一个切面*/
package com.ctomdsotr.study.aspectj.user.aspect;
import com.ctomdsotr.study.aspectj.user.User;
public aspect FirstAspect {
    pointcut addUser(String userId, String userName): execution(void User.addUser(String, String)) && args(userId, userName);
    after(String userId, String userName) returning : addUser(userId, userName){
        System.out.println(userId + ":" + userName);
    }
}

/**最后运行User，控制台输出*/
added 1
1:2
```

# 参考
> 
AspectJ Doc http://www.eclipse.org/aspectj/doc/released/adk15notebook/ataspectj.html
Maven Plugin http://www.mojohaus.org/aspectj-maven-plugin/index.html
Maven Jar http://mvnrepository.com/artifact/org.aspectj
AspectJ 安装文件 http://download.csdn.net/detail/smaiiboy/9477346
两个挺好的博客 http://blog.csdn.net/zl3450341/article/details/7673942 http://blog.csdn.net/ccj659/article/details/53302951

