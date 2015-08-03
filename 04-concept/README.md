## Concept
### How
#### Server
#### Client
![treeview](https://www.dropbox.com/s/985tkrkpx67nyuk/treeview.png?dl=1)

#### Interaction
### Problems
#### Synchronization Process

> “I conclude that there are two ways of constructing a software
design: One way is to make it so simple that there are obviously
no deficiencies and the other way is to make it so complicated
that there are no obvious deficiencies. The first method is far
more difficult.  
- Hoare Turing Award Lecture 1980

React is utilized by SceGraToo to render the tree-view that gives a more structured view of the scene-graph than the rendered scene.

The scene-graph is the most important part of SceGraToo, it shows the the structure rather than the visual representation.
Different off-the-shelf solutions, like angular or JQuery plugins, were tested against theses requirements:

2. custom elements as part of tree nodes (multiple checkboxes or multiple inputs)
3. ability to listen to changes to tree nodes
4. binding to an arbitrary model, that can recover inconsistent state

with mixed results:

**Partially not met requirements:**

* custom elements as part of tree nodes
* ability to listen to changes to the tree node

**Requirements none of the tested tools met:**

* binding to an arbitrary model, that can recover inconsistent views

#### Easy Synchronization to a Model
The most complicated part is keeping the tree-view in sync with the scene-graph while the scene-graph is being modified and vice versa.

The tree-view is made up of two distinct parts.
1. a javascript object that encapsulates the scene-graph parsing and rendering to the DOM
2. the rendered visual representation of the tree view in the DOM

The aim is to keep the tree-view's DOM representation consistent with the scene-graph.

First approach would be to create the tree-view while traversing the scene-graph.
For each scene-graph node create a corresponding tree-view node.
Each tree-view node would observe its corresponding scene-graph node for attribute mutations and added or removed child nodes and change it's view accordingly.

Depending on how the scene-graph is mutated 3 main cases must be differentiated.
1. a scene-graph node is added  
  ↪️ add a new tree node to the parent's corresponding tree-view node
2. a scene-graph node is deleted  
  ↪️ remove the corresponding tree-view node
3. a scene-graph node is mutated  
  ↪️ alter the corresponding tree node.

1\. and 2. would be handled by the tree-view itself whereas 3. would be handled by each node respectively.


<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/naiveTreeView.md.svg></object>  
<sup>the dotted lines represent events, the solid one method calls</sup>

1. An initial tree-view is created by traversing the scene-graph and creating the tree-view nodes adhoc.
2. The tree-view and the tree nodes replay all mutations to themselves to the scene-graph, also they replay all changes observed from the scene-graph to themselves.
3. It's assumed that the updates will always lead to consistent state, where the scene-graph and the tree node converge.

The synchronization process has no ability to detect if updates lead to a consistent state.
It also has no ability to recover from that state, though without the ability to detect inconsistencies this is less of a problem.
The state is copied and there is not one model where all information comes from. The scene-graph should be that model.

The problem is that the tree-view maintains it's own state.

This design leads to brittle code that is hard to maintain and hard to adapt to new use cases.

The first culrpit in this design is that the tree-view mutates itself.

Let's externalize the mutation observation and update capabilities into another actor: a mutation observer.

<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/externalObserver.md.svg></object>

The mutation observer can map each graph node to its corresponding tree node and thus change the tree node to represent the matching graph node again.

The tree-view does not mutate itself anymore.
All the mutation logic that synchronizes the tree-view with the scene-graph is in the mutation observer.
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
The other important feature of react is it's way to build the virtual DOM.
A is a factory that returns
A virtual DOM node is the return value of component's render function, the render function can nest other virtual [DOM] nodes in its return value.

A quick examples:
```javascript
// TreeNodeAttributeList :: [Attribute] -> div
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

// TreeNodeAttributeList :: [Attribute] -> div
var TreeNodeAttributeList = React.createClass({
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

##### Data Binding
<!--  TODO: remove konunktiv -->
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

##### Why the chosen approach will work
After trying different solutions it turned out the a declarative solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon the first version of it suffocated under it's own complexity (probably also due to the incompetence of its user - me).
It turned out that MVC had serious flaws when trying to use it to describe an ever changing declarative scene-graph.
The naive solution would be to observe the X3D node and rerender the whole tree structure whenever it changed, that would also mean to rerender every time an attribute is changed.
To make that clear, that means rerendering everytime an object is moved with the mouse, since the translation is an attribute.
This thesis will not contain any benchmarks proving that manipulating the DOM from javascript is slow, rather than that it is left as an exercise to the reader to research it if she wants.
React solves this issue quite elegantly by creating the dom structure in javascript and diffing it with the [DOM], only applying the minimum of changes to the DOM to realize the corresponding result.
That made it possible to use the X3D node of the DOM as the only source of truth and minimize the state that needs to be kept to make the tree-view work.

### How to solve
