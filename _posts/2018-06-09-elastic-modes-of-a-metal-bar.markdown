---
layout: post
title:  "Elastic Modes of a Metal Bar"
date:   2018-06-09 20:42:56 +0100
categories: science physics opensource
---

So, I was planning to publish one of these tutorial-style posts per month, but things slipped a bit. But I did not forget about this, and this post is the last episode of the Modelling Acoustics with Open Source Software series.

So, without further due, let's dig into the modelling of elastic modes with Open Source Software.


# Why Elastic Modes
In the [last episode](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html)  we examined the analytical solution of the acoustic modes of a rectangular room. The idea was that now we would move on to use FEM, and ensure that we understand how to use it by comparing the approximate solution from FEM with the analytical one.

However, I decided to backtrack a little and instead follow a slightly different approach. At the time of writing, ElmerFEM (the FEM solver that we will use) does not have an Helmholtz eingensolver. Hence, studying modes is a bit more complicated than would it should be for a first introduction to ElmerFEM.

Since I remember how hard it was to get started with ElmerFEM, I decided then to propose a problem that is simpler to handle within the program: that of the elastic modes of solid things. This will allow us to get started with ElmerFEM in the most gentle way possible, by only using the GUI.

And of course, it is all very well related to acoustics. In fact, modes of solids are of crucial importance. Exactly as [we observed for acoustic modes](https://crocoduckoducks.github.io/science/physics/opensource/2018/12/31/acoustic-modes-of-a-rectangular-room.html), a sum of modes is what defines the steady state vibration of elastic bodies, and the vibration of elastic bodies is one of the most important causes of sound radiation in fluids. As an example, see this beautiful simulation made with ElmerFEM by [Ben Qui](https://www.youtube.com/channel/UCMcP1V6o9MfDyJSp2te6xkA)  on the soundfield radiated by the acoustic modes of a tuning fork.

[![Tuning Fork - FEM Modal & Vibroacoustic simulation with Elmer](http://img.youtube.com/vi/i5bJL4CbAxk/0.jpg)](http://www.youtube.com/watch?v=i5bJL4CbAxk)

We will not try a simulation like that one (yet). We will study only the modes of a very simple bar of  metal, without coupling it with air.

# Accompanying Videos
To make things easier to follow, I decided to record a bunch of videos in a tutorial style which you can follow. I have put them into [one playlist on YouTube](https://www.youtube.com/playlist?list=PLgBFAun2MIdNjX9wL7g6ILFV7DQTwp6Eg).


You will probably remember from <a href="https://crocoduckoducks.github.io/science/physics/opensource/2018/10/27/modelling-acoustics-with-opensource-software.html" target="_blank" rel="noopener">Modelling Acoustics with Open Source Software</a> that we discussed how to install the software on Ubuntu Linux. However, my videos are recorded on Arch Linux. This is way much more convenient for me as I do not have an Ubuntu computer locally, but only remote one. So, to be able to record with good quality, I prefer to use my local Arch Linux box. Nothing really changes in the way the programs are used.

<h1>The Workflow</h1>
Simple problems are good as they allow us to introduce the basic workflow for numerical problem solving in physics. We will go through the following steps:
<ol>
	<li>Geometry Setup.</li>
	<li>Meshing.</li>
	<li>Solver Setup.</li>
	<li>Solution.</li>
	<li>Visualization and Post-processing.</li>
</ol>

<h1>Geometry Setup</h1>
The first step is always to define the geometry. There are many ways to do this, but we will use FreeCAD. The reason being that very often, when we model real systems, we will start from CAD files. So, it is best to get used to CAD programs.

In this example, we will create just a simple bar with square cross section, with a 10 mm by 10 mm base and a height of 100 mm.

Watch the video below to see how to setup the geometry.

[![Modes of a Metal Bar: Geometry](http://img.youtube.com/vi/eYiGw6cc8lg/0.jpg)](http://www.youtube.com/watch?v=eYiGw6cc8lg)

<h1>Meshing</h1>
So, we mentioned that we will be using FEM to solve for the modes of vibration of our physical system. But what that means exactly?

This is probably not the best place to cover FEM in great detail, but we can do a simple introduction.

You will remember that in the episode <a href="https://crocoduckoducks.github.io/science/physics/opensource/2018/11/30/what-is-acoustic-modelling.html" target="_blank" rel="noopener">What is Acoustic Modelling</a> we said that most physical phenomena are governed by Partial Differential Equations (PDEs) and that we use solvers to get approximated numerical solutions for them. FEM, shorthand for Finite Element Method, is a very important kind of solver.

FEM is able to provide an approximated solution to our governing PDE by altering the <em>domain</em> to which we apply it. The domain is no other than the portion of space in which we apply the PDE, the bar itself in this case. The first step of FEM is that of <em>Meshing</em>: we cut the domain in many small not overlapping subdomains. They need not to have the same size or shape. At the vertices of these domains we put <em>nodes</em>, single points. For higher order meshes we will have nodes also inside. Then, we define basis functions over the inside of the domain, typically a family of polynomials whose order is linked to the mesh order. Simply speaking, the approximated solution to our problem is defined by a weighted sum of these basis functions, and the weights are found by solving matrix equations involving the value of the unknown field at the nodes, which is then the real unknown of the problem. Given that the number of nodes is then the number of unknowns, this number is also called the number of degrees of freedom. In essence, FEM works by simplifying a problem over infinite points (all the points inside the domain) to a problem over a finite set of points (the nodes). However, thanks to the basis functions, the solution is actually defined everywhere in the original domain through interpolation.

This might seem very complicated at first, and in fact it is quite advanced mathematics, but the important takeaway here is this:

Given that meshing is the process at the very core of the FEM method (although there is more to it) we should make it right.

Now, that of <em>convergence</em> of the FEM solution to the real one is another advanced topic. Very high order and very high mesh density might actually make convergence harder. Moreover, the higher the order or the density the higher the computational requirements. On the other hand, accuracy will be reduced for low order and density.

Since this is our first attempt, let's just do a simple coarse, but second order, mesh, and see whether by critically looking at the results we can see whether we can trust them. This is a very sane approach: <span style="text-decoration:underline;">never just blindly trust the solver</span>!

Watch how to use Salome to mesh our geometry in the video below. There are many online resources if you want to lookup more info on FEM and meshing. One that I would recommend is the <a href="https://www.comsol.com/multiphysics" target="_blank" rel="noopener">COMSOL's multiphysics cyclopedia</a>.

[![Modes of a Metal Bar: Meshing](http://img.youtube.com/vi/r6k66HekS3c/0.jpg)](http://www.youtube.com/watch?v=r6k66HekS3c)

<h1>Solver Setup and Solution</h1>
Now, ElmerFEM is the only bit of software that I did not get to work on Arch Linux. So, I will be using the virtual machine available <a href="https://github.com/ElmerCSC/elmerfem/wiki/Packages" target="_blank" rel="noopener">here</a>. This is, by the way, another reason why I kept the mesh coarse on the videos. When I will need to solve problems with finer mesh I will do so on a remote Ubuntu machine I set up somewhere else. Remember that you can refer to <a href="https://crocoduckoducks.github.io/science/physics/opensource/2018/10/27/modelling-acoustics-with-opensource-software.html" target="_blank" rel="noopener">Modelling Acoustics with Open Source Software</a> to read how to install all the programs mentioned here on Ubuntu.

Follow the video to check how the solution process works.

[![Modes of a Metal Bar: Solving](http://img.youtube.com/vi/R9MTok7IAsA/0.jpg)](http://www.youtube.com/watch?v=R9MTok7IAsA)

<h1>Visualization and Post-processing</h1>
Now that the problem is solved, we can look at the solution itself.

[![Modes of a Metal Bar: Post-processing](http://img.youtube.com/vi/T_Fx-QZ2unA/0.jpg)](http://www.youtube.com/watch?v=T_Fx-QZ2unA)

By having a look around with ParaView it feels like the last mode, at highest frequency, has a bit of a strangely looking solution, with some weird stuff going on inside. Perhaps, to solve with good accuracy at such a high frequency, we need a finer mesh. In fact, at that mode, the metal bar is twisting around its own axis. So, to calculate the rotation of the bar as well as possible, we should have a high density of meshes going from the axis to the borders. This is where FEM comes handy: we can have different degrees of mesh density along different directions. But let's keep that for later.

<h1>Can we Trust it?</h1>
This physical problem, that of the modes of a metal bar, is already quite not trivial. Other solutions available somewhere are still approximate. However, we used a bar, and not some other very complicated object such as a car door, for example, as we wanted to compare with other solutions, to build a sense of confidence in it. The goal is to find the correct way to mesh the geometry and set the solver so that we can trust the solution, and then we will use what we learn on more complex problems.

So. let's dig out other approximate solutions. One is reported <a href="http://hyperphysics.phy-astr.gsu.edu/hbase/Music/barres.html" target="_blank" rel="noopener">here</a>. The solution in there holds for thin bars, while ours is a pretty thick one. Also, the boundary condition is not the same, as the clamped condition also applied to the derivative of the field. But let's have a look anyway.

Let's calculate the first 3 resonance frequencies of the bar according to the thin bar formula.

{% include mathjax.html %}

From the Elmer material library, we know the Young's modulus and density of the bar:
<p style="text-align:center;">$$ Y=7\cdot10^{10}\,\textrm{Pa}$$</p>
<p style="text-align:center;">$$ \rho=2700\,\frac{\textrm{kg}}{\textrm{m}^{3}}$$</p>
While length and thickness are known from our geometry:
<p style="text-align:center;">$$ L=0.1\,\textrm{m}$$</p>
<p style="text-align:center;">$$ a=0.01\,\textrm{m}$$</p>
So, in the thin bar approximation the fundamental frequency is:
<p style="text-align:center;">$$ f_{1}\approx824.86\,\textrm{Hz}$$</p>
Which is very very close to what our ElmerFEM model predicts, with two close first modes at 824.50 Hz and 824.57 Hz respectively. As we noted in the video, the two modes are most likely degenerate, i.e. actually related to the same modal frequency, which we can think of being 824.53 Hz (the mean value). So, all good for the first modal frequency!

For the second, the thin bar theory predicts:
<p style="text-align:center;">$$ f_{2}\approx5171.89\,\textrm{Hz}$$</p>
While we got a prediction more akin to 4941.93 Hz, by reasoning along the same lines as above. Whilst there is clearly a difference, it is not as dramatic as it might look like, amounting to just 78.92 <a href="https://en.wikipedia.org/wiki/Cent_(music)" target="_blank" rel="noopener">cents</a>. So, the difference is pretty much 80% of a semitone, not huge but significant, and it would be definitely audible by a human.

Finally, let's look at the third frequency:
<p style="text-align:center;">$$ f_{3}\approx14476.36\,\textrm{Hz}$$</p>
Wow! That's now hugely different with respect what Elmer gave us, which is pretty much 7291 Hz. So, what is going on?

Well, the thin bar modal theory that we are using as a benchmark not only works well with thin bars, but also if displacement is only happening in one direction, the so called transverse vibration. In fact, the benchmark solution seems consistent with the theory detailed, for example, in <a href="https://www.mapleprimes.com/DocumentFiles/206657_question/Transverse_vibration_of_beams.pdf" target="_blank" rel="noopener">here</a>. Now, if you recall the FEM solution, at 7291 Hz our bar was twisting around its own axis. So, clearly we are not talking about a transverse mode. In fact, if we look at our <a href="http://hyperphysics.phy-astr.gsu.edu/hbase/Music/barres.html" target="_blank" rel="noopener">reference page again</a>, we see that while the first two modes in the picture kinda look like our first fours from Elmer, the third really doesn't look like our fifth one from Elmer. Interestingly, 7291 Hz is almost half of the third resonance frequency for the transverse vibration of a thin bar...

So, it looks like two things are going on here:

The most widely known analytical solution for the problem, the one which we are using as a benchmark, is derived in more special conditions with respect our FEM model, so direct comparison is hard. At the same time, our mesh size might be introducing errors at the higher modes, as we seen in the video, and we should try to reduce it. We ought to expected lower accuracy the higher the frequency, as the bar will be taking more complex vibration patterns, which hence would need a higher mesh density to be solved for accurately (just think of how many little mesh volumes we have per twist of the bar).

Still, the first mode frequency is quite accurate. The lowest mode is where we can expect all approximate models to agree, so it looks like we are going in the right direction.

In the next episode we will we use all of this information to refine the model.
<h1>The Takeaway</h1>
Even when the problem looks simple, we should not just trust the solver. It is a good idea to solve a simple case first, and compare with other solutions available in literature. Once we are satisfied about the simple solution, we can apply the same meshing paradigm and solver setting to other problems, with much higher confidence in the results we will get.

I will see you on the next episode then!