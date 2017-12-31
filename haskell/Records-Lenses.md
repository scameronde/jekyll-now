---
published: true
---
Definition von Mengen über Record-Syntax
-------------------------------------------

Der Umgang mit solchen Werten kann bei vielen Parametern etwas mühsam sein. Als erstes ist die Reihenfolge der Parameter wichtig und deren Bedeutung nicht immer sichtbar. Außerdem muss der Zugriff auf die Parameter programmiert werden:

```haskell
getA (Paar a _) = a
getB (Paar _ b) = b
```

Der Compiler kennt deswegen eine Abkürzung:

```haskell
data RPaar atype btype = RPaar { a :: atype, b :: btype}
```
  
Neben der normalen Art einen Wert zu erzeugen

```haskell
rpaar1 = RPaar 5 6
```
  
kann zur besseren Dokumentation die Record-Schreibweise genutzt werden. Dann ist auch die Reihenfolge der Paremeter egal:

```haskell
rpaar2 = RPaar { a = 5, b = 6 }
rpaar3 = RPaar { b = 6, a = 5 }
```

Außerdem werden Accessor-Funktionen für die Parameter erzeugt:
  
```haskell
a1 = a rpaar1
a2 = a rpaar2
a3 = a rpaar3
```

Zugriff auf Records über Lenses
----------------------------------

tbd.

[[TOP]](/haskell/Preface) 
