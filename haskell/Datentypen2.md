---
layout: page
published: true
---
## Datentypen / Mengen

Eine gute Programmiersprache sollte das Ändern von Daten (mutability) so schwer wie möglich, das Erzeugen von neuen Typen so einfach wie möglich machen.

Es gibt zwei Arten wie ein Datentyp in Haskell definiert werden kann:

1. über die Menge seiner Elemente (`data`, `type` und `newtype`)
2. über die Eigenschaften der Elemente (`class`)

Dabei müssen wir nicht komplett bei Null anfangen (auch wenn wir das könnten), sondern können uns auf eingebaute Datentypen stützen.

### Eingebaute Datentypen

- Bool --> True, False
- Char  --> Unicode
- String  --> Liste von Char
- Integer  --> beliebige Größe (wie in Python)
- Int  --> mindestens -2^29 bis 2^29
- Float  --> einfache Genauigkeit (4 Byte)
- Double  --> doppelte Genauigkeit (8 Byte)
- Unit
- List
- Tuple

### Typ-Aliase

- type MyChar = Char

Werte von unterschiedlichen Aliasen eines Typs sind untereinander kompatibel.

### Aufzählungstypen

#### normal



#### rekursiv

#### algorithmisch

### Kombinationstypen

### Typklassen




Definition von Mengen (data) und Zugriff auf Teile von Werten (Pattern Matching)
-----------------------------------------------------------------------------------

Wie kann ich eine simple Funktion wie Addition schreiben, wenn ich nur einen Parameter an eine Funktion übergeben kann? 

Wir müssen wohl beide Summanden irgendwie in einen Datentyp verpacken, der beide Werte halten kann und gegenüber der Funktion als ein Parameter auftritt. Ein Tupel, eine Liste, ein Record, ein Objekt, irgend etwas halt.

Wie geht das in Haskell? In Haskell sind Typen einfach Datenmengen. Und eine Datenmenge kann ich am einfachsten durch Aufzählung der Elemente festlegen:

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
addP :: Paar Int Int -> Int
addP (Paar x y) = x + y

addedP = addP (Paar 5 6)
```

Die Art den Parameter in der Funktionsdefinition zu schreiben ist vergleichbar mit den partiellen Funktionsdefinitionen und nutzt ein Sprachfeature, dass sich Pattern Matching nennt. So kommt man also an den "Inhalt" eines parametrisierten Wertes.

[[PREV]](/haskell/Funktionen) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Funktionen-als-Werte)

