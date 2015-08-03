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

#### states
TODO: explain the DOM

Using something imperative like backbone it would be necessary to create a model (copying the information already in the DOM), a view (rendering that information to the DOM again) and a controller keeping track of the state changing the model where necessary and rerendering the view, and also keeping track of all it's children and removing them when they disappear or creating new ones whenever a new X3D element appears.

#### Functional Programming
All SceGraToo does is listen to changes of the X3D node and whenever it changes, may it be an attribute that changes or nodes that are added or removed, it calls one function that evaluates the X3D node (basically traversing it) and return the new structure.
That structure is diffed against the tree-view that is already in the DOM and react calculates the minimal changes necessary to make the tree-view's old structure coherent with the newly calculated one.


### Koa
Koa is pretty much boring unless I'd explain node.js's take on asynchronicity, callbacks, promises and then generators. Still, the server is way to simple to be of any interest for this project.

### Related Work

#### Collaborative Work
#### [3D Meteor][0]
![3dmeteor](https://www.dropbox.com/s/173i8xo8xkxdmq7/3dmeteor.png?dl=1)

#### [Blender Plugin][10]


#### 3D Widgets
##### [Tilt Brush][20]
Tilt Brush was lauded for t

#### x3d gizmos

#### [Component Editor][30]

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
