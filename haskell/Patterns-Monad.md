---
layout: page
published: true
---

Monad
--------

```haskell
realMirrorX :: Punkt -> Maybe Punkt
realMirrorX p = if (x == 0) then Nothing else Just (mirrorX p) where (Punkt x _) = p
```



Die do-Notation einführen, in dem ein Maybe Punkt in ein Maybe (x, y) transformiert
werden soll, aber nur wenn x und y positiv sind. Dazu wird die Funktion 
assertPositive :: Int -> Maybe Int genutzt


[[PREV]](/haskell/Patterns-Applicative) [[TOP]](/haskell/Preface)

