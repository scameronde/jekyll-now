---
layout: page
published: true
---

Haskell und Lazy Evaluation
--------------------------------

Es wird Zeit über eine Besonderheit von Haskell zu sprechen. Haskell ist nicht strikt. Was bedeutet das? Ich darf hier mal aus der Wikipedia zitieren:


> Lazy Evaluation (bequeme [faule] Auswertung) bezeichnet in der Informatik eine Art der Auswertung von Ausdrücken, bei der das Ergebnis des auszuwertenden Ausdrucks nur so weit berechnet wird, wie es gerade benötigt wird.
>
> Ein Vorteil einer solchen Auswertungsstrategie ist Zeitersparnis, da Funktionsaufrufe ganz vermieden oder zumindest teilweise eingespart werden können. Außerdem gibt Lazy Evaluation dem Programmierer die Möglichkeit, unendliche Datenstrukturen zu verwenden.

Das ist mal ganz anders als alles was ich bisher so gesehen habe. In allen mir bekannten imperativen Sprachen gilt eine strikte Ausführung, d.h. dass die Reihenfolge in der Statements oder Expressions aufgeführt werden auch die Ausführungsreihenfolge bestimmt und dass Funktionen direkt ausgeführt wertden. 

Soweit ich weiß ist Haskell die einzige "General Purpose Language" die nicht-strikt ist. Sogar andere funktionale Sprachen wie Elm, F# oder Lisp sind alle strikt.

Aber was bedeutet das konkret? Nehmen wir mal folgende Expression:

```haskell
a :: [Int]
a = [ length "Hello", lenght "World!" ]
```

In Haskell werden die Längen der Strings nicht berechnet. Jedenfalls solange nicht wie sie nicht benötigt werden. Wenn jetzt jemand später nur `length a` benötigt und nie auf die Elemente von `a` zugreift, dann werden die Längen der Strings nie berechnet.

Das ist jetzt erst einmal erstaunlich, stellt aber bisher kein Problem dar. Was interessiert mich denn, ob und wann eine Funktion aufgerufen wurde oder nicht, solange ich ihren Wert nicht benötige. Denn - wir erinnern uns - das Einzige was eine Funktion kann, ist einen Rückgabewert berechnen.

Dafür erlaubt "lazy evaluation" eine ganz andere Ausdrucksstärke bei Datentypen und Berechnungen. Nehmen wir uns mal folgendes Beispiel:

```haskell
infinite = [1..]

dividableBy3 :: [Int] -> [Int]
dividableBy3 [] = []
dividableBy3 (x : xs) | (x `rem` 3 == 0) = (x : dividableBy3 xs)
dividableBy3 (x : xs) | (x `rem` 3 /= 0) = dividableBy3 xs

numSummandsDividableBy3Until :: Int -> Int
numSummandsDividableBy3Until upper =
  calc 0 0 (dividableBy3 infinite)
  where
    calc numSummands sum _        | (sum >= upper) = numSummands
    calc numSummands sum (x : xs) | (sum <  upper) = calc (numSummands+1) (sum+x) xs

erg = numSummandsDividableBy3Until 100000
```

Fangen wir mit der ersten Ungeheuerlichkeit an. Der Wert `infinite` ist eine Liste aller ganzen Zahlen von `1` bis `2^29-1`. Wenn diese Liste tatsächlich befüllt würde, hätten wir unser erstes Speicherproblem. Aber selbst wenn wir eine clevere Lösung dafür hätten, würde uns spätestes beim Aufruf von `dividableBy3 infinite` der Speicher explodieren. 

Das alles kann nur funktionieren, weil Haskell faul ohne Ende ist. Haskell berechnet nur die Werte (führt nur die Funktionsaufrufe durch), die tatsächlich von jemandem benötigt werden. Und da unsere Funktion `numSummandsDividableBy3Until` recht früh terminiert (bei 100.000 als Grenze werden gerade mal 258 Summanden benötigt), werden von der riesigen Menge an Zahlen in `infinite` nur wenige benötigt und somit berechnet.

Bei einer strikten Evaluierung müssten wir entweder eine sinnvolle Obergrenze für `infinite` abschätzen, oder algorithmisch ganz anders vorgehen. Bei beidem würde die Ausdrucksstärke und Verständlichkeit leiden.

[[TOP]](/haskell/Preface)

