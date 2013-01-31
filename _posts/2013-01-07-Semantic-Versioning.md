---
layout: post
description: "Semantic Versioning - Ein Regelset zur Vergabe von Versionsnummern"
title: "Semantic Versioning"
---

Semantic Versioning (SemVer) ist ein Regelset zur Vergabe von Versionsnummern, das von Tom 
Preston Wernen, dem Mitbegründer und CEO von GitHub formuliert wurde. Die Innovation dabei besteht darin, aus 
der Vielfalt der gängigen Regeln einen konstistenten Set auszuwählen, welcher der Problemstellung gerecht wird und
diesen im Detail auszuarbeiten. Bei der Pflege des Regelset wird konsequenter Weise Semantic Versioning angewandt. 
 
Hier das Rational für SemVer:

>In systems with many dependencies, releasing new package versions can quickly become 
>a nightmare. If the dependency specifications are too tight, you are in danger of 
>version lock (the inability to upgrade a package without having to release new versions 
>of every dependent package). If dependencies are specified too loosely, you will 
>inevitably be bitten by version promiscuity (assuming compatibility with more future 
>versions than is reasonable). Dependency hell is where you are when version lock 
>and/or version promiscuity prevent you from easily and safely moving your project 
>forward.
>
>As a solution to this problem, I propose a simple set of rules and requirements 
>that dictate how version numbers are assigned and incremented.
 
Überall da, wo Software produziert und gewartet wird, sollte es bereits eine implizite oder explizite Praxis
des Semantic Versioning geben. Allerdings stellt die Verschiedenheit der angewandten Regelsets ein großes
Problem dar, da sich der Haupt-Vorteil des Semantic-Versioning erst aus der strikten Einhaltung des duch SemVer
festgelegten Kontracts ergibt.

>This is not a new or revolutionary idea. In fact, you probably do something close to this already. The 
>problem is that "close" isn't good enough. Without compliance to some sort of formal specification, 
>version numbers are essentially useless for dependency management. By giving a name and clear definition 
>to the above ideas, it becomes easy to communicate your intentions to the users of your software. Once 
>these intentions are clear, flexible (but not too flexible) dependency specifications can finally be made.    
 
Hier vereinfachend zusammengefasst, wie Semantic Versioning funktioniert:

Ausgangspunkt ist ein publiziertes API bzw. ein textuell formullierter logisch konsistenter Regelset. 
 
Releases werden mit 3 Zahlen, getrennt durch Punkte, bezeichnet:

{% highlight bash %}
	<major>.<minor>.<patch>
{% endhighlight %}

Die wichtigsten Regeln lauten wie folgt:

* Bei Bruch der Rückwärtskompatibilität muß die major-Nummer inkrementiert werden. 

* Hinzufügen von Funtionalität ohne Bruch der Rückwärtskompatibilität erfordert ein Inkrement der minor-Nummer
und Ein Rücksetzen der Patch-Nummer. 

* Bei Bug Fixes und Änderungen der internen Logik, welche keinen
Einfluß auf das Public-API haben, wird die Patch-Nummer inkrementiert. 

Mehr Informationen über Sematic-Versioning findet man unter [http://semver.org](http://semver.org).