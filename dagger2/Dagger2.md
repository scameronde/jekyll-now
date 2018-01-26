---
layout: page
published: true
---

# Dagger-2 Grundlagen

Dagger2 ist ein Compile-Time-Injection Framework. D.h. es findet keinerlei Reflection zur Laufzeit statt. Das hat mehrere Vorteile:
 
- besseres Laufzeitverhalten
- einfachere Stacktraces
- Fehler im Objektgraph werden schon vom Compiler erkannt
- ob Implementierungen von Interfaces tatsächlich genutzt werden, kann durch statische Codeanalyse festgestellt werden
 
 
Dagger2 baut auf wenigen Konzepten auf:
 
- Components
- Modules
- Scopes
- Qualifiers


Ein Injection-Framework wird für 2 Use-Cases benötigt:

1. ich benötige einen vollständig erzeugten und initialisierten Objektgraphen
2. ich habe bereits ein Objekt dessen Abhängigkeiten aber noch erfüllt werden müssen


## Components, oder die Wurzel von allem

Components liefern mir die „vollständig erzeugten und initialisierten Objektgraphen“, und sind in der Lage bereits erzeugte Objekte mit deren Abhängigkeiten zu versorgen.

### Components erzeugen und initialisieren Objekte

Ein Interface

```java
@Component
interface MyComponent {
  Objectgraph1 getObjectgraph1();
  Objectgraph2 getObjectgraph2();
}
```

weist Dagger an, zur Compilezeit eine Klasse `DaggerMyComponent` zu generieren, die mir die Objektgraphen erzeugt. Die so erzeugte Klasse wird wie folgt verwendet:

```java
MyComponent component = DaggerMyComponent.builder().build();
Objectgraph1 graph1 = component.getObjectgraph1();
```

Jeder Aufruf von `component.getObjectgraph1()` liefert mir eine neue Instanz des `Objectgraph1`.

### Components initialisieren Objekte, erzeugen diese aber nicht

Wenn ein Objekt schon vorhanden ist, kann es über die Component nachträglich mit Anhängigkeiten versorgt werden. Durch folgende Signaturen im Component-Interface

```java
@Component
interface MyComponent {
  Objectgraph1 inject1(Objectgraph1 instance);
  Objectgraph2 inject2(Objectgraph2 instance);
}
```

werden passende Methoden zur Compilezeit generiert, die wie folgt verwendet werden können:

```java
MyComponent component = DaggerMyComponent.builder().build();
Objectgraph1 graph1 = getFromElsewhere();
component.inject1(graph1);
```

## Erzeugen von Objekten

### per Annotation

Wenn es nur um Klassen geht und ich den Code in der Hand habe, dann kann man Dagger per Annotation zeigen, wie es welche Klasse erzeugen kann. Dazu muss nur genau ein Constructor der Klasse - wir betrachten jetzt nur den Default-Constructor - mit der Annotation `@Inject` versehen werden.

```java
class A {
  @Inject
  A() { ... };
}


@Component
interface MyComponent {
  A getA();
}

```


### explizit über Module

Wenn ich an den Code der Klassen nicht herankomme, ich gegen ein Interface programmiert habe oder die Instanz aufwändig zu erzeugen ist, dann muss ich Module verwenden. 

```java
interface A {
}


class MyA implements A {
  MyA() { ... };
}


@Component(modules = { MyModule.class })
interface MyComponent {
  A getA();
}


@Module
class MyModule {
  @Provides
  A provideA() {
    return new MyA();
  }
}
```

Generell würde ich empfehlen **immer** Module zu verwenden und auf die Erzeugung per Annotation zu verzichten. Die Bedeutung dieser Empfehlung wird im Kontext von Scopes noch einmal deutlicher.


## Initialisieren von Objekten / Erfüllen von Abhängigkeiten

Für einzelne Objekte müsste man den Aufwand nicht betreiben. Spannend wird das alles, wenn Abhängigkeiten erfüllt werden müssen.


### per Annotation

Auch hier gibt es mehrere Möglichkeiten.

Ich kann die Erzeugung und die Initialisierung über Constructor-Injection gemeinsam erledigen:

```java
class C {
  A a;
  B b;

  @Inject
  C(A a, B b) {
    this.a=a; this.b=b;
  }
}
```

oder ich kann Method-Injection nehmen. Das hat den Vorteil, dass sie auch in den Fällen funktioniert, bei denen das Objekt bereits anderweitig erzeugt wurde:


```java
class C {
  A a;
  B b;

  @Inject // optional!
  C() {
  }

  @Inject
  void setA(A a) { ... };

  @Inject
  void setB(B b) { ... };
}
```

Method-Injection von Dagger funktioniert sogar mit mehreren Parametern:

```java
class C {
  A a;
  B b;

  @Inject // optional!
  C() {
  }

  @Inject
  void setAll(A a, B b) { ... };
}
```

Für alle **nicht-private** Fields geht auch Field-Injection:

```java
class C {
  @Inject
  A a;

  @Inject
  B b;

  @Inject // optional!
  C() {
  }
}
```


### explizit über Module

Aktuell weiß ich nur, dass man über Module Erzeugung **plus** Initialisierung abbilden kann. Eine reine Initialisierung, für den Fall dass das Objekt anderweitig erzeugt wurde, ist meines Wissens nach nich möglich.

```java
@Module
class MyModule {
  @Provides
  C provideC(A a, B b) {
    return new MyC(a, b);
  }
}
```

## Besonderheiten bei der Erzeugung/Initialisierung von Objekten

### Nullable

Dagger achtet darauf, dass kein Provider `null` zurückliefert bzw. dass keine Dependency (egal über Constructor-, Method- oder Field-Injection) mit `null` belegt wird.

Sollte dies trotzdem gewünscht sein, müssen die entsprechenden Stellen mit der Annotation `@Nullable` versehen werden.


### Lazy

Nicht immer benötigt man eine Dependency direkt zum Zeitpunkt der Injection. Evtl. benötigt man sie situationsabhängig überhaupt nicht. Sollte die Erzeugung teuer sein, kann man diese verzögern. Dazu muss man nur als Field btw. Methodenparameter statt dem Typ `T` den Typ `Lazy<T>` angeben.

```java
Lazy<T> lt;

T t;

t = lt.get(); // das Objekt wird erzeugt
assert (t == lt.get());  // danach wird immer das selbe Objekt zurück gegeben
```


### Provider

`Provider` funktioniert ganz ähnlich wie `Lazy`, nur dass bei jedem Zugriff ein neues Objekt erzeugt wird.

```java
Provider<T> lt;

T t;

t = lt.get(); // das Objekt wird erzeugt
assert (t != lt.get());  // es wird wieder eine neue Instanz erzeugt
```
 

###  Qualifyer

Wenn es mehrere Implementierungen zu einem Interface gibt, muss man Dagger mitteilen, welche Variante an welcher Stelle injected werden soll. Dies kann man - sowohl mit `@Inject` Annotationen als auch in Modulen - mit Hilfe von Qualifyern erledigen. Ein Standardqualifyer ist `@Named`. Man kann aber auch eigene definieren.

```java
interface A {
}


class FirstA implements A {
  FirstA() { ... };
}

class SecondA implements A {
  SecondA() { ... };
}


@Module
class MyModule {
  @Provides
  @Named(„first“)
  A provideFirstA() {
    return new FirstA();
  }

  @Provides
  @Named(„second“)
  A provideSecondA() {
    return new SecondA();
  }

  @Provides
  C provideC(@Named(„first“) A a) { ... };
}
```

oder mit eigenem Qualifyer

```java
@Qualifier
@Retention(RUNTIME)
public @interface First {
}

@Qualifier
@Retention(RUNTIME)
public @interface Second {
}

@Provides
C provideC(@First A a) { ... };
```

## Module kennen Module

Module enthalten nur Rezepte zur Erzeugung und Initialisierung. Sie haben keinen Scope und müssen auch nicht vollständig definiert sein. Erst eine Component muss in sich vollständig sein. Dazu kann sie eine beliebige Anzahl von Modulen anziehen. Manchmal ist dies aber lästig und repetiv. Deswegen gibt es die Möglichkeit, dass ein Modul andere Module inkludiert und deren Know-How weiter gibt. Aber keine Sorge, irgend so etwas kompliziertes wie Namensräume oder Sichtbarkeiten werden dabei nicht eingeführt. Es ist rein ein Organisationsfeature.

```java
@Module(includes = { MyModule2.class })
class MyModule1 {
}
```

## Scopes



## Geschachtelte Scopes


