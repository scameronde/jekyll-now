---
published: true
---

Functor
---------

Jetzt nehmen wir an, unsere Punkte seien optional. Es kann also sein, dass es einen Punkt gibt, oder nicht. In objektorientierten Sprachen drückt man das häufig (glücklicherweise wird das auch immer seltener) durch den Wert 'null' aus. Einen solchen, allgemein gültigen Wert, der das Nichtvorhandensein ausdrückt, gibt es in Haskell nicht. Er müsste für jede Menge definiert werden. Deswegen gibt es den Typ "Maybe", der diese Aufgabe erfüllt. Seine Definition ist:

```haskell
data Maybe a = Nothing | Just a
```
  
Wie müsste jetzt die Funktion `mirrorX` aussehen, wenn sie ein `Maybe Punkt` entgegen nehmen und wieder zurück liefern würde?

```haskell
mMirrorX :: Maybe Punkt -> Maybe Punkt
mMirrorX Nothing = Nothing
mMirrorX (Just (Punkt x y)) = Just (Punkt (-x) y)
```

Oder wenn man die vorhandene Funktion wiederverwenden möchte:

```haskell
mMirrorX' :: Maybe Punkt -> Maybe Punkt
mMirrorX' Nothing = Nothing
mMirrorX' (Just p) = Just (mirrorX p)
```

Wir sehen, dass wir mehr mit Fallunterscheidung, "auspacken" und "einpacken" beschäftigt sind, als mit der eigentlichen Logik.

Außerdem wandern wir in das Land des Boilerplates, wenn wir auch noch ein `mirrorY` einführen:

```haskell
mirrorY :: Punkt -> Punkt
mirrorY (Punkt x y) = Punkt x (-y)

mMirrorY' :: Maybe Punkt -> Maybe Punkt
mMirrorY' Nothing = Nothing
mMirrorY' (Just p) = Just (mirrorY p)
```

Die Funktionen `mMirrorX'` und `mMirrorY'` sind nahezu identisch. Sie könnte, mit der konkret aufzurufenden Funktion `mirrorX` oder `mirrorY` als Parameter, generalisiert werden:

```haskell
maybeMap :: (Punkt -> Punkt) -> Maybe Punkt -> Maybe Punkt
maybeMap f Nothing = Nothing
maybeMap f (Just x) = Just (f x)

mMirrorX'' = maybeMap mirrorX
mMirrorY'' = maybeMap mirrorY
```

Und warum sollte man das auf Punkte reduzieren?

```haskell
maybeMap' :: (a -> b) -> Maybe a -> Maybe b
maybeMap' f Nothing = Nothing
maybeMap' f (Just x) = Just (f x)
```

Nachdem wir das für so ziemlich alles, was man in einem `Maybe´ verpacken kann, verallgemeinert haben, können wir uns die Frage stellen, ob das auch für andere Datentypen außer `Maybe` interessant sein könnte.

Die Antwort ist Ja, auch wenn die nächsten Beispiele vielleicht nicht auf den ersten Blick passen.

Nehmen wir eine Liste von Integern. Wir wollen jeden Integer auf 'even' testen. Das Ergebnis soll eine Liste von Bool sein. Wir müssten jetzt in unserer zu schreibenden Funktion über die Liste der Ints iterieren und dabei eine Ergebnisliste erstellen, oder wir schaffen uns eine Funktion, die das generalisiert. Versuchen wir es doch mal:
 
```haskell
listMap :: (a -> b) -> [a] -> [b]
listMap f la = iterate f la []
  where
    iterate :: (a -> b) -> [a] -> [b] -> [b]
    iterate f [] lb = lb
    iterate f (x : xs) lb = iterate f xs ((f x) : lb) 

intList = [1, 2, 3, 4]
boolList = listMap even intList
```

Da Strings auch nur Listen von Chars sind, kann ich das gleich für folgendes nutzen:

```haskell
helloWorld = "Hello World!"
upperHelloWorld = listMap toUpper helloWorld
```

Wenn wir suchen, werden wir mit Sicherheit jede Menge Datentypen finden, die so eine Funktion, die beliebige Funktionen auf den "Inhalt" der Datentypen anwendet, anbieten können.

Es ist zwar offensichtich, dass die Implementierung je nach Datentyp unterschiedlich ist, die Signatur ist aber - könnte man sie genügend generalisieren - identisch. In objektorientierten Sprachen währe der Zeitpunkt gekommen ein Interface mit einer entsprechenden Methode, nennen wir sie mal `fmap`, einzuführen.

Haskell kann das auch. Allerdings geht man hier kein Interface, sondern eine Klasse von Datentypen definieren. Alle Datentypen einer Klasse von Datentypen müssen eine vorgegebene Struktur erfüllen und Implementierungen für vorgegebene Funktionen beisteuern. Das ganze sieht dann wie folgt aus:

```haskell
class MyFunctor mf where
  myMap :: (a -> b) -> mf a -> mf b

instance MyFunctor Maybe where
  myMap f Nothing = Nothing
  myMap f (Just x) = Just (f x)
  
instance MyFunctor [] where
  myMap f la = iterate f la [] where
    iterate :: (a -> b) -> [a] -> [b] -> [b]
    iterate f [] lb = lb
    iterate f (x : xs) lb = iterate f xs ((f x) : lb) 
```
    
Natürlich gibt es in den Standardbibliotheken schon eine entsprechende Implementierung des Patterns "Functor" mit folgender Definition:

```haskell
class Functor mf where
  fmap :: (a -> b) -> mf a -> mf b
  (<$>) = fmap
```
  
Die Infix-Variante von fmap ist besonders interessant, weil sie folgende Schreibweise erlaubt, die schon mehr an einen klassischen Funktionsaufruf erinnert:

```haskell
upperHelloWorld' = toUpper <$> helloWorld
```

[[PREV]](/haskell/Patterns) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Patterns-Applicative)
