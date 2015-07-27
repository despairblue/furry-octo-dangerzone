<!-- TOC depth:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [SceGraToo](#scegratoo)
	- [Prelude](#prelude)
		- [Motivation](#motivation)
		- [Scope](#scope)
	- [Basics](#basics)
		- [Scene Graph](#scene-graph)
			- [Culling](#culling)
			- [Transformations](#transformations)
			- [X3D](#x3d)
				- [x3dom](#x3dom)
		- [SSIML](#ssiml)
		- [Roundtrip 3D](#roundtrip-3d)
		- [[React](https://facebook.github.io/react)](#reacthttpsfacebookgithubioreact)
			- [states](#states)
			- [Functional Programming](#functional-programming)
		- [Koa](#koa)
		- [moaarrr stuff that needs explaining](#moaarrr-stuff-that-needs-explaining)
	- [Related Work](#related-work)
		- [Collaborative Work](#collaborative-work)
		- [[3D Meteor][0]](#3d-meteor0)
		- [[Blender Plugin][10]](#blender-plugin10)
		- [3D Widgets](#3d-widgets)
			- [[Tilt Brush][20]](#tilt-brush20)
			- [x3d gizmos](#x3d-gizmos)
		- [[Component Editor][30]](#component-editor30)
	- [Concept](#concept)
		- [How](#how)
			- [Server](#server)
			- [Client](#client)
			- [Interaction](#interaction)
		- [Problems](#problems)
		- [How to solve](#how-to-solve)
	- [Implementation](#implementation)
		- [Used Tech](#used-tech)
		- [Problems](#problems)
			- [Backbone and why it sucks](#backbone-and-why-it-sucks)
			- [Why Frameworks and Boilerplate in particular suck](#why-frameworks-and-boilerplate-in-particular-suck)
	- [Case Study](#case-study)
		- [Tasks](#tasks)
		- [Results](#results)
	- [Results](#results)
		- [Why Scegratoo is the best since sliced Bread](#why-scegratoo-is-the-best-since-sliced-bread)
<!-- /TOC -->

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
~~Usually their is a convention about origin of coord system, scaling and orientation~~ (no there isn't).
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
Scene graphs are a way to organize 3d objects and concerning transformations in a directed graph, most of the times a directed acyclic graph (DAG), though also directed cyclic graphs (DCG) exist.
It starts with a root node that is associated with one or more children. Each child can be an object or a group, again containing more children.
A group can contain associated transformation information, like *translation*, *rotation* or *scaling*. This structure has certain advantages compared to applying all transformations to the the raw meshes and sending everything to the [GPU].<sup>[50] (2015-07-23 15:59)</sup>

[SSIML] scene graphs differ from above definition scene graph: Usually, there are three types of nodes:
- transform
- geometry
- group

#### Culling
Before using structures like scene graphs, all polygon would be sent to the [GPU] and the [GPU] would need to test what polygons are actually in the view and thus need to be rendered. The problem with that approach was that this information was only known after doing a lot of calculations for every polygon already.  
With a scene graph it's possible to start from the root and traverse the graph, testing the bounding box of each group and only sending it to the [GPU] it it's completely visible. If it ain't, the whole subtree isn't sent to the [GPU]. If it's partially visible the same process is applied to the subtree.  
By using a structure that retains more information about what it represents it's possible to let the [CPU] do more of the heavy lifting and unburden the [GPU].

>Rather than do the heavy work at the OpenGL and polygon level, scenegraph architects realized they could better perform culling at higher level abstractions for greater efficiency. If we can remove the invisible objects first, then we make the hardware do less work and generally improve performance and the all-important frame-rate.  
– [50] (2015-07-23 15:59)

#### Transformations
Another advantage is the way transformations work. Instead of applying them to the meshes directly, and keeping track of what meshes belong to the same object (like the chassis and the tires and the windows of a car), they can simply be nested under the same transformation group. The transformation thus applies to all objects associated with that group.

#### Reuse
With the ability to address nodes it's possible to reuse their information. If you have a car, it would be enough to have only one node containing the meshes for a tire, all other tires are merely addressing the tire with the geometry information:

* Group
	* Transform
		* Geomatry "Chassis"
	* Transform
		* Geometry "Tire"
	* Translate
		* Use Geometry "Tire"
	* Translate
		* Use Geometry "Tire"
	* Translate
		* Use Geometry "Tire"

That way the memory footprint of an application can be reduced.

#### X3D
X3D is the [XML] representation of [VRML] which was designed as a universal interactive 3D exchange format, much like html is for written documents or SVG for vector graphics.
Due to its XML structure it can be integrated in html documents, thus the Frauenhofer Institute pursued to implement a runtime that could interpret and visualize x3d in the browser, by using a [WebGL] context. It's called [x3dom](http://www.x3dom.org/) and it's extensively used by SceGraToo, the tool that arose from this thesis.

##### x3dom
As said in the previous chapter x3dom was developed by the Frauenhofer Institute to realize the vision that started VRML in the first place: *mark up interactive 3D content for the web*. On the web there are to entirely different approaches to describe the same thing:
* imperative
* declarative

The following matrix classifies x3dom together with other common web technologies <sup>[40] (2015-07-23 13:00)</sup>:

|                | 2D       | 3D      |
| :------------- | :------- | :------ |
| Declarative    | [SVG]    | [X3DOM] |
| Imperative     | [Canvas] | [WebGL] |

As can be seen x3dom complements the already existing technologies perfectly.

### SSIML
Heterogeneous developer groups, groups that are comprised of people from different backgrounds (programmers, 3D designers) have hard time working together. <sup>[GF15]</sup>

SSIML is a graphical approach to unify the scene graph model and the the application model, thus making the communication between the different parties easier.

It also serves as a code generation template.

![A SSIML Diagram](https://www.dropbox.com/s/v7tpvhvqdqbw4mi/SSIML.png?dl=1)  
<sup>A SSIML Diagram <sup>[60] (2015-07-24 18:17)</sup></sup>

### Roundtrip 3D
As stated above, when developing 3D applications, many different developers are involved, i.e. 3D designers, programmers and, ideally, also software designers (see figure n)

![Roundtrip Process](https://www.dropbox.com/s/komaipe44oguh9q/csrd2014.svg?dl=1)  
<sup>Roundtrip3D proposed process <sup>[JLV15]</sup></sup>

Roundtrip3D was research project that, amongst others, resulted in a graphical editor for [SSIML] models.
It offers an approach for merging a developer's changes back into the main model.
After all working copies are merged back into the main model (dropping unwanted or conflicting changes), all working copies are regenerated and delivered to the individual developers.
After each roundtrip every developer has a copy of the project that is consistent with everyone else's.


### [React](https://facebook.github.io/react)
As part of SceGraToo as tree view
React is utilized by SceGraToo to render the tree view that gives a more structured view of the scene graph than the 3 dimensional representation.

#### Objective description of the problem
The most important part of SceGraToo is the tree view, that visualizes the x3d scene graph in a more comprehendible.

**Requirements:**
1. has to work right away <sup>[80]</sup>
2. custom elements as part of tree nodes (multiple checkboxes or multiple inputs)
3. ability to listen to changes to tree nodes
4. *easy* synchronization to a model
5. flexible input parsing

#### Why some approaches might not work
First a couple off-the-shelf solutions were tried, like [angular] or [JQuery] plugins with mixed results:

**Partially not met requirements:**
* did not work right away
* custom elements as part of tree nodes
* ability to listen to changes to the tree node

**Requirements none of the tested tools met:**
* flexible input parsing
* easy synchronization to a model

The ones that met most requirements still required that the input data is passed in as a specifically structured javascript object and offered no to little help on keeping the tree in sync with the graph.
<!-- If going down that road, how would one act on changes to the initial model? -->

<!-- ##### Flexible Input Parsing
That posed some problems, because it meant that I'd have to traverse the x3d scene and construct that specifically structured object.

```javascript
[
    {
        label: 'node1', id: 1,
        children: [
            { label: 'child1', id: 2 },
            { label: 'child2', id: 3 }
        ]
    },
    {
        label: 'node2', id: 4,
        children: [
            { label: 'child3', id: 5 }
        ]
    }
]
```
<sup>jqTree's input format <sup>[70]</sup></sup>

Traversing the scene graph and creating that structure isn't exactly complicated and I didn't expect to find a solution that supported rendering a part of the [DOM] back into the [DOM] out of the box, so this requirement can be neglected. -->

##### Easy Synchronization to a Model
The far more complicated part is keeping the tree view in sync with the scene graph while the scene graph is being modified and also the other way around.

First approach would be to create the tree view while traversing the scene graph.
Each tree node would observe the its corresponding scene graph node for attribute mutations.

Depending on how the scene graph is mutated 3 main cases must be differentiated.
1. a scene graph node is added
	* :arrow_right_hook: add a new tree node to the map and the tree view.
2. a scene graph node is deleted
	* :arrow_right_hook: remove it from the map and the tree view.
3. a scene graph node is mutated
	* :arrow_right_hook: alter the corresponding tree node.

1. and 2. would be handled by the tree view itself whereas 3. would be handled by each node respectively.

![Naive Approach](https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/naiveTreeView.md.svg)

Let's recap.
An initial tree view is created by traversing the scene graph.
The tree view and the tree nodes replay all mutations to themselves to the scene graph, also they replay all changes observed from the scene graph to themselves.
It's assumed that the updates will always lead to consistent state, where the scene graph and the tree node converge.

So what's the single source of truth ([SSOT])?  
Scene Graph or the Tree View?  
What if the code performing the update is faulty (maybe neglecting an edge case)?

There is no [SSOT] and if the update code is faulty they diverge and they stay inconsistent for an unforeseeable time (or even forever).

This design leads to brittle code that is hard to maintain and hard to adapt to new use cases.

Let's try to find a design that's more maintainable and easier to reason about. Keep also in mind that the diagram above only represents the most basic interactions that could be provided by the tree view.

The first culrpit in this design is that the tree view mutates itself.

Let's externalize the mutation observation and update capabilities into another actor: a mutation observer.

![External Observer](https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/externalObserver.md.svg)

The mutation observer can map each graph node to its corresponding tree node and thus change the tree node to represent the matching graph node again.

The tree view does not mutate itself anymore.
All the mutation logic that synchronizes the tree view with the scene graph is in the mutation obserser.
But the scene graph and the tree view can still diverge, since moving the mutation logic from the tree view to the mutation observer does not make it easier to reason about and thus less error prone.

Assuming that the first operation (traversing the scene graph and creating the tree view) is correctly implemented and resilient, the easiest way to make sure the tree view and the scene graph are in sync would be to recreate the tree view whenever the scene graph changes.

![rerender on every change](https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/rerenderOnChange.md.svg)

At first sight that seems to be awfully slow, that needn't to be true.
Instead of recreating the the whole tree using [DOM] elements, a model could be created using simple javascript objects.
This virtual tree view can then be compared to the tree view already in the [DOM] and only the changes need to be applied.

That might sound like it's no easier than doing the synchronization manually like in the first diagram, but the diff algorithm doesn't actually need to know what its diffing.
Meaning it could be developed and tested once and be used by all kinds of projects.
That's what mainly is react is.
React calls the virtual representation of what will be rendered *virtual DOM*.
The other important feature of react is it's way to build the virtual [DOM].
A is a factory that returns
A virtual [DOM] node is the return value of component's render function, the render function can nest other virtual [DOM] nodes in its return value.

A quick examples:
```javascript
var TreeNodeAttributeList = React.createClass({
  render: function () {
    // the scene graph node that was passed `createElement` by the caller
    var node = this.props.node;

    var whitelist = ['def', 'diffusecolor', 'orientation', 'position', 'render', 'rotation', 'scale', 'translation', 'url'];

    var attributesToRender = node.attributes.filter(function (attribute) {
      return propsToRender.includes(attribute.name.toLowerCase());
    })

    return (
      <div>
        {
          attributesToRender.map(function (attribute) {
            return <TreeNodeAttribute attribute={a} owner={node}/>
          })
        }
      </div>
    );
  }
});
```
The usage of HTML tags is just syntactic sugar, it's transpiled into:
```javascript
'use strict';

var TreeNodeAttributeList = React.createClass({
  displayName: 'TreeNodeAttributeList',

  render: function render() {
    // the scene graph node that was passed `createElement` by the caller
    var node = this.props.node;

    var whitelist = ['def', 'diffusecolor', 'orientation', 'position', 'render', 'rotation', 'scale', 'translation', 'url'];

    var attributesToRender = node.attributes.filter(function (attribute) {
      return propsToRender.includes(attribute.name.toLowerCase());
    });

    return React.createElement(
      'div',
      null,
      attributesToRender.map(function (attribute) {
        return React.createElement(TreeNodeAttribute, { attribute: a, owner: node });
      })
    );
  }
});
```

This is code is the a simplified version of the code that renders a graph's nodes attributes into the tree view. This component simply decides what attributes should be rendered into the tree. `TreeNodeAttribute` is another component that will renders different elements depending on what attribute is passed in.

Because the outputted html is only a function of its input it easy to parse the scenegraph:
1. choose a graph node as the root
2. call the node component with that graph node
3. if the graph node has child nodes call the node component again with each child node and return their return values wrapped in an element
4. if the graph node has no children return an empty element

##### Other Solutions
Another idea is to utilize data binding.
Frameworks like [angular] or web components like [polymer] support templates and two way data binding.
An idea would be to make every part of the scene graph a component (polymer) or directive (angular).
Nesting is also possible so it should be possible to compose the scene graph.
The initial scene graph would need to be parsed for each element a directive/componten would be created, binding its attributes to a model.
The data binding would than ensure that when the scene graphs attributes are changed the model is kept in sync and the other way around.
The tree view could be bound to the same model and should thus update automatically respectively all updates to the tree view would be propagated to the model again:
It would probably work, but the complexity is too daunting.
* nesting directives/components indefinitely deep makes it harder to reason about them.
* the x3d scene couldn't be dumped just be dumped into the [DOM] it would have to be parsed and componentized

<!-- What that means is to traverse the x3d scene, create a tree node for every eligible part of the scene graph and hook up event listeners that update that tree node when the scene graph node changes and the other way around, update the scene graph when the tree node is updated. -->

<!-- Even if one of them would have worked the main problem would have remained: How to keep the tree view in sync with the [DOM]? -->

#### Why the chosen approach will work
After trying different solutions it turned out the a declarative solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon the first version of it suffocated under it's own complexity (probably also due to the incompetence of its user - me).
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
[MDD]: Model-Driven-Development

[0]: http://3d.meteor.com/
[10]: http://www.researchgate.net/publication/221610780_A_Blender_Plugin_for_Collaborative_Work_on_the_Articiel_Platform
[20]: http://www.tiltbrush.com/
[30]: https://github.com/x3dom/component-editor
[40]: http://www.x3dom.org
[50]: http://www.realityprime.com/blog/2007/06/scenegraphs-past-present-and-future/
[60]: http://vr.tu-freiberg.de/roundtrip3d/
[70]: https://mbraak.github.io/jqTree/#node-functions-getnextnode
[80]: http://blogs.msdn.com/b/brada/archive/2003/10/02/50420.aspx
[GF15]: http://dx.doi.org/10.1007/s00450-014-0256-x
[JLV15]: http://dx.doi.org/10.1007/s00450-014-0258-8
