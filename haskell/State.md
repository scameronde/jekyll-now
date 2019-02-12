---
layout: page
published: false
---

# State

In FP gibt es keinen State. Trotzdem kann es sinnvoll sein, Teile der Werte einer Berechnung als State zu betrachten.

Beispiel: ich möchte die Summe der Quadrate der ersten n Zahlen berechnen. Das Summieren erfolgt schrittweise. Die aktuelle Zahl x aus 1..n ist bei der Berechnung der State. Die Berechnungsvorschrift könnte wie folgt aussehen:

- lese den Wert aus dem State
- erhöhe den Wert des States um +1
- berechne das Quadrat des Wertes
- mache das n mal und addiere anschließend alle erhaltenen Quadrate


Für die Berechnung - und allgemein für den Umgang mit State - haben wir also:

- einen Anfangsstate
- eine Berechnung, die ein Ergebnis aus dem State erzeugt
- eine Berechnung die einen neuen State erzeugt

Das kann ich natürlich alles per Hand verlöten, ich kann aber auch ein Pattern verwenden, damit der Code in allen Fällen ähnlich und dadurch wiedererkennbar aussieht. Muster helfen durch Abstraktion beim Verständnis.

Eine per Hand verlötete Lösung könnte wie folgt aussehen:

```haskell
type State = Int
type Wert  = Int

nextState :: State -> State
nextState s = s+1

calcResult :: State -> Wert
calcResult s = s * s

squareSum :: Int -> State -> Wert
squareSum 1 state = calcResult state
squareSum numSteps state = calcResult state + squareSum (numSteps-1) (nextState state)

initialState :: State
initialState = 1

value :: Wert
value = squareSum 5 initialState
```

Sie klappt, es sieht aber nicht sehr nach unserer Berechnungsvorschrift aus. Und die Tatsache, dass einer der Werte einen State darstellen soll wird auch nicht ersichtlich. Zusätzlich fehlt mir noch ein einfacher Weg den State nach der Berechnung weiter zu verwenden.

Glücklicherweise haben wir die State Monade als Pattern, die uns hilft das ganze expressiver formulieren zu können. Hier erst mal die Anwendung:

```haskell
import Control.Monad.State

type Count = Int
type Wert = Int

nextState :: Count -> Count
nextState s = s + 1

calcResult :: Count -> Wert
calcResult s = s * s

squareCounter :: State Count Wert
squareCounter = do
                  s <- get
                  put (nextState s)
                  return (calcResult s)

squareSum :: Int -> Count -> Wert
squareSum 1 c = evalState squareCounter c
squareSum numSteps c = evalState squareCounter c + squareSum (numSteps-1) (execState squareCounter c)

initialState :: Count
initialState = 1

value :: Wert
value = squareSum 5 initialState

-- oder

squareSum' :: Int -> Count -> Wert
squareSum' numSteps c = sum (evalState (sequence (replicate numSteps squareCounter)) c)

value' :: Wert
value' = squareSum' 5 initialState

```
