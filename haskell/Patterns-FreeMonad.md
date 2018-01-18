---
layout: page
published: false
---

Freie Monaden
-----------------

Durch die `IO` Monade und anderen Monaden bekommt man schnell den Eindruck, dass Monaden immer irgend etwas tun müssten, im wildesten Fall sogar etwas mit Seiteneffekten. Aber wieso sollte das so sein? Als allererstes ist eine Monade nichts weiter als ein Datentyp mit ein paar Funktionen und Regeln.

Rekapitulieren wir doch noch mal, was eine Monade ausmacht.

Als erstes ist eine Monade sowohl ein Functor als auch ein Applicative. D.h. dass folgende Funktionen definiert sein müssen:

```haskell
fmap  :: (a -> b) -> f a -> f b

pure  :: a -> f a 
(<*>) :: f (a -> b) -> f a -> f b
```

Die Monade bringt dann noch folgende Funktion mit:

```haskell
(>>=) :: m a -> (a -> m b) -> m b
```

Kann ich denn folgenden simplen Datentyp zu einem Functor, Applicative und Monad machen?

```haskell
data Test a = Test a deriving Show

instance Functor Test where
  fmap f (Test s) = Test (f s)

instance Applicative Test where
  pure = Test
  (<*>) (Test f) (Test s) = Test (f s)

instance Monad Test where
  (>>=) (Test s) f = f s
```

Diese Monade macht eigentlich überhaupt nichts außer eine Datenstruktur zu sein, erfüllt aber die Bedingungen für eine Monade. Somit ist sie die kleinste, denkbare Monade. Eben eine **"Freie Monade"**.

Aber was soll ich mit so einem Konstrukt? Warum mache ich mir überhaupt die Mühe den Typ `Test` zu einer Monade zu machen?

Als erste einmal ist es gut, Daten und Funktionalität voneinander zu separieren. Dadurch kann ich unterschiedliches Verhalten für ein und die selben Daten abbilden. Das alleine macht aber noch keine Monade notwendig. Interessant sind Monaden, weil sie so schön linear kombinierbar sind. Mit Hilfe einer freien Monade kann man einfach eine DSL abbilden - sogar mit dem Vorteil der `do`-Notation - und den Interpreter der DSL unabhängig von der DSL implementieren.

Und genau das machen wir jetzt mal. Wir bauen uns eine einfache DSL, die wir mit der `do`-Notation verwenden wollen und schauen dann, wie wir sie in Haskell mit freien Monaden abbilden können.

Unsere DSL soll die Ein- und Ausgabe von Daten erlauben.

Zuerst versuchen wir es mal ganz platt ohne `do`-Notation und Monaden. Wir definieren uns eine Datenstruktur, mit deren Hilfe wir unsere DSL abbilden und einen Interpreter, der die Datenstruktur abarbeitet und 'ausführt'.

```haskell
data SimpleIO = Output String SimpleIO
                | Input (String -> SimpleIO) 
                | End

p1 = Output "Hello" (Output "World!" End)
p2 = Output "Name: " (Input (\name -> Output name End))

run :: SimpleIO -> IO ()
run (Output s next) = putStrLn s >> run next
run (Input f)       = getLine >>= run . f
run End             = return ()

main :: IO ()
main = run p1  -- oder: run p2
```

Die Programme sehen zwar nicht schön aus, und sind etwas mühsam zu schreiben, der Interpreter `run` kann sie aber ausführen.

Leider sieht es mit der Kombinierbarkeit der Programme nicht so gut aus. Ich kann weder `p1` mit `p2` verketten, noch `p1` in `p2` aufrufen. Aber dafür ist die Monade ja gut geeignet. 

Erweitern wir unsere DSL zu einer Monade.

Als Ziel wollen wir folgendes schreiben können:

```haskell
p1 = do
       output "Hello"
       output "World!"

p2 = do
       output "Name: "
       name <- input
       output name
```

Und wir wollen `p1` mit `p2` kombinieren und ausführen können, also

```haskell
main :: IO ()
main = run do
            p1
            p2
```

Um es gleich vorneweg zu sagen: das ganze wird eine holprige Reise. Ich werde erst versuchen aus unserem Datentyp `SimpleIO` eine Monade zu machen. Dazu muss sie erst einmal ein Functor sein. Allein damit werde ich bei einer naiven Herangehensweise scheitern. Ich werde in die funktionale Trickkiste greifen müssen, um das Ziel zu erreichen. Der Applicative ist dann relativ einfach. Aus dem Applicative eine Monade zu machen wird mich dann wieder vor Herausforderungen stellen. Wenn wir die alle gemeistert haben, schauen wir mal, wie wir das so verallgemeinern können, dass wir das so nie wieder machen müssen.

Los geht es! Um unsere `do`-Notation abbilden zu können, muss unser Typ `SimpleIO` eine Monade sein, und wir brauchen Funktionen die irgendwie wie folgt aussehen: 

```haskell
output :: String -> SimpleIO  -- String ist ein Input-Parameter
output s = Output s End

input  :: String -> SimpleIO  -- (String -> SimpleIO) ist eine Funktion die mit der Eingabe aufgerufen wird
input f = Input f
```

HIER KOMMT NEUER TEXT HIN (gleich einen Functor, Applicative und Monad aus SimpleIO machen)

Zuerst machen wir aus SimpleIO einen Functor. Ein Functor muss aber parametrisiert sein.

(Über den Ein-/Ausgabetyp parametrisieren) (warum sollte ich über den next-Typ parametrisieren?)




DAS IST ALT

Und für die `do`-Notation müssen wir die Funktionen `>>` und `>>=` bestimmen (es reicht `return` und `>>=` zu bestimmen, zur Klarstellung unserer Intention definieren wir aber alle Funktionen).

```haskell
(>>)  :: SimpleIO -> SimpleIO -> SimpleIO
(>>)  step1 step 2 = fmap (\_ -> step2) step1
```

Wir müssen hier die beiden Steps / Funktionen miteinander verbinden. Und damit wir uns die Fallunterscheidung was `step1` ist und das Aus- und Einpacken hier sparen können, soll das `fmap` erledigen. Dazu muss aber unser Typ `SimpleIO` ein Functor werden. Ein Functor muss jedoch parametrisiert sein über den Typ, auf dem `fmap` arbeiten soll. Leider können wir den Typ nicht einschränken (währe cool wenn wir ihn auf `SimpleIO` einschränken könnten, doch leider kann Haskell das nicht.

Unser angepasste Definition von `SimpleIO` muss also wie folgt aussehen:

```haskell
data SimpleIO next = Output String next
                    | Input (String -> next) 
                    | End

instance Functor SimpleIO where
  fmap f (Output s n) = Output s (f n)
  fmap f (Input g)    = Input (f . g)
  fmap f (End)        = End
```

Jetzt haben wir zwar einen Functor, haben aber ein Problem mit dem Typ. Schauen wir uns mal die Typen folgender Ausdrücke an:

```haskell
p1 = Output "Hello" End
p2 = Output "Hello" (Output "World!" End)
p3 = Output "Hello" (Output "World!" (Output "von mir" End))

p1 :: SimpleIO (SimpleIO next)
p2 :: SimpleIO (SimpleIO (SimpleIO next))
p3 :: SimpleIO (SimpleIO (SimpleIO (SimpleIO next)))
```

Nicht gut. Das können wir so nicht verwenden. Glücklicherweise ist da eine Regelmäßigkeit drin, die öffter vorkommt. Eine Lösung gibt es dafür auch: den Fixpunkt eines Functors. Und der sieht wie folgt aus:

```haskell
data Fix f = Fix (f (Fix f))
```

Wenn wir unseren Datentyp darin einpacken, bekommen wir stabile Typen:

```haskell
p1 = Fix (Output "Hello" (Fix End))
p2 = Fix (Output "Hello" (Fix (Output "World!" (Fix End))))
p3 = Fix (Output "Hello" (Fix (Output "World!" (Fix (Output "von mir" (Fix End))))))

p1 :: Fix SimpleIO
p2 :: Fix SimpleIO
p3 :: Fix SimpleIO
```

