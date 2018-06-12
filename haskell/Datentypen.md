---
layout: page
published: true
---
## Datentypen / Mengen

Definition von Mengen (data) und Zugriff auf Teile von Werten (Pattern Matching)
-----------------------------------------------------------------------------------
  
Wie kann ich eine simple Funktion wie Addition schreiben, wenn ich nur einen Parameter an eine Funktion übergeben kann? 

Wir müssen wohl beide Summanden irgendwie in einen Datentyp verpacken, der beide Werte halten kann und gegenüber der Funktion als ein Parameter auftritt. Ein Tupel, eine Liste, ein Record, ein Objekt, irgend etwas halt.

Wie geht das in Haskell? In Haskell sind Typen einfach Datenmengen. Eine Datenmenge kann ich am einfachsten durch Aufzählung der Elemente festlegen:

```haskell
data Aufzaehlung = Eins | Zwei | Drei | Viele
```
    
So einfach wie das ist, so mühsam kann das sein. Deswegen kann man Elemente auch parametrisieren.

```haskell
data Aufzaehlung' = Eins' | Zwei' | Drei' | Viele' Int
```

Gültige Elemente von Aufzaehlung' sind also z.B. `Zwei'`, `Viele' 5` und `Viele' -20`

Es muss aber nicht bei einem Parameter bleiben. Es sind beliebig viele möglich:

```haskell
data Paar = Paar Int Int
```

Wie bei Funktionen kann das auch generisch erfolgen:

```haskell
data Paar' a = Paar' a a
```

Oder sogar mit unterschiedlichen generischen Typen:

```haskell
data Paar'' a b = Paar'' a b
```

Jetzt kann ich eine Addition abbilden

```haskell
addP :: Paar'' Int Int -> Int
addP (Paar'' x y) = x + y

addedP = addP (Paar'' 5 6)
```

Die Art den Parameter in der Funktionsdefinition zu schreiben ist vergleichbar mit den partiellen Funktionsdefinitionen und nutzt ein Sprachfeature, dass sich Pattern Matching nennt. So kommt man also an den "Inhalt" eines parametrisierten Wertes.

[[PREV]](/haskell/Funktionen) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Funktionen-als-Werte)

