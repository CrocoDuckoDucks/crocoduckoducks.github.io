---
layout: post
title:  "Refining the Metal Bar Model"
date:   2020-02-15 20:00:00 +0100
categories: science physics opensource
---

Right, this episode was supposed to be published only a few weeks after the [last one](https://crocoduckoducks.github.io/science/physics/opensource/2019/06/09/elastic-modes-of-a-metal-bar.html). Clearly, things drifted a little bit...

Anyway, in the last episode we seen that our model solution for the modes of vibration of a metal bar wasn't looking particularly good for the highest mode. To understand this we have to step back a little and think about FEM some more.

FEM does not actually solve the PDEs that govern our model. That is too complicated. Instead, FEM sets up a simplified problem. The full machinery of how FEM works is beyond the scope of this post, but you can read it [here](https://www.comsol.it/multiphysics/finite-element-method) (as well as many other places on the internet, in more or less advanced ways). One thing we can say, though, is that for single physics problems, that is, problems described by a single PDE, FEM is guaranteed to converge to the PDE solution as the meshing is made denser and/or the higher the order of the mesh is. If you are not familiar with the concept of element order, we can explain it in few words. In FEM the field solved at the nodes (the points at which the mesh elements touch) is used to compute the field within the elements by making use a set of basis functions. The higher the order of the element, the more complex these functions, and hence the higher the accuracy of the model. In reality the highest order found in most applications is 2, meaning that the basis functions are quadratics. The linear elements are the ones most used.

So, the first port of call when our solution looks dodgy is the mesh: is it fine enough to model the system properly? In our case the model seemed to face some issues when solving for the twisting mode of the metal bar. If we look at our mesh, we will see that the cross-section of the bar is tiled by only 8 elements, the edges being fairly long (one quarter of the cross-section edge). In reality, we would like to have at least 10 elements along the cross-section edge, so that we can capture well all sorts of motions that involve the cross-section as a whole.

The more the better. However, keep in mind that the computational requirements of the model will go up the denser the mesh and the higher the order. Also, ElmerFEM (but also any other solver) will face numerical issues for very high density meshes, especially if they aren't linear. So, in reality, we need to compromise.

# Let's Remesh!

We can open up our Salome study from the previous episode. Then, let's go to the Mesh Module. If this seems strange, just re-watch the [meshing vide](https://www.youtube.com/watch?v=r6k66HekS3c) from the previous episode, **Meshing** section.

If you are editing the old Salome study, you can create a mesh alongside the old one. Otherwise just create a new study as in the video, but when you get to the point of creating the mesh, follow the procedure below.

We will be using a different mesh: a hexahedral mesh. We do this because our solid is a big hexahedron itself, and the nature of linear elasticity makes regular meshing of regular objects quite appropriate. Also, by selecting as a meshing parameter the segments per edge we will get a mesh that is denser in the cross-section than on the sides of the bar, meaning that our mesh will be refined to tackle our particular problem: no need to raise the computational costs by making the mesh denser everywhere! Finally, hexahedral meshes are very easy and fast to compute, so we get our improvements without needing to sit 15 minutes staring at Salome screen.

Once you are in the *Mesh* Module in Salome, do the following:
+ Select the *Solid_1* Geometry item from the *Object Browser* on the left.
+ On the upper Menu, select *Mesh* and then *Create Mesh*. A window will popup.
+ On the *3D* tab, select *Hexahedron (i,j,k)*.
+ On the *2D* tab, select *Quadrangle Mapping*.
+ On the *1D* tab, select *Wire Discretization*. Click the gear wheel to the right of *Hypothesis* and select *Number of Segments*. Type *20* in the the *Number of Segments* edit box and select *Equidistant distribution* in the *Type of distribution* list box. Click *OK*.
+ Click *Apply and Close*.
+ Now, right click your newly created mesh, which is sitting in the *Object Browser* and select *Create Groups from Geometry*. This will open up a new window.
+ While the new window is still open, select *Solid_1* and all the faces (*Face_1*, *Face_2*, ...) from the *Object Browser*. These will now appear in the *Elements* list in the new window. Click *Apply and Close*.
+ You can now right click the your mesh from the *Object Browser* and select *Compute*.
+ A window will pop up after the mesh computation has finished. Dismiss it.
+ Now, save your study (*File* > *Save*) and export your mesh. Right click on it from the *Object Browser* and select *Export* and then *UNV File*.

At this point you can use your newly exported mesh into Elmer just like explained in the previous episode, in the **Solver Setup and Solution** section.

Note that the mesh we calculated is linear, that is, the basis functions are linear. You can convert it to quadratic by right clicking on it from the *Object Browser* and selecting *Convert to/from quadratic*. However, I found that once the mesh gets a bit dense ElmerFEM will struggle and fail to find a solution due to denormal floating point values being generated by the calculation. So, I recommend keeping the meshes linear after a certain density, unless the particular solver actually works better with other kind of element orders (this is normally reported in the [Elmer Models Manual](https://www.nic.funet.fi/index/elmer/doc/ElmerModelsManual.pdf)).

Also, as mentioned, note that it is not universal that hexahedral meshes are better. Here, they are made better by the simplicity of both the domain and the PDE we are solving for. In general, tetrahedral meshes are the most flexible, allowing even mesh optimisation. So, if in doubt, go with tetrahedral.

# The Result

You can now follow along the previous episode's **Visualization and Post-processing** section to load the simulation results into ParaView and check them out. Let's compare the fifth eigenmode of our new solution with the fifth eigenmode of the previous solution.

![Fifth Mode Comparison]({{ site.baseurl }}/res/pictures/2020-02-15-refining-the-metal-bar-model/Figure_1.png  "Fifth Mode Comparison")

Looks like we were successful! In our refined mesh results we do not see any nasty unexpected displacement field discontinuity, or dodgy bubbles in the middle of the domain, a sign that now that we have way more than 10 elements across the edges of the cross-section things are way more realistic.

# Conclusion

In general, when you expect your field to vary significantly along a certain length, your mesh size should be lower than a tenth of that length. But it doesn't have to be that small everywhere! If you know, from a simplified analytical model or from a simplified FEM run with a coarse mesh, that the field will vary significantly only along a certain direction you can have your mesh to be denser along that direction only and coarser along the others. The FEM method is powerful also due to this flexibility.

Clearly, this is not an exhaustive explanation about how to mesh your domain properly. In fact, you probably noticed how many advanced options the Salome *Mesh Module* has. However, we now know what to look at first every time things smell fishy. We also know that meshing is, in reality, the most important aspect of solving a FEM problem: bad mesh equals bad solution.

On the next episode we will be moving out from vibration and we will simulate some simple acoustics!
