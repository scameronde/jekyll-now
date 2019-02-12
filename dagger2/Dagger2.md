---
layout: page
published: true
title: An introduction to dependency injection with Dagger-2
---

Why bother looking at a dependency injection framework for Java when you can use Spring or the standard implementation of your application server? Because it does something notably different: it is a compile time dependency injection framework.

What does that mean? Spring and many other DI frameworks operate while your programm is running. Their knowledge about the object tree that has to be build is constructed at runtime. They scan the classpath for annotated classes, read and parse configuration files and add information you provided by using the API of your framework of choice. Consequently, while building the object graph, they create and inject dependencies using reflection. The advantages of this modus operandi are the same as with dynamic typing: you have great flexibility and very low overhead while you are programming. Unfortunately, the disadvantages are also the same: it all falls apart if you did something wrong. The program crashes if a dependency can not be satisfied. Also static code analyses are nearly impossible. What is missing? What is superfluous and can be deleted? What depends on what?

The answer to this problems are the same as with typing: ditching dynamic for static.

Dagger-2 does all what other DI frameworks are doing, but at compile time. It scans the codebase for information about the object graph and checks if all dependencies can be satisfied without any ambiguities. If this is not the case, compiling fails. If everything is fine, Dagger-2 generates source code that builds the object graph at runtime.

Besides the obvious advantage of having the guaranty that dependency injection will not fail at runtime, there are more benefits to the static approach: object tree creating is faster because no reflection is used and static code analysis tools can do their job.

But how does it all work?

## Dagger-2 concepts

Dependency injection for Java uses some standarized concepts, represented by annotations:

- @Inject
- @Scope
- @Qualifyer
  
Dagger-2 adds three more concepts:

- @Component
- @Module
- @Provides


## Basic object tree creation

Let's start with a simple object tree.

<svg src="SimpleObjectGraph.svg"></svg>

With Dagger-2 you can use three types of dependency injection:

**constructor injection**

```java
class A {
  B b;

  @Inject
  public A(B b) {
    this.b = b;
  }
}
```

**field injection**

```java
class A {
  @Inject
  B b;

  public A() {
  }
}
```

**method injection**

```java
class A {
  B b;

  public A() {
  }

  @Inject
  void setB(B b) {
    this.b = b;
  }
}
```

All three methods work only for non-private constructors/fields/methods, because the generated code has to have access.

Because most of the time it is a good idea to combine object creating with object initialization, we will use constructor injection for most examples. So our code looks like this:

```java
class A {
  B b;

  @Inject
  public A(B b) {
    this.b = b;
  }
}

class B {
  C c;

  @Inject
  public B(C c) {
    this.c = c;
  }
}

class C {
  @Inject
  public C() {
  }
}
```

But how do we create an instance of the object graph? We have to tell Dagger-2 for what root element it should generate the neccessary code. This is done using an interface and `@Component`.

```java
@Component
interface Root {
  A getA();
}
```

Dagger-2 now generates a class named `DaggerRoot` that implements the marked interface `Root`. You can get an instance of the object graph by simply doing:

```java
Root root = DaggerRoot.create();
A a = root.getA();
```

## Initializing existing objects

## Working with interfaces

## Working with classes that you can not change
