---
published: true
---

Funktionsdefinition
-----------------------

Eine Funktiondefinition muss erst den Parametertyp und dann den Typ des Rückgabewerts angeben. Danach folgt die eigentliche Funktionslogik.

```haskell
f :: String -> Int
f s = length s
```

Funktionsaufruf
------------------
  
Eine Funktion wird einfach wie folgt aufgerufen. Eine Klammer um den Parameter ist weder notwendig noch üblich.

```haskell
y = f "Hello World!"
```      


Generische Funktionsdefinition
---------------------------------

Natürlich kann ich für jeden möglichen Typ eine Funktion (hier die Identität) schreiben.

```haskell
idI :: Int -> Int
idI v = v
```
  
aber auch

```haskell
idS :: String -> String
idS v = v
```

Man sich die Arbeit aber auch sparen, in dem man generische Funktionen schreibt. Hier ist der Ein- und Ausgabetyp durch einen nicht näher spezifizierten Platzhalter 'a' ersetzt. So wie hier definiert, funktioniert die Funktion 'idA' mit jedem Typ. Sie garantiert aber, dass der Ausgabetyp mit dem Eingabetyp identisch ist. Natürlich gibt es Möglichkeiten die Eigenschaften des generischen Platzhalters näher zu beschreiben. Das aber später.

```haskell
idA :: a -> a
idA v = v
``` 
  
  
Partielle Funktionsdefinition
--------------------------------

Funktionen können partiell, also nur für einen bestimmten Teil der Eingabemenge definiert werden. Man kann, muss aber nicht, auf die vollständige Abdeckung der Eingabemenge achten.

```haskell
printBool :: Bool -> String
printBool True = "Wahr"
printBool False = "Falsch"
```   


Partielle Funktionsdefinition mit Guards
-------------------------------------------

Die partielle Funktionsdefinition war im obigen Beispiel noch einfach durch reine Aufzählung zu erreichen. Bei größeren Mengen geht dies nicht mehr. Stattdessen kann man dies über die Beschreibung der Eigenschaft der Elemente erreichen.

```haskell
printOdd :: Int -> String
printOdd x | (even x) = "Gerade"
printOdd x | (odd x) = "Ungerade"
```


Kombinieren von Funktionsaufrufen
------------------------------------

Ein Funktionsaufruf alleine macht noch kein Programm. Funktionen müssen nacheinander aufgerufen werden. Dabei ist die korrekte Klammerung zu beachten, sonst werden die falschen Dinge als Parameter betrachtet.

Richtig:
```haskell
odd (length (printBool True))
```

Falsch:
```haskell
odd length printBool False
```

[[PREV]](/haskell/fp-vs-oo) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Datentypen)
