# Furry Octo Dangerzone

## Web-basierte Komposition von 3D-Szenen für deren serverseitige Weiterverarbeitung im Roundtrip3D Projekt (Web-based 3D-scene composition for further server-sided processing in the Roundtrip3D project)
Für das Projekt Roundtrip3D soll unter Verwendung aktueller Webtechniken ein Web-basiertes 3D-Kompositionswerkzeug erstellt werden. Dadurch soll die Nachbearbeitung generierter X3D-Szenen bzw. deren Weiterverarbeitung innerhalb des Round-trip Prozesses stark vereinfacht werden. Das Werkzeug soll folgende Sichten und Funktionalitäten bereitstellen:0
* eine 3D-Ansicht zur Visualisierung und Bearbeitung (Komposition) von 3D-Szenen im X3D
Format
* eine Baumansicht zur Darstellung und Bearbeitung von 3D-Szenen im X3D Format
* eine Textansicht mit Bearbeitungsmöglichkeiten für JavaScript Dokumente
* eine Diagramm Ansicht für SSIML (Scene Structure and Integration Modelling Language) Modelle
* es soll möglich sein, Aufrufe von bereits existierenden Transformationen zwischen SSIML
Modellen und den Zielsprachen (X3D und JavaScript), über das Webinterface an eine Eclipse-Instanz weiterzuleiten Sämtliche Modelle/ Daten werden auf einem (Web-) Server gespeichert, auf dem auch die Eclipse-Instanz läuft. Geeignete Methoden aus aktuellen Arbeiten zum kollaborativen Arbeiten an Modellen sowie geeignete Eingabemethoden für 3D-Editoren sollen beleuchtet und ggf. umgesetzt werden.

Anforderungen: Webentwicklung, JavaScript Programmierung, 3D-Modellierung, Eclipse

### General
What would be important for a solution so that we could adopt it and extend it. instead of writing our own.

1. accept X3D as input format any other conversion could lead to errors when merging back (lost IDs for example)
2. needs to e able to manipulate the data without rewriting it.


## Technologies

### [ParaViewWeb](http://paraviewweb.kitware.com/)
Simple to use out of the box, but needs a paraview server instance and paraview does not support X3D as an input format.
So using this is sadly not possible unless I would write an input filter. And even with this there are a lot of conversion steps involved which could be omitted when using x3dom.
Also it uses WebGL primitives, so there is no scene graph.


### [GEFGWT](https://github.com/ghillairet/gef-gwt)
Completely undocumented and there seems to be a custom wrapper for GEF. I would have to write a wrapper for GMD for this to work, at least I suppose so. Without any documentation it seems unfeasible to pursue this approach.


### [XML3D](http://xml3d.org/)
Purely declarative design. Without dynamic elements like the routes and the DOM-Profile proposed by X3DOM developers. The XML3D data buffers are optimized for modern graphics hardware and thus need no further conversion. So XML3D would be slightly more performant, because X3DOM has to preprocess the IndexedFaceSets before it can render them. X3DOM tackles this problem with it's aopt tool which preprocesses and optimizes an X3D file before embedding it (e.g. moving meshes into png files) in an html file. Other more drastic performance boosts can be expected when implementing XML3D natively into the browsers (like they did with their patched versions of chromium and firefox), but unless these changes are merged upstream this isn't a reason to choose XML3D over X3DOM.

X3DOM is more widely used, more actively developed and uses X3D as it's native format. So we wouldn't need to convert our outputted X3D or incorporate XML3D as an output format to R3D.


## Papers

### [Real-time Collaborative Scientific WebGL Visualization with WebSocket](http://dl.acm.org/citation.cfm?id=2338714.2338721)
Using WebSockets instead of AJAX is an interesting approach. Especially the cut down on latency. It's over all very similar to the approach I was thinking about. Except that they propagate only some information, like the camera state and position. We would have to synchronize the whole scene. So to have spectators like they have, we either have to send the whole Model to the server after each change or use some kind of tree diff.

They visualized a specific dataset in a specific format (**Which?**). We only need to visualize X3D data, so there is no reason to use some generic visualization toolkit, x3dom does a pretty decent job doing this. On the other hand we not only have to visualize the data, but also maniulate it and this is way easier in a declarative form instead of manipulating WebGL scenes or data formatas like VTK. Well, and X3D is the only output format supported by r3d right now. So in many ways my approach will be like theirs, but goes beyond that.

### [Remote Visualization of Large Datasets with MIDAS and ParaViewWeb](http://dl.acm.org/citation.cfm?id=2010425.2010450)
