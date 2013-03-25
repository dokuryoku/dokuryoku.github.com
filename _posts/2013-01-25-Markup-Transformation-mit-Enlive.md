---
layout: post
description: "Anhand von Beispielen wird kurz beschrieben, wie die clojure-library Enlive zur Transformation von Markup (HTML/XML) verwendet werden kann"
title: "Markup-Transformation mit Enlive"
---

[Enlive](http://github.com/cgrand/enlive), eine von [Christophe Grand](http://github.com/cgrand) 
programmierte clojure-library, gibt es schon seit mehr als 2 Jahren. Enlive ist für die Verarbeitung
von Markup (XML, HTML ...) gedacht und kann für Scraping, Templating und jegliche Art von Markup-Transformation
eingesetzt werden.

Ein wesentlicher Vorteil von Enlive ist, dass es dem Programmierer erlaubt, Design und Content 
sehr weitgehend zu separieren und so eine Zusammenarbeit zwischen Programmierer und Web-Designer
erlaubt, die vom Designer nicht unbedingt Kenntnisse im Umgang mit clojure-Code verlangt. Enlive 
nutzt intensiv die Möglichkeiten von clojure aus und erreicht durch funktionale Komposition und 
Macros eine große Mächtigkeit bei einem Code-Umfang von "nur" ~1000 Lines netto.   

Nun zu dem Anwendungs-Beispiel, welches Enlive für das Erzeugen von HTML aus Templates verwendet.
Es werden 2 Templates verwendet, die als Files mit der Extension HTML bereitgestellt sind. 

Das Template `layout.html` definiert den Rahmen für das zu erzeugende HTML-Dokument. 

{% highlight html %}
<html>
  <head>
    <title>Default Title</title>
  </head>
  <body>
    <div id="header" class="column">
      Default Header
    </div>
    <div id="notice"></div> 
    <div id="main" class="column">
      Default Body
    </div>
    <div id="footer" class="column">
      Default Footer
    </div>
  </body>
</html>
{% endhighlight %}

Das Template `page.html` definert den Inhalt.
     
{% highlight html %}
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <div id="main">
      <div>Hello <_name_ /></div>
    </div>
  </body>
</html>
{% endhighlight %}

Durch Transformation und Einfügen von zusätzlicher Information wird das fertige 
HTML-Dokument erzeugt, das wie folgt aussieht: 

{% highlight html %}
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <div class="column" id="header">Page Header</div>
    <div id="notice">take care !</div> 
    <div class="column" id="main"><div>Hello World</div></div>
    <div class="column" id="footer">Default Footer</div>
  </body>
</html>  
{% endhighlight %}

Der Clojure-Code, welcher dies leistet sieht wie folgt aus:

{% highlight clojure %}
(ns enlive-test.template1.core
  (:require [net.cgrand.enlive-html :as e]))

(defmacro maybe-content
  ([expr] `(if-let [x# ~expr] (e/content x#) identity))
  ([expr & exprs] `(maybe-content (or ~expr ~@exprs))))

(defn layout
  [{:keys [title notice main header footer]}]
  (let [layout (e/html-resource "enlive_test/template1/layout.html")]
		(e/at layout
		  [:title] (maybe-content title)
		  [:div#notice] (maybe-content notice) 
		  [:div#main] (e/content main)
		  [:div#header] (maybe-content header)
		  [:div#footer] (maybe-content footer))))

(defn page-t 
  [{:keys [page name]}]
  (e/at page 
    [:_name_] (e/substitute name)))

(defn page [{:keys [notice] :as params}]
  (let [page-r (e/html-resource "enlive_test/template1/page.html")
        page (page-t (assoc params :page page-r))]                                                               
    (layout {:title (e/select page [:title :> e/text-node])  
             :notice notice
             :main (e/select page [:div#main :> :*])
             :header "Page Header" 
             :footer nil})))

(print (apply str (e/emit* (page {:notice "take care !" :name "World"}))))
{% endhighlight %}


In dem Ausdruck `[:_name_] (e/substitute name)` ist `[:_name_]` ein Selector und `(e/substitute name)` eine Transformation. Selektoren
können via embedded DSL in einer den CSS3-Selektoren ähnlichen Syntax formuliert werden.
Eine Transformations-Funktion ist eine Funktion, die als Argument Transformations-Parameter erhält 
und als Rückgabewert eine Funktion liefert, die Nodes verarbeitet. Eine Transformation ist ein
Sequenz von Paaren aus Selektor und Transformations-Funktion. Transformations-Funktionen können
cascadiert werden.  
     
Fazit: Enlive ist im Vergleich zu anderen Varianten der Markup-Verarbeitung (XSLT etc.) eine
einfache und elegante Lösung. Es gibt allerdings eine bedeutende Einschränkung: Markup, welche
Namespaces verwendet, kann mit Enlive 1.0.x nicht verarbeitet werden.

Ein interessantes Projekt, das sich auf Enlive bezieht, ist [Enfocus](http://github.com/ckirkendall/enfocus) 
Enfocus ist in [Clojurescript](http://github.com/clojure/clojurescript) programmiert und bietet
Enlive-ähnliche Möglichkeiten der Markup-Verarbeitung auf dem Browser-Client. Intern verwendet
Enlive [Domina](https://github.com/levand/domina.git) ;-) das sich wiederum auf das mit
Clojurescript automatisch mitkommende [Google Closure](https://developers.google.com/closure/library/) abstützt.         