---
published: true
---

Komposition von Funktionen
-----------------------------
  
Jetzt wo wir gezeigt haben, dass man mit einer funktionalen Programmiersprache auch Funktionen mit mehreren Parametern abbilden kann, können wir eine besondere Funktion mit zwei Parametern einführen.
  
Ihre Definition ist:

```haskell
(.) :: (b -> c) -> (a -> b) -> (a -> c)
```
    
Sie nimmt also zwei Funktionen und kombiniert diese zu einer neuen Funktion.
  
Schauen wir uns das Beispiel von vorher an, bei dem wir mehrere Funktionen in Folge ausführen:
  

```haskell
odd (length (printBool True))
```
    
Das können wir jetzt auch anders machen:
  

```haskell
(odd . length . printBool) True
```
    
Wir können der kombinierten Funktion sogar einen Namen geben:
  

```haskell
oddLengthOfBool = odd . length . printBool

oddLengthOfBool True
```


Warum ist das wichtig? Weil funktionale Programmierung nichts anderes ist, als die Komposition von Funktionen. 
  
[[PREV]](/haskell/Funktionen-als-Werte) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Kontrollstrukturen)

