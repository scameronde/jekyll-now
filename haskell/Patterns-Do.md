---
layout: page
published: true
---

Do-Notation bei Monaden
---------------------------

Um die Do-Notion bei Monaden erklären zu können, benötige ich ein etwas komplexeres Beispiel. Die Do-Notation hat nämlich den Zweck, durch syntaktischen Zucker, schwierig zu lesenden mit Monaden arbeitenden Code etwas lesbarer zu gestalten. 

Um den Einstieg in das Beispiel etwas leichter zu machen, arbeiten wir erst einmal mit `Functor` und `Applicative` und steigen dann auf `Monad` um.

Nehmen wir folgenden Code:

```haskell
mp1 = Just (Punkt 3 4)
mp2 = Just (Punkt 0 2)
mp3 = Just (Punkt 2 0)

getX :: Punkt -> Int
getX (Punkt x _) = x

getY :: Punkt -> Int
getY (Punkt _ y) = y
```

Wenn wir jetzt - nur mit diesen Vorgaben ausgerüstet - aus einem `Maybe Punkt` ein `Maybe (Int, Int)` machen wollen, also nur ein Tuple mit den Koordinaten möchten, dann ist das mit Hilfe von `Applicative` einfach zu erledigen. Wir brauchen nur eine Funktion, die aus zwei Parametern ein Tuple macht:

```haskell
tuple :: Int -> Int -> (Int, Int)
tuple x y = (x, y)
```

(Aufmerksame Leser werden bemerkt haben, dass die Funktion `(,)` das schon macht. Ich brauche hier aber die spezielle Variante um etwas zu verdeutlichen).

Unser Ziel erreichen wir dann wie folgt:

```haskell
tuple <$> (getX <$> mp1) <*> (getY <$> mp1)
```

Das ganze geht aber auch mit einem simplen `Functor` und benötigt kein `Applicative`, da beide Parameter auf dem selben Wert arbeiten. Schauen wir mal wie:

```haskell
(\p -> tuple (getX p) (getY p)) <$> mp1
```

Oh mein Gott, was haben wir da gemacht und wie? Als erstes haben wir eine sogenannte "anonyme Funktion" hingeschrieben. Diese hat einen Parameter `p` - der offensichtlich vom Typ `Punkt` zu seien scheint - und erzeugen dann ein Tuple aus `x` und `y`. Und diese anonyme Funktion übergeben wir dem `Functor`.

Klapp auch.

Ich möchte das jetzt noch weiter treiben, und die Funktion `tuple` ebenfalls durch eine anonyme Funktion ersetzen.

```haskell
(\p -> (\x y -> (x, y)) (getX p) (getY p)) <$> mp1
```

Und da sich jede Funktion mit zwei Parametern als eine Funktion mit einem Parameter, der wiederum eine Funktion zurückliefert, ausdrücken lässt, möchte ich das auch mit der anonymen Funktion machen:

```haskell
(\p -> (\x -> (\y -> (x, y))) (getX p) (getY p)) <$> mp1
```

Ich hoffe das Muster ist deutlich geworden. Ich übergebe dem `Functor` eine Funktion, die den im `Functor` gespeicherten Wert übergeben bekommt. Auf diesen Wert werden `getX` und `getY` losgelassen. Die daraus resultierenden Werte werden in eine Funktion gestecht, die daraus ein Tupel macht. Der `Functor` verpackt dann das Tupel wieder.


Jetzt kommt aber die große Frage: wie geht das, wenn wir das monadisch machen wollen, also Funktionen haben, die eine Monade zurückgeben?

Hier erst mal die beiden monadischen Funktionen:

```haskell
realGetX :: Punkt -> Maybe Int
realGetX (Punkt 0 _) = Nothing
realGetX (Punkt x _) = Just x

realGetY :: Punkt -> Maybe Int
realGetY (Punkt _ 0) = Nothing
realGetY (Punkt _ y) = Just y
```

Wir wollen, dass wenn `mp1` gleich `Nothing` ist, dass das Ergebnis `Nothing` ist. Ebenso soll, wenn `realGetX` und/oder `realGetY` `Nothing` zurück liefert das Ergebnis `Nothing` sein. Im Grunde ist der Aufbau analog des Aufbaus mit dem `Functor`. Da die `>>=` Funktion die Parameter in der anderen Reihenfolge erwartet, müssen wir diese nur umdrehen. 

```haskell
mp1 >>= (\p -> realGetX p >>= (\x -> realGetY p >>= (\y -> Just (x, y))))
```

Und da dieses Vorgehen in Haskell sehr oft vorkommt, haben sich die Compilerbauer etwas einfallen lassen. Allein durch syntaktische Umformungen (die ich hier nicht beschreiben will), kann man das analog wie folgt formulieren:

```haskell
do
  p <- mp1
  x <- realGetX p
  y <- realGetY p
  Just (x, y)
```

Und um das ganze noch auf die Spitze zu treiben, hat eine Monade ein Alias auf die Funktion `pure` des `Applicative` - die ja nur einen Wert in den umliegenden Typ wrapped, also quasi der Standardkonstruktor - mit dem Namen `return`. Dadurch kann man den Code wie folgt schreiben:

```haskell
do
  p <- mp1
  x <- realGetX p
  y <- realGetY p
  return (x, y)
```

Jetzt sieht das ganze fast aus wie imperative Programmierung. Allerdings sollte man sich davon nicht beirren lassen. `realGetX p` und `realGetY p` können weiterhin unabhängig voneinander und sogar parallel ausgewertet werden.

STOP! STOP! STOP! Wieso "unabhängig" und "parallel"? Da stehen doch Statements untereinander. Das heißt sie werden nacheinander ausgeführt! Dem ist in Haskell nicht so. Und weil das ganze extrem wichtig ist, hat es ein eigenes Kapitel verdient ([lazy evaluation](/haskell/Lazy)).

[[PREV]](/haskell/Komposition-Patterns) [[TOP]](/haskell/Preface)
