---
layout: post
title:  "Modelling Acoustics With Open Source Software"
date:   2018-10-27 20:42:56 +0100
categories: science physics opensource
---

As part of my profession as an acoustician, I often make use of Open Source software. It might surprising to few that the Open Source ecosystem is actually filled with very good packages for this task. As a result, I decided to try to publish a little series of tutorials about the topic, and this would be its first post.

So let's first have a look ad the most important numerical simulation programs and technical computing languages that can be used to simulate acoustics. There are many programs out of there, but I mainly made use of ElmerFEM. ElmerFEM will be the focus of the series then, and we will look at how to set up Ubuntu 18.04 to be our workstation!

<!--more-->
<h1>The Ecosystem</h1>
There is plenty of Open Source packages that are either designed to simulate acoustics, can simulate acoustic if one wants to, or can be used to simulate certain parts of multi-physics systems, or can be used for pre and post processing. Here few examples.
<h3>Technical Computing Languages</h3>
There are many technical computing languages, or languages that can be useful for technical/scientific computing. The most commonly used are:
<ul>
	<li><a href="https://www.gnu.org/software/octave/" target="_blank" rel="noopener">GNU Octave</a>. This is very useful also for having compatible syntax to that of Matlab, but it is not really my favorite, as it is a bit slow.</li>
	<li><a href="https://www.python.org/" target="_blank" rel="noopener">Python</a> (in conjunction with many packages such as  <a href="http://www.numpy.org/" target="_blank" rel="noopener">Numpy,</a> <a href="https://www.scipy.org/" target="_blank" rel="noopener">SciPy</a>...). Python is very nice to code with, and there are tons of scientific packages for it. Moreover, certain numerical software programs and frameworks allow Python scripting (ParaView, as an example, see below) or have Python interfaces. So, it is a language that it is worth having a grasp of.</li>
	<li><a href="https://julialang.org/" target="_blank" rel="noopener">Julia</a>. OK, I am not sure it "among the most used", but it is a new exciting language that feels like what would happen if Octave and Python had a baby. I really like it, and we will probably make a lot of use of it. The main reason being that this is a good excuse for me to study Julia.</li>
</ul>
This list is not exhaustive: there are many more examples, such as <a href="https://www.r-project.org/" target="_blank" rel="noopener">R</a> and <a href="http://www.scilab.org/" target="_blank" rel="noopener">Scilab</a>, and there actually are many more open source scientific frameworks such as <a href="https://root.cern.ch/" target="_blank" rel="noopener">ROOT</a>. However, I decided to mainly stick with what I am most familiar with.

Also, I should mention <a href="http://maxima.sourceforge.net/" target="_blank" rel="noopener">Maxima</a>, which I like to use with the front-end <a href="https://wxmaxima-developers.github.io/wxmaxima/" target="_blank" rel="noopener">wxMaxima</a>. It is a very useful computer algebra system which I often used for symbolic computation.
<h3>Physical Modelling DSP</h3>
The <a href="https://faust.grame.fr/" target="_blank" rel="noopener">Faust programming language</a> is a very powerful language to program real-time DSP algorithms. In its library there are numerous functions for physical modelling of music instruments, Moreover, it has an entire <a href="https://ccrma.stanford.edu/~rmichon/pmFaust/" target="_blank" rel="noopener">Physical Modelling Toolkit</a>. Although I am not a Faust expert, and I sporadically use it, it would be nice to try to use it to turn a simulation result into a plugin!
<h3>Numerical Simulation and Analysis Packages</h3>
These packages are suites for numerical modelling, using various methods.
<ul>
	<li><a href="https://www.csc.fi/web/elmer" target="_blank" rel="noopener">ElmerFEM</a> is a multiphysical FEM solver which is maybe a bit hard to use, but really powerful. We will be using this a lot. It has very good solvers for various acoustics problems, and we can study vibroacoustics coupling too, and much more.</li>
	<li><a href="http://acousto.sourceforge.net/" target="_blank" rel="noopener">AcouSTO</a> is an interesting BEM solver for acoustics that I regrettably still have to try.</li>
	<li><a href="http://www.openpstd.org/index.html" target="_blank" rel="noopener">OpenPSTD</a> is an acoustics simulation program that uses the Pseudo Spectral Time Domain Method. It integrates with Blender.</li>
	<li><a href="http://i-simpa.ifsttar.fr/" target="_blank" rel="noopener">I-Simpa</a> is open source software designed to simulate 3D acoustic propagation. Another one to add in my must try list.</li>
	<li><a href="http://www.femm.info/wiki/HomePage" target="_blank" rel="noopener">FEMM</a> is a finite element modelling program for magnetics. This can be very useful to model and design speaker magnets, but I never used it myself.</li>
	<li><a href="http://mesh2hrtf.sourceforge.net/" target="_blank" rel="noopener">Mesh2HRTF</a> seems to a very interesting package for the numerical simulation of Head Related Transfer Functions (HRTF). Unfortunately, it has Matlab as a dependency. Still, this is pretty high in my must try list.</li>
	<li><a href="http://www.signal.uu.se/Toolbox/dream/" target="_blank" rel="noopener">The DREAM Toolbox</a> is a Matlab/Octave toolbox for the simulation of ultrasonic speaker arrays.</li>
	<li><a href="http://qucs.sourceforge.net/" target="_blank" rel="noopener">Qucs</a> is a nice, quite easy to use, circuit simulator. It can be used to simulate, of course, electronic circuits to use for audio applications, but it can also be used to build lumped acoustic models through electrical analogies (see <a href="https://en.wikibooks.org/wiki/Engineering_Acoustics/Electro-Mechanical_Analogies" target="_blank" rel="noopener">here</a>, for example).</li>
</ul>
There are actually few more solvers, such as <a href="http://www.calculix.de/" target="_blank" rel="noopener">Calculix</a>, <a href="https://code-aster.org/spip.php?rubrique2" target="_blank" rel="noopener">Code_Aster</a>, <a href="https://openfoam.org/" target="_blank" rel="noopener">OpenFOAM</a>, <a href="http://getfem.org/" target="_blank" rel="noopener">GetFEM++</a>, <a href="http://www.freefem.org/" target="_blank" rel="noopener">Freefem++</a> and <a href="http://onelab.info/" target="_blank" rel="noopener">ONELAB</a> but again I decided to stick with either those I tried, or those that are most relevant to acoustics.

There are cool things too more in the domain pf pyschoacoustcs and analysis:
<ul>
	<li>The <a href="http://amtoolbox.sourceforge.net/" target="_blank" rel="noopener">Auditory Modelling Toolbox</a> is a cool open source Matlab/Octave toolbox for auditory modelling.</li>
	<li>The <a href="http://ltfat.github.io/" target="_blank" rel="noopener">The Large Time-Frequency Analysis Toolbox</a> is a Matlab/Octave toolbox that provides many useful transforms for signal analysis that can be very useful to understand acoustic signals.</li>
	<li>The <a href="http://psychtoolbox.org/" target="_blank" rel="noopener">Psychtoolbox</a> is another Matlab/Octave toolbox that can be used for realtime analysis of auditory stimuli.</li>
	<li><a href="https://cmusphinx.github.io" target="_blank" rel="noopener">CMUSphinx</a> is an open source toolbox for speech recognition.</li>
</ul>
<h3>3D Modelling and CAD</h3>
In order to solve a problem, one must first create its geometry. There are really many programs out of there which can be used for this, but those I actually know best are:
<ul>
	<li><a href="https://www.freecadweb.org/" target="_blank" rel="noopener">FreeCAD</a>, this is very good to prepare geometries for FEM solving. FreeCAD can also provide an interface to solvers.</li>
	<li><a href="https://www.blender.org/" target="_blank" rel="noopener">Blender</a>, this is more for graphics, but certain solvers (such as AcouSTO) actually have plugins for Blender. Blender is another good example of software that allows internal Python scripting.</li>
</ul>
<h3>Pre and Post Processing</h3>
In order to use any solver, our geometry needs to be prepossessed. Moreover, our results from the numerical solvers might need further postprocessing. This is why we need these packages:
<ul>
	<li><a href="http://gmsh.info/" target="_blank" rel="noopener">gmsh</a> is a great, and very lightweight, tool which does the job nicely.</li>
	<li><a href="http://www.salome-platform.org/" target="_blank" rel="noopener">Salome Platform</a> is for now my favorite tool when it comes to pre-processing. To be noticed that Code_Aster distributes Salome-Meca, a modelling suite based on Salome Platform incorporating the Code_Aster solver.</li>
	<li><a href="https://www.paraview.org/" target="_blank" rel="noopener">ParaView</a> is instead my favorite post-processor.</li>
</ul>
<h1>Getting Started on Ubuntu 18.04</h1>
Now that we seen what the ecosystem has to offer, let's try to build a simple simulation workstation based around ElmerFEM, with which I am most familiar. So, let's install the various packages on Ubuntu and let's prepare our environment. We will use the command line to install the packages.

But first, why Ubuntu? Well, it turns out that, even though Ubuntu is missing few of the packages we want to use in its repo, all packages are pretty easy to install, especially ElmerFEM and Salome, which can be quite complicated in other distros.
<h3>FreeCAD</h3>
It is best to install from the <a href="https://launchpad.net/~freecad-maintainers/+archive/ubuntu/freecad-stable" target="_blank" rel="noopener">FreeCAD stable releases PPA</a>:

[code language="bash"]
sudo add-apt-repository ppa:freecad-maintainers/freecad-stable
sudo apt-get update
sudo apt-get install freecad
[/code]

This should pull in the latest stable FreeCAD version. There is a not very outdated version in the official Ubuntu repos, but it does not seem to work on Ubuntu 18.04 at the time of writing.
<h3>Salome</h3>
To install Salome we should head to the <a href="http://www.salome-platform.org/downloads/current-version" target="_blank" rel="noopener">download page</a>. At the moment of writing there isn't a version for Ubuntu 18.04, but I tried the universal version and it seems to work fine after a cursory test. So, let's just download the universal binaries for Linux, and then let's open a terminal in the download folder. These commands should take care of the installation:

[code language="bash"]
chmod +x Salome-V8_5_0-univ_public.run
./Salome-V8_5_0-univ_public.run
[/code]

You might need to adapt the commands to match the filename of your download. The installer will ask a few questions. Answering the default by pressing enter worked just fine in my case.
<h3>ElmerFEM</h3>
ElmerFEM can be very conveniently installed from the <a href="https://launchpad.net/~elmer-csc-ubuntu/+archive/ubuntu/elmer-csc-ppa" target="_blank" rel="noopener">Ubuntu PPA</a> with these commands:

[code language="bash"]
sudo apt-get install libqt5xml5
sudo apt-add-repository ppa:elmer-csc-ubuntu/elmer-csc-ppa
sudo apt-get update
sudo apt-get install elmerfem-csc-eg
sudo cp -rs /usr/share/ElmerGUI/edf-extra/* /usr/share/ElmerGUI/edf/
[/code]

The last command is meant to create links to the various ElmerGUI elements that by default are not included in the GUI. It could be best to run it after each upgrade.
<h3>Julia</h3>
It is now time to install Julia. At the moment, there aren't official packages for Ubuntu 18.04, so let's follow the <a href="https://julialang.org/downloads/platform.html" target="_blank" rel="noopener">Julia installation tips</a> for generic Linux operating systems. Let's download the Linux binary from <a href="https://julialang.org/downloads/" target="_blank" rel="noopener">here</a>. Then extract the archive. I would suggest to do it in a reasonable folder, where you will not risk to remove Julia by mistake. Then, let's create a symbolic link to the Julia executable to a folder in the PATH variable. /usr/local/bin should be a good choice. In this example, I extracted the archive in my home folder, resulting in the folder julia-1.0.1:

[code language="bash"]
sudo ln -fs ~/julia-1.0.1/bin/julia /usr/local/bin/julia
[/code]

As above, you might need to adapt the command to reflect the extracted folder name for future versions. If you now issue the command "julia" to the terminal, you should be able to see the julia REPL. use exit() to quite Julia.
<h1>Other Distributions?</h1>
It is possible, of course, to set any distribution to be a numerical modelling workstation. There is also a ready to use Linux distribution for the purpose: <a href="https://caelinux.com/CMS3/" target="_blank" rel="noopener">CAELinux</a>.
<h1>What's next?</h1>
I will try to publish some baby step tutorials regularly. We will maybe first try to model a rectangular room, and compare it with the known analytical solution, of which we will build a Julia version.

So, let's just hope I can keep up with this commitment!
