---
layout: page
published: true
---

Patterns zum vereinfachten Umgang mit Daten
----------------------------------------------

Da wir in der Funktionalen Programmierung alle, wirklich alle benötigten Daten an die Funktion übergeben müssen, haben wir es recht schnell mit komplexen, ineinander verschachtelten Datenstrukturen zu tun. Da diese auch noch immutable sind, ist es wichtig möglichst effizient mit ihnen arbeiten zu können. Deswegen haben sich - wie sollte es auch anders sein - ein paar immer wieder angewante Patterns herauskristallisiert, die ich im folgenden vorstellen möchte.  

Um die Patterns zu erklären, nehmen wir folgendes Beispiel:

```haskell
data Punkt = Punkt Int Int

mirrorX :: Punkt -> Punkt
mirrorX (Punkt x y) = Punkt (-x) y
```

[[PREV]](/haskell/Komposition-Funktionen) [[TOP]](/haskell/Preface) [[NEXT]](/haskell/Patterns-Functor)

