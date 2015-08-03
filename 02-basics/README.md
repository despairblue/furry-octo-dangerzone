## Basics
### Scene-Graph
Scene-graphs can be used to group and organize 3D objects, their properties and concerning transformations. A directed acyclic graph (DAG) can be used to represent a scene-graph.
It starts with a root node that is associated with one or more children. Each child can be an object or a group, again containing more children.
A group can contain associated transformation information, like *translation*, *rotation* or *scaling*. This structure has certain advantages compared to applying all transformations to the raw meshes and sending everything to the [GPU].<sup>[50] (2015-07-23 15:59)</sup>

[SSIML] scene-graphs differ from the above definition: there are three types of nodes:
- transform
- geometry
- group

#### Culling
Before using structures like scene-graphs, all polygon would be sent to the GPU and the GPU would need to test what polygons are actually in the view and thus need to be rendered. The problem with that approach was that this information was only known after doing a lot of calculations for every polygon already.  
With a scene-graph it's possible to start from the root and traverse the graph, testing the bounding box of each group and only sending it to the GPU it it's completely visible. If it ain't, the whole sub-tree isn't sent to the GPU. If it's partially visible the same process is applied to the sub-tree.  
By using a structure that retains more information about what it represents it's possible to let the [CPU] do more of the heavy lifting and unburden the [GPU].

>Rather than do the heavy work at the OpenGL and polygon level, scene-graph architects realized they could better perform culling at higher level abstractions for greater efficiency. If we can remove the invisible objects first, then we make the hardware do less work and generally improve performance and the all-important frame-rate.  
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
Due to its XML structure it can be integrated in html documents, thus the Frauenhofer Institute pursued to implement a runtime that could interpret and visualize X3D in the browser, by using a [WebGL] context. It's called [x3dom](http://www.x3dom.org/) and it's extensively used by SceGraToo, the tool that arose from this thesis.

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

SSIML is a graphical approach to unify the scene-graph model and the the application model, thus making the communication between the different parties easier.

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
React is utilized by SceGraToo to render the tree view that gives a more structured view of the scene-graph than the 3 dimensional representation.

#### Objective description of the problem
The most important part of SceGraToo is the tree-view, that visualizes the X3D scene-graph in a more comprehendible.

React is utilized by SceGraToo to render the tree-view that gives a more structured view of the scene-graph than the rendered scene.

The scene-graph is the most important part of SceGraToo, it shows the the structure rather than the visual representation.
Different off-the-shelf solutions, like angular or JQuery plugins, were tested against theses requirements:

1. has to work right away <sup>[80]</sup>
2. custom elements as part of tree nodes (multiple checkboxes or multiple inputs)
3. ability to listen to changes to tree nodes
4. *easy* synchronization to a model, that can recover inconsistent state
5. flexible input parsing

with mixed results:

**Partially not met requirements:**

* has to work right away
* custom elements as part of tree nodes
* ability to listen to changes to the tree node

**Requirements none of the tested tools met:**

* flexible input parsing
* easy synchronization to a model, that can recover inconsistent state

##### Easy Synchronization to a Model
The far more complicated part is keeping the tree-view in sync with the scene-graph while the scene-graph is being modified and also the other way around.

First approach would be to create the tree-view while traversing the scene-graph.
Each tree node would observe the its corresponding scene-graph node for attribute mutations.

Depending on how the scene-graph is mutated 3 main cases must be differentiated.
1. a scene-graph node is added  
  ↪️ add a new tree node to the and the tree-view.
2. a scene-graph node is deleted  
  ↪️ remove it from the and the tree-view.
3. a scene-graph node is mutated  
  ↪️ alter the corresponding tree node.

1\. and 2. would be handled by the tree-view itself whereas 3. would be handled by each node respectively.


<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/naiveTreeView.md.svg></object>

Bei der Implementierung muss sichergestellt werden, dass selbst nach einem Syncfehler zwischen TreeView und 3D View Änderungen eines Datenmodells in das jeweils andere übertragen werden und so wieder eine konsistente Datenbasis hergestellt wird

Let's recap.
An initial tree-view is created by traversing the scene-graph.
The tree-view and the tree nodes replay all mutations to themselves to the scene-graph, also they replay all changes observed from the scene-graph to themselves.
It's assumed that the updates will always lead to consistent state, where the scene-graph and the tree node converge.

So what's the single source of truth ([SSOT])?  
Scene-graph or the Tree View?  
What if the code performing the update is faulty (maybe neglecting an edge case)?

There is no [SSOT] and if the update code is faulty they diverge and they stay inconsistent for an unforeseeable time (or even forever).

This design leads to brittle code that is hard to maintain and hard to adapt to new use cases.

Let's try to find a design that's more maintainable and easier to reason about. Keep also in mind that the diagram above only represents the most basic interactions that could be provided by the tree-view.

The first culrpit in this design is that the tree-view mutates itself.

Let's externalize the mutation observation and update capabilities into another actor: a mutation observer.

<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/externalObserver.md.svg></object>

The mutation observer can map each graph node to its corresponding tree node and thus change the tree node to represent the matching graph node again.

The tree-view does not mutate itself anymore.
All the mutation logic that synchronizes the tree-view with the scene-graph is in the mutation obserser.
But the scene-graph and the tree-view can still diverge, since moving the mutation logic from the tree-view to the mutation observer does not make it easier to reason about and thus less error prone.

Assuming that the first operation (traversing the scene-graph and creating the tree-view) is correctly implemented and resilient, the easiest way to make sure the tree-view and the scene-graph are in sync would be to recreate the tree-view whenever the scene-graph changes.

<object data="https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/rerenderOnChange.md.svg"></object>

At first sight that seems to be awfully slow, that needn't to be true.
Instead of recreating the the whole tree using [DOM] elements, a model could be created using simple javascript objects.
This virtual tree-view can then be compared to the tree-view already in the [DOM] and only the changes need to be applied.

That might sound like it's no easier than doing the synchronization manually like in the first diagram, but the diff algorithm doesn't actually need to know what it's diffing.
Meaning it could be developed and tested once and be used by all kinds of projects.
That's one thing react provides.
The rendering pipeline looks finally like this:  
<object data="https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/react.md.svg"></object>

React calls the virtual representation of what will be rendered *virtual DOM*.
The other important feature of react is it's way to build the virtual [DOM].
A is a factory that returns
A virtual [DOM] node is the return value of component's render function, the render function can nest other virtual [DOM] nodes in its return value.

A quick examples:
```javascript
var TreeNodeAttributeList = React.createClass({
  render: function () {
    // the scene-graph node that was passed `createElement` by the caller
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
    // the scene-graph node that was passed `createElement` by the caller
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

This is code is the a simplified version of the code that renders a graph's nodes attributes into the tree-view. This component simply decides what attributes should be rendered into the tree. `TreeNodeAttribute` is another component that will renders different elements depending on what attribute is passed in.

Because the outputted html is only a function of its input it easy to parse the scene-graph:
1. choose a graph node as the root
2. call the node component with that graph node
3. if the graph node has child nodes call the node component again with each child node and return their return values wrapped in an element
4. if the graph node has no children return an empty element

##### Other Solutions
Another idea is to utilize data binding.
Frameworks like [angular] or web components like [polymer] support templates and two way data binding.
An idea would be to make every part of the scene-graph a component (polymer) or directive (angular).
Nesting is also possible so it should be possible to compose the scene-graph by either:
* constructing a template and rendering it
* using recursive directives or components, which are at least in the case of angular not as simple to achieve as with react <sup>the author has no knowledge of web components or polymer and can thus form no opinion about how hard this approach would be by using that technology</sup>

The initial scene-graph would need to be parsed for each element a directive/componten would be created, binding its attributes to a model.
The data binding would than ensure that when the scene-graphs attributes are changed the model is kept in sync and the other way around.
The tree-view could be bound to the same model and should thus update automatically respectively all updates to the tree-view would be propagated to the model again:
It would probably work, but the complexity is too daunting.
* nesting directives/components indefinitely deep makes it harder to reason about them.
* the X3D scene couldn't be dumped just be dumped into the [DOM] it would have to be parsed and componentized

<!-- What that means is to traverse the X3D scene, create a tree node for every eligible part of the scene-graph and hook up event listeners that update that tree node when the scene-graph node changes and the other way around, update the scene-graph when the tree node is updated. -->

<!-- Even if one of them would have worked the main problem would have remained: How to keep the tree-view in sync with the [DOM]? -->

#### Why the chosen approach will work
After trying different solutions it turned out the a declarative solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon the first version of it suffocated under it's own complexity (probably also due to the incompetence of its user - me).
It turned out that MVC had serious flaws when trying to use it to describe an ever changing declarative scene-graph.
The naive solution would be to observe the X3D node and rerender the whole tree structure whenever it changed, that would also mean to rerender every time an attribute is changed.
To make that clear, that means rerendering everytime an object is moved with the mouse, since the translation is an attribute.
This thesis will not contain any benchmarks proving that manipulating the DOM from javascript is slow, rather than that it is left as an exercise to the reader to research it if she wants.
React solves this issue quite elegantly by creating the dom structure in javascript and diffing it with the [DOM], only applying the minimum of changes to the DOM to realize the corresponding result.
That made it possible to use the X3D node of the DOM as the only source of truth and minimize the state that needs to be kept to make the tree-view work.

#### states
TODO: explain the DOM

Using something imperative like backbone it would be necessary to create a model (copying the information already in the DOM), a view (rendering that information to the DOM again) and a controller keeping track of the state changing the model where necessary and rerendering the view, and also keeping track of all it's children and removing them when they disappear or creating new ones whenever a new X3D element appears.

#### Functional Programming
All SceGraToo does is listen to changes of the X3D node and whenever it changes, may it be an attribute that changes or nodes that are added or removed, it calls one function that evaluates the X3D node (basically traversing it) and return the new structure.
That structure is diffed against the tree-view that is already in the DOM and react calculates the minimal changes necessary to make the tree-view's old structure coherent with the newly calculated one.


### Koa
Koa is pretty much boring unless I'd explain node.js's take on asynchronicity, callbacks, promises and then generators. Still, the server is way to simple to be of any interest for this project.

### moaarrr stuff that needs explaining

[40]: http://www.x3dom.org
[50]: http://www.realityprime.com/blog/2007/06/scenegraphs-past-present-and-future/
[60]: http://vr.tu-freiberg.de/roundtrip3d/
[70]: https://mbraak.github.io/jqTree/#node-functions-getnextnode
[80]: http://blogs.msdn.com/b/brada/archive/2003/10/02/50420.aspx
[GF15]: http://dx.doi.org/10.1007/s00450-014-0256-x
[JLV15]: http://dx.doi.org/10.1007/s00450-014-0258-8
