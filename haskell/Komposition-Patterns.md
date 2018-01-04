---
layout: page
published: true
---

Komposition bei Functor, Applicative und Monad
-------------------------------------------------

### Functor

Wir gehen noch einmal zurück zu den Functors. Unser Beispiel sah wie folgt aus:

```haskell
data Punkt = Punkt Int Int

mirrorX :: Punkt -> Punkt
mirrorY :: Punkt -> Punkt

p1 = Punkt 3 4
mp1 = Just p1
```

Die Anwendung von einer Funktion auf den Punkt sah wie folgt aus:

```haskell
mirrorX p1
```

bzw.

```haskell
mirrorX <$> mp1
```

Wenn ich jetzt in beide Richtungen spiegeln möchte, dann kann ich das ohne das `Maybe` wie folgt machen:

```haskell
mirrorY (mirrorX p1)
```

Um mir die Klammerung zu ersparen, gibt es eine Funktion `$` in Haskell, die ausdrückt, dass erst rechts von `$` alles ausgewertet werden soll, bevor es der Funktion links vom `$` übergeben wird. Damit ist folgende Schreibweise möglich:

```haskell
mirrorY $ mirrorX p1
```

oder auf die Spitze getrieben

```haskell
mirrorY $ mirrorX $ p1
```

Jetzt sehen wir auch, warum die Infix-Variante der Functor-Funktion `<$>` heißt, denn der entsprechende Code für den `Maybe Punkt` sieht wie folgt aus:

```haskell
mirrorY <$> mirrorX <$> mp1
```

Es gibt ja aber noch die Funktionskomposition mit der Funktion `.`

```haskell
(mirrorY . mirrorX) $ p1
```

Wenn wir das mit einem Functor probieren, sehen wir, dass das genau so klappt:

```haskell
(mirrorY . mirrorX) <$> mp1
```

Dass der Punkt im `Maybe` genau wie der normale Punkt an beiden Achsen gespiegelt wurde, ist aber nicht selbstverständlich. Ich kann mir ohne Mühe Implementierungen von `<$>` vorstellen, die das nicht gewährleisten. Allerdings ist es eine Regel des Patterns `Functor`. Alles was sich `Functor` nennen will, muss dies sicherstellen.


### Applicative

Wie sieht das jetzt bei `Applicative` aus? 

Nun, der steuert zur Komposition nichts wirklich interessantes bei, da er nur die Applikation von Parametern übernimmt.


### Monad

Und bei Monad?

Beim Functor haben wir mit `mirrorX` und `mirrorY` gearbeitet. Probieren wir das mit einer `Maybe` Monade und `realMirrorX` und `realMirrorY`.

```haskell
mp1 >>= realMirrorX >>= realMirrorY
```

Also das klappt schon mal ganz intuitiv. Die Richtung ist zwar eine andere als beim `Functor`, aber das ist nur Kosmetik.

Was im Gegensatz zum `Functor` bei der Monade nicht geht ist

```haskell
mp1 >>= (realMirrorX . realMirrorY)
```

Weil Ein- und Ausgabetypen nicht passen, muss es zur Kombination von solchen Funktionen eine eigene Funktion passend zur Monade geben. Zufälligerweise gibt es die (Kleisli arrow):

```haskell
mp1 >>= (realMirrorX >=> realMirrorY)
```


[[PREV]](/haskell/Patterns-Monad) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Patterns-Do)

