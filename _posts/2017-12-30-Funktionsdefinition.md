---
published: false
---

Funktionsdefinition
-----------------------

Eine Funktiondefinition muss erst den Parametertyp und dann den Typ des Rückgabewerts angeben. Danach folgt die eigentliche Funktionslogik.

    f :: String -> Int
    f s = length s
  

Funktionsaufruf
------------------
  
Eine Funktion wird einfach wie folgt aufgerufen. Eine Klammer um den Parameter ist weder notwendig noch üblich.

    y = f "Hello World!"
      

Generische Funktionsdefinition
---------------------------------

Natürlich kann ich für jeden möglichen Typ eine Funktion (hier die Identität) schreiben.

    idI :: Int -> Int
    idI v = v
  
aber auch
  
    idS :: String -> String
    idS v = v
  
Allerdings kann man sich die Arbeit sparen, in dem man generische Funktionen schreibt. Hier ist der Ein- und Ausgabetyp durch einen nicht näher spezifizierten Platzhalter 'a' ersetzt. So wie hier definiert, funktioniert die Funktion 'idA' mit jedem Typ. Sie garantiert aber, dass der Ausgabetyp mit dem Eingabetyp identisch ist. Natürlich gibt es Möglichkeiten die Eigenschaften des generischen Platzhalters näher zu beschreiben. Das aber später.
  
    idA :: a -> a
    idA v = v
  
  
Partielle Funktionsdefinition
--------------------------------
  
    printBool :: Bool -> String
    printBool True = "Wahr"
    printBool False = "Falsch"
   

Partielle Funktionsdefinition mit Guards
-------------------------------------------
 
    printOdd :: Int -> String
    printOdd x | (even x) = "Gerade"
    printOdd x | (odd x) = "Ungerade"


Kombinieren von Funktionsaufrufen
------------------------------------
  
    odd (length (printBool True))
    
aber nicht
    
    odd length printBool False
    
