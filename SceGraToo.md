[toc]
# SceGraToo
## Prelude
The goal of the thesis is to create a 3d editor, using web technologies, which enables the user to postprocess x3d scenes.
These scenes are the result of a tool which was implemented as part of the DFG research project [Roundtrip3D](http://elrond.informatik.tu-freiberg.de/Roundtrip3D/).

### Motivation
As part of the Roundtrip3D project an application (hereafter referred to as *R3D*) was built containing an editor that enabled the user to describe 3D scenes in a visual UML-like language.
Then the R3D can be used to generate boilerplate code for multiple programming languages and an X3D file describing the scene.
The X3D file may contain references to other X3D files containing the actual 3D data (e.g. a car and the corresponding tires), hereafter called *inlines* .
These files are created by exporting objects from a 3D computer graphics software (e.g. [Blender](https://www.blender.org/), [Maya](http://www.autodesk.com/products/maya/overview) or [3DS Max](http://www.autodesk.com/products/3ds-max/overview)).  
The problem that arises is that each object is usually in the center of it's own coordinate system. So they need to be translated, rotated and maybe even scaled to result in desired scene structure (e.g. a car where the tires are in the places they belong to and not in the center of the chassis).  

![Picture of a car with the tires in the center]()  
![Picture of a car with the tires where they belong to]()  


This can be achieved by adding *translate*, *rotate* and *scale* properties to the transformation node containing the individual objects.
To exacerbate this problem further, the orientation the 3d artists chose for it's object is simply unknown. The following pictures demonstrate the problem:

![wheel1](https://www.dropbox.com/s/xhs8gbur6pa8lgd/wheel1.png?dl=1)
![wheel2](https://www.dropbox.com/s/0gyfkmpam5o8riq/wheel2.png?dl=1)
![wheel3](https://www.dropbox.com/s/gg362w2h9p93wdz/wheel3.png?dl=1)  
These are common orientations, since it's disputable which of these would be normal.
But if one depends on art from 3rd parties, the orientation and position could be completely arbitrary:
![wheel4](https://www.dropbox.com/s/nuoge1q1bxv8ygj/wheel4.png?dl=1)  

These properties can be added in R3D application through a property list/table view, but the resulting workflow is less than ideal:

1. generate the scene (takes a couple of minutes)
2. evaluate the scene and think about what objects need to go where and whether they need to be scaled
3. type random translation, rotation and scale
4. regenerate the scene
5. evaluate whether the transformations did the right thing (since one does not know anything about the orientation of the object)
6. go back to 3. until all objects are placed correctly

The tool I build as part of this thesis addresses this usability issue by making it possible to load the root X3D file and change all transformations, containing the inlines, by using the mouse as well. For fine grained placement a tree view still exists to visualize the structure of the scene and allow the input of exact coordinates.
![treeview](https://www.dropbox.com/s/985tkrkpx67nyuk/treeview.png?dl=1)

![Picture of the R3D property list view]()  

## Basics
### Scene Graph
**what is a scene graph why is it usefull**

### Roundtrip 3D
Roundtrip3D was research project that resulted in an editor implementing [SSIML](http://edoc.ub.uni-muenchen.de/9459/). SSIML stands for *Scene Structure and Integration Modeling Language* and is a visual UML-like language for describing 3D scene as a [scene-graph](https://en.wikipedia.org/wiki/Scene_graph). A scene starts with a scene node that is the parent of describing attributes like the viewpoint or the light and also contains all object the scene contains.

![SSIML-Diagram](https://www.dropbox.com/s/v7tpvhvqdqbw4mi/SSIML.png?dl=1)

### X3D
X3D is the [XML](https://en.wikipedia.org/wiki/XML) represntation of [VRML](https://en.wikipedia.org/wiki/VRML) which was designed as a universal interactive 3d exchange format, much like html is for written documents or svg for vector graphics.
Due to its XML structure it can be integrated in html documents, thus the Frauenhofer Institute pursued to implement a runtime that could interpret and visualize x3d in the browser. It's called [x3dom](http://www.x3dom.org/) and it's exensively used by SceGraToo, the tool that arose from this thesis.

#### x3dom
As said in the previous chapter x3dom was developed by the Frauenhofer Institute to realize the vision that started VRML in the first place: *mark up interactive 3D content for the web*. On the web there are to entirely different approaches to describe the same thing:
* imperative
* declerative

The following matrix classifies x3dom together with other common web technologies:

|                | 2D       | 3D    |
| :------------- | :------- | :---- |
| Declerative    | [SVG]    | X3DOM |
| Imperative     | [Canvas] | WebGL |

As can be seen x3dom complements the already existing technologies perfectly

### [React](https://facebook.github.io/react/)
After trying different solutions it turned out the a declarative solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon it suffocated under it's own complexity (probably also due to the incompetence of its inventor - me).
It turned out that MVC had serious flaws when trying to use it to describe an ever changing declarative scene graph.
The naive solution would be to observe the X3D node and rerender the whole tree structure whenever it changed, that would also mean to rerender every time an attribute is changed.
To make that clear, that means rerendering everytime an object is moved with the mouse, since the translation is an attribute.
This thesis will not contain any benchmarks proving that manipulating the DOM from javascript is slow, rather than that it is left as an exercise to the reader to research it if she wants.
React solves this issue quite elegantly by creating the dom structure in javascript and diffing it with the [DOM](https://en.wikipedia.org/wiki/Document_Object_Model), only applying the minimum of changes to the DOM to realize the corresponding result.
That made it possible to use the X3D node of the DOM as the only source of truth and minimize the state that needs to be kept to make the tree view work.

#### states
Using something imperative like backbone it would be necessary to create a model (copying the information already in the DOM), a view (rendering that information to the DOM again) and a controller keeping track of the state changing the model where necessary and rerendering the view, and also keeping track of all it's children and removing them when they disappear or creating new ones whenever a new X3D element appears.

#### functional programming
React enables one to represent the view as a function of its input, which is the X3D node.
All SceGraToo does is listening to changes of the X3D node and whenever it changes, may it be an attribute that changes or nodes that are added or removed, it calls one function that evalute the X3D node and return the new structure.
That structre is diffed against the tree view that is already in the DOM and react calculates the minimal changes necessary to make the tree view coherent with the X3D node again.


### koa
### moaarrr stuff that needs explaining
## Related Work
### Collaborative Work
### 3D Widgets
#### [Tilt Brush](http://www.tiltbrush.com/)
#### x3d gizmos
### [Component Editor](https://github.com/x3dom/component-editor)
### [3D Meteor](http://3d.meteor.com/)
## Concept
### How
#### Server
#### Client
#### Interaction
### Problems
### How to solve
## Implementation
### Used Tech
### Problems
#### Backbone and why it sucks
#### Why Frameworks and Boilerplate in particular suck
## Case Study
### Tasks
### Results
## Results
### Why Scegratoo is the best since sliced Bread
