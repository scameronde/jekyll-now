---
layout: page
published: true
---

Ein- und Ausgabe
-------------------

Wenn wir auf die purity verzichten und in Haskell in Kauf nehmen, dass wir - analog F# - Funktionen mit Seiteneffekten haben, dann hätten wir trotzdem ein Problem:

```haskell
a = [ putStrLn "Hello", putStrLn "World!" ]
```

Wie schon im Kapitel über lazy evaluation erklärt, ist weder klar ob und wenn ja, in welcher Reihenfolge die Seiteneffekte ausgelöst werden.

Die `do`-Notation der Monade sieht im Grunde schon mal wie eine strikte Ausführung aus, ist es aber nicht, wie folgende Umformung zeigt:

```haskell
do
  putStrLn "Hello"
  putStrLn "World!"
```

ist äquivalent zu

```haskell
(putStrLn "Hello") >> (putStrLn "World!")
```

Um die Umformung zu verstehen, müssen wir uns die Signaturen von `putStrLn` und `>>` ansehen.

```haskell
putStrLn :: String -> IO ()
(>>)     :: Monad m => m a -> m b -> m b
```

`putStrLn` nimmt einen String und erzeugt einen Wert des Typs `IO`, getypt mit "boring". `(>>)` nimmt zwei Monaden und verknüpft diese, in dem es die erste scheibar wegwirft und die zweite zurück gibt. Ich sage hier mit Absicht "scheinbar", da die Signatur NICHTS darüber sagt, dass der zweite Parameter zurückgegeben wird. Es wird nur gesagt, dass der Rückgabewert vom selben Typ wie der zweite Parameter ist.

Ohne jede weitere Annahme - nur mit den Signaturen bewaffnet - wird deutlich, dass die Reihenfolge in der die beiden `putStrLn` Seiteneffekte ausgeführt werden nicht bestimmbar ist.

Erweitern wir das Beispiel um eine Eingabe:

```haskell
do
  hello <- getLine
  putStrLn hello
  world <- getLine
  putStrLn world
```

ist äquivalent zu

```haskell
getLine >>= (\hello -> (putStrLn hello) >> getLine >>= (\world -> (putStrLn world)))
```

Hier sehen wir, dass zumindest die erste Ausgabe von der ersten Eingabe abhängig ist. Allerdings kann die zweite Eingabe durchaus vor der ersten Ausgabe erfolgen. Was wir also erreichen müssen, ist dass die Reihenfolge immer gewährleistet ist. Und das gelingt uns in nicht-strikten Systemen ausschließlich durch den Datenfluss (Erinnerung: Datenfluss ist gleich Kontrollfluss).

Und genau das stellt die `IO` Monade in ihren Implementierungen der Funktionen `(>>=)`, `(>>)` und `(>=>)` sicher. Wie genau das geschieht sind Interna der jeweiligen Haskell Implementierung, es könnte aber z.B. durch einen Zähler geschehen, der aus dem jeweils linken `IO` Wert gelesen, hochgezählt, mit dem rechten `IO` Wert kombiniert und in dem Ergebnis `IO` Wert abgelegt wird. Jedenfalls lässt sich so sicherstellen, dass immer erst die linke Seite und dann die rechte Seite der Funktionen `(>>=)`, `(>>)` und `(>=>)` evaluiert wird.

Damit haben wir schon einmal zwei der drei Behauptungen des letzten Kapitels bewiesen:

- nicht-pure Aspekte sind von puren Aspekten unterscheidbar: wenn alles was Seiteneffekte verursacht, und dementsprechend eine fixen Reihenfolge benötigt, über die `IO` Monade abgewickelt wird, dann haben wir unseren Marker für nicht-puren Code.
- das Problem mit der nicht-strikten Ausführung ist gelöst

Jetzt geht es nur noch um die Frage, ob das ganze durch die `IO` Monade wieder pur wird, oder nicht. Ganz ehrlich, dass ist jetz Philosophie und ist für die tägliche Nutzung komplett irrelevant. Aber rein der Signatur der Funktionen nach, sind `putStrLn`, `getLine` und `(>>=)`, `(>>)`, `(>=>)` pur. Die nicht puren Aspekte finden innerhalb der `IO` Monade statt. Aber wie gesagt, aus praktischer Sicht ist das so ziemlich egal.


[[PREV]](/haskell/Seiteneffekte) [[TOP]](/haskell/Preface)

