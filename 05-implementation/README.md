## Implementation
### Used Tech

#### angular
* dependency management
* resource management
* less dynamic templates

#### react
* highly dynamic views (like the tree view)

### Problems

#### synchronization Process
This design leads to brittle code that is hard to maintain and hard to adapt to new use cases.

The first culrpit in this design is that the tree-view-controller mutates itself.

Let's externalize the mutation observation and update capabilities into another actor: a mutation observer.

<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/externalObserver.md.svg></object>

The mutation observer can map each graph node to its corresponding tree node and thus change the tree node to represent the matching graph node again.

The tree-view-controller does not mutate itself anymore.
All the mutation logic that synchronizes the tree-view-controller with the scene-graph is in the mutation observer.
But the scene-graph and the tree-view-controller can still diverge, since moving the mutation logic from the tree-view-controller to the mutation observer does not make it easier to reason about and thus less error prone.

Assuming that the first operation (traversing the scene-graph and creating the tree-view-controller) is correctly implemented and resilient, the easiest way to make sure the tree-view-controller and the scene-graph are in sync would be to recreate the tree-view-controller whenever the scene-graph changes.

<object data="https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/rerenderOnChange.md.svg"></object>

At first sight that seems to be awfully slow, that needn't to be true.
Instead of recreating the the whole tree using [DOM] elements, a model could be created using simple javascript objects.
This virtual tree-view-controller can then be compared to the tree-view-controller already in the [DOM] and only the changes need to be applied.

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

This is code is the a simplified version of the code that renders a graph's nodes attributes into the tree-view-controller. This component simply decides what attributes should be rendered into the tree. `TreeNodeAttribute` is another component that will renders different elements depending on what attribute is passed in.

Because the outputted html is only a function of its input it easy to parse the scene-graph:
1. choose a graph node as the root
2. call the node component with that graph node
3. if the graph node has child nodes call the node component again with each child node and return their return values wrapped in an element
4. if the graph node has no children return an empty element

##### Why the chosen approach will work
After trying different solutions it turned out the a functional solution would also suit SceGraToo the best.
First Chaplin (a backbone successor) was used to implement SceGraToo, but soon the first version of it suffocated under it's own complexity (probably also due to the incompetence of its user - me).
It turned out that MVC had serious flaws when trying to use it to describe an ever changing declarative scene-graph.
The naive solution would be to observe the X3D node and rerender the whole tree structure whenever it changed, that would also mean to rerender every time an attribute is changed.
To make that clear, that means rerendering every time an object is moved with the mouse, since the translation is an attribute.
This thesis will not contain any benchmarks proving that manipulating the DOM from javascript is slow, rather than that it is left as an exercise to the reader to research it if she wants.
React solves this issue quite elegantly by creating the dom structure in javascript and diffing it with the [DOM], only applying the minimum of changes to the DOM to realize the corresponding result.
That made it possible to use the X3D node of the DOM as the only source of truth and minimize the state that needs to be kept to make the tree-view-controller work.

A solution that handles state manually can work, but requires more discipline and a more complicated mental model.
The programmer has to keep all side effects and cascading effects in mind when changing parts of the program.
