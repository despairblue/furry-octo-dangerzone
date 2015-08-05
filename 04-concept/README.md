<!-- TOC depth:6 withLinks:1 updateOnSave:1 orderedList:0 -->

  - [Concept](#concept)
    - [How](#how)
      - [Server](#server)
      - [Client](#client)
      - [Interaction](#interaction)
    - [Problems](#problems)
      - [Synchronization Process](#synchronization-process)
      - [Easy Synchronization to a Model](#easy-synchronization-to-a-model)
        - [Terminology](#terminology)
        - [Data Binding](#data-binding)
    - [How to solve](#how-to-solve)
<!-- /TOC -->

## Concept

skizze wie dom rendering funktioniert

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
- Hoare, Turing Award Lecture 1980

React is utilized by SceGraToo to render the tree-view-controller that gives a more structured view of the scene-graph than the rendered scene.

The scene-graph is the most important part of SceGraToo, it shows the the structure rather than the visual representation.
Different off-the-shelf solutions, like angular or JQuery plugins, were tested against theses requirements:

1. custom elements as part of tree nodes (multiple checkboxes or multiple inputs)
2. ability to listen to changes to tree nodes
3. binding to an arbitrary model, that can recover inconsistent state

with mixed results:

**Partially not met requirements:**

* custom elements as part of tree nodes
* ability to listen to changes to the tree node

**Requirements none of the tested tools met:**

* binding to an arbitrary model, that can recover inconsistent views

None of the off-the-shelf solutions could satisfy expectations.

#### Easy Synchronization to a Model
The most complicated part is keeping the tree-view-controller in sync with the scene-graph while the scene-graph is being modified and vice versa.

##### Terminology

**scene-graph:**  
The HTML/XML/X3D representation of the scene:
```html
<x3d version="3.0" profile="Interaction" width="708px" height="354px">
  <!-- id=69b81d54-6e7a-4967-acca-b8c89ba90782 -->
  <scene render="true" bboxcenter="0,0,0" bboxsize="-1,-1,-1" pickmode="idBuf" dopickpass="true">
    <worldinfo def="generatedWorldInfo1" title="Orgel" info="">
    </worldinfo>

    <background skycolor="0.3 0.3 0.3" groundcolor="" groundangle="" skyangle="" backurl="" bottomurl="" fronturl="" lefturl="" righturl="" topurl=""></background>

    <viewpoint def="V0" fieldofview="0.7" position="-1.9504748689304945 1.180349823049051 -3.724484991086488" orientation="0.10730618820578387 0.9854647428763489 0.13169898450784276 3.8513980449760714" centerofrotation="0 0 0" znear="-1" zfar="-1">
    </viewpoint>

    <!-- id=8d3f0a8a-b6d7-4acc-922b-ea59364443fa -->
    <group def="G1" render="true" bboxcenter="0,0,0" bboxsize="-1,-1,-1">
      <transform def="generatedTransform3" scale="1 1 1" rotation="0 0 1 0" translation="0 0 0" render="true" bboxcenter="0,0,0" bboxsize="-1,-1,-1" center="0,0,0" scaleorientation="0,0,0,0">
        <!-- id=8459b736-5a9d-4688-b624-e519857a92fd -->
        <inline def="O1_3" namespacename="O1_3" url="projects/Red Box/src/redBox.x3d" render="true" bboxcenter="0,0,0" bboxsize="-1,-1,-1" load="true">
          <Shape render="true" bboxCenter="0,0,0" bboxSize="-1,-1,-1" isPickable="true">
            <Appearance sortType="auto" alphaClipThreshold="0.1">
              <Material diffuseColor="1 0 0" ambientIntensity="0.2" emissiveColor="0,0,0" shininess="0.2" specularColor="0,0,0"></Material>
            </Appearance>
            <Box solid="true" ccw="true" useGeoCache="true" lit="true" size="2,2,2"></Box>
          </Shape>
        </inline>
      </transform>
    </group>
  </scene>
</x3d>
```

**tree-view-controller:**  
Loosely defined term that comprises of all objects, methods, functions, event-listeners and callbacks related to parsing the scene-graph and creating the initial  tree-view-view.

**tree-view-node-controller:**
Loosely defined term that comprises of all objects, methods, functions, event-listeners end callbacks related to keeping a subtree of the scene-graph uptodate.

**tree-view-view:**  
The HTML representation of the tree-view-controller (html output shortened and simplified):  
![treeview](https://www.dropbox.com/s/985tkrkpx67nyuk/treeview.png?dl=1)
```html
<div>
  <li>
    <div>
      <div></div>
      <a>
        <span></span>
        <span>SCENE</span>
      </a>
    </div>
    <div>
      <div>
        <div>
          <div>
            <span>render</span>
            <span>:</span>
          </div>
          <div>
            <input type="checkbox">
          </div>
        </div>
      </div>
      <ol>
        <li>
          <div>
            <div></div>
            <a>
              <span></span>
              <span>WORLDINFO</span>
            </a><a>X</a></div>
          <div>
            <div>
              <div>
                <div>
                  <span>def</span>
                  <span>:</span>
                </div>
                <div>generatedWorldInfo1</div>
              </div>
            </div>
            <ol></ol>
          </div>
...
      </ol>
    </div>
  </li>
</div>
```

The aim is to keep the tree-view-view a consistent representation of the scene-graph, where interesting scene-graph nodes and their properties are presented in an up to date and editable form.

First design approach is to let the tree-view-controller create an initial tree-view-view by traversing it and creating tree-view-node-controllers ad hoc.
These tree-view-node-controllers create the corresponding tree-view-node-views while traversing the scene-graph.
For each scene-graph node create a corresponding tree-view-view node.
Each tree-view-node-controller observes its corresponding scene-graph node for attribute mutations and added or removed child nodes and change it's view accordingly.

Depending on how the scene-graph is mutated 3 main cases can be differentiated.
1. a scene-graph node is added  
  ↪️ a new tree-view-node-controller is instantiated and renders a new tree-view-node-view
2. a scene-graph node is deleted  
  ↪️ the corresponding tree-view-node-controller is destroyed
3. a scene-graph node is mutated  
  ↪️ the corresponding tree-view-node-view is altered

<!-- 1\. and 2. would be handled by the tree-view-controller itself whereas 3. would be handled by each node respectively. -->

TODO: remove or update
<object data=https://rawgit.com/despairblue/furry-octo-dangerzone/master/assets/naiveTreeView.md.svg></object>  
<sup>the dotted lines represent events, the solid one method calls</sup>

Tree-view-node-views can also be used to edit a show scene-graph node's properties.  
When a tree-view-node-views is edited it's tree-view-node-controller is notified and applies the new properties to the corresponding scene-graph node.
It's assumed that the updates will always lead to consistent state, where the scene-graph and the tree node converge.

The synchronization process has no ability to detect if updates lead to a consistent state.
It also has no ability to recover from that state, though without the ability to detect inconsistencies this does not really matter.

**Problem 1:** keeping the tree-view consistent with the scene-graph
The difficulty to make sure that the incremental updates are error-free exacerbated even more when further functionality gets added to the tree-view.

**Problem 2:** implementation effort  
For every new functionality 4 things have to implemented:  
1. code for parsing the the scene-graph
2. code to generate the tree-view-node-view
3. code to synchronize changes from the scene-graph to the tree-view-node-view
4. code to synchronize changes from the tree-view-node-view to the scene-graph

This design bears more complexity than reparsing and regenerating the complete tree-view-view on every change would.

Problem 1 is gone and problem 2 is reduced to two steps:  
1. code for parsing the scene-graph and generating the tree-view-node-view
2. code to synchronize changes from the tree-view-node-view to the scene-graph

Rerendering everything on every change is usually inefficient.
React is used to circumvent this problem.
React calculates a lightweight representation of what is going to be rendered to the DOM and compares that to what is already rendered.
It calculates a set of patches and only applies these to the DOM.

That means as long as the code that parses the scene-graph and generates the lightweight representation is correct, the tree-view-view is correct.



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
The tree-view-controller could be bound to the same model and should thus update automatically respectively all updates to the tree-view-controller would be propagated to the model again:
It would probably work, but the complexity is too daunting.
* nesting directives/components indefinitely deep makes it harder to reason about them.
* the X3D scene couldn't be dumped just be dumped into the [DOM] it would have to be parsed and componentized

<!-- What that means is to traverse the X3D scene, create a tree node for every eligible part of the scene-graph and hook up event listeners that update that tree node when the scene-graph node changes and the other way around, update the scene-graph when the tree node is updated. -->

<!-- Even if one of them would have worked the main problem would have remained: How to keep the tree-view-controller in sync with the [DOM]? -->



### How to solve
