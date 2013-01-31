---
layout: post
description: "Ein Anwendung-Beispiel für state-monads"
title: "Ein Anwendungsbeispiel für State-Monads: Zufallsdaten"
---

Die Implementierung von [Clojure Monads](http://github.com/clojure/algo.monads) enthält ein Anwendungsbeispiel, 
das ich mir dieser Tage näher angeschaut habe. In dem Beispiel werden Zufallsdaten mit Hilfe von State-Monads
erzeugt. Das Beispiel zeigt, wie mächtig State-Monads sind und wofür sie zweckmäßig eingesetzt werden können.

Vorab der Versuch einer textuellen Beschreibung von State-Monads:

>A state monad item represents a computation that changes a state and returns a value. Its structure is a function that takes a state argument
>and returns a two-item list containing the value and the updated state. It is important to realize that everything you put into a state monad
>expression is a state monad item (thus a function), and everything you get out as well. A state monad does not perform a calculation, it
>constructs a function that does the computation when called.

Im Zentrum des Zufallszahlen-Beispiels stehen 2 Funktionen. Die Funktion `rng`, die Pseudo-Zufallszahlen aus dem Interval
[0, 1] erzeugt:

{% highlight clojure %}
(defn rng [seed]
  (let [m      259200
        value  (/ (float seed) (float m))
        next   (rem (+ 54773 (* 7141 seed)) m)]
    (print (str value ", "))
    [value next]))
{% endhighlight %}
  
und die Funktion `value-seq`, die eine Lazy-Seq aus den von `rng` erzeugten Zahlen liefert:  

{% highlight clojure %}
(defn value-seq [f seed]
  (lazy-seq
    (let [[value next] (f seed)]
      (cons value (value-seq f next)))))
{% endhighlight %}

Für eine Sequenz von Zufallszahlen kann man Mittelwert und Varianz wie folgt berechnen:

{% highlight clojure %}
(defn sum [xs] (apply + xs))

(defn mean [xs] (/ (sum xs) (count xs)))

(defn variance [xs]
  (let [m (mean xs)
        sq #(* % %)]
    (mean (for [x xs] (sq (- x m))))))
{% endhighlight %}
  
Diese Funktionen haben noch nichts mit State-Monads zu tun. Die Funktion
`rng` hat aber Eigenschaften, welche die näherungsweise Erzeugung von normalverteilten Zahlen mit
Mittelwert 0 und Standardabweichung 1 erlauben. Mit Hilfe von State-Monads kann man das in eleganter Weise 
zu tun, indem man 12er-Tupel aus einem mit `rng` erzeugten Zahlenstrom bildet und deren Mittelwert berechnet.

{% highlight clojure %}
(def gaussian
  (m/with-monad m/state-m
    ((m/m-lift 1 #(- % (float 6)))
      (m/m-reduce + (repeat 12 rng)))))
{% endhighlight %}

Überprüfen kann man die Eigenschaften der von der Funktion `gaussian` erzeugten Zahlen 
beispielsweise so:

{% highlight clojure %}
(defn- close-to [a b tolerance] 
  (let [sqr #(* % %)]
    (< (sqr(- a b)) tolerance)))

; the mean should be close to zero.
(assert
  (close-to
    (float 0)
    (mean (take 1000 (value-seq gaussian 1)))
    0.001))

(assert
  (close-to
    (float 1)
    (variance (take 1000 (value-seq gaussian 1)))
    (float 0.00001)))
{% endhighlight %}

Man kann auch normalverteilte Zufallszahlen für 2 oder 3 Dimensionen erzeugen:

{% highlight clojure %}
(def rng2D 
  (m/with-monad m/state-m
    (m/m-seq (list rng rng))))

; drei 2D Zufallszahlen 
(take 3 (value-seq rnd2D 1))   

(def gaussian3D  
  (m/with-monad m/state-m
    (m/m-seq (list gaussian gaussian gaussian))))
    
; fünf 3D Zufallszahlen normalverteilt mit Mittelwert 0 und Sigma 1      
(take 5 (value-seq gaussian3D 1))   
{% endhighlight %}

Mit Hilfe von `fetch-state` und `set-state` ist es möglich, Teil-Sequenzen von Zufallszahlen
zu wiederholen:
 	    	    
{% highlight clojure %}
(def identical-random-seqs
  (m/domonad m/state-m
    [seed (m/fetch-state)
     x1   rng
     x2   rng
     _    (m/set-state seed)
     y1   rng
     y2   rng]
    (list [x1 x2] [y1 y2])))  
{% endhighlight %}

## Zusammenfassung
	    
Man sieht, wie State-Monads es möglich machen, Zufallszahlen in eleganter Technik
zu erzeugen. Hierbei ist der Fakt, dass die Funktion `rng` mit der von den State-Monads 
erwarteten Struktur/Signatur für Monadic Values identisch ist, bedeutsam.

Die Verwendung von State-Monads bewirkt, dass die Daten durch die programmierte Logik 
"fließen", wobei die monadische Logik dafür sorgt, dass Funktionen einfach verkettet
werden können.

Die Implementierung von Monads in ihrer ganzen Tiefe zu verstehen, ist ein hartes Brot, aber
wenn man sich auf die reine Nutzerperspektive beschränkt und nur die durch colojure.algo.monads
vordefinierten Monads nutzt, kann man sich auf das Schreiben von Monadic-Functions und die Nutzung
gemäß vorgegebener Muster beschränken, was einem die Sache bedeutend erleichtert. Allerdings sollte
man von einer Verwendung von Monads absehen, wenn es eine konventionelle Alternative gibt.
Zumindest so lange wie das volle Verständnis, wie Monads funkionieren nicht Allgemeingut in der
Welt der Programmierer ist -> Wartbarkeit in den Dimensionen Mensch und Zeit. 

## Links

* [A monad tutorial for Clojure programmers](http://onclojure.com/2009/03/05/a-monad-tutorial-for-clojure-programmers-part-1)
* [State Monads erklärt](http://www.clojure.net/2012/02/10/State)
* [The State Monad: A Tutorial for the Confused](http://brandon.si/code/the-state-monad-a-tutorial-for-the-confused)
* [Using State Monads for Validation](http://www.leonardoborges.com/writings/2013/01/04/bouncer-validation-lib-for-clojure)