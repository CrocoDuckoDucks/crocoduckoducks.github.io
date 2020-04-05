---
layout: post
title:  "Rigid Walled Room"
date:   2020-03-28 20:00:00 +0100
categories: science physics opensource
---

In the [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html) episode we explored the analytical model of a rigid walled room with some Julia code. We focused on finding the resonance frequencies (or eigenfrequencies) of the room and calculating the related modal patterns (eigenfunctions). Now that we know how to setup Helmholtz problems with ElmerFEM we can approach the problem with the FEM method. In this episode we will solve for the modal patterns of a rectangular rigid walled room and compare their accuracy to that of the analytical solution we discussed in [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html). We will see that this is actually hard to do directly, as ElmerFEM does not have an Eigensolver, but we will obtain useful information anyway!

{% include mathjax.html %}

# GitHub Repository
All the code and models for this episode are stored under the folder [2020-03-28-rigid-walled-room](https://github.com/CrocoDuckoDucks/Acoustic-Models/tree/master/2020-03-28-rigid-walled-room) of the [Acoustic-Models](https://github.com/CrocoDuckoDucks/Acoustic-Models) repository. The contents of the directory are shown below.

```text
2020-03-28-rigid-walled-room/
├── 1_vs_2_anim.avi
├── 1_vs_2_anim.gif
├── 1_vs_2.pvsm
├── elmerfem_1
│   ├── case0001.vtu
│   ├── case0002.vtu
│   ├── case0003.vtu
│   ├── case0004.vtu
│   ├── case0005.vtu
│   ├── case0006.vtu
│   ├── case0007.vtu
│   ├── case0008.vtu
│   ├── case0009.vtu
│   ├── case0010.vtu
│   ├── case.sif
│   ├── egproject.xml
│   ├── ELMERSOLVER_STARTINFO
│   ├── Mesh_1.unv
│   ├── mesh.boundary
│   ├── mesh.elements
│   ├── mesh.header
│   ├── mesh.nodes
│   ├── netgen.prof
│   ├── solver.log
│   └── solver.png
├── elmerfem_2
│   ├── anim.avi
│   ├── anim.gif
│   ├── case0001.vtu
│   ├── case0002.vtu
│   ├── case0003.vtu
│   ├── case0004.vtu
│   ├── case0005.vtu
│   ├── case0006.vtu
│   ├── case0007.vtu
│   ├── case0008.vtu
│   ├── case0009.vtu
│   ├── case0010.vtu
│   ├── case.sif
│   ├── egproject.xml
│   ├── ELMERSOLVER_STARTINFO
│   ├── Mesh_2.unv
│   ├── mesh.boundary
│   ├── mesh.elements
│   ├── mesh.header
│   ├── mesh.nodes
│   ├── netgen.prof
│   ├── postprocessing.pvsm
│   ├── solver.log
│   └── solver.png
├── error
│   ├── error_0.csv
│   ├── error_1.csv
│   ├── error_2.csv
│   ├── error_3.csv
│   ├── error_4.csv
│   ├── error_5.csv
│   ├── error_6.csv
│   ├── error_7.csv
│   ├── error_8.csv
│   ├── error_9.csv
│   └── postproc.pvsm
├── error_anim.avi
├── error_anim.gif
├── error_norm.png
├── exact.jl
├── example.jl
├── export
│   ├── data_0.csv
│   ├── data_1.csv
│   ├── data_2.csv
│   ├── data_3.csv
│   ├── data_4.csv
│   ├── data_5.csv
│   ├── data_6.csv
│   ├── data_7.csv
│   ├── data_8.csv
│   └── data_9.csv
├── frequencies
│   ├── f_10.csv
│   ├── f_1.csv
│   ├── f_2.csv
│   ├── f_3.csv
│   ├── f_4.csv
│   ├── f_5.csv
│   ├── f_6.csv
│   ├── f_7.csv
│   ├── f_8.csv
│   └── f_9.csv
├── geometry_1.brep
├── geometry_2.brep
├── geometry.FCStd
├── geometry.FCStd1
├── Mesh_1.unv
├── Mesh_2.unv
├── meshing.hdf
├── modal_behaviour.png
├── postproc.pvsm
└── validate.jl

```

The study was performed with two different source locations (see below for details). The suffixes `_1` and `_2` denote files for source in the middle and source in the corner respectively.

The folders `elmerfem_` contain the ElmerFEM projects.

The folder `error` contains the error analysis output performed by the Julia script `validate.jl`.

The folder `export` contains data exported from the `_2` solution by using ParaView, to be used for error analysis.

The folder `frequencies` contains ParaView importable frequency arrays useful to annotate frequency in the animations.

The various _pvsm_ files are ParaView state files containing various postprocessing.

The _gif_ files are exported ParaView animations.

The `error_norm` _png_ file contains a plot of the modal error.

`exact.jl` contains a Julia implementation of functions to calculate the theoretical modes.

`example.jl` is a Julia script showing how to use the methods defined in `exact.jl`.

`geometry.FCStd` and `geometry.FCStd1` are FreeCAD files defining the model geometry.

The `geometry_` _brep_ files contain the exported geometry for the simulations.

The `Mesh_` _unv_ files are the meshes for the study.

`meshing.hdf` is the Salome study.

`modal_behaviour.png` is a plot of the analytical modal behaviour of the room.

Finally, `validate.jl` is a Julia script that compares the data stored in the `export` files to the exact solutions and produce comparison plots.

Note that to run `validate.jl` you will need to install the Julia [Plots package](https://docs.juliaplots.org/latest/).

# Approach
Ideally, we would use a Helmholtz Eigensolver to find the resonance frequencies and their associated modal patterns. Unfortunately ElmerFEM does not have a Helmholtz Eigensolver (at the time of writing), so we will have to approach the problem in a different way. We will:

1. Calculate the expected resonance frequencies with Julia.
2. Stimulate our room model at the various resonance frequencies.
3. Compare the results with the modal patterns.

Point 2 implies that we will need an active flux boundary condition on our solution, so we will need to place a source in our room. We will use a small pulsating sphere with uniform displacement in the corner of the room (spoiler: the placement of the source is important).

Point 3 assumes that the response of the room, when driven exactly at a certain frequency, is the related modal shape. In reality this is wrong, as the response is instead a _modal superposition_. We will see where this assumption breaks down.

# The Room
The parameters of the room are reported below:

## Room Geometry

| Name   | Symbol  | Value    |
|--------|---------|----------|
| Length | $$L_x$$ | 5 meters |
| Width  | $$L_y$$ | 4 meters |
| Height | $$L_z$$ | 3 meters |

## Medium Properties

|Parameter Name              | Symbol       | Value | Unit                      |
|----------------------------|--------------|-------|---------------------------|
| Medium Sound Phase Speed   | $$c_{0}$$    | 343   | meters per second         |
| Medium Equilibrium Density | $$\rho_{0}$$ | 1.205 | kilograms per cubic meter |

## Sphere Source Parameters

|Parameter Name               | Symbol       | Value                              | Unit   |
|-----------------------------|--------------|------------------------------------|--------|
| Source Radius               | $$a$$        | 0.005                              | meters |
| Source Frequency            | $$f$$        | 10 different resonance frequencies | hertz  |
| Source Surface Displacement | $$d$$        | 0.001                              | meters |

# Building the FEM Model

## Geometry
As always, let's open FreeCAD and follow the procedure below:

* Select _File_ from the top menu and then _New_.
* Switch to the _Part_ _Workbench_.
* In the toolbar you should see a button with a yellow cube. Its tooltip says _Create a cube solid_. Click on it. This will create a cube.
* On the _Combo View_ on the left, select the newly created cube. On the bottom you will see the cube properties. Input the following properties:
    * Under _Box_:
        * _Length_: 5000 mm
        * _Width_: 4000 mm
        * _Height_: 3000 mm
    * Under _Base_ > _Placement_ > _Position_ (we do this to centre the room in the origin of the coordinate system):
        * _x_: -2500 mm 
        * _y_: -2000 mm 
        * _z_: -1500 mm
* In the toolbar you should see a button with a yellow sphere. Its tooltip says _Create a sphere solid_. Click on it. This will create a sphere.
* On the _Combo View_ on the left, select the newly created sphere. On the bottom you will see the sphere properties.
    * This sphere will represent our source, so under _Sphere_ > _Radius_ we will keep the radius of _5 mm_.
    * Under _Base_ > _Placement_ > _Position_ (we do this to centre the sphere on one corner of the room):
        * _x_: 2500 mm 
        * _y_: 2000 mm 
        * _z_: 1500 mm
* Now, click in any empty space in the _Combo View_ to un-select all objects. Then, while holding `Ctrl`, select the room first and the small sphere second. On the toolbar you will see a button showing two overlapping balls, one blue and the other one white. Its tooltip says _Make a cut of two shapes_. Click this button. The resulting object will be our room with the volume of our smaller sphere cut out of it.
* We are done. You can save the FreeCAD project, but do not forget to export our final object as _BREP_. To do so, select the final object in the _Combo View_ on the left. Then, select _File_ from the top men and then _Export_. On the _Files of type_ combo, select _BREP format_ and save your file to disk.

Note that, similarly to what we did in [The Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) episode, we are only modelling the air enclosed within the room, so the source itself is omitted from the model. We will model the influence of the source on the air as a boundary condition.

Note that our model is essentially a rectangular room with a small corner cut off that will act as the source. Why it is important to put the source in one corner? We will see below.

## Meshing
We need to make some choices here. What we want to model? Let's first have a look at our modal frequencies. We can generate a list of many modal frequencies as follows:

First, checkout the repository and head to the _2020-03-28-rigid-walled-room_ sub-directory. Open a terminal here and call `Julia` to open up Julia.

At the Julia prompt, type the following commands:

```julia 
include("exact.jl")

A, B, C = indexGrid(100, 100, 100)

Lx = 5
Ly = 4
Lz = 3

F = modeFrequency.(A, B, C, Lx, Ly, Lz)
```

The 3D array `F` contains the modal frequencies found in a cube in the mode-numbers space. If we sort this array we get the various modal frequencies of our room in increasing order.

```julia 
julia> sort(F[:])
1030301-element Array{Float64,1}:
    0.0             
   34.3             
   42.87500000000001
   54.90679033598667
   57.16666666666666
   66.66721666439793
   68.6             
   71.45833333333333
   79.26401076641136
   80.89638820738537
   85.75000000000001
   89.29718796119941
   92.35557644235674
   99.05681906248442
  102.89999999999999
  103.05867395701235
  108.61666666666667
  109.81358067197334
  111.47500000000001
  114.33333333333331
  117.71337127861803
  119.36750441854394
  122.10805352682969
  123.80246474839576
    ⋮               
 7841.058022358972  
 7842.658830613683  
 7844.341593006883  
 7844.721219291529  
 7847.137065874726  
 7850.731863990629  
 7853.709714381976  
 7855.67352045641   
 7857.5708105878875 
 7859.1802103274595 
 7862.0405519575515 
 7865.4789638189095 
 7868.002660636166  
 7870.411360913736  
 7873.7620695829255 
 7880.347761379429  
 7882.4189458425635 
 7885.2708577448375 
 7888.466115690857  
 7896.957894517217  
 7903.291666666668  
 7911.618830415036  
 7926.401076641138 
```

By using the [Plots Julia package](https://docs.juliaplots.org/latest/) we can make some cool plot, for example:

![Modal Behavior](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/modal_behaviour.png  "Modal Behavior")

On the top panel we just reported the first 1000 elements of the sorted `F` array. As we can see, the modal frequencies rise steeply, telling us that there are more and more modes per unity frequency the higher the mode number.

On the bottom panel instead we reported the distance, in [cents](https://en.wikipedia.org/wiki/Cent_(music)), between adjacent modal frequencies, which in Julia we can compute with the following lines:

```Julia 
# First 1000 modal frequencies:
f = sort(F[:])[1:1000] 

# Distance in cents between adjacent frequencies:
c = 1200 .* log2.(f[2:end] ./ f[1:(end - 1)])
```

We can see that the modal frequencies get very close together as the mode number (and hence the frequency) rises, the difference being less than 100 cents (1 semitone distance) already at the 10th mode. This is why modal frequencies are normally an issue for room acoustics only at low frequency. In fact, at low frequency (low mode number) the resonance frequencies are far away from each other, so they produce strong filtering (in human perception) while the various resonances sort of merge together at high frequency, producing a more uniform perception.

Hence, we will focus only on the first 10 nonzero modes, which we can get like this:
```julia 
julia> sort(F[:])[2:11]
10-element Array{Float64,1}:
 34.3             
 42.87500000000001
 54.90679033598667
 57.16666666666666
 66.66721666439793
 68.6             
 71.45833333333333
 79.26401076641136
 80.89638820738537
 85.75000000000001
```

As we learned in [The Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) episode, our mesh size $$s$$ should be lower than one tenth of the wavelength:

<p style="text-align: center;">$$s<\frac{\lambda}{10}=\frac{c_{0}}{10f}$$</p>

Here we have many frequencies, so we will use the highest frequency as a reference for our mesh calculation (85.75 Hz). If we choose a mesh size of 0.3 m we will be able to model up to 114.34 Hz, which is safely higher than 85.75 Hz. So, we will go with that.

Now that we know what to do, let's open Salome and follow the procedure below:

* On the toolbar, you will see the _Modules_ combo box. Select the _Geometry_ module.
* Select _File_ from the top menu, then _Import_ and then _BREP_. This will open the _Import BREP_ window. Select the _BREP_ file you exported when following the **Geometry** section.
* In the _Object Browser_ on the left you will see your newly imported object. Select it.
* Select _New Entity_ from the top menu, and then _Explode_. This will open the _Sub-shapes Selection_ window. On the _Syb-shapes Type_ combo, select _Solid_. Then choose _Apply and Close_. This will create a new solid object under our original object tree.
* Select the newly created solid object from the _Object Browser_ (expand the original object tree if necessary).
* Select _New Entity_ from the top menu, and then _Explode_. This will open the _Sub-shapes Selection_ window. On the _Syb-shapes Type_ combo, select _Face_. Then choose _Apply and Close_. This will create new face objects under the solid object tree.
* On the toolbar, you will see the _Modules_ combo box. Select the _Mesh_ module.
* From the object browser, make sure to select the solid object that we created during our first explosion, and not the top root imported _BREP_ object. Expand the tree if necessary.
* Once the correct object is selected, select _Mesh_ from the top menu and then _Create Mesh_. This will open the _Create mesh_ window. In the _Algorithm_ combo, select _NETGEN 1D-2D-3D_ Then click on the button with the gear wheel next to the _Hypothesis_ combo. From the popup menu, select _Netgen 3D Parameters_.
    * Input the following parameters. Type _300_ in the _Max. size_ edit box. Then type _1_ in the _Min. size_ edit box. You should interpret these sizes as millimetres. Select _Fine_ for the _Fineness_. Finally, tick the _Second order_ tickbox and click _OK_.
* Once you done with the options above, click on _Apply and Close_ in the _Create mesh_ window.
* Select your newly created mesh on the _Object Browser_ and right click with your mouse. A menu will popup. Select _Create Groups from Geometry_. This will open a new window. With this window open, select the solid and face entities that were exploded in the previous steps. Then click on _Apply and Close_.
* At this point we are ready to compute the mesh. Select and right click on the mesh on the _Object Browser_. A menu will popup. Select _Compute_ and wait until it is done.
* After the computation is complete, select and right click on the mesh on the _Object Browser_. A menu will popup. Select _Export_ and then _UNV File_. Save the UNV file wherever you want on your hard drive.

## Solving
We are now ready to solve our problem. We will make use of MATC expressions again, as in [The Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) episode, but we will also do a scanning simulation to solve for all of our frequencies right one after the other. A scanning simulation works by creating a dummy time variable that takes discrete values (1, 2, ...) up until a maximum that we specify. Then, simulations will be repeated for each value of the dummy time. If we do something with this dummy time at each iteration, then we will have a nice parameter sweep. Here we will put our 10 resonance frequencies into a vector and, at each iteration, use the dummy time to index into the vector and use the frequency value as input to the solver, so that at the end we have a collection of solutions, each one for each of our target resonance frequencies.

As a note, we mentioned above that we will be setting our source to be an uniform displacement source. This is in contrast with [The Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) episode, where we used uniform velocity. The [Elmer Model Manual](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf#chapter.11) tells us how to setup such boundary conditions. The flux will be given by:

<p style="text-align: center;">$$g=-\left(2\pi f\right)^{2}\rho_{0}d$$</p>

Where $$g$$ is the flux and $$d$$ is the normal surface displacement (we do not have any other component of the displacement in our simulation). We will set $$d$$ to 1 millimetre. Note that this flux is purely real.

Finally, to model a perfectly rigid wall, we will impose a null flux on the boundaries.

Let's follow the procedure below to setup our model.

Open up ElmerGUI by typing `ElmerGUI` in the terminal. Then, proceed as follows.

* Select _File_ from the top menu and then _Open_. Navigate to the location of your _UNV_ mesh to open it. This will load the mesh into the 3D view.
* Select _Model_ from the the top menu and then _Setup_.
    * In the _Coordinate Scaling_ edit box, type _0.001_, so that ElmerFEM understands that the units of our UNV mesh are millimetres.
    * In the _Simulation type_ combo, select _Scanning_.
    * In the _Timestep intervals_ editbox, type _10_.
    * In the _Free text_ edit box type the code in **Snippet 1** below. This code contains the simulation variables. To change anything about the simulation, it is sufficient to change any value here.
    * Click _Apply_.
* Select _Model_ from the the top menu, then _Equation_ then _Add_. This will open the _Equation_ window.
    * Select the _Helmholtz Equation_ tab.
    * Click the tick beside _Active_.
    * At the bottom, under _Apply to bodies_, tick the body to which apply our equation (there should be only one).
    * In _Free text input_ type the code in **Snippet 2** below. This code sets the simulation frequency at each scanning step.
    * Click the _Edit Solver Settings_ button.
        * In the _Linear system_ tab, select the _Iterative_ method and, from the combo, select _BiCGStabl_. Below, from the _Preconditioning_ combo, select _ILUT_. In my experience, these parameters work well with Helmholtz simulations.
        * I suggest you also tick the box _Abort if the solution did not converge_, so that ugly solutions will not be saved if the solver cannot converge.
        * In the _Nonlinear system_ tab, type _1_ in the _Max. iterations_ editbox (our problem is linear).
        * Click on _Apply_.
    * Click on _OK_.
* Select _Model_ from the the top menu, then _Material_ then _Add_.
    * Click the _Material library_ button.
        * Select _Air (room temperature)_.
        * Click _OK_.
    * In the _General_ tab, replace the value of _Density_ with the MATC expression in **Snippet 3**.
    * At the bottom, under _Apply to bodies_, tick the body to which apply our equation (there should be only one).
    * Click _OK_.
* Select _Model_ from the the top menu, then _Boundary condition_ then _Add_.
    * This will be our radiation condition, so I suggest you edit the _Name_ edit box at the bottom and type _Radiator_.
    * Then, select the _Helmholtz Equation_ tab.
    * Under _Real part of the flux_, insert the code in **Snippet 4**.
    * Click _OK_.
* Select _Model_ from the the top menu, then _Boundary condition_ then _Add_.
    * This will be our rigid wall condition, so I suggest you edit the _Name_ edit box at the bottom and type _Rigid_.
    * Then, select the _Helmholtz Equation_ tab.
    * Under _Real part of the flux_, type _0_.
    * Under _Imag part of the flux_, type _0_.
    * Click _OK_.
* Select _Model_ from the the top menu, then _Set boundary properties_.
* Rotate the domain until you find the corner where you cut out the spherical source. Double click on it (see below for an example). A window will popup. From the _Boundary condition_ combo, select _Radiator_.
* Double click on any wall. A window will popup. From the _Boundary condition_ combo, select _Rigid_. Repeat this for every remaining wall.
* Select _File_ from the top menu and then _Save project_. A window will popup.
    * In the new window, click on the _Create Folder_ button. This folder is actually the project folder, and will contain all our files. Give it a catchy name.
    * Once in the new folder click on the _Open_ button. This will actually save all the project files in the new folder.
    
The last points about saving the project are a bit strange, so ensure you follow them.

![Source Location]({{ site.baseurl }}/res/pictures/2020-03-28-rigid-walled-room/Figure_1.png "Source Location")

To check that everything is alright, select _Sif_ from the top menu and then _Edit_. This will open the Solver Input File generated by the GUI. If everything is OK, it should look like [this](https://github.com/CrocoDuckoDucks/Acoustic-Models/blob/master/2020-03-28-rigid-walled-room/elmerfem_2/case.sif).

If everything is OK, you area ready to press the _Start solver_ button on the toolbar of the ElmerGUI program. This will start the solver and open the converge plot viewer and the log file viewer. Sit back while the simulation runs!

You will probably see a convergence plot like the one below, but that is not really a convergence plot as it reports results from unrelated solver runs, each one with different frequency. You can dismiss the plot.

![Converge (Not Really) Plot](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/elmerfem_2/solver.png  "Convergence (Not Really) Plot")

### Snippet 1
MATC definitions for the simulation. The code below defines a vector of 10 elements `f`, which contains the first 10 resonance frequencies of our room. For this reason, we set the _Timestep intervals_ to _10_ in the _Setup_ window. `p` is the medium density, in kilograms per cubic meter, and `d` is the uniform source surface displacement, in meters.

```text 
$ f = 34.3 42.87500000000001 54.90679033598667 57.16666666666666 66.66721666439793 68.6 71.45833333333333 79.26401076641136 80.89638820738537 85.75000000000001
$ p  = 1.205
$ d = 0.001
```

### Snippet 2
MATC expression to set the frequency for the Helmholtz solver. The scanning simulation works by advancing a variable, called `time`, by 1 in each step. This variable starts at 1, but MATC arrays are 0 indexed. For this reason, we index into `f` by using `tx - 1`. So, in order to select the correct value of frequency for each step, we do the following:

```text 
Frequency = Variable time; Real MATC "f(tx - 1)"
```

#### Snippet 3
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the medium density.
```text
Real MATC "p"
```

#### Snippet 4
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the real part of the _Flux_ boundary condition. This is the only part of the _Flux_ boundary condition for this problem.
```text
Variable time; Real MATC "-((2 * pi * f(tx - 1))^2) * p * d""
```

# Results
Yay! Our simulation is solved. But is it good? Let's figure it out. Let's head over again to the [Elmer Models Manual](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf#chapter.11) to have a better look at what we calculated. The solver gave us a complex field defined over the whole of the room, called $$P$$. But what does a complex field mean? First of all, let's remember that we actually solved for 10 different fields, so let's add a suffix $$n$$ to $$P$$, so that we can identify them all: $$P_{n}$$, for $$n$$ going from 1 to 10. Then, remember that that the Helmholtz solver gives us the spatial part of the steady state solution, and the spatial part only depends on space variables: $$P_{n} = P_{n}\left(x,y,z\right)$$. So, If we want to include time, we need to multiply this spatial part by this complex sine wave:

<p style="text-align: center;">$$\exp\left(j2\pi f_{n}t\right)$$</p>

Where $$j$$ is the imaginary unit, $$f_{n}$$ are the resonance frequencies we used as input for the system, and $$t$$ is time, measured in seconds. Now we have all the ingredients to derive our steady state pressure waves $$p_{n}\left(x,y,z,t\right)$$. According to the Elmer Models Manual the physical solution is the real part of the complex field after multiplication with the complex sine wave:

<p style="text-align: center;">$$p_{n}\left(x,y,z,t\right)=\Re\left[P_{n}\left(x,y,z\right)\exp\left(j2\pi f_{n}t\right)\right]$$</p>

This is a very common convention, and people often say "we use complex numbers as they make algebra simpler, but the physical solution is the real part, so at the end we discard the imaginary part". Well, this is not completely accurate. Let's expand the equation above:

<p style="text-align: center;">$$p_{n}\left(x,y,z,t\right)=\Re\left[P_{n}\left(x,y,z\right)\right]\cos\left(2\pi f_{n}t\right)-\Im\left[P_{n}\left(x,y,z\right)\right]\sin\left(2\pi f_{n}t\right)$$</p>

Where $$\Re$$ and $$\Im$$ denote the real and imaginary parts respectively. As you can see, to build our final solution we use both real and imaginary parts of $$P_{n}$$. But also note this: if $$t=\frac{k}{2f}$$, with $$k$$ any integer number, then the sine term will be $$0$$, while the cosine term will be either $$1$$ or $$-1$$. This means the real part of $$P_n$$ is controlling the vibration at those times. The opposite happens instead at $$t=\frac{2k+1}{4f}$$, where instead the cosine is $$0$$ and the sine is either $$1$$ or $$-1$$. For all the other times $$p_n$$ is a _superposition_ of real and imaginary parts, with _weights_ decided by the trigonometric functions. So, the imaginary parts of things matter, if we discarded the imaginary part of $$P_{n}$$ prematurely we would ended up with a solution which is good only for certain times (for example, $$t=0$$).

Cool, now let's build the benchmark. In [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html) we seen that the modal shape of the room is given by:

<p style="text-align: center;">$$ p\left(x,y,z,t\right)=S\left(x,y,z\right)T\left(t\right)$$</p>

Where $$S$$ is the spatial part of the mode, and $$T%% is a cosine wave. Remembering that we have 10 of them, and to avoid confusion the FEM solution above, let's denote the mode shape with an overline:

<p style="text-align: center;">$$\overline{p}_{n}\left(x,y,z,t\right)=S_{n}\left(x,y,z\right)T_{n}\left(t\right)$$</p>

Where $$S$$ and $$T$$ are given as follows:

<p style="text-align: center;">$$ S\left(x,y,z\right)=\cos\left(\frac{n_{x}\pi}{L_{x}}x\right)\cos\left(\frac{n_{y}\pi}{L_{y}}y\right)\cos\left(\frac{n_{z}\pi}{L_{z}}z\right)$$</p>

<p style="text-align: center;">$$T_{n}\left(t\right)=\cos\left(2\pi f_{n}t\right)$$</p>

Where $$n_x$$, $$n_y$$ and $$n_z$$ are the mode numbers associated with $$f_n$$ (for more info refer back to [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html).

Now, we should really not forget that the benchmark field we put together above is a single mode (for each $$n$$). But the solution from FEM is different. What we solved for with ElmerFEM is not a single modal shape, but a field driven by a source instead. Turns out that for modal systems the steady state vibration, when driven by a source, is a weighted sum of _all_ the possible modal shapes:

<p style="text-align: center;">$$p_{Expected}\left(x,y,z,t\right)=\sum_{n=0}^{\infty}a_{n}\overline{p}_{n}\left(x,y,z,t\right)$$</p>

Where $$a_{n}$$ are the coefficients of the superposition, and depend on where the source is located, and the properties of the source. And, they are also very hard to calculate analytically.

Still, two things are going on in this simulation:

* The walls are rigid.
* We are modelling low frequency.

The two things above, in combination, tell us that the modal superposition will be either pure (i.e. the only mode in the superposition is that related to the resonance frequency in input) or only "contaminated" by neighbouring modal shapes. This because rigid walls make resonances sharp, so the influence of each mode is narrow in frequency, making its ability to contaminate other modes reduced. But also, and more importantly, because at low frequency the distance between modes is larger, further a part than the resonance width, so modes do not really interact much in the superposition.

So, in other words: the response of the room is a modal superposition, but for rigid rooms at low frequency the superposition is going to be pure. But at what frequency this will stop? It is hard to tell, but we can find it by comparing the FEM result against the pure modal shapes as computed by the exact model. The lowest frequency modes should match the pure modes closely. If at some frequency we see mismatch, then we know that the superposition is not pure.

Wait, but could other things cause mismatch, like solver accuracy? As we seen in [The Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) episode we know that our mesh size should grant us low errors. Moreover, if the error patterns that we observe have a modal taste to them, and are not random or uniform, we know that the source of mismatch must be contamination from neighbouring modes.

So, to sum it up, ElmerFEM cannot solve for the exact modal shapes, but can solve for the steady state pressure as driven by a source. This pressure is given by a modal superposition, and by comparing it with the pure mode, given that our room is rigid and our frequencies are low, we will:

* Understand how ElmerFEM is accurate by checking how close the fundamental mode is.
* Understand at which frequency the modal superposition stops being pure, and more than one modes contribute to the steady state pressure.

Even though modal shapes and modal superposition are not the same thing, we will still refer to the comparison between the superposition delivered by ElmerFEM and the pure modes delivered by Julia as _error_.

Only one thing is left to figure out. Since in our FEM model we have a driving source, the amplitude of the resulting field is dictated by the source. But in our benchmark solution (the pure mode) the amplitude is $$1$$. In order to compare the two, we will normalise the FEM solution as follows:

* Find the peak of the exact mode shape.
* Multiply the FEM solution by a factor that brings the value at the same position to that of the exact solution.

So, let's go ahead and let's have a look at our FEM solution first with ParaView. Open ParaView, select _File_ from the top menu and then _Open_. Navigate to the Elmer project folder. You should see the entry _case..vtu_. If you expand it you will see that it is just a collection of vtu files (_case0001.vtu_, _case0002.vtu_, ...). These files are generated by ElmerFEM for each time step of our scanning simulation. If we select _case..vtu_ we will import them all into ParaView. Remember to click on _Apply_ in the _Properties_ browser on the left.

On the toolbar, you will see a combo that, by default, says _Solid Color_. Click on it and select _pressure wave 1_. This is the real part of the pressure field, while _pressure wave 2_ is the imaginary part. If you use the play/stop buttons at the top you can cycle through all 10 solutions. This will show you all the different pressure fields. The animation below shows all the fields, but normalised to take only values between $$1$$ and $$-1$$ so that they can be plotted on the same scale.

![Result Fields](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/elmerfem_2/anim.gif  "Result Fields")

The animation was performed with ParaView by selecting only the real part of the field. As we seen above, we can think of this as being the result field at $$t=0$$. The various surfaces are surfaces of constant normalised pressure. They help us understanding the shape of the field. As you probably remember from [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html), the shapes do really look similar to modes. You will probably also see some strange surfaces close to the corner (the source location) or the walls. Those exists due to the negative peak value of the field not being exactly $$-1$$.

Now to validation. To validate, we should export the solution into CSV files, so that we can read it with Julia. In the _Pipeline Browser_, select the data you imported. Then, from the top menu choose _File_ and _Save Data_. If you want to use the `validate.jl` script, save them in the _export_ sub-directory of the repository, type _data_ in the _File name:_ editbox and choose _csv_ from the _File of type:_ combo. This will open the _Configure Writer (CSVWriter)_ window. Tick _Write Time Steps_. Tick _Choose Arrays to Write_ and untick _GeometryIds_. In _Precision_ type _30_. Click _OK_. The exported CSV files will contain the value of the fields at the nodal points of the mesh.

The validation is carried on by the Julia script `validate.jl` by using the exported CSV files. The script will use the equations above to calculate two metrics of error. One is the norm. This metric is calculated by putting the FEM solution and the exact one in two arrays, taking the difference, summing the square of the difference and taking the squared root of the result. Finally, the ratio in dB between the FEM solution and the exact one is exported in output CSV files. This allows us to see where the FEM solution differs, in dB, most from the exact solution. The error field used to compute the norm are also saved in the CSV files. 

The result for the norm is reported below:

![Error Norm](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/error_norm.png  "Error Norm")

It is possible to see that the error is overall rising with frequency. This is to be expected because the higher the frequency the higher contamination from neighbouring modes we should expect.

We can inspect the error and the dB error of the fields by importing it into Paraview. Select _File_, then _Open_. Then, navigate to the _error_ subdirectory in the repository, where `validate.jl` has exported all the error fields as CSV files. Select _error__..csv_ to import them all. This will import a data table into ParaView available in the _Pipeline Browser_ as _error*_. Select it, then go _Filters_ > _Alphabetical_ > _Table To Points_. In the _Properties_ tab, for the _X Column_, _Y Column_ and _Z Column_ combos select _x_, _y_ and _z_ respectively. You will now see all the nodal points of the mesh displayed in space. In the colouring combo at the top, you can select either _error_field_ or _dB__error__field_ to colour the points according to how big the selected error field is. My results are shown in the animations below.

![Error Fields](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/error_anim.gif  "Error Fields")

It is possible to see that the error in Pascals (left panel) is very low except at round 66 Hz and 80 Hz. When we look at the pattern of the error we clearly see something that is not casual, and that has a striking resemblance to the modal patterns themselves. As we look at the frequencies at which the highest errors are recorded, we clearly see that there are other modal frequencies in close proximity. This suggests that the resulting field is deviating from the target not because model or computational errors, but because neighbouring modes are activated by the source. If we look at the dB error field on the right we will see that the error is always very small a part for the highest frequency solution field. This tells us that the contribution from other modes, even though we can detect it at around 66 Hz, starts becoming actually significant only at around 80 Hz. So, for this rigid walled rectangular room, we can say that at 80 Hz the modal superposition cannot be considered pure.

_By the way, note that if we had to drive the room at frequencies different than exact modal frequencies, we will never get a pure superposition, not even a low frequency, as we will activate the modes closest to the driving frequency_

Now, you could do the same analysis as we did for any other source location. Will anything change? The animation below compares two identical simulations, the only difference being that in the one on the left the source is at the centre of the room.

![Error Fields](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-28-rigid-walled-room/1_vs_2_anim.gif  "Error Fields")

This animation, still prepared with ParaView, shows the [sound pressure level](https://en.wikipedia.org/wiki/Sound_pressure#Sound_pressure_level) associated to the various solution fields. To give some context to the numbers, 40 dB SPL is the level you would record in a very quiet room, while 130 dB SPL is the threshold of pain. You can see that if we put the source in the middle the room is extremely quiet, only reaching high level SPL for the last frequency. Why?

Well, from [Acoustic Modes of a Rectangular Room](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html) you will probably remember that modes have nodal surfaces, surfaces in which the mode shape is 0. If we put a source there, we can hardly activate the mode at all, as we are trying to drive it when it can only be 0, so no wave can start. Turns out that the middle of the room is in a nodal surface for most of the low frequency modes we are considering. We can still see a bit of sound developing because our source is a sphere, not an exact point, so it is driving air also just outside the nodal surface. The higher the frequency the sharper the variation of pressure close to the nodal surface, so high pressure can still be found close to the nodal surface. hence, at higher frequency the sphere will become more able to drive pressure even by being located in a nodal surface due to its non infinitesimal extent. By contrast, on the corners the modes always have a peak, so it is the ideal place to drive all modes or, if you want to control them, to put a bass trap in. This has the con that, in our simulation, the modal superposition gets dirty with many modes pretty soon in frequency.

# Conclusion
This was another long one, it took more time than what I anticipated to finish. Still, we learned something valuable:

* ElmerFEM does not have, at the moment, an Helmholtz Eigensolver. This complicates things for us when we want to study modal behaviour of enclosures, but we can work around it. So, even when your solver does not support what you want to do, you can be creative. Or even better, write an Eigensolver for ElmerFEM, since the project is opensource. One day...
* When we drive an enclosure with a sound source the steady state vibration of the enclosed air is a modal superposition. Even for rigid walled rooms, the superposition will be pure (one single mode) only at low frequency. For our room, we can expect one single mode only when driving at resonance frequencies lower than 80 Hz.
* The response of a room depends on its shape (which dictates the shape of the modes) but also on the position of the source. Also, the pressure is not uniform in the room, meaning that the pressure recorded by different microphones at different points will be most likely different. In other words, the transmission line between a source and a receiver, when placed inside a room, depends on the locations of the source and receiver, even when they are omnidirectional!

In the next episodes we will maybe have a look at ParaView and the basics of how to use it for postprocessing. Then, we will make more realistic rooms and investigate their response.
