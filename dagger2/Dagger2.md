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

### Module über Injection

Informationen die in einem Module stehen haben für Dagger immer Vorrang vor den annotierten Informationen.

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

Bis jetzt wurden immer neue Instanzen erzeugt, wenn eine der Methoden des oder der Components aufgerufen wurde. Manchmal möchte man aber Singletons haben. Dazu gibt es eine Scope-Annotation `@Singleton`. Je nachdem ob man die Erzeugung per Annotation oder über ein Module macht, muss die Annotation `@Singleton` an einer anderen Stelle stehen.

Wichtig ist, dass die Eigenschaft *Singleton* nur für eine Component-Instanz gilt. Unterschiedliche Instanzen einer Component haben neue Singletons:

```java
@Component
@Singleton
interface MyComponent {
  ASingleton getASingleton();
  ANormalObject getANormalObject();
}

MyComponent component1 = DaggerMyComponent.builder().build();
MyComponent component2 = DaggerMyComponent.builder().build();

assert(component1.getANormalObject() != component1.getANormalObject());
assert(component1.getASingleton() == component1.getASingleton());
assert(component1.getASingleton() != component2.getASingleton());
```

Wenn man also ein systemweites Singleton haben will, muss man dafür sorgen, dass man immer mit der selben Instanz der Component arbeitet. Später mehr dazu.

Um Singletons zurückgeben zu können, muss die Component entsprechend annotiert werden. Das bedeutet jetzt nicht, dass die Component ausschließlich Singletons zurück gibt, sondern eben auch. Welches Objekt dabei ein Singleton ist und welches nicht, wird an anderer Stelle bestimmt.


### Erzeugung per Annotation

Wenn die Erzeugung über einen mit `@Inject` annotierten Konstruktor geschieht, muss man die Klasse mit `@Singleton` annotieren:

```java
@Singleton
class ASingleton {
  @Inject ASingleton() {};
}
```

### Erzeugung per Module

Wenn die Erzeugung über ein Modul geschieht, muss die Provider-Methode mit `@Singleton`annotiert werden:

```java
class MyModule {
  @Provide
  @Singleton
  ASingleton provideASingleton() { ... };

  @Provide
  ANormalObject provideANormalObject() { ... };
}
```

### Eigene Scope-Annotationen

Man kann auch eigene Scope-Annotationen erstellen. Diese haben aber exakt die gleiche Bedeutung wie @Singleton, können aber zur besseren Dokumentation verwendet werden. Warum wird im nächsten Abschnitt deutlich.

```java
@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface SystemSingleton {
}


@Scope
@Retention(RetentionPolicy.RUNTIME)
public @interface TenantSingleton {
}
```

Pro Component kann aber nur eine Annotation verwendet werden. Pro Modul können zwar unterschiedliche Annotationen verwendet werden, diese dürfen aber nicht bei der selben Component in Verwendung kommen.


## Geschachtelte Scopes

Jede halbwegs komplexe Serverapplikation hat mehrere Scopes. Jeder Scope bestimmt die Lebensdauer eines Objekts. Die Scopes sind in der Regel hierarchisch aufgebaut. Hier ein Beispiel:

- @SystemSingleton
  Das Objekt wird vom ganzen System verwendet und lebt so lange der Prozess lebt.
- @TenantSingleton
  Das Objekt lebt im Kontext eines Tenant/Mandanten und ist nur für diesen gültig. Es lebt und stirbt mit dem Mandant. Ein Tenant-Scoped Objekt kann natürlich alle System-Scoped Objekte verwenden.
- @SessionSingleton
  Das Objekt lebt im Kontext einer User-Session und lebt und stirbt mit der Session. Ein Session-Scoped Objekt kann alle Tenant-Scoped und System-Scoped Objekte verwenden.
- @RequestSingleton
  Das Objekt lebt im Kontext eines Server-Aufrufs. Ein Request-Scoped Objekt kann alle Session-Scoped, Tenant-Scoped und System-Scoped Objekte verwenden.

Da jede Applikation anders ist und jede Programminfrastruktur anders ist, unterstütz Dagger solche geschachtelten Scopes mit Objektlebensdauer und Zugriffshierarchie nicht direkt, sondern gibt einem nur die Werkzeuge in die Hand, um leicht selber etwas passendes bauen zu können.

Das Werkzeug der Wahl ist dabei Component-Dependencies. In unserem Beispiel sieht das wie folgt aus:

```java
@SystemSingleton
@Component
interface SystemServices {
}

@TenantSingleton
@Component(dependencies = { SystemServices.class })
interface TenantServices {
}

@SessionSingleton
@Component(dependencies = { TenantServices.class })
interface SessionServices {
}

@RequestSingleton
@Component(dependencies = { SessionServices.class })
interface RequestServices {
}
```

Die Komponenten werden dann wie folgt erzeugt:

```java
SystemServices systemServices = DaggerSystemServices.builder().build();

TenantServices tenantServices = DaggerTenantServices.builder().systemServices(systemServices).build();

SessionServices sessionServices = DaggerSessionServices.builder().tenantServices(tenantServices).build();

RequestServices requestServices = DaggerRequestServices.builder().sessionServices(sessionServices).build();
```

## Kontext des Scopes den Objekten mitgeben

Irgendwie brauchen die Objekte bzw. Services der Scopes passende Kontextinformationen, damit sie z.B. den Reseller oder den User oder die Session-Locale kennen. Dies ist der Scope-Kontext, der fast allen in einem Scope erzeugten Objekten mitgegeben werden muss.

### über die Module

Die einfachste, aber nicht bevorzugte Methode, ist die über Module. Man gibt dem Modul einfach den Kontext als Fields mit:

```java
@Module
class TenantContextModule {
  TenantId tenantId;

  TenantContextModule(TenantId tenantId) { ... };

  @Provide
  @TenantSingleton
  ATenantSingletonObject provideATenantSingletonObject(ADependency aDependency) {
    return new ATenantSingletonObject(this.tenantId, aDependency);
  }
}

@Component(modules = { TenantContextModule.class }, dependencies = { SystemServices.class })
@TenantSingleton
interface TenantServices {
  ATenantSingletonObject getATenantSingletonObject();
} 
```

Allerdings muss die Component dann anders erzeugt werden, da sie das Modul nicht selbst erzeugen kann:

```java
TenantServices ts = DaggerTenantServices.builder().systemServices(systemServices)
                                                  .tenantContextModule(new TenantContextModule(aTenantId))
                                                  .build();
```


### über die Component

Der von Dagger bevorzugte Weg geht direkt über die Component. Dazu muss man zwar der Component einen eigenen Builder verpassen, das hat aber den Vorteil, dass der Kontext wie jedes andere Objekt injected werden kann und so auch von per Annotation erzeugten Objekten verwendet werden kann.

```java
@Module
class TenantContextModule {
  @Provide
  @TenantSingleton
  ATenantSingletonObject provideATenantSingletonObject(TenantId tenantId, ADependency aDependency) {
    return new ATenantSingletonObject(this.tenantId, aDependency);
  }
}


@Component(modules = { TenantContextModule.class }, dependencies = { SystemServices.class })
@TenantSingleton
interface TenantServices {
  ATenantSingletonObject getATenantSingletonObject();

  @Component.Builder
  interface Builder {
    @BindsInstance Builder tenantId(TenantId tenantId);
    TenantServices build();
  }
} 
```

Dagger generiert dann den notwendigen Code, damit die Component wie folgt erzeugt werden kann:

```java
TenantServices ts = DaggerTenantServices.builder().systemServices(systemServices)
                                                  .tenantId(aTenantId)
                                                  .build();
```


## Instanzen von Components entsprechend des Lebenszykluses speichern

Wenn die gewählte Server- oder Programmstruktur keinen nativen Ort für die erzeugten Component-Instanzen bietet, müssen wir diese selber speichern. Eine Möglichkeit dafür ist das Locator-Pattern:

```java

public final class SystemServicesLocator {
  private static final SystemServices systemServices = DaggerSystemServices.builder().build();

  public static SystemServices get() {
    return systemServices;
  }
}

```

Alle kontextbehafteten Instanzen können in einer Map abgelegt werden, die von der jeweils höheren Instanz geliefert wird:
 
```java

@Singleton
public final class TenantServicesLocator {
  private final Map<TenantId, TenantServices> services = new ConcurrentHashMap<>();
  
  SystemServices systemServices;
  
  public TenantServices get(TenantId id) {
    return services.computIfAbsent(id, tenantId -> DaggerTenantServices.builder()
                                                                       .systemServices(systemServices)
                                                                       .tenantId(tenantId)
                                                                       .build());
  }

  public void drop(TenandId id) {
    services.remove(id);
  }
}

```

