[toc]
# SceGraToo
## Prelude
The goal of the thesis is to create a 3d editor, using web technologies, which enables the user to postprocess x3d scenes. These scenes are the result of a tool which was implemented as part of the DFG research project [Roundtrip3D](http://elrond.informatik.tu-freiberg.de/Roundtrip3D/).

### Motivation
Roundtrip3D was research project that resulted in an editor implementing [SSIML](http://edoc.ub.uni-muenchen.de/9459/). SSIML is a
The result of the project Roundtrip
As part of the Roundtrip3D project an application (hereafter referred to as *R3D*) was built containing an editor that enabled the user to describe 3D scenes in a visual UML-like language. Then the R3D can be used to generate boilerplate code for multiple programming languages and an X3D file describing the scene. The X3D file may contain references to other X3D files containing the actual 3D data (e.g. a car and the corresponding tires), hereafter called *inlines* . These files are created by exporting objects from a 3D computer graphics software (e.g. [Blender](https://www.blender.org/), [Maya](http://www.autodesk.com/products/maya/overview) or [3DS Max](http://www.autodesk.com/products/3ds-max/overview)).  
The problem that arises is that each object is usually in the center of it's own coordinate system. So they need to be translated, rotated and maybe even scaled to result in desired scene structure (e.g. a car where the tires are in the places they belong to and not in the center of the chassis).  

![Picture of a car with the tires in the center]()  
![Picture of a car with the tires where they belong to]()  


This can be achieved by adding *translate*, *rotate* and *scale* properties to the transformation node containing the individual objects.
To exacerbate this problem further, the orientation the 3d artists chose for it's object is simply unknown. The following pictures demonstrate the problem:
![wheel1](https://www.dropbox.com/s/xhs8gbur6pa8lgd/wheel1.png?dl=1)
![wheel2](https://www.dropbox.com/s/0gyfkmpam5o8riq/wheel2.png?dl=1)
![wheel3](https://www.dropbox.com/s/gg362w2h9p93wdz/wheel3.png?dl=1)  
These are common orientations, since it's disputable which of these would be normal. But if one depends on art from 3rd parties, the orientation and position could be completely arbitrary:
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
### Roundtrip 3D
Roundtrip3D was research project that resulted in an editor implementing [SSIML](http://edoc.ub.uni-muenchen.de/9459/). SSIML is a
### X3D
#### x3dom
### react
#### states
#### functional programming
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
