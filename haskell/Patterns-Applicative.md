---
layout: page
published: true
---

Applicative
-------------



Bisher haben wir nur Funktionen mit einem Parameter mit Hilfe der fmap Funktion auf den Functor losgelassen. Was aber passiert, wenn ich eine Funktion mit 2 Parametern nehme?

```haskell
gradient :: Punkt -> Punkt -> Float
gradient (Punkt x1 y1) (Punkt x2 y2) = fromIntegral (x2-x1) / fromIntegral (y2-y1)

whatIsThis = gradient <$> Just (Punkt 5 6)
```

Genau. `whatIsThis` ist jetzt vom Typ `Maybe (Punkt -> Float)`
    
D.h. im Maybe ist eine Funktion verpackt. Mit <$> bzw. fmap kann man aber nur eine Funktion auf einen verpackten Wert anwenden. Wir brauchen also noch das Gegenstück, welches eine verpackte Funktion auf einen verpackten Wert anwendet.

Dafür gibt es das Patter "Applicative" in der Standardbibliothek mit folgender Definition:
  
```haskell
class Functor f => Applicative f where
  <*> :: f (a -> b) -> f a -> f b
```
      
Wie wir sehen, ist ein Applicative auch immer ein Functor. Dadurch wird folgendes möglich:


```haskell
maybeGradient = gradient <$> Just (Punkt 5 6) <*> Just (Punkt 7 8)
```


Manchmal möchte man die Funktion schon ohne erste Applikation eines Parameters im Datentyp haben. Deswegen hat Applicative noch eine weitere Funktion:

```haskell
class Functor f => Applicative f where
  <*> :: f (a -> b) -> f a -> f b
  pure :: a -> f a  


maybeGradient' = pure gradient <*> Just (Punkt 5 6) <*> Just (Punkt 7 8)
```


Und warum sind die beiden Funktionen nicht schon Teil des Pattern "Functor"?
  
Weil es Functors gibt, für die die beiden Funktionen keinen Sinn ergibt. Ein Beispiel ist der Type String. Er ist zwar ein Functor, aber kein Applicative. Ein String ist zwar eine Liste, aber eben nur eine Liste von Char. Er kann nicht in eine Liste von Funktionen umgewandelt werden. Jedenfalls nicht als Functor.
  

[[PREV]](/haskell/Patterns-Functor) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Patterns-Monad)

