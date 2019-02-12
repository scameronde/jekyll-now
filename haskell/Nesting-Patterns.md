---
layout: page
published: true
---

Verschachtelung von Functor, Applicative und Monad
------------------------------------------------------

Was mache ich, wenn ich Patterns wie Functor, Applicative oder Monad ineinenader verschachtelt habe? Also z.B. ein

```haskell
list = [Just "Hello", Just "World!"]
```

Mir fallen drei Möglichkeiten ein:

```haskell
l1 = fmap (fmap length) list
```

```haskell
l2 = do
  e <- list
  pure (fmap length e)
```

```haskell
import Data.Functor.Compose

l3 = getCompose (fmap length (Compose list))
```

Irgendwie ist die `do`-Notation am hässlichsten. Das ich die Werte explizit aus- und wieder einpacke find ich kontraproduktiv. Genau dafür sind Functors ja da, dass ich mir das sparen kann. Die Verwendung von `Data.Functor.Compose` ist mit Sicherheit dann sinnvoll, wenn ich jede Menge Funktionen auf den Functor loslassen muss. Für eine einfache Anwendung sollte die erste Möglichkeit reichen. Da kann ich mir ja auch einen passenden Operator basteln:

```haskell
(.:) :: (Functor f1, Functor f2) => (a -> b) -> f2 (f1 a) -> f2 (f1 b)
(.:) = fmap . fmap
```

[[TOP]](/haskell/Preface)
