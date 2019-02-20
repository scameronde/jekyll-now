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

In **Working with interfaces** we had an interface with two implementations, but used only one for the object graph. But what if we want to use both implementations in different contexts? How do we distinguish both?

<img src="../InterfaceImplementation2.svg"/>

The Java standard for injection uses `@Qualifyer` for this use-case. We will now see how that works in Dagger-2.

First we have do define two qualifyers:

```java
@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface One {
}

@Qualifier
@Documented
@Retention(RetentionPolicy.RUNTIME)
public @interface Two {
}
```

Our module and component should look like that:

```java
@Module
class RootModule {
  @Provides
  @One
  B provideBImpl1(BImpl1 impl) {
    return impl;
  }

  @Provides
  @Two
  B provideBImpl2(BImpl2 impl) {
    return impl;
  }
}

@Component(modules = { RootModule.class } )
interface Root {
  A1 getA1();
  A2 getA2();
}
```

And our classes `A1` and `A2` should look like that:

```java
class A1 {
  B b;

  @Inject
  public A1 (@One B b) {
    this.b = b;
  }
}

class A2 {
  B b;

  @Inject
  public A2 (@Two B b) {
    this.b = b;
  }
}
```

More realistically we do not want our constructor parameters to be littered with annotations. Most of the time we do not know which implementation of B we should use at the time of coding `A1` and `A2`. Therefore we should do it like this:

```java
class A1 {
  B b;

  public A1 (B b) {
    this.b = b;
  }
}

class A2 {
  B b;

  public A2 (B b) {
    this.b = b;
  }
}

@Module
class RootModule {
  @Provides
  @One
  B provideBImpl1(BImpl1 impl) {
    return impl;
  }

  @Provides
  @Two
  B provideBImpl2(BImpl2 impl) {
    return impl;
  }

  @Provides
  A1 provideA1(@One B b) {
    return new A1(b);
  }

  @Provides
  A2 provideA1(@Two B b) {
    return new A2(b);
  }
}
```

## Working with scopes

When calling a method on a component, each time a new instance of the requested object tree is created.

```java
A a1 = root.getA();
A a2 = root.getA();
assert (a1 != a2)
assert (a1.b != a2.b)
assert (a1.b.c != a2.b.c)
```

But what do we do if `C` for example is a stateful service that should be shared by all instances of `A`?

This is when `@Scope` comes into play. When a class that is instantiated in a Dagger-2 component has a scoped annotation, Dagger-2 creates exactly one instance of the class returned by that function for each component instance. Alternatively, one can create and annotate a `@Provides` method. Let's do some examples:

First, we create our scope:

```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface Scope1 {
}
```

Now we can either annotate the class:

```java
@Scope1
class C {
  @Inject
  public C() {
  }
}
```

Or we can annotate a `@Provides` method:

```java
@Module
class RootModule {
  @Provides
  @Scope1
  C getC() {
    return new C();
  }
}

@Component( modules = { RootModule.class } )
@Scope1
interface Root {
  A getA();
}
```

Now the following is true:

```java
A a1 = root.getA();
A a2 = root.getA();
assert (a1 != a2)
assert (a1.b != a2.b)
assert (a1.b.c == a2.b.c)  // changed from != to ==
```

The component creates only one instance of class `C` because it is annotated by a `@Scope` annotation.

Please notice, that this behavior is local to an instance of a component. Each component instance handles its own instances:

```java
Root root1 = DaggerRoot.create();
Root root2 = DaggerRoot.create();
A a1 = root1.getA();
A a2 = root2.getA();
assert (a1 != a2)
assert (a1.b != a2.b)
assert (a1.b.c != a2.b.c)  // changed back to ==
```

One last thing: the component interface has to be annotated by the scope, too.

```java
@Component( modules = { RootModule.class } )
@Scope1
interface Root {
  A getA();
}
```

This is because of an important concept of Dagger-2: a component can only work with one scope. The scope a component works with must be annotated at the component interface. Every module a component works with can only provide objects without a scope or with the scope the component works with. Everything else results in a compiler error.

## Working with a hierarchy of scopes

A business application typically has more than one scope for objects. Possible scopes could be **system** (one instance per running process), **tenant** (one instance per registered tenant) and **session** (one instance per user session). These scopes also have a hierarchy: objects living in the session scope have a shorter live than objects living in the system scope. In our example the hierarchy would be **system->tenant->session**

How can we provide different, hierarchical scopes if a Dagger-2 component can only handle one scope?

The answer is easy: we have to work with hierarchically organized components. It is time to introduce `@Subcomponent`.

The term **subcomponent** might be misleading. **Sub** is not meant like 'part of something larger' but more like 'subclass'. A subcomponent inherits all information from its parent component and adds more information to it. In addition, a subcomponent must have a different scope than that of its parent. The scope of the subcomponent is also smaller than that of its parent. Confusing? An example will bring some clarity.

These are our scopes:

```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface SystemScope {
}

@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface TenantScope {
}

@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface SessionScope {
}
```

Our (sub)components have to look like that:

```java
@Component( modules = { SystemModule.class } )
@SystemScope
interface SystemScopedServices {
  ASystemScopedService getASystemScopedService();
  TenantScopedServices getTenantScopedServices(TenantModule module);
}

@Subcomponent( modules = { TenantModule.class } )
@TenantScope
interface TenantScopedServices {
  ATenantScopedService getATenantScopedService();
  SessionScopedServices getSessionScopedServices(SessionModule module);
}

@Subcomponent( modules = { SessionModule.class } )
@SessionScope
interface SessionScopedServices {
  ASessionScopedService getASessionScopedService();
}
```

with their corresponding modules looking like this:

```java
@Module
public class SystemModule {
  public SystemModule() {
  }

  @Provides
  @SystemScope
  public ASystemScopedService provideService() {
    return new SystemScopedService();
  }
}

@Module
public class TenantModule {
  private final Tenant tenant;

  public TenantModule(Integer tenantId) {
    this.tenant = new Tenant(tenantId);
  }

  @Provides
  @TenantScope
  Tenant provideTenant() {
    return this.tenant;
  }

  @Provides
  @TenantScope
  public ATenantScopedService provideService(Tenant tenant) {
    return new TenantScopedService(tenant);
  }
}

@Module
public class SessionModule {
  private final Session session;

  public SessionModule(Integer sessionId) {
    this.session = new Session(sessionId);
  }

  @Provides
  @SessionScope
  Session provideSession() {
    return this.session;
  }

  @Provides
  @SessionScope
  public ASessionScopedService provideService(Session session) {
    return new SessionScopedService(session);
  }
}
```

The only component is the `SystemScopedServices` component. The other two are subcomponents. The hierarchy of components is declared by the functions that return the subcomponents (`TenantScopedServices` and `SessionScopedServices` respectively).

You must be aware, that every call to `SystemScopedServices#getTenantScopedServices(...)` and `TenantScopedServices#getSessionScopedServices(...)` will return a new Instance of the corresponding subcomponent. The same way you are responsible to keep and provide a system-wide singleton of the `SessionScopedServices` you are responsible to keep, provide and destroy the `TenantScopedServices` and `SessionScopedServices`.
