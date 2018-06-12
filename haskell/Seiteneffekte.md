---
layout: page
published: true
---


Seiteneffekte
-----------------

### Vorwort
Gleich vorneweg: für die Seiteneffekte bediene ich mich hemmungslos bei einem wunderparen Paper von Simon Peyton Jones mit dem Titel "Tackling the Awkward Squad: monadic input/output, concurrency, exceptions, and foreign-language calls in Haskell" vom 7. April 2010. Das Paper kann [hier](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/mark.pdf) gelesen werden.


### Jetzt aber ...

So ziemlich jedes erste Beispiel für eine Programmiersprache ist ein "Hello World!". Bis jetzt habe ich mich darum gedrückt.

Aber warum eigentlich? Kann doch nicht so schwer sein! Schauen wir doch mal:

```haskell
main = putStrLn "Hello, World!"
```

Na, war doch gar nicht schwer. Das ist ein Hello-World Programm in Haskell. Es kann in eine Datei `HelloWorld.hs` geschrieben werden, die mit dem Compiler in eine ausführbare Datei übersetzt werden kann (`ghc -o HelloWorld HelloWorld.hs`).

Das Problem ist nur, dass der obige Code in Haskell eigentlich unmöglich ist und gar nicht funktionieren dürfte.

Wir erinnern uns: Haskell ist pur funktional, d.h. es gibt keine Seiteneffekte. Das ist im Grunde genommen ein wenig peinlich, denn wenn es etwas gibt wofür wir Programme brauchen, dann ist es für die Interaktion mit der Umwelt. Wir wollen Werte von Außen bekommen, wir wollen Ausgaben auf die Konsole oder ein Fenstersystem machen, wir möchten Daten in eine Datenbank schreiben und Webservices ansprechen. All das sind aber Seiteneffekte, und eine pure funktionale Programmiersprache kann dies per Definition nicht.

Jetzt gibt es mehrere Wege damit umzugehen. Einer ist der, den Elm und erste Versionen von Haskell gegangen sind. Seiteneffekte werden konsequent nach "draußen" verlagert. D.h. dass das pure, funktionale Programm von einem umliegenden System in einer Art Endlosschleife immer wieder aufgerufen wird. Wenn das Programm einen Seiteneffekt wünscht, kehrt es mit einer entsprechenden Anfrage an das umliegende System zurück. Das umliegende System - meist ein C Programm - führt den Seiteneffekt aus und ruft das pure, funktionale Programm mit den Ergebnissen wieder auf. 

In Haskell könnte das wie folgt aussehen:

```haskell
main :: [Response] -> [Request]
```

Das Problem bei dem Ansatz ist, dass jeder neue Seiteneffekt (heute Konsolenausgabe, morgen SQL, übermorgen No-SQL) irgendwie in `Request` und `Response` kodiert werden muss und das umliegende System diese ausführen können muss. Im Zweifelsfall muss ich also eine neue Art von Seiteneffekt in C oder C++ programmieren und in Haskell `Response` und `Request` entsprechend erweitern.

Andere funktionale Sprachen - wie z.B. F# - pfeifen einfach auf die purity und leben damit, dass manche Funktionen impure sind und einen Seiteneffekt produzieren. Leider werfen sie damit auch alle Vorteile der purity über Bord. 

Man könnte das Problem aber auch damit lösen, dass man sein Programm - sichtbar für den Programmierer, den Compiler und andere Werkzeuge - in einen puren und einen nicht puren Teil aufteilt. 

In aktuellen Versionen von Haskell wird tatsächlich letzterer Weg beschritten, und zwar mit der Hilfe einer Monade, der `IO` Monade.

Dies hat einige Vorteile:
- pure und nicht pure Teile eines Programms können leicht erkannt werden
- man kann sich belügen in dem man sagt, dass die `IO` Monade die nicht puren Teile versteckt und das Programm dadurch wieder durchgehen pur ist.
- man kann das Haskell eigene Problem der nicht strikten Ausführung lösen

Was bedeutet das? Wie geht das? Am besten klären wir das im nächsten Kapitel am Beispiel von IO.

[[PREV]](/haskell/Lazy) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Seiteneffekte-IO)



