---
layout: page
published: true
---

Monad
--------

Die bei `Functor` und `Applicative` verwendeten Funktionen haben an dem umgebenden Datentyp nichts verändert. Wenn zum Beispiel das `Maybe` ein `Just` war, dann blieb es nach der Anwendung der Funktion ein `Just`.

Was ist aber, wenn ich den Wert des umgebenden Datentyps beeinflussen möchte? Hier ein Beispiel:

```haskell
realMirrorX :: Punkt -> Maybe Punkt
realMirrorX p = if (x == 0) 
                  then Nothing 
                  else Just (mirrorX p) 
                    where (Punkt x _) = p
```

Es werden also nur die Punkte an der X-Achse gespiegelt, die nicht auf der X-Achse liegen. Alle Punkte die auf der X-Achse liegen werden eleminiert.

Eine solche Funktion würde für einen Functor nur wenig bringen. Wir hätten dann ein `Maybe` in einem `Maybe`. Wir brauchen also ein neues Pattern, welches die Anwendung einer solchen Funktion erlaubt. Also etwas in der Art von:

```haskell
class Applicative m => MyMonad m where
  myMMap :: (a -> m b) -> m a -> m b
```

Damit währe wir dann symmetrisch zu:

```haskell
class Functor f where
  (<$>) :: (a -> b) -> f a -> f b

class Functor f => Applicative f where
  (<*>) :: f (a -> b) -> f a -> f b
```

Und selbstverständlich gibt es so etwas in der Standardbibliothek. Allerdings sind bei der Mapping-Funktion die ersten beiden Parameter vertauscht. Warum? Es bricht zwar die Symmetrie zu `Functor` und `Applicative`, ermöglicht aber ein paar sinnvolle Konstrukte, die ich später noch zeigen werde.

```haskell
class Applicative m => Monad m where
  (>>=) :: m a -> (a -> m b) -> m b
```

Und da der Typ `Maybe` ganz zufälligerweise auch eine Monade ist, können wir folgendes schreiben:

```haskell
maybeMirroredPoint = Just (Punkt 0 3) >>= realMirrorX
```


[[PREV]](/haskell/Patterns-Applicative) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Komposition-Patterns)

