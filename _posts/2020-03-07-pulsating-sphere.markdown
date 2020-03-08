---
layout: post
title:  "The Pulsating Sphere"
date:   2020-03-07 20:00:00 +0100
categories: science physics opensource
---

![FEM Model of a Spherical Source](https://raw.githubusercontent.com/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/paraview.png  "FEM Model of a Spherical Source")

{% include mathjax.html %}

In this episode we will build a model of a pulsating sphere source. The pulsating sphere source is an ideal source which forms the base for the development of point-sources. In essence, a point source is a pulsating sphere in the limit of $$a$$, the radius of the sphere, approaching 0. For this reason, although abstract, the pulsating sphere is a very powerful theoretical tool that enables the study of point -sources which in turn, through integration and wave propagation principles, enable to study of any arbitrary acoustic field source.

In this episode we will study the point source radiating in infinite space in the domain of FEM analysis. Again, the study of simplified problems allows us to understand, through comparison with theory, how to tune our FEM problem to maximise accuracy. We will be then able to carry over this knowledge to more advanced problems.

Similarly to what done in previous episodes, we will:

* Dig out a theoretical model for our system from literature.
* Use the Julia programming language to create an exact model.
* Use ElmerFEM to create an approximate model.

# GitHub Repository
All the code and models for this post are stored under the folder [2020-03-07-pulsating-sphere](https://github.com/CrocoDuckoDucks/Acoustic-Models/tree/master/2020-03-07-pulsating-sphere) of the [Acoustic-Models](https://github.com/CrocoDuckoDucks/Acoustic-Models) repository. The contents of the directory are shown below.

```text
2020-03-07-pulsating-sphere/
|-- elmerfem_1
|   |-- case0001.vtu
|   |-- case.sif
|   |-- egproject.xml
|   |-- ELMERSOLVER_STARTINFO
|   |-- Mesh_1.png
|   |-- Mesh_1.unv
|   |-- mesh.boundary
|   |-- mesh.elements
|   |-- mesh.header
|   |-- mesh.nodes
|   |-- netgen.prof
|   |-- solver.log
|   `-- solver.png
|-- elmerfem_2
|   |-- case0001.vtu
|   |-- case.sif
|   |-- egproject.xml
|   |-- ELMERSOLVER_STARTINFO
|   |-- Mesh_2.png
|   |-- Mesh_2.unv
|   |-- mesh.boundary
|   |-- mesh.elements
|   |-- mesh.header
|   |-- mesh.nodes
|   |-- netgen.prof
|   |-- solver.log
|   `-- solver.png
|-- errors_1.png
|-- errors_2.png
|-- exact.jl
|-- geometry.brep
|-- geometry.FCStd
|-- geometry.FCStd1
|-- linedata_1.csv
|-- linedata_2.csv
|-- Mesh_1.unv
|-- Mesh_2.unv
|-- meshing.hdf
|-- paraview.png
|-- postproc.pvsm
`-- validate.jl

```

The study was performed with a coarse and a fine mesh. The suffixes `_1` and `_2` denote files for coarse and fine mesh respectively.

The folders `elmerfem_` contain the ElmerFEM projects.

The `errors_` _png_ files show a plot of the error of the FEM simulation against a theoretical model.

`exact.jl` contains a Julia implementation of functions to calculate the theoretical solution.

`geometry.brep` contains the _BREP_ geometry for the problem, while `geometry.FCStd` and `geometry.FCStd1` are FreeCAD files.

The `linedata_` _csv_ files are fields exported from the FEM solutions, while the `Mesh_` _unv_ files are the meshes for the study.

`meshing.hdf` is the Salome study.

`paraview.png` is a screenshot from the post processing example `postproc.pvsm`, which is a ParaView state file.

Finally, `validate.jl` is a Julia script that compares the data stored in the `linedata_` files to the exact solutions and produce comparison plots.

Note that to run `validte.jl` you will need to install the Julia [Plots package](https://docs.juliaplots.org/latest/).

# Theoretical Model 
Many equivalent formulations of the field radiated by a pulsating sphere exist. In the context of this post we will make use of the formulation presented in the first chapter of [Active Control of Sound](https://www.elsevier.com/books/active-control-of-sound/nelson/978-0-12-515425-3) by P.A. Nelson and S.J. Elliot, to which you can refer for more information, although similar derivations will be available for sure in many other books and websites. We will not derive the result, but just take the bits we need.

When we think about acoustic wave propagation, two quantities are of interest. One is the pressure disturbance, or acoustic pressure, $$p$$, which is the pressure disturbance with respect the equilibrium pressure $$p_{0}$$. The other is the particle velocity $$u$$, which is the velocity of an element of fluid deriving from the propagation of the acoustic field (remember that we are talking about continuum fluid dynamic, so our fluid is modelled as a continuum and particles are not molecules, but infinitesimal fluid elements). Both $$p$$ and $$u$$ will change in space and time, and the amplitude and phase relationship between them will also change in space and time. The most convenient mathematical way to represent them is then complex numbers, as they "encode" amplitude and phase information simultaneously in a single complex value. Hence:

<p style="text-align: center;">$$p=p\left(x,y,z,t\right)\in\mathbb{C}$$</p>
<p style="text-align: center;">$$u=u\left(x,y,z,t\right)\in\mathbb{C}$$</p>

Both the relationships above are valid for every $$x$$, $$y$$, $$z$$ and $$t$$. The amplitude and phase relationship between acoustic pressure and particle velocity is nontrivial, and depends on the nature of the source and domain, as in essence it depends on the field itself. To capture this relationship, one can define the specific acoustic impedance as follows:

<p style="text-align: center;">$$z=\frac{p}{u}$$</p>

Since $$p$$ and $$u$$ are both complex functions of space and time, so is the specific acoustic impedance. It is possible to show that, for spherical waves propagating in an infinite uniform medium, the specific acoustic impedance is:

<p style="text-align: center;">$$z\left(r\right)=\rho_{0}c_{0}\frac{jkr}{1+jkr}$$</p>

Where $$r=\sqrt{x^{2}+y^{2}+z^{2}}$$ is the distance from the centre of the source emanating the spherical wave, such is our pulsating sphere. $$j$$ is the imaginary unit and $$k$$ is the wavenumber:

<p style="text-align: center;">$$k=\frac{2\pi f}{c_{0}}$$</p>

Where $$f$$ is the frequency at which the source is pulsating, and $$c_{0}$$ is the phase speed of sound in the medium. Finally, $$\rho_{0}$$ is the density of the medium. Note that, for a spherical wave propagating in free field (infinite space without any other obstacle) the specific acoustic impedance does not depend on time.

The specific acoustic impedance is an ingredient which defines the sound radiated from a spherical source, as we will see just below, but it is also a property that we can use to make a finite domain to behave as an infinite one when we do FEM, as we will see later.

As we said, a pulsating sphere will emit a spherical wave, which then will be characterised by the specific acoustic impedance defined above. Let's assume that we have a sphere of radius $$a$$ pulsating at the frequency $$f$$ with a surface velocity $$U$$. Note that the surface velocity is, by definition, orthogonal to the surface of the source. Then the pressure disturbance is given by:

<p style="text-align: center;">$$p=p\left(r,t\right)=z\left(a\right)\frac{aU}{r}\exp\left(j\left[\omega t-k\left\{ r-a\right\} \right]\right)$$</p>

where $$\omega=2\pi f$$ as the angular frequency. To be noted that, due to the symmetry of the problem, the spatial dependency of the acoustic pressure field is radial, that is, points at the same distance $$r$$ from the source centre have the same pressure disturbance.

# Source and Medium Parameters
For the rest of the post, we will assume the following source and medium parameters. You can repeat the study with any choice of parameters.

|Parameter Name              | Symbol       | Value | Unit                      |
|----------------------------|--------------|-------|---------------------------|
| Source Radius              | $$a$$        | 0.005 | meters                    |
| Source Frequency           | $$f$$        | 1000  | hertz                     |
| Source Surface Velocity    | $$U$$        | 0.75  | meters per second         |
| Medium Sound Phase Speed   | $$c_{0}$$    | 343   | meters per second         |
| Medium Equilibrium Density | $$\rho_{0}$$ | 1.205 | kilograms per cubic meter |

The chosen medium parameters above are that for air in equilibrium at ordinary room temperature (20 Celsius Degrees).

# Julia Model
The code implementing the equations above is written in the file `exact.jl` available at the repository. The following paragraphs will illustrate how to use the functions in the code.

Once you clone the repo, navigate to the `2020-03-07-pulsating-sphere` directory with your terminal. Open Julia by issuing the command `julia`. At the Julia REPL, do:

```julia
julia> include("exact.jl")
```

Now the functions defined in `exact.jl` are loaded into the Julia workspace. For example, to compute the impedance at 1 meter from the source:

```julia
julia> sphericalImpedance(1000, 1, 1.205, 343)
412.08694629151245 + 22.49588634867694im
```

Or, to compute the impedance at a set of equally spaced distances between 0.005 meters and 1 meter, with a resolution of 0.001 meters:

```julia
julia> sphericalImpedance.(1000, 0.005:0.001:1, 1.205, 343)
996-element Array{Complex{Float64},1}:
  3.438464634081654 + 37.54125692082816im 
  4.933330796152837 + 44.88520764425224im 
 6.6859932275747145 + 52.14133471072512im 
  8.689694549377409 + 59.296461203974424im
 10.936816885631023 + 66.33804720893404im 
 13.418952281912064 + 73.25425572657367im 
 16.126978587079495 + 80.03400974817295im 
 19.051139570106578 + 86.66704005848428im 
 22.181128024567666 + 93.14392352178388im 
  25.50617062354156 + 99.4561117850073im  
 29.015113325049185 + 105.59595050321417im
   32.6965061898067 + 111.5566893503921im 
  36.53868655556306 + 117.33248321972421im
 40.529859611718095 + 122.91838513937483im
  44.65817553018512 + 128.31033153091624im
 48.911802429442304 + 133.5051205168856im 
   53.2789945744125 + 138.50038404157235im
  57.74815534149099 + 143.29455460562127im
  62.30789460235908 + 147.88682743163895im
   66.9470802992786 + 152.27711887640334im
  71.65488409597712 + 156.46602188756788im
   76.4208210901329 + 160.45475927119824im
  81.23478366451711 + 164.2451354935025im 
                    ⋮                     
 412.03124854725024 + 22.998819837131627im
  412.0338616598966 + 22.97547339214939im 
  412.0364668102936 + 22.952174149442268im
  412.0390640307233 + 22.92892196645721im 
  412.0416533533047 + 22.905716701212768im
 412.04423480999424 + 22.882558212296207im
   412.046808432587 + 22.859446358860694im
  412.0493742527182 + 22.836381000622502im
  412.0519323018632 + 22.813361997858152im
  412.0544826113393 + 22.79038921140168im 
  412.0570252123062 + 22.767462502641838im
  412.0595601357671 + 22.74458173351936im 
 412.06208741256955 + 22.721746766524205im
   412.064607073407 + 22.698957464692874im
  412.0671191488184 + 22.676213691605657im
  412.0696236691907 + 22.65351531138398im 
  412.0721206647585 + 22.63086218868773im 
 412.07461016560563 + 22.60825418871258im 
 412.07709220166583 + 22.585691177187382im
 412.07956680272395 + 22.563173020371515im
  412.0820339984162 + 22.540699585052277im
 412.08449381823146 + 22.51827073854228im 
 412.08694629151245 + 22.49588634867694im 
```

The function `sphericalWave` can be similarly used. For example, this is the complex pressure disturbance at 1 meter and 0 seconds:

```julia
julia> sphericalWave(0.75, 0.005, 1000, 0, 1, 1.205, 343) 
-0.07164793924384269 + 0.12186780546373502im
```

And at the previous set of equally spaced distances:

```julia
julia> sphericalWave.(0.75, 0.005, 1000, 0, 0.005:0.001:1, 1.205, 343)
996-element Array{Complex{Float64},1}:
   2.5788484755612404 + 28.15594269062112im  
   2.5784638811553324 + 23.419984385921428im 
    2.577447564216482 + 20.030421070628968im 
    2.575928568123367 + 17.48236749878261im  
    2.573978809358829 + 15.495333621210861im 
   2.5716416698416644 + 13.901026983151116im 
   2.5689449933704016 + 12.592353813596183im 
    2.565907583772156 + 11.497919569528525im 
   2.5625427038311743 + 10.568298459334644im 
   2.5588600746002603 + 9.768187455810947im  
    2.554867074910161 + 9.071698703619745im  
     2.55056949098419 + 8.45941727362481im   
   2.5459720014034053 + 7.916497356569299im  
    2.541078500336285 + 7.431393058575405im  
   2.5358923186147004 + 6.9949899977960435im 
    2.530416378405386 + 6.599997421816778im  
   2.5246533036072263 + 6.24051400555941im   
    2.518605500057459 + 5.911712067710895im  
   2.5122752147313987 + 5.609604165258298im  
   2.5056645800587423 + 5.330868039183739im  
   2.4987756475202665 + 5.072713572995965im  
    2.491610413407431 + 4.832780452951799im  
    2.484170838773508 + 4.609058570271543im  
                      ⋮                      
 -0.11626019807233158 + 0.08589527124416463im
   -0.114550195977554 + 0.08792053472881003im
 -0.11280528539998785 + 0.08991219324792896im
 -0.11102611882218058 + 0.09186965187271595im
 -0.10921335874434179 + 0.0937923282007353im 
 -0.10736767744639013 + 0.09567965252245855im
 -0.10548975674727247 + 0.09753106798332205im
  -0.1035802877616337 + 0.09934603074126734im
 -0.10163997065392487 + 0.10112401011971621im
 -0.09966951439002934 + 0.10286448875594613im
 -0.09766963648649898 + 0.10456696274481922im
 -0.09564106275748302 + 0.10623094177783032im
 -0.09358452705943339 + 0.10785594927744079im
 -0.09150077103368005 + 0.10944152252665723im
 -0.08939054384696062 + 0.11098721279382501im
 -0.08725460192998967 + 0.11249258545260821im
 -0.08509370871416262 + 0.11395722009711984im
 -0.08290863436648038 + 0.11538071065217743im
 -0.08070015552278063 + 0.11676266547865974im
 -0.07846905501937268 + 0.11810270747393418im
 -0.07621612162316302 + 0.11940047416733444im
   -0.073942149760358 + 0.12065561781066901im
 -0.07164793924384269 + 0.12186780546373502im
```

The file `validate.jl` makes use if these functions to generate comparison plot between the exact solution, as implemented in Julia, and the ElmerFEM results.

# Building the FEM Model
We will now build the FEM model. As always we start with the Geometry.

## Geometry

This is really very simple to build. Open FreeCAD and follow the procedure below.

* Select _File_ from the top menu and then _New_.
* Switch to the _Part_ _Workbench_.
* In the toolbar you should see a button with a yellow sphere. Its tooltip says _Create a sphere solid_. Click on it. This will create a sphere.
* On the _Combo View_ on the left, select the newly created sphere. On the bottom you will see the sphere properties. This sphere will represent our source, so we will keep the radius as _5 mm_.
* Click again on the _Create a sphere solid_ button. This will create a new sphere.
* On the _Combo View_ on the left, select the newly created sphere. On the bottom you will see the sphere properties. Click on the _Radius_ value, type _100_ and press `Enter`. This bigger sphere will model the extent of air that we include in the model.
* Now, click in any empty space in the _Combo View_ to un-select all objects. Then, while holding `Ctrl`, select the bigger sphere first and the smaller sphere second. On the toolbar you will see a button showing two overlapping balls, one blue and the other one white. Its tooltip says _Make a cut of two shapes_. Click this button. The resulting object will be our bigger sphere with the volume of our smaller sphere cut out of it.
* We are done. You can save the FreeCAD project, but do not forget to export our final object as _BREP_. To do so, select the final object in the _Combo View_ on the left. Then, select _File_ from the top men and then _Export_. On the _Files of type_ combo, select _BREP format_ and save your file to disk.

So, it will be clear to you that essentially our geometry is a big sphere with a spherical hole in the centre. So, in essence, we are only modelling the body of air around the source, up to 0.1 meters, omitting the source itself. This because the source is, in reality, nothing but a boundary condition for the air in contact with it. This is how we will model the source with ElmerFEM.

## Meshing
We will now mesh our geometry. Open your Salome installation and follow the procedure below.

* On the toolbar, you will see the _Modules_ combo box. Select the _Geometry_ module.
* Select _File_ from the top menu, then _Import_ and then _BREP_. This will open the _Import BREP_ window. Select the _BREP_ file you exported when following the Geometry section.
* In the _Object Browser_ on the left you will see your newly imported object. Select it.
* Select _New Entity_ from the top menu, and then _Explode_. This will open the _Sub-shapes Selection_ window. On the _Syb-shapes Type_ combo, select _Solid_. Then choose _Apply and Close_. This will create a new solid object under our original object tree.
* Select the newly created solid object from the _Object Browser_ (expand the original object tree if necessary).
* Select _New Entity_ from the top menu, and then _Explode_. This will open the _Sub-shapes Selection_ window. On the _Syb-shapes Type_ combo, select _Face_. Then choose _Apply and Close_. This will create two new face objects under the solid object tree. One is the inner face, the other is the outer face.

After this geometry manipulation, we are ready to mesh. As a rule of thumb, our mesh size $$s$$ should always be smaller, along every direction, than one tenth of the wavelength $$\lambda$$. Hence:

<p style="text-align: center;">$$s<\frac{\lambda}{10}=\frac{c_{0}}{10f}$$</p>

At 1 kHz this "critical size" is 3.43 centimetres. We will see that this size is already sufficient to provide accurate results, but we can also run a higher density mesh study to see what we gain from it. Let's then move on and create our mesh.

* On the toolbar, you will see the _Modules_ combo box. Select the _Mesh_ module.
* From the object browser, make sure to select the solid object that we created during our first explosion, and not the top root imported _BREP_ object. Expand the tree if necessary.
* Once the correct object is selected, select _Mesh_ from the top menu and then _Create Mesh_. This will open the _Create mesh_ window. In the _Algorithm_ combo, select _NETGEN 1D-2D-3D_ Then click on the button with the gear wheel next to the _Hypothesis_ combo. From the popup menu, select _Netgen 3D Parameters_.
    * For a coarse mesh, proceed as follows. Type _29_ in the _Max. size_ edit box. Then type _1_ in the _Min. size_ edit box. You should interpret these sizes as millimetres. Select _Fine_ for the _Fineness_. Finally, tick the _Second order_ tickbox and click _OK_.
    * For a fine mesh, proceed as follows. Type _5_ in the _Max. size_ edit box. Then type _1_ in the _Min. size_ edit box. You should interpret these sizes as millimetres. Select _Fine_ for the _Fineness_. Finally, tick the _Second order_ tickbox and click _OK_.
* Once you done with the options above, click on _Apply and Close_ in the _Create mesh_ window.
* Select your newly created mesh on the _Object Browser_ and right click with your mouse. A menu will popup. Select _Create Groups from Geometry_. This will open a new window. With this window open, select the solid and face entities that were exploded in the previous steps. Then click on _Apply and Close_.
* At this point we are ready to compute the mesh. Select and right click on the mesh on the _Object Browser_. A menu will popup. Select _Compute_ and wait until it is done.
* After the computation is complete, select and right click on the mesh on the _Object Browser_. A menu will popup. Select _Export_ and then _UNV File_. Save the UNV file wherever you want on your hard drive.

Both the coarse and fine mesh parameters outlined above produce meshes with sizes below a tenth of a wavelength.

Pictures of the coarse and fine mesh are shown below. They are obtained with Salome. If you want to get similar views, just right click on your mesh in the 3D view and choose _Clipping_. A window will popup allowing to introduce any clipping plane you might like. Note that this clipping plane is for visualisation only, and does not affect the mesh. You can see that NETGEN meshes adapt their size where needed, and are finer where the curvature is higher (like in the centre).

![Coarse Mesh](https://raw.githubusercontent.com/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/elmerfem_1/Mesh_1.png  "Coarse Mesh")

![Fine Mesh](https://raw.githubusercontent.com/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/elmerfem_2/Mesh_2.png  "Fine Mesh")

## Solving
It is finally the time to solve our study. For this study we will use a little more advanced feature of ElmerFEM, which is MATC expressions. MATC is a library for the evaluation of numerical expressions that is included with ElmerFEM. Its documentation is available [here](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/MATCManual.pdf). With MATC it is possible to add flexibility to ElmerFEM and do all sorts of cool things, like parametric sweep studies or post-processing. You can run MATC expressions on the fields, at each solver iteration, but beware that MATC is pretty slow. It is best used to set up the simulation parameters, as we will do here.

### Understanding the Problem
But first, what equation we have to solve? Look again at our theoretical solution:

<p style="text-align: center;">$$p\left(r,t\right)=z\left(a\right)\frac{aU}{r}\exp\left(j\left[\omega t-k\left\{ r-a\right\} \right]\right)$$</p>

This solution holds for a spherical source at steady state, and it is found by using the wave equation we outlined towards the end of[this episode](https://crocoduckoducks.github.io/science/physics/opensource/2018/11/30/what-is-acoustic-modelling.html). Turns out that ElmerFEM can solve for that equation. But also, notice this property:

<p style="text-align: center;">$$p\left(r,t\right)=z\left(a\right)\frac{aU}{r}\exp\left(-jk\left[r-a\right]\right)\exp\left(j\omega t\right)$$</p>

Thanks to the properties of exponential functions, we can factor the function in one part, up to the first exponential, that depends on space only ($$r$$) and a second part (the second exponential) that depends on time only which is just a complex sine wave at our source frequency. When this happens, we can actually use a variant of the linearized wave equation (as it is called) that allows to solve for the spatial part only of our disturbance. This equation is the Helmholtz equation, and we will user ElmerFEM Helmholtz model.

Note that this factorisation is typical of steady state propagation, and it is possible to demonstrate that the time dependant part is always the same complex exponential function for every problem (for this type of PDE). So you should use the Helmholtz solver every time you are after a steady state solution, so that time will not complicate your analysis and potentially introduce more errors. If you need to model steady state time behaviour all you have to do is multiply your final space solution by $$\exp\left(j\omega t\right)$$. Time dependent solvers are best suited to study _transients_, that is, the evolution between steady states.

Since we will be using the Helmholtz model, we should always refer to its documentation [here](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf#chapter.11).

Now, in order to set up the model properly we will use a FEM trick. If you were reading carefully, you must have observed that our theoretical model is for a spherical source radiating in free field, which is an infinite space. However, our geometry is a ball of air of 0.1 meters radius! How can we simulate our spherical source realistically with such a small volume?

Well, remember the spherical wave specific acoustic impedance defined above? That holds for a spherical wave propagating in an infinite medium. So, at a certain distance $$r$$ from the source the impedance will be $$z\left(r\right)$$. Now, imagine there was a wall in our infinite medium, at some distance from the sphere source. A wall is some object that offer its own impedance to the wave. This will cause part of the wave to be transmitted into the wall, and part to reflect. So, impedance controls what happens to the field. Let's think about the outer boundary of our domain again. If we assign to it the very same impedance that the infinite spherical wave would have at that radius, that is $$z\left(r\right)$$, with $$r$$ equal to 0.1 meters, than the outer boundary behaves exactly as if it was perfectly transparent to the wave and there was infinite space behind it. This because its impedance _matches_ exactly that of the wave, meaning that no reflection happens, and transmission through the boundary is the same as that in free field. Hence, we will do just that!

By referring to the Helmholtz model manual, we can see that the model allows two types of boundary conditions, _Flux_ and _impedance_. Clearly, we want the second one for our outer surface, but note that the quantity $$Z$$ evidenced in the manual is not our specific acoustic impedance $$z$$. However, it is easy to calculate $$Z$$ is it just the specific acoustic impedance divided by the density of the medium:

<p style="text-align: center;">$$Z\left(r\right)=\frac{z\left(r\right)}{\rho_{0}}=c_{0}\frac{jkr}{1+jkr}$$</p>

So, all we need to do is calculate this for $$r$$ equal to 0.1 meters (the radius at which our outer spherical boundary is placed) and use it as boundary condition. Since ElmerFEM requires the quantity to be input as a real-imaginary parts pair, here the same quantity but with the two parts separated:

<p style="text-align: center;">$$Z\left(r\right)=c\frac{\left(kr\right)^{2}}{1+\left(kr\right)^{2}}+jc\frac{kr}{1+\left(kr\right)^{2}}$$</p>

This is an expression we will evaluate with MATC.

Finally, we need to assign a boundary condition at the inner surface of our medium, that in contact with the spherical source. This boundary condition must be that a spherical source would produce. The Elmer model manual already gives us the _Flux_ boundary condition that we need:

<p style="text-align: center;">$$g=j\omega\rho_{0}U$$</p>

We will implement this too as a MATC expression.

### Model Setup
We are ready to go ahead then!

Open up ElmerGUI by typing `ElmerGUI` in the terminal. Then, proceed as follows.

* Select _File_ from the top menu and then _Open_. Navigate to the location of your _UNV_ mesh to open it. This will load the mesh into the 3D view.
* Select _Model_ from the the top menu and then _Setup_.
    * In the _Coordinate Scaling_ edit box, type _0.001_, so that ElmerFEM understands that the units of our UNV mesh are millimetres.
    * In the _Free text_ edit box type the following code in **Snippet 1** below. This code contains the simulation variables. To change anything about the simulation, it is sufficient to change any value here.
    * Click _Apply_.
* Select _Model_ from the the top menu, then _Equation_ then _Add_. This will open the _Equation_ window.
    * Select the _Helmholtz Equation_ tab.
    * Click the tick beside _Active_.
    * At the bottom, under _Apply to bodies_, tick the body to which apply our equation (there should be only one).
    * In _Free text input_ type the code in **Snippet 2** below. This code sets the simulation frequency.
    * Click the _Edit Solver Settings_ button.
        * In the _Linear system_ tab, select the _Iterative_ method and, from the combo, select _BiCGStabl_. Below, from the _Preconditioning_ combo, select _ILUT_. In my experience, these parameters work well with Helmholtz simulations.
        * I suggest you also tick the box _Abort if the solution did not converge_, so that not ugly solution will be saved if the solver cannot converge.
        * Click on _Apply_.
    * Click on _OK_.
* Select _Model_ from the the top menu, then _Equation_ then _Add_.
    * Click the _Material library_ button.
        * Select _Air (room temperature)_.
        * Click _OK_.
    * In the _General_ tab, replace the value of _Density_ with the MATC expression in **Snippet 3**.
    * In the _Helmholtz Equation_ replace the value of _Sound speed_ with the MATC expression in **Snippet 4**.
    * At the bottom, under _Apply to bodies_, tick the body to which apply our equation (there should be only one).
    * Click _OK_.
* Select _Model_ from the the top menu, then _Boundary condition_ then _Add_.
    * This will be our field radiation condition, so I suggest you edit the _Name_ edit box at the bottom and type _Radiator_.
    * Then, select the _Helmholtz Equation_ tab.
    * Under _Imag part of the flux_, insert the code in **Snippet 5**.
    * Click _OK_.
* Select _Model_ from the the top menu, then _Boundary condition_ then _Add_.
    * This will be our far field condition, so I suggest you edit the _Name_ edit box at the bottom and type _Outlet_.
    * Then, select the _Helmholtz Equation_ tab.
    * Under _Real part of the impedance_, insert the code in **Snippet 6**.
    * Under _Imag part of the impedance_, insert the code in **Snippet 7**.
    * Click _OK_.
* Select _Model_ from the the top menu, then _Set boundary properties_.
* Zoom into the domain with the mouse wheel until the view penetrates the outer surface. The inner surface should now be visible. Double click on it. A window will popup. From the _Boundary condition_ combo, select _Radiator_.
* Zoom out until the outer surface of the domain is visible. Double click on it. A window will popup. From the _Boundary condition_ combo, select _Outlet_.
* Select _File_ from the top menu and then _Save project_. A window will popup.
    * In the new window, click on the _Create Folder_ button. This folder is actually the project folder, and will contain all files. Give it a catchy name.
    * Once in the new folder click on the _Open_ button. This will actually save all the project files in the new folder.
    
The last points about saving the project are a bit strange, so ensure you follow them.

To check that everything is alright, select _Sif_ from the top menu and then _Edit_. This will open the Solver Input File generated by the GUI. If everything is OK, it should look like [this](https://github.com/CrocoDuckoDucks/Acoustic-Models/blob/master/2020-03-07-pulsating-sphere/elmerfem_2/case.sif).

If everything is OK, you area ready to press the _Start solver_ button on the toolbar of the ElmerGUI program. This will start the solver and open the converge plot viewer and the log file viewer. Sit back while the simulation runs!

    
#### Snippet 1
Variables of the simulation. From top to bottom: frequency (hertz), source surface velocity (meters per second), phase speed of sound in the medium (meters per second), density of the medium (kilograms per cubic meter) and radius at which the domain terminates (meters).
```text
$ f = 1000.0
$ U = 0.75
$ c = 343.0
$ p = 1.205
$ r = 0.1
```

#### Snippet 2
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the frequency.
```text
Frequency = Real MATC "f"
```

#### Snippet 3
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the medium density.
```text
Real MATC "p"
```

#### Snippet 4
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the medium phase speed of sound.
```text
Real MATC "c"
```

#### Snippet 5
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the imaginary part of the _Flux_ boundary condition. This is the only part of the _Flux_ boundary condition for this problem. This expression is the same as that outlined in the section **Understanding the Problem** above.
```text
Real MATC "2 * pi * f * p * U"
```

#### Snippet 6
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the real part of the _impedance_ boundary condition. This expression is the same as that outlined in the section **Understanding the Problem** above.
```text
Real MATC "((2 * pi * f * r)^2) / (c * (1 + (((2 * pi * f * r)^2) / (c^2))))"
```

#### Snippet 7
This code instructs to assign the Real value resulting from the MATC expression between quotes to the required parameter. In this case, the imaginary part of the _impedance_ boundary condition. This expression is the same as that outlined in the section **Understanding the Problem** above.
```text
Real MATC "(2 * pi * f * r) / (1 + (((2 * pi * f * r)^2) / (c^2)))"
```

## Results

Once the simulation is finished, we can use ParaView to look at the results. Open ParaView.

Select _File_ from the top menu and then _Open_. Navigate to your ElemerFEM project directory. In the directory, select the _vtu_ file and then click _OK_.

In the properties browser on the left, you will see that the _vtu_ file contains the _pressure wave 1_ and _pressure wave 2_ fields. They are the real and imaginary part of the complex acoustic pressure field. Click on the _Apply_ button. Now the data is loaded into ParaView.

There are tons of good things you can do with ParaView. A ParaView state is included in the repo as a reference for you. Here I will just show you how we can export the field values along a line. If you look at the exact solution for the field, you will see that it depends only from $$r$$. So, to do a simple comparison with theory, we can simply export the field along a radius of the spherical domain.

* From the top menu, select _Filters_.
* Select _Alphabetical_ and then _Plot Over Line_.
* In the _Line Parameters_, under the _Properties_ tab on the left, input the following values:
  | Point1 | 0.005 | 0 | 0 |
  | Point2 | 0.1   | 0 | 0 |
  This will make a radial line along the $$x$$ axis, extending from the source to the edge of the domain.
* Click _Apply_. A new view will open showing a plot of all the fields in the _vtu_ file as a function of the position along the line.

This data can be exported. To do so, select the result of the _Plot Over Line_ filter from the _Pipeline Browser_ on the left. Then, click on _File_ from the top menu and choose _Save Data_. If you follow the steps below, you will be able to use the Julia code `validate.jl` to calculate the error between the fields sampled along the line with the theoretical expected values.

* In the _File name_ edit box type a name with _csv_ extension, for example _linedata.csv_.
* Click _OK_.
* In the _Precision_ editbox, type _30_. This is actually overkill, but it will ensure that all digits of our numbers are written to file.
* Then tick the _Choose Arrays To Write_ tickbox.
* Un-select everything but _pressure wave 1_ and _pressure wave 2_.
* Click _OK_.

You could now edit `validate.jl` so that it points to your _csv_ file. I suggest you put the _csv_ file in the same directory of `validate.jl` and change line 12 with the name of your file. For example, if you called your file _linedata.csv_:

```julia
linedata = readdlm("linedata.csv", ',')
```

You can run this file from the Julia REPL by issuing this command:

```julia 
julia> include("validate.jl")
```
This will work only if Julia was open in the same directory as `validate.jl`.

The script will produce a plot and hang after it displays it until `Enter` is pressed. It might take some time to run the code for the first time, as Julia will have to compile it. The plots show various metrics of error. I will present my results below.

### Coarse Mesh 
![Coarse Mesh](https://raw.githubusercontent.com/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/errors_1.png  "Coarse Mesh")

### Fine Mesh
![Coarse Mesh](https://raw.githubusercontent.com/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/errors_2.png  "Coarse Mesh")

### Discussion
The error can be computed in few different ways. One way is to check the error of the complex numbers. We do this by subtracting the exact solution from the FEM solution. The results of this operation are shown in the 4 plots on the left side of the figures, as real, imaginary, magnitude and phase. We can see that overall the errors are always very small also for the coarse mesh. The real part of the error tends to grow with distance, while the imaginary part is more stable.

The magnitude of the error overall shrinks with distance while its phase show significant variation that, with a magnitude so low, is not considered of concern.

Whilst useful to point out trends with distance, the error of the field is actually more interesting.

If we take the quotient between the FEM solution and the exact solution we obtain an "error field" which quantifies the accuracy of the solution. The magnitude of this field will then be the ratio of the FEM solution magnitude and exact solution magnitude. If the accuracy is high, this number will be very close to 1. Or, if we take the dB value of it (20 times the base 10 logarithm) then it will be close to 0 for very high accuracy. This quantity is represented on the top right panel.

The phase of the error field is instead the difference between the FEM field phase and the exact field phase. For very high accuracy, this difference will be 0. This quantity is represented in the bottom right panel.

It is possible to see that the error is in all cases vanishing small, only a tiny fraction of dB or a radiant. These values are, in all cases, way smaller than the lowest values one can hope to measure in real life (0.1 dB for magnitude, few radiant for phase). Hence, if we had a real spherical source in an anechoic space the values we would measure would agree with all 3 of our models: the exact one, the coarse FEM one and the fine FEM one, telling us that they are all equivalent.

We can then conclude that, despite the gain in accuracy by using the fine mesh is definitely measurable, the coarse solution is already very accurate, confirming that using mesh sizes much lower than one tenth of a wavelength does not bring particular advantages.

# Conclusion
This was quite a long one. However, we learned a few things:

* How to use MATC expression to configure Elmer.
* How to setup impedance boundary conditions with Elmer the right way.
* How to setup source boundary conditions with Elmer the right way.
* That the solution will most likely be good already with a mesh size just smaller than one tenth of a wavelength.
* That we can simulate infinite spaces with finite geometries if we are clever with the boundary conditions.

In reality, the symmetry of the problem would have allowed for many more hacks FEM allows, like solving the problem only on one quarter of the domain. Also, we checked the solution only along one direction and it would be more interesting to check it in the entire domain. But we will do that in a next episode, when we get back to the modes of a room.

But perhaps the next episode should focus on more solver details. Why we set the solver settings the way we did? What is the convergence plot? How do we use it?
