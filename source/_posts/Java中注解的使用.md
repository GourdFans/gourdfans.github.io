---
title: Java中注解的使用
date: 2017-11-06 10:48:55
tags: Java
---

在日常的开发中，经常会使用到类似这样的东西 @Test @Override等东西，特别是在使用第三方框架的时候比如Spring中就会有大量用到注解的地方。关于什么是注解？怎么去使用注解？如何实现一个注解？这些地方以前没有了解过，今天就把Java中关于注解的使用给理一理。

注解是在J2SE 1.5版本中引入的，主要是提供了一种机制，这种机制允许程序员在编写代码的时候可以定义一些元数据。所谓的元数据这个词，可以这样来及理解。元数据就是定义自身的数据。注解就是代码的元数据，它们包含了代码的信息.

Annotation其实是一种接口，通过Java的反射机制相关的API来访问Annotation信息，相关类根据这些信息来决定如何使用程序的元素或者改变它们的行为。但是当一个类直接继承java.lang.annotation.Annotation接口时，仍是接口，而非注解。要想成为注解，只能通过关键字@interface来声明

@interface声明一个注解的时候，其中每一个方法实际都声明了一个配置参数，方法名就是参数名称，返回值就是参数的类型，可以通过default来实现默认的值

在Java中，关于注解，有四个元注解，同样所谓的元注解，就是可以用来定义注解的注解。包含一下四中

* @Retention 主要用来表示注解的生命周期
* @Target 主要用来表示注解的作用域
* @Documented 用来生成Java doc文档中的内容
* @Inerited 用来定义使用注解的类的继承相关的东西.

@Retention是用来说明如何村已经被标记的注解，有三种可以选择的值

* SOURCE：标志注解会被编译器忽略，并只会保存在源代码中
* CLASS：表明这个注解会通过编译留在CLASS文件中，这样一些操作CLASS相关的代码都可以从CLASS中获取到改信息
* RUNTIME：表示该注解会被JVM中获取，这样在运行中我们就可以通过反射来获取到改注解

下面的这段代码，就定义了一个注解，并且指定可以在运行时拿到该注解信息

```java 
@Retention(RetentionPolicy.RUNTIME)
public @interface NeedLogin {

    boolean required() default true;
}
```
@Target注解用来定义被注解的类型

* ANNOTATION_TYPE 表示该注解可以使用在注解上
* CONSTRUCTOR表示可以用到构造器上
* FIELD表示可以用到作用域上，
* METHOD表示可以用在方法上
* LOCAL_VARIABLE表示可以使用到局部变量上。
* METHOD可以使用到方法级别的注解上。
* PACKAGE可以使用到包声明上。
* PARAMETER可以使用到方法的参数上
* TYPE可以使用到一个类的任何元素上。

注解的作用域比较多，可以根据需要，选择上面的一个或者多个来使用,下面这样定义注解的时候，就表示了改注解，只能定义在方法上和类上了.
```
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NeedLogin {

    boolean required() default true;
}
```

@Documented表示该注解类型的元素会通过javadoc类似的工具进行文档化，该注解没有成员，用于注解那些影响客户使用代注解的元素声明的类型。如果类型使用@Documented来注解的，这种类型的注解被标注未程序元的公共API。
```
@Documented
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NeedLogin {

    boolean required() default true;
}
```

@Inherited注解时一个标记注解，阐述了某个被标注的类型是被继承的，如果使用了@Inherited修饰的注解被用于一个class，则这个注解将被用于该class的子类。需要注意的是，@Inherited类型是被标注过的class的子类所继承，类并不从他继承的interface中继承该注解，方法并不从他重载的method中继承该注解。如果我们在代码中使用反射的时候，在去查询一个@Interited注解类型的时候，反射代码检查器class和父类的class，直到发现制定的注解或者到达继承结构的顶层为止
```
@Documented
@Inherited
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface NeedLogin {

    boolean required() default true;
}
```

上面给出了几种元注解的使用和如何定义自己的注解。在定义了注解以后，只是相当于给某段代码或者某个运行中的类，或者参数方法等打上了一个标签，具体怎么使用改标签来影城程序的运行，需要自己在代码中实现才可以。例如上面我们定义了一个@NeedLogin的注解，有一个参数，required默认未true，表示需要登陆的，现在我们就来使用该注解,在Student上使用该注解，表示学生需要登陆才可以.
```
@Data
@NeedLogin(required = false)
@NoArgsConstructor
public class Student {
    private Integer age ;

    private Integer score;
}

```

我们可以在代码中通过反射的方式来拿到该注解，并通过注解的值来影响程序的运行
```
Student student = new Student();
NeedLogin needLogin = Student.class.getAnnotation(NeedLogin.class);
if (needLogin != null && !needLogin.required()) {
    System.out.println("no need login");
} else {
    System.out.println("need login");
}
```

在Java8中，注解的能力又进一步增强，在Java 8之前，只能允许在声明式前使用注解，而在Java 8中。注解可以用在使用Type的任何地方，比如对象初始化，对象转换 使用implements表达式或者使用throw表达式的时候
```
//初始化对象时
String myString = new @NotNull String();

//对象类型转化时
myString = (@NonNull String) str;

//使用 implements 表达式时
class MyList<T> implements @ReadOnly List<@ReadOnly T>{
                    …
}

//使用 throws 表达式时
public void validateValues() throws @Critical ValidationFailedException{
                    ...
}
```
定义的时候也比较简单，只需要指定Target为 ElementType.TYPE_PARAMETER 或者 ElementType.TYPE_USE，或者同时指定这两个 Target。 

在Java8中，同时允许Repeating Annotation 重复注解。因为在实际情况中，比如一个权限的注解中，在一个地方可能需要多个权限，在以前只能在一个注解上通过列表来实现，在Java8中可以允许一个位置有多个重复的注解来解决该问题。
由于兼容性的缘故，Repeating Annotation 并不是所有新定义的 Annotation 的默认特性，需要开发者根据自己的需求决定新定义的 Annotation 是否可以重复标注。Java 编译器会自动把 Repeating Annotation 储存到指定的 Container Annotation 中。而为了触发编译器进行这一操作，开发者需要进行以下的定义：

首先，在需要重复标注特性的 Annotation 前加上 @Repeatable 标签，示例如下：
```
@Repeatable(AccessContainer.class)
public @interface Access {
        String role();
}
```
@Repeatable 标签后括号中的值即为指定的 Container Annotation 的类型。在这个例子中，Container Annotation 的类型是 AccessContainer，Java 编译器会把重复的 Access 对象保存在 AccessContainer 中。


AccessContainer 中必须定义返回数组类型的 value 方法。数组中元素的类型必须为对应的 Repeating Annotation 类型。具体示例如:
```java 
public @interface AccessContainer {
     Access[] value()
}
```

可以通过 Java 的反射机制获取注解的 Annotation。一种方式是通过 AnnotatedElement 接口的 getAnnotationByType(Class<T>) 首先获得 Container Annotation，然后再通过 Container Annotation 的 value 方法获得 Repeating Annotation。另一种方式是用过 AnnotatedElement 接口的 getAnnotations(Class<T>) 方法一次性返回 Repeating Annotation。

Repeating Annotation 使得开发者可以根据具体的需求对同一个声明式或者类型加上同一类型的注解，从而增加代码的灵活性和可读性。

数量的使用注解，能够减少开发中的复杂性，而且能够使代码更加优雅清晰。在Java中定义Bean的时候，就可以使用 lombok来减少Get Set方法的定义。在Junit中和Spring中都有大量使用注解的地方，具有很好的参考意义。

参考文档：
https://www.ibm.com/developerworks/cn/java/j-lo-java8annotation/index.html 
http://linbinghe.com/2017/ac8515d0.html
http://ifeve.com/java-annotations-tutorial/
