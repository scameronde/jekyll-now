---
published: true
---
Funktionen als Werte
-----------------------

Jetzt kommt was abgefahrenes. Funktionen als Parameter und Rückgabewerte. Eine unserer ersten Funktionen war

```haskell
idA :: a -> a
idA v = v
```

und wir haben bisher nur 'normale' Werte übergeben und zurück gegeben. Was aber passiert, wenn wir folgendes machen?

```haskell
(idA addP) (Paar 5 6)
```
  
Das Ergebnis ist das gleiche wie bei

```haskell
addP (Paar 5 6)
```
  
D.h. die Funktion idA kann auch eine Funktion als Parameter und als Rückgabewert haben.

Wenn wir keine generische Funktion, sondern eine nur mit Funktionen arbeitende Variante von idA erstellen wollten, hätte sie folgende Definition:

```haskell
idF :: (a -> b) -> (a -> b)
idF f = f
```


Funktionen mit mehreren Parametern
-------------------------------------

Können wir damit auch etwas sinnvolleres anfangen? Wie währe es mit folgender Funktion?
  
```haskell
evenOrOdd :: Bool -> (Int -> Bool)
evenOrOdd True  = even
evenOrOdd False = odd
    
(evenOrOdd True)  5
(evenOrOdd False) 5
```
    
`evenOrOdd` gibt je nach dem übergebenen Wert eine Funktion zurück, die auf den Wert 5 angewendet wird.
  
Was passiert eigentlich, wenn wir in der Ausführung die Klammern weg lassen?

```haskell
evenOrOdd True 5
evenOrOdd False 5
```
    
Es klappt! Wir haben - irgendwie - eine Funktion mit zwei Parametern. Im Übrigen können wir bei der Funktionsdefinition die Klammern ebenfalls weg lassen:
  
```haskell
evenOrOdd :: Bool -> Int -> Bool
evenOrOdd True  v = even v
evenOrOdd False v = odd v
```
  

Mit dem Wissen bewaffnet können wir auch eine 'normale' add Funktion schreiben:
  
```haskell
add :: Int -> Int -> Int
add a b = a + b
```
    
Die Anwendung mit zwei Parametern ergibt dann ein Int:
  
```haskell
eleven = add 5 6
```
    
Und die Anwendung mit nur einem Parameter eine Funktion:
  
```haskell
add5 = add 5
twelve = add5 7
```

[PREV](/haskell/Datentypen) [NEXT](/haskell/Komposition-Funktionen)
