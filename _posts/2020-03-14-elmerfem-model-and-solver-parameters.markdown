---
layout: post
title:  "ElmerFEM Model and Solver Parameters"
date:   2020-03-07 20:00:00 +0100
categories: science physics opensource
---

{% include mathjax.html %}

In the previous episodes we solved a few equations with ElmerFEM. We did some choices when we setup the solver parameters. What those parameters do, and how should we set them? This is perhaps the trickiest part in FEM (beside making the mesh right) and definitely the one I am the least expert about. Still, in this episode we will step back and look at those options more closely. This post is really not meant to be an exhaustive explanation. For that, refer to the [ElmerFEM documentation](http://www.elmerfem.org/blog/documentation/).

# Linear System and Nonlinear System

Every time we set the ElmerFEM Solver Settings, we typically do it through the _Solver control_ window of ElmerGUI. ElmerGUI is very convenient, espcially to start getting our hands dirty with FEM, but it is essentially a GUI layer around various utilities ElmerFEM provides. One of the most important operations it does is to prepare a solver input file (sif) for us. As we will see later in the series, not all ElmerFEM models are supported by the GUI, so we will need to write at least parts of the sif by ourselves. In any case, what Solver settings we should choose?

Turns out this question is actually quite hard to answer.

## Keep your Equation in Mind
We should not think of ElmerFEM, or any other FEM solver for the matter, as a replacement for our brain. Rather, we should try to understand our problem as best as possible to setup the solver in the best possible way. Turns out that this is not always easy, and maybe we will need some trial and error of different solver options, but figuring out the details of our problem will give us a jump-start. With ElmerFEM, most issues you might face are about not setting up the solver properly.

### Your Equation
Look at the equation you aim to solve. For example, so far we dealt with Linear Elasticity and Helmholtz equations, which I am going to report below as formulated as ElmerFEM models (notation might be slightly different with respect the last episodes, but it will be clarified here).

#### Linear Elasticity Equation

<p style="text-align: center;">$$\rho\frac{\partial^{2}\mathbf{d}}{\partial t^{2}}-\nabla\cdot\tau=\mathbf{f}$$</p>

Where $$\rho$$ is the volumetric mass density of the body we are modelling, $$\mathbf{d}$$ is the displacement (from rest) field (a 3D vector), $$\tau$$ is the stress tensor (more details [here](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf#chapter.6)) and $$\mathbf{f}$$ is a body force field, a force value we apply to each point of the body we are modelling (it is a 3D vector and each point of the body can have a different force applied to it).

To make things clearer, let's expand the divergence:

<p style="text-align: center;">$$\rho\frac{\partial^{2}\mathbf{d}}{\partial t^{2}}-\sum_{i=x,y,z}\sum_{k=x,y,z}\frac{\partial\tau_{i,k}}{\partial k}=\mathbf{f}$$</p>


Where $$\tau_{i,k}$$ are the various components of the stress tensor.

#### Helmholtz Equation

<p style="text-align: center;">$$\left(k^{2}-jkD\right)P+\nabla^{2}P=0$$</p>

Where $$k$$ is the wave number, $$D$$ is an optional dumping factor, $$j$$ is the imaginary unit and $$P$$ is the spatial part of the acoustic pressure field. To make things clearer let's expand the Laplace operator:

<p style="text-align: center;">$$\left(k^{2}-jkD\right)P+\frac{\partial^{2}P}{\partial x^{2}}+\frac{\partial^{2}P}{\partial y^{2}}+\frac{\partial^{2}P}{\partial z^{2}}=0$$</p>

See [here](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf#chapter.11) for more information.

### First Question: are they Linear?
To understand whether the PDE is linear, we need to look at what happens to our unknown field. Being a PDE, our unknown field will undergo differentiation. If **all** the terms of the PDE are linear, then the PDE is linear. A linear term is a term in which a derivative is used _as is_, or eventually multiplied by a number or any function **different** from the unknown field. For example, if $$F$$ was an unknown scalar field (but it is the same for vector fields) the terms below would be all examples of linear terms:

<p style="text-align: center;">$$\frac{\partial F}{\partial x},\,4\frac{\partial^{7}F}{\partial z^{7}},\,x^{4}\frac{\partial^{3}F}{\partial y^{3}},\,g\left(x,y,z\right)F$$</p>

While the terms below are nonlinear:

<p style="text-align: center;">$$F\frac{\partial F}{\partial x},\,\ln\left(\frac{\partial^{5}F}{\partial x^{5}}\right),\,\left(\frac{\partial^{3}F}{\partial y^{3}}\right)^{6},\,\frac{g\left(x,y,z\right)}{F}$$</p>

Let's look at the Helmholtz equation first. $$P$$ is our unknown field, and clearly there are only linear terms in the Helmholtz equation. So, the equation is linear.

Let's look now at the linear elasticity equation instead. The equation itself is, well, linear, as the derivatives of the unknown field are linear terms. However, there might be a catch! The stress tensor $$\tau$$ depends on the strain, and the strain in turns depend on our unknown field $$\mathbf{d}$$ and its derivatives. In linear materials, or if we use a linearised formulation, then the dependency is still linear, and in the double sum above we will have linear terms of mixed derivatives of the unknown field. So, the linear elasticity equation is linear as long as we treat the material as linear. Things simplify even more if the material is isotropic (same elasticity properties along every dimension) as in that case we can figure out the stress tensor out of two numbers: Poisson ratio and Young modulus.

Note that we could say a similar thing about acoustics: the acoustic pressure will disturb the material properties. However, the Helmholtz equation is derived from the wave equation which, in turn, is derived in the approximation of adiabatic wave propagation, so these effects are not accounted for. We will see how things change for more advanced acoustic equations.

So, be sure you understand the material parameters too, as they might depend non-linearly on the unknown field. In our previous studies we specified isotropic linear parameters for linear elasticity, so our equations were linear.

## The Linear System Parameters
![Linear System Parameters in ElmerGUI]({{ site.baseurl }}/res/pictures/2020-03-14-elmerfem-model-and-solver-parameters/figure_1.png  "Linear System Parameters in ElmerGUI")

Now that we know whether our equation is linear or nonlinear, we can understand better what the various parameters of our solver mean.

Linear PDEs are turned by FEM into linear systems of equations that are formulated as matrix equations. These matrices are big, as they are assembled from the various equations at each single node of the mesh.

These matrix equations can be solved directly. We did so for linear elasticity. This is a very good method, as it will solve the matrix equation exactly within numerical precision (see [here](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerSolverManual.pdf#chapter.4)), but they do not scale up for very big problems (very fine meshes and/or high mesh order) as they will require massive amounts of memory and CPU time. If your problem is small enough, end even better, if it is 2D, i would recommend you start off with the _Umfpack_ direct method.

To work around this, iterative methods can be used, where the linear system is solved by iterative algorithms. This is where things become complicated.

Iterative methods bring in convergence. So we ideally want two things:

* Converge in a small number of iterations.
* Compute each iteration in a reasonably small time.

But what convergence mean? It means that the solution is not varying a lot between consecutive iterations, hence we are reaching the final correct solution of the linear system. We can specify the convergence tolerance for ElmerFEM, and I would suggest to start from the default value of $$10^{-10}$$. Additionally, we can specify the maximum number of iterations. After the number of iterations is reached, ElmerFEM will stop, but be aware that our solution will most likely be inaccurate if we did not reach convergence, for this reason we should tick the _Abort if the solution did not converge_ box.

There are many different iterative methods that one could use, each one best suited for a certain type of linear system. I would recommend to start with _BiCGStabl_, with a _BiCGStabl order_ of $$2$$. This method should give you a smoother convergence in most cases.

However, there is one more thing: preconditioning. Preconditioning is an operation carried on the matrices to aid convergence rate. The best preconditioning has to be searched by trial and error. I normally start with _ILUT_ and a default _ILUT tolerance_ of $$10^{-3}$$.

Finally, ElmerFEM provides Multigrid methods, but I am not familiar with those.

For more information, please refer to the [Elmer Solver Manual](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerSolverManual.pdf), especially [this chapter](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerSolverManual.pdf#chapter.4). As it is not easy to nail down what the matrices for complex geometries will look like, trial end error becomes important. For big problems you will use iterative or multigrid methods. I recommend you test many simpler (smaller) problems with different methods and preconditioning, until you are satisfied you minimised both the number of iterations needed for convergence and the time each iteration needs. You will see this by looking at the ElmerFEM log. Be aware that not all iterative methods bring smooth convergence.

## The Nonlinear System Parameters
![Nonlinear System Parameters in ElmerGUI]({{ site.baseurl }}/res/pictures/2020-03-14-elmerfem-model-and-solver-parameters/figure_2.png  "Nonlinear System Parameters in ElmerGUI")

Well, equations can be nonlinear, as we seen. So, how can the maths of linear systems deal with them? Well, the nonlinear system can be linearized and solved iteratively. These are the so called nonlinear iterations. Most solvers for nonlinear equations have tunable linearization methods, so we will see most of this stuff later. For our linear equations we could (and perhaps should) set the _Max. iterations_ to $$1$$, as we essentially do not need them. We did so for linear elasticity, but I left the default value for our spherical source because ~~I forgot about it~~ on purpose so that I could explain a few things here.

![Convergence Plot for Spherical Source](https://media.githubusercontent.com/media/CrocoDuckoDucks/Acoustic-Models/master/2020-03-07-pulsating-sphere/elmerfem_2/solver.png  "Convergence Plot for Spherical Source")

ElmerFEM reports the nonlinear convergence plot during the solver execution, which is shown above for our [Pulsating Sphere](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/07/pulsating-sphere.html) problem. We can see that, in fact, we did only one nonlinear iteration before exiting, as the solution did not change in the slightest between the first solution and the first nonlinear iteration, as expected for linear problems.

We will touch more on this when we get to nonlinear PDEs.

# Solver Specific Options
Well, these are solver specific, so they change between solvers. There is very little general stuff to say about this. It is best to refer to the [Elmer Models Manual](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf) to check what we can do. With the Solver Specific Options we can change what we solve for (for example, setup _Eigen Analysis_ for linear elasticity, or set special numerical algorithms which can aid accuracy and convergence as well.

# Conclusion
This post was more about raising awareness about how nontrivial it is to solve problems with FEM. The best possible way to ensure that we know how to solve a problem properly is do a few studies with benchmark solutions, as we are doing, and compare them to known analytical solutions. The steps to setup our study properly are as follows:

* Read the [Elmer Models Manual](http://www.nic.funet.fi/pub/sci/physics/elmer/doc/ElmerModelsManual.pdf) and figure out whether you need or not the Nonlinear iterations. Be careful that non-linearity can be hidden into the materials dependency on the unknown fields.
* Set a few benchmark problems against known analytical solutions (such as our spherical source or room model) to test various Direct (if the problem doesn't have too many nodes) Iterative or Multigrid solvers, with different preconditioning, as well as different solver specific settings. We want:
    * Convergence in a reasonably small number of iterations.
    * Reasonably fast iterations.
note that in reality it is possible to have hundreds of iterations that might take a few minutes, if the problem is big. I had to run models overnight in the past (but it is possible to cut execution time with parallelization, as we shall see).
 * Once we have figured out what solver parameters give us the best results, consistently among different benchmarks, then we can take the same parameters to more complex problems, knowing they will work alright (solver specific settings are probably going to be tightly related to particular problems, though).

So, now that we are aware of this, we will discuss our solver settings choices a little bit more in the next episodes.
