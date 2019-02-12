---
layout: page
published: true
title: An introduction to dependency injection with Dagger-2
---

Why bother looking at a dependency injection framework for Java when you can use Spring or the standard implementation of your application server? Because it does something notably different: it is a compile-time dependency injection framework.

What does that mean? Spring and many other DI frameworks operate while your program is running. Their knowledge about the object tree that has to be built is constructed at runtime. They scan the classpath for annotated classes, read and parse configuration files and add information you provided by using the API of your framework of choice. Consequently, while building the object graph, they create and inject dependencies using reflection. The advantages of this modus operandi are the same as with dynamic typing: you have great flexibility and very low overhead while you are programming. Unfortunately, the disadvantages are also the same: it all falls apart if you did something wrong. The program crashes if a dependency cannot be satisfied. Also, static code analysis is nearly impossible. What is missing? What is superfluous and can be deleted? What depends on what?

The answer to these problems is the same as with typing: ditching dynamic for static.

Dagger-2 does all that other DI frameworks are doing but at compile time. It scans the codebase for information about the object graph and checks if all dependencies can be satisfied without any ambiguities. If this is not the case, compiling fails. If everything is fine, Dagger-2 generates source code that builds the object graph at runtime.

Besides the obvious advantage of having the guaranty that dependency injection will not fail at runtime, there are more benefits to the static approach: object tree creating is faster because no reflection is used and static code analysis tools can do their job.

But how does it all work?

## Dagger-2 concepts

Dependency injection for Java uses some standardized concepts, represented by annotations:

- @Inject
- @Scope
- @Qualifyer
  
Dagger-2 adds three more concepts:

- @Component
- @Module
- @Provides


## Basic object tree creation

Let's start with a simple object tree.

<img src="../SimpleObjectGraph.svg"/>

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

All three methods work only for non-private constructors/fields/methods because the generated code has to have access.

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

But how do we create an instance of the object graph? We have to tell Dagger-2 for what root element it should generate the necessary code. This is done using an interface and `@Component`.

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

But Dagger-2 can do more. Let's say you already have an object, maybe created by another framework, that is not fully initialized. Dagger can initialize that object, creating any dependencies as needed.

For this example, we have to change class `A` like so:

```java
class A {
  @Inject
  B b;

  public A() {
  }
}
```

The component interface should look like this:

```java
@Component
interface Root {
  A inject(A a);
}
```

And the code using Dagger-2 looks like:

```java
Root root = DaggerRoot.create();
A a = someOtherService.getUninitializedA();
a = root.inject(a);
```

## Working with interfaces

Object graphs made out of simple classes are fine and dandy, but many times we have to work with services that are defined by an interface and realized by one or more implementations. Again we are working with a simple example:

<img src="../InterfaceImplementation.svg"/>

There is no way for Dagger-2 to know, which implementation of `B` it should inject, without us telling it what to do. It is time to introduce `@Provides` and `@Module`.

Here we have our classes:

```java
class A {
  B b;

  @Inject
  public A (B b) {
    this.b = b;
  }
}

interface B {
}

class BImpl1 implements B {
  @Inject
  public BImpl1() {
  }
}

class BImpl2 implements B {
  @Inject
  public BImpl2() {
  }
}
```

Every time, a Dagger-2 component does not know how to do something, you can provide it with one or more **modules** that provide the missing knowledge.

```java
@Module
class RootModule {
  @Provides
  B provideBImpl1(BImpl1 impl) {
    return impl;
  }
}

@Component(modules = { RootModule.class } )
interface Root {
  A getA();
}
```

The module now tells Dagger-2 which implementation of `B` it should use. See how you do not have to instantiate `BImpl1` on your own? Dagger-2 can do that for you. You just have to provide the knowledge, and the component has to reference the module containing the knowledge.

You can also provide an instance of `BImpl1` yourself if you would like to.

```java
@Module
class RootModule {
  @Provides
  B provideBImpl1() {
    return new BImpl1();
  }
}
```

But how does Dagger-2 know how to create an instance of `RootModule` for the component `Root`? In this example, it is easy because `RootModule` has a default constructor. So everything is as it used to be:

```java
Root root = DaggerRoot.create();
```

But we could also do it like this:

```java
Root root = DaggerRoot.builder()
                      .rootModule(new RootModule())
                      .build();
```

Using the second approach, we could work with modules that need some additional data to be initialized (something we will explore later).

## Working with classes that you can not change or are difficult to setup

How do we work with classes that are either complicated to set up, or that we only have the binary of? The solution is the same as above: using modules.

Let's use the classes `A`, `B` and `C` again, but this time without annotations:

```java
class A {
  B b;

  public A(B b) {
    this.b = b;
  }
}

class B {
  C c;

  public B(C c) {
    this.c = c;
  }
}

class C {
  public C() {
  }
}
```

We have to provide a module that looks like this:

```java
@Module
class RootModule {
  @Provides
  C provideC() {
    return new C();
  }

  @Provides
  B provideB(C c) {
    return new B(c);
  }

  @Provides
  A provideA(B b) {
    return new A(b);
  }
}

@Component(modules = { RootModule.class } )
interface Root {
  A getA();
}
```

The usage stays the same:

```java
Root root = DaggerRoot.create();
A a = root.getA();
```

## Working with more than one implementation

## Working with scopes

## Working with a hierarchy of scopes

