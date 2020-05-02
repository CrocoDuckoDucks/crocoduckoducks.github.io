---
layout: post
title:  "Home Studio - 1"
date:   2020-04-25 20:00:00 +0100
categories: science physics opensource
---

In the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) episode we seen how to model a rectangular room with rigid walls. We driven the room at the modal frequencies and compared the solution field with the theoretical modal shapes, finding that the results matched single modal shapes real well until, at a frequency high enough, the contribution of multiple modes (in addition to that related to the driving frequency) became important. In this episode we will look at making the model more realistic. To do so, we will investigate the **low frequency** response of a home studio.

{% include mathjax.html %}

# Getting less Step by Step

So far all episodes involved detailed step-by-step instructions. This makes sense as I want this series to act as a tutorial, and some explanation is needed as these tools are not very straightforward to learn. However, the previous episodes cover the basics in great detail, so we can avoid repeating the procedure to setup geometries, meshes and solver and cut to the chase. From this episode on step by step guidance will be given only when dealing with something not already covered in the previous episodes.

# GitHub Repository
See [here](https://github.com/CrocoDuckoDucks/Acoustic-Models/tree/master/2020-04-25-home-studio-1).

# How to add Complexity

The best way to add complexity to a model is through incremental steps, going from simple to complex. If we start off a model with a high complexity, and look at the solution, how do we understand that it makes sense? Could we have made an error in setting one of the many parameters? And what is the impact of the many parameters and features of the model in isolation? These, and many more questions, become tricky to answer when everything is entangled in a final field.

So, we should do something different. We already seen how to model the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html), one of the simplest systems in acoustics (and already a pretty complex one). Then we should proceed with baby steps. For example, something like this would be nice:

1. Make the shape more realistic, putting an omni-directional uniform velocity source in the room but keeping the walls rigid.
2. Modify the above and make the impedance of the surfaces (walls, floor, ceiling, doors and windows) more realistic.
3. Modify the above and simulate the behaviour of furniture.
4. Modify the above to simulate the behaviour of acoustic treatment for the room.
5. Modify the above to include more realistic sound radiators.

The list could go on, and actually many steps can be broken in many substeps, but this should give you the idea.

# A Note for Arch Users

Looks like Salome Platform 9.3.0 does not like the latest mesa version. In order to work with it, I had to downgrade to `mesa-19.3.4-2`. So, if you get this kind of error when opening the _Geometry_ module:

```text 
OpenGL_Window::CreateWindow: glxCreateContext failed.
```

you can try to downgrade your mesa. For guidance about downgrading packages, refer to [the Arch Linux wiki](https://wiki.archlinux.org/index.php/downgrading_packages).

# Wait, Low Frequency Response?

Yes, for the time being. The reason is that we are solving for the acoustic field inside a large room. Now, remember our rule for the maximum mesh size given the frequency at which we are running the simulation:

<p style="text-align: center;">$$s<\frac{\lambda}{10}=\frac{c_{0}}{10f}$$</p>

We can plot it as a function of frequency (assuming $$c_{0}$$ to be 343 meters per second):

![Maximum Mesh Size VS Simulation Frequency]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_1.png  "Maximum Mesh Size VS Simulation Frequency")

Note that the scale of both axes is logarithmic. As you can see, the maximum mesh size rapidly drops with frequency. This means that more elements will be needed for higher frequency. At 1 kHz the maximum mesh size is already as small as 3.4 centimetres. This is a very small number, and solving for a realistic room size will be very hard. For the room we will be considering, this amounts to a total number of elements in the order of 6 millions! Solving for such a system is pretty computationally intensive. For this reason, when it comes to acoustics, Finite Element Analysis (FEA) for large volumes (such as rooms, for example) is rarely employed at frequencies above 1 kHz. Ray tracing methods are preferred at high frequencies instead.

So, why not to just use ray tracing? Well, ray tracing works in the so called optical approximation of acoustics, that holds at high frequency but not at low frequency. Wave methods, such as FEA, are then preferred ones at low frequency.

## Frequencies Under Study

We will be solving for the steady state acoustic field inside our room at a number of different frequencies. Hence, our mesh size should allow for good accuracy also at the highest frequency of the study. However, the plot above shows to us that a fine mesh will be overkill for frequencies that are significantly lower than the maximum frequency for which the mesh is tuned. Hence, it will be best to split the study in frequency ranges.

In the following study we will sweep across the [third octave nominal centre frequencies](https://en.wikipedia.org/wiki/Octave_band#One-third_octave_bands) up to 400 Hz. We will do so by using two meshes as follows:

|Mesh Name     | Mesh Size  | Accurate up to | Frequency Range of Usage          |
|--------------|------------|----------------|-----------------------------------|
| mesh_125_hz  | 274.40 mm  | 125 Hz         | from 16 Hz to 100 Hz              |
| mesh_500_hz  | 68.60 mm   | 500 Hz         | from 125 Hz to 400 Hz             |

Notice how we will not use one mesh up to the maximum third octave centre frequency it can work with, but the one just below. We do this as a sort of "safety margin" for accuracy. The meshes above will be prepared in the usual way (_NETGEN 1D-2D-3D_ algorithm and _Second Order_ elements) but we will see that we will face convergence issues. But let's not get ahead of ourselves.

# Geometry

![Geometry]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_2.png  "Geometry")

This time I will not cover the geometry of the system in a step by step fashion. If you are new to FreeCAD you can refer to the previous episodes. If you need help to model your room in FreeCAD, refer to the online FreeCAD resources (such as the [forum](https://forum.freecadweb.org/) and [documentation](https://wiki.freecadweb.org/Getting_started)). You can also refer to the FreeCAD file included in the [repository](https://github.com/CrocoDuckoDucks/Acoustic-Models/tree/master/2020-04-25-home-studio-1) for guidance.

In the previous episodes we made a CAD model of the air enclosed within the room, and only that. In this episode we will model the air of the room but we will put another 3D object inside it, to act as our source. In the model of the repository the room volume is generated by tracing a 2D plant of a room. Then, two surfaces, flush with the walls, are used to act as doors. You can model your room as you like, and once you have done you can put a sphere anywhere inside it. This sphere will act as our omni-directional radiator. Once you have done, select both the room and the sphere object (as shown below) and export this selection as _BREP_.

![Solids to Export as BREP]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_3.png  "Solids to Export as BREP")

# Meshing

As always, you can import your _BREP_ file, that now contains more than one solid, inside the _Geometry_ module in Salome. You can now proceed to explode the imported geometry into _Solid_ entities, as always. However, there is now an important thing to consider.

We have two 3D objects in our geometry. We must make sure that they are meshed properly, with "continuity", that is, the mesh of one fades with continuity into the mesh of the other, without gaps. Also, the surface between the two 3D objects must "belong" to both objects, so that it can be used to apply boundary conditions to the both of them or, in multi-physics problems, to act as a _Structure Interface_. This is a so called _conformal_ mesh, and the way we do this is through domain partition.

To do a partition select, among the solids we just exploded from the original geometry, the 3D object representing the room (_Solid_2_ in my case) and proceed as following:

* From the top menu, choose _Operations_ and then _Partition_. This will open a new window.
* In the new window, click the arrow on the left of _Tool Objects_ and select the sphere source solid (_Solid_1_ in my case), as shown below.

![Partition]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_4.png  "Partition")

This will create a partition object, which has two solid sub-entitites. You can now proceed and explode each one of these sub-entities in faces. The hierarchy of results is shown below:

![Objects Hierarchy]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_5.png  "Objects Hierarchy")

So, to sum up:

* We imported the file `geometry.brep`, thus creating the _geometry.brep_1_ object.
* We exploded _geometry.brep_1_ into two solids, _Solid_1_ and _Solid_2_ (see the previous episodes for instructions on how to explode things).
* We created a partition from _Solid_2_ (the room volume) by using _Solid_1_ (the source model) as a tool, resulting in the partition _Partition_1_.
* We exploded each of the solids in _Partition_1_ in faces. You will note that the spherical surface enclosing the sphere source appears under _Solid_3_ (which is the sphere) **and** _Solid_4_ (which is the room). This is the crucial bit.

Good, so now we only have to do the mesh(es). Switch to the _Mesh_ module, and make sure you select the partition object from the _Object Browser_ (as shown in the figure above). Then define your mesh in the usual way (see to the previous episodes for guidance). We will use our old trustworthy _NETGEN 1D-2D-3D_ algorithm with the maximum sizes defined above, a minimum size of 0.01 mm, _Fine_ fineness and make sure it is _Second order_ (see below for an example). Remember that Salome wants the sizes to be specified in millimetres.

![Meshing Parameters Example]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_6.png  "Meshing Parameters Example")

Now, make sure that once the mesh is setup, you create _Create Groups from Geometry_ in the usual way. But we will need to include in the groups **all** the sub-entities of our partition, as shown below.

![Groups from Geometry]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_7.png  "Groups from Geometry")

The solid entities will become _Groups of Volumes_ and the face entities will become _Groups of Faces_, as shown below.

![Resulting Groups]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_8.png  "Resulting Groups")

We then finally compute and export our mesh as _UNV_. When we load the _UNV_ file into ElmerFEM then ElmerFEM will treat the _Groups of Volumes_ as bodies and the _Groups of Faces_ as boundaries. This means that we will be able to apply different governing equations to different bodies, and couple equations, but we will not do this today. We will leave the sphere itself ungoverned (which will generate various warnings when we solve), and just apply an uniform velocity boundary condition, to its surface, in the usual way.

Below is a picture of the resulting mesh as visualised with Salome. You can see that it looks pretty conformal.

![Conformal Mesh]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_9.png  "Conformal Mesh")

# Solving

This largely goes like the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html), so I will just list the model setup briefly, using the study from 16 Hz to 100 Hz as an example.

## Model Setup

* Put the following in the _Free text_:

```text
$ f = 16 20 25 31.5 40 50 63 80 100
$ p = 1.205
$ U = 10
```

`f` is the vector of third octave centre frequencies we will be solving for, `p` is the density of air at room temperature, and `U` is the surface velocity of our sphere source.

* Set the _Simulation type_ to _Scanning_.
* Set the _Timestep intervals_ to the number of elements in `f` (_9_ in this case).
* Set the coordinate scaling to _0.001_, as the coordinates in our mesh are millimetres instead of meters.

## Equation

Add an Helmholtz equation in the usual way (see the previous episodes if you need guidance). Ensure that it is applied to the correct body, i.e. the room volume. One way to do so is to choose _Model_ > _Set body properties_ and double click on any surface of the room. This will allow to assign the equation to it.

Also, since we will be solving at many frequencies, I suggest you tick _Abort if the solution did not converge_ in the _Linear system_ solver settings, as shown below. This will stop the solver as soon as it cannot converge to a solution, so that we do not risk to go great lengths analysing a solution that is, most likely, not that good.

For low frequency a value of _Max. iterations_ of 500 will be OK, but after 125 Hz you will see the study having an harder time converging to a stable solution. With 2000 iterations (as shown below) you should be able to solve up to 315 Hz (see the next section for more observations).

![Abort if Not Converged]({{ site.baseurl }}/res/pictures/2020-04-25-home-studio-1/Figure_10.png  "Abort if Not Converged")

## Material

Add _Air (room temperature)_ in the usual way. Remember to put this MATC expression for _Density_:

```text
Real MATC "p"
```

## Boundary Condition

Add 2 boundary conditions, a rigid wall and an uniform velocity radiator.

The rigid one has 0 flux, for both real and imaginary parts.

The radiator has this MATC expression for the imaginary part of the flux:

```text 
Variable time; Real MATC "2 * pi * f(tx - 1) * p * U"
```
To apply them, I suggest you choose _Model_ > _Set boundary properties_. Then, double click all the external boundaries of the room (including the doors) and set them to rigid. Finally, open the boundary condition editor again for the radiator condition. There should be only one boundary left to which you can apply it.

Cool! Now you can press the _Start solver_ button and sit back while the solver crunches out numbers.

## Sif Files

You can see my sif files [here](https://github.com/CrocoDuckoDucks/Acoustic-Models/blob/master/2020-04-25-home-studio-1/elmerfem_16_to_100_hz/case.sif) (16 hz to 100 Hz) and [here](https://github.com/CrocoDuckoDucks/Acoustic-Models/blob/master/2020-04-25-home-studio-1/elmerfem_125_to_400_hz/case.sif) (125 Hz to 400 Hz) to check that you setup the model the same way I did.

# Results

You will see that when we deal with frequencies lower than 125 Hz ElmerFEM can converge to a solution pretty easily (less than 30 iterations). However, things get quite harder to deal with at higher frequency. In fact, I could not get the solution at 400 Hz to converge at all!

But first, let's have a look.

![Results](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-04-25-home-studio-1/anim.gif  "Results")

The animation above, prepared with ParaView (see [here](https://crocoduckoducks.github.io/science/physics/opensource/2020/04/11/intro-to-paraview.html) for an introduction and few tricks) shows the steady state Sound Pressure Level (SPL) in the room for each of our study frequencies, with the exception of 400 Hz, since the solver could not converge there. The room is shown transparent, and two slices are operated in the domain, one vertical and one horizontal, passing through the source. The curves inside the room are curves of constant SPL.

Well, clearly a surface velocity of 10 meter per second is pretty crazy, and yields pressures up to 130 Hz. But we can see that the steady state field grows in complexity the higher the frequency. Few features are seen at 16 Hz. In fact, all the SPL variation is concentrated close to the source, and the rest of the air has an pretty uniform SPL value. But at high frequencies we start seeing many nodal surfaces of very complex shape, cutting zones of high pressures. This complexity of the field is one of the causes of the convergence issues at high frequency.

# Conclusion

We seen in this episode how to create models with more than one body. We also seen how computationally intensive it is to solve for high frequency problems, due to the mesh size needing to me smaller, and we seen that the field becomes more complex the higher the frequency. Finally, we seen that convergence at high frequency is more complicated to achieve. So, before we dig into the study of the results some more, and go ahead in adding additional complexity, we should stop for a moment and investigate these two issues:

* What can we do to aid convergence?
* What can we do to shorten computation times?

So, we will look at the problem of convergence in the next episode!

