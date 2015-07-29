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
