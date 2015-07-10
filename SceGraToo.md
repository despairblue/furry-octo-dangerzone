[toc]
# SceGraToo
## Prelude
The goal of the thesis is to create a 3D editor, using web technologies, which enables its users to postprocess x3d scenes.
These scenes are the result of a tool which was implemented as part of the DFG research project [Roundtrip3D].
The scene are automatically derived from software models ...

### Motivation
As part of the Roundtrip3D project, a round-trip framework (hereafter referred to as *R3D*) was developed. This framework also includes a graphical editor for SSIML models, to describe 3D applications.
<!-- TODO: csrd paper -->
Then the R3D can be used to generate boilerplate code for multiple programming languages, such as JavaScript, Java or C++, and an X3D file describing the scene.
The X3D file may contain references to other X3D files containing the actual 3D data (e.g. a car and the corresponding tires), hereafter called *inlines* .
These files are created by exporting objects from a 3D computer graphics software (e.g. [Blender], [Maya] or [3DS Max]).  
The problem that arises is that each object is usually in the center of it's own coordinate system. So they need to be translated, rotated and maybe even scaled to result in desired scene (e.g. a car where the tires are in the places they belong to and not in the center of the chassis). The structure is mostly generated. Attribute values of respective nodes, such as transformation nodes need to be adjusted in order to compose the overall 3D scene.

![Picture of a car with the tires in the center]()  
![Picture of a car with the tires where they belong to]()  


This can be achieved by adjusting *translate*, *rotate* and *scale* properties to arrange 3D objects.
To exacerbate this problem further, the orientation the 3D artists chose for it's object is simply unknown, if their is no convention for 3D modeling.
~~Usually their is a convention about origin of coord system, scaling and orientation.~~ (no there isn't)
However, we cannot assume that these conventions are always met.
The main problem is, that 3D transformations, such as translation, orientation and scale of single 3D geometries, need to be adjusted.
So far, there is no graphical tool that meets both of these requirements:

* user friendly and straight forward compose functionalities of X3D scenes
* and preservation of (generated) information, such as node names or comments (necessary to merge the changed files back into the source model)

The following pictures demonstrate the problem:  
![wheel1](https://www.dropbox.com/s/xhs8gbur6pa8lgd/wheel1.png?dl=1)
![wheel2](https://www.dropbox.com/s/0gyfkmpam5o8riq/wheel2.png?dl=1)
![wheel3](https://www.dropbox.com/s/gg362w2h9p93wdz/wheel3.png?dl=1)

These are common orientations, since it's disputable which of these could be considered be the norm.  
But if one depends on art from 3rd parties, the orientation and position could be completely arbitrary:
![wheel4](https://www.dropbox.com/s/nuoge1q1bxv8ygj/wheel4.png?dl=1)  

These properties could be added and adjusted via any text editor by opening the generated x3d file, but the resulting workflow isn't user-friendly:

1. model 3D application, including 3D scene structure
2. generate 3D scene and application code
3. run application and evaluate the scene and think about what objects need to go where and whether they need to be scaled
4. type random translation, rotation and scale
5. run the application again and evaluate whether the transformations did the right thing (since one does not know anything about the orientation of the object)
6. go back to 4. until all objects are placed correctly

Tools like Maya or Blender could also be used for this, but their import and export filter discard important metadata that is necessary for the round-trip transformation.
This is what SceGraToo is meant to be.

SceGraToo addresses both of these issues.
It allows for loading the root X3D file and changing all transformations, containing the inline nodes, using mouse interactions. For fine grained control SceGraToo also contains a tree view that allows the user to input exact attributes for translations, rotations and scale.

### Scope
This thesis addresses two issues:

1. allowing for composing generated X3D scenes (with focus on usability). E. g., besides a 3D view,
a tree view for fine grained editing should not only visualize the 3D scene's structure, but also allow for entering concrete coordinate values.
2. during the editing process, the preservation of all information/metadata must be guaranteed.

## Basics
### Scene Graph
**what is a scene graph why is it usefull**
Scene graphs are a way to organize 3d objects and concerning transformations in a treelike structure, mostly a directed acyclic graph (DAG). It stats with a root node that is associated with one or more children. Each child can be an object or a group, again containing more children. A group can contain associated transformation information, like *translation*, *rotation* or *scaling*. This structure has certain advantages compared applying all transformations to the the raw meshes and sending everything to the [GPU].

#### Culling
Before using structures like scene graphs, all polygon would be sent to the [GPU] and the [GPU] would need to test what polygons are actually in the view and thus need to be rendered. The problem with that approach was that this information was only known after doing a lot of calculations for every polygon already.  
With a scene graph it's possible to start from the root and traverse the graph, testing the bounding box of each group and only sending it to the GPU it it's completely visible. If it ain't, the whole subtree isn't sent to the GPU. If it's partially visible the same process is applied to the subtree.  
By using a structure that retains more information about what it represents it's possible to let the CPU do more of the heavy lifting and unburden the GPU.

#### Transformations
Another advantage is the way transformations work. Instead of applying them to the meshes directly, and keeping track of what meshes belong to the same object (like the chassis and the tires and the windows of a car), they can simply be nested under the same transformation group. The transformation thus applies to all objects associated with that group.

#### X3D
X3D is the [XML] representation of [VRML] which was designed as a universal interactive 3D exchange format, much like html is for written documents or SVG for vector graphics.
Due to its XML structure it can be integrated in html documents, thus the Frauenhofer Institute pursued to implement a runtime that could interpret and visualize x3d in the browser, by using a [WebGL] context. It's called [x3dom](http://www.x3dom.org/) and it's extensively used by SceGraToo, the tool that arose from this thesis.

##### x3dom
As said in the previous chapter x3dom was developed by the Frauenhofer Institute to realize the vision that started VRML in the first place: *mark up interactive 3D content for the web*. On the web there are to entirely different approaches to describe the same thing:
* imperative
* declarative

The following matrix classifies x3dom together with other common web technologies:

|                | 2D       | 3D    |
| :------------- | :------- | :---- |
| Declerative    | [SVG]    | [X3DOM] |
| Imperative     | [Canvas] | [WebGL] |

As can be seen x3dom complements the already existing technologies perfectly

### SSIML
TODO: web3d paper

### Roundtrip 3D
TODO: csrd paper

Roundtrip3D was research project that, amongst others, resulted in a graphical editor for [SSIML] models.
SSIML stands for *Scene Structure and Integration Modeling Language* and graphical DSL to model 3D applications as a scene-graph.
A scene's root node is a scene node that is the parent of describing attributes like the viewpoint or the light and also contains all object the scene contains.

![SSIML-Diagram](https://www.dropbox.com/s/v7tpvhvqdqbw4mi/SSIML.png?dl=1)

### [React]
As part of SceGraToo a treeview

TODO:
1. Objective description of the problem
2. Why some approaches might not work
3. Why the chosen approach will work

After trying different solutions it turned out the a declarative solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon it suffocated under it's own complexity (probably also due to the incompetence of its inventor - me).
It turned out that MVC had serious flaws when trying to use it to describe an ever changing declarative scene graph.
The naive solution would be to observe the X3D node and rerender the whole tree structure whenever it changed, that would also mean to rerender every time an attribute is changed.
To make that clear, that means rerendering everytime an object is moved with the mouse, since the translation is an attribute.
This thesis will not contain any benchmarks proving that manipulating the DOM from javascript is slow, rather than that it is left as an exercise to the reader to research it if she wants.
React solves this issue quite elegantly by creating the dom structure in javascript and diffing it with the [DOM], only applying the minimum of changes to the DOM to realize the corresponding result.
That made it possible to use the X3D node of the DOM as the only source of truth and minimize the state that needs to be kept to make the tree view work.

#### states
TODO: explain the DOM

Using something imperative like backbone it would be necessary to create a model (copying the information already in the DOM), a view (rendering that information to the DOM again) and a controller keeping track of the state changing the model where necessary and rerendering the view, and also keeping track of all it's children and removing them when they disappear or creating new ones whenever a new X3D element appears.

#### Functional Programming
React enables one to represent the view as a function of its input, which is the X3D node.
All SceGraToo does is listen to changes of the X3D node and whenever it changes, may it be an attribute that changes or nodes that are added or removed, it calls one function that evaluates the X3D node (basically traversing it) and return the new structure.
That structure is diffed against the tree view that is already in the DOM and react calculates the minimal changes necessary to make the tree view's old structure coherent with the newly calculated one.


### Koa
Koa is pretty much boring unless I'd explain node.js's take on asynchronicity, callbacks, promises and then generators. Still, the server is way to simple to be of any interest for this project.

### moaarrr stuff that needs explaining

## Related Work

### Collaborative Work
### [3D Meteor][0]
![3dmeteor](https://www.dropbox.com/s/173i8xo8xkxdmq7/3dmeteor.png?dl=1)

### [Blender Plugin][10]


### 3D Widgets
#### [Tilt Brush][20]
Tilt Brush was lauded for t

#### x3d gizmos

### [Component Editor][30]
## Concept
### How
#### Server
#### Client
![treeview](https://www.dropbox.com/s/985tkrkpx67nyuk/treeview.png?dl=1)

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





[Roundtrip3D]: http://elrond.informatik.tu-freiberg.de/Roundtrip3D/
[Blender]: https://www.blender.org/
[Maya]: http://www.autodesk.com/products/maya/overview
[3DS Max]: http://www.autodesk.com/products/3ds-max/overview
[XML]: http://www.w3.org/TR/REC-xml/
[X3D]: http://www.web3d.org/standards
[VRML]: http://www.web3d.org/standards
[SSIML]: https://dl.acm.org/citation.cfm?id=1122591.1122610
[React]: https://facebook.github.io/react
[DOM]: http://www.w3.org/TR/dom/
[SVG]: http://www.w3.org/TR/SVG11/
[WebGL]: https://www.khronos.org/registry/webgl/specs/latest/2.0/
[X3DOM]: http://www.x3dom.org/
[Canvas]: https://html.spec.whatwg.org/multipage/scripting.html#the-canvas-element

[0]: http://3d.meteor.com/
[10]: http://www.researchgate.net/publication/221610780_A_Blender_Plugin_for_Collaborative_Work_on_the_Articiel_Platform
[20]: http://www.tiltbrush.com/
[30]: https://github.com/x3dom/component-editor
