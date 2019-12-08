---
layout: post
title:  "Acoustic Modes of a Rectangular Room"
date:   2018-12-31 20:42:56 +0100
categories: science physics opensource
---

In this episode about Open Source Acoustic Modelling we will look at how to make a simple Julia model of one of the simplest systems in acoustics, a rectangular room with rigid walls, assuming adiabatic wave propagation. Even if this system is among the simplest in acoustics, it is actually already very complicated. As such, we will focus only on the modes, one part of the problem, without attempting impulse response simulation or other fancy things like that, for now.

<h1>Motivation</h1>
We use Julia to implement the model by taking advantage of the fact that a closed form solution exists for this problem. So, we will not use a solver. This might feel strange, as in the previous episodes we covered the used of solvers to solve complex equations. Also, the usefulness of playing with a simplified model might seem weak, given that most rooms are not rectangular, the walls are not rigid (but propagation is quite adiabatic at low sound pressure levels). So, why are we doing this?

The reason is that these simple systems constitute very good benchmarks for our solvers. In fact, solvers can only provide us with approximate solutions, and the quality of the approximation will depend on how well we set the solver parameters and input data. But if we don't know the solution, how can we know how good an approximation of the solution is? This is where systems with known solutions come into place: we can directly compare solvers outputs with the known analytical solution, thus being able to assert whether the solver is being accurate and reliable. Once we figure out how to configure our solver thanks to the benchmark model, we can then move to more complex problems.

<h1>The Equation</h1>
We will very briefly look at the equation. A comprehensive solution and analysis is out of the scope of the post, but we will look at the most important parts so to clarify what we are doing. If you are an expert, just skip the section.  If you instead want to know more, <a href="https://www.wiley.com/en-us/Fundamentals+of+Acoustics%2C+4th+Edition-p-9780471847892" target="_blank" rel="noopener">Fundamentals of Acoustics</a> would be a good reference.

{% include mathjax.html %}

The equation governing adiabatic wave propagation in a compressible fluid (such as air) is the so called d'Alembert equation:
<p style="text-align: center;">$$ \nabla^{2}p=\frac{1}{c^{2}}\frac{\partial^{2}p}{\partial t^{2}}$$</p>
where $$ c$$ is the speed of sound in air, related to the air thermodynamic properties. At room temperature and standard pressure, it is 343 meters per second. This equation will describe air in a rectangular room as soon as we define the unknown pressure disturbance field $$ p$$ over all points of a 3D rectangular volume (a Parallelepiped, to be clear). We can describe length, width and height of the volume with the 3 numbers $$ L_x$$, $$ L_y$$ and $$ L_z$$. One of the room corners can be the center of our reference frame, and we can align the reference frame axes to run along the edges of the room. We will call the Cartesian coordinates $$ x$$,  $$ y$$ and $$ z$$. Then, we make the walls rigid by imposing the normal component of particle velocity to be null at the walls. Since particle velocity is proportional to the spatial partial derivatives of the pressure disturbance, those are what we put to zero. In other words, our boundary condition is on the pressure field spatial derivatives at the walls.

Now, we could place some kind of field source in the room, but for the time being we will only look at the normal modes of the room. In fact, due to geometry, there are special frequencies at which waves bouncing off the walls superpose just right to produce a stationary wave, that is, a wave localized in space, whose amplitude changes in time at the relative special frequency.

This will become clearer as we go along, but for the time being, let's clarify why we search for stationary waves. The reason is that every steady state vibration of air in the room can be represented by a sum of all these special modes, with various weighting coefficients. Hence, in terms of modal superposition, we can express the frequency response of a room. For this reason, and many additional ones, room modes are of crucial importance in room acoustics.

Property of modes is that spatial and temporal properties of the wave are factorized, which means that the pressure disturbance is the product of a function that depends only on space coordinates and a function that depend only on time:
<p style="text-align: center;">$$ p\left(x,y,z,t\right)=S\left(x,y,z\right)T\left(t\right)$$</p>
To search for solutions of this kind we can substitute the equation above into the wave equation, which, after some manipulation, brings us to the Helmholtz equation for the spatial part:
<p style="text-align: center;">$$ \nabla^{2}S=-k^{2}S$$</p>
Which has the form of an <em>eigenvalue problem</em>, as we are searching for the all special $$ S$$ and $$ k$$ for which the action on $$ S$$ of the differential operator on the left equates to multiplying by a scalar real value $$ -k^{2}$$. Another equation will hold for $$ T$$, which has a simple solution: an harmonic wave. The solution for $$ S$$ has instead this shape:
<p style="text-align: center;">$$ S\left(x,y,z\right)=\cos\left(\frac{n_{x}\pi}{L_{x}}x\right)\cos\left(\frac{n_{y}\pi}{L_{y}}y\right)\cos\left(\frac{n_{z}\pi}{L_{z}}z\right)$$</p>
Where we normalized the amplitude to $$ 1$$ both in value and unit. While, for the resonance frequencies:
<p style="text-align: center;">$$ f_{n_{x},n_{y},n_{z}}=\frac{c}{2}\sqrt{\left(\frac{n_{x}}{L_{x}}\right)^{2}+\left(\frac{n_{y}}{L_{y}}\right)^{2}+\left(\frac{n_{z}}{L_{z}}\right)^{2}}$$</p>
$$ n_x$$, $$ n_y$$ and $$ n_z$$ are the so called mode numbers. They are 3 integer numbers that identify a single mode. So, one mode could be 0 0 0 (a rather not interesting one) or 1 0 0, or 4 5 2 and so on and so forth. For each choice of the mode numbers we get one spatial part $$ S$$ and one resonance frequency $$ f$$. It isn't clear as we did not do all the math, but the resonance frequencies are related to the eigenvalues $$ -k^{2}$$ (they turn out to be infinite, but discrete, too, and parametrized by our 3 integers), and the reason why we have infinite, but discrete, modes parametrized by integer mode numbers is the boundary conditions.

$$ S$$ is the spatial part of the stationary wave, while $$ T$$, the temporal part of the stationary wave, can be a cosine wave vibrating at the related mode frequency:
<p style="text-align: center;">$$ T\left(t\right)=\cos\left(2\pi f_{n_{x},n_{y},n_{z}}t\right)$$</p>
Depending on $$ L_x$$, $$ L_y$$ and $$ L_z$$, it is possible that different modes (different choices of $$ n_x$$, $$ n_y$$, $$ n_z$$) could have the same frequency, although different $$ S$$. This is known as degeneracy.

<h1>The Julia Model</h1>
The equations above were inmplemented in few Julia functions available in <a href="https://github.com/CrocoDuckoDucks/RectangularRoom" target="_blank" rel="noopener">this GitHub repo</a>. I will add all rectangular room simulation material as we go ahead with more episodes, so this will not be a Julia package. Just clone the repo normally. Open a terminal and issue:

{% highlight bash %}
git clone https://github.com/CrocoDuckoDucks/RectangularRoom.git
cd RectangularRoom
{% endhighlight %}

And launch Julia from the same terminal session:

{% highlight bash %}
julia
{% endhighlight %}

It will be best if we add some packages to Julia right away. <a href="https://github.com/JuliaPlots/Plots.jl/" target="_blank" rel="noopener">Plots</a> will be essential to play with some plots and run the example code in the repo. To add press ] to have the REPL go into package mode, then issue:

{% highlight julia %}
v(1.0) pkg> add Plots
{% endhighlight %}

This could take some time. Once finished, press CTRL+C to exit package mode. As a note, notice that often Julia code needs a bit of time when it is the first time it is run, or when a package is loaded.

Now that we have Plots, let's compute a bunch of resonance frequencies. This is easier said than done, as there are few caveats. Let's say we want to compute the first 10 resonance frequencies of the room. Let's look at the equation for the resonance frequencies again:
<p style="text-align: center;">$$ f_{n_{x},n_{y},n_{z}}=\frac{c}{2}\sqrt{\left(\frac{n_{x}}{L_{x}}\right)^{2}+\left(\frac{n_{y}}{L_{y}}\right)^{2}+\left(\frac{n_{z}}{L_{z}}\right)^{2}}$$</p>
Each frequency depends on 3 integer numbers. However, it is a bit hard to figure out how to choose these numbers so that we get, say, 10 frequencies in ascending order of magnitude. Although it is possible to came up with good efficient algorithms, let's do something maybe a bit computationally expensive, but easy. In the Julia REPL, just include the modelling functions:

{% highlight julia %}
julia> include("RectangularRoom.jl")
{% endhighlight %}

next, let's define a 3D grid of integers:

{% highlight julia %}
julia> A, B, C = indexGrid(100, 100, 100)
{% endhighlight %}

In essence, we could think of $$ n_x$$, $$ n_y$$, $$ n_z$$ as 3D coordinates of points in a 3D space, and A B and C hold these coordinates. We chosen to include up to 101 values for $$ n_x$$, $$ n_y$$, $$ n_z$$ (the indices start from 0) so to make sure we have 1030301 of the infinite room modes (in fact, $$ n_x$$, $$ n_y$$, $$ n_z$$ can grow indefinitely). Among those 1030301 we should really be able to find the first lowest 10 (or perhaps even the first few hundreds or thousands, but then we will be missing many high frequency modes afterwards, since the higher the mode numbers the more combinations are possible to produce high frequency modes).

By the way, if you press ? in the REPL, the REPL will go into help mode. I included help for all the functions in the repo. For example, to see the help for indexGrid:

{% highlight julia %}
help>; indexGrid
search: indexGrid

  indexGrid(Nx, Ny, Nz)

  Returns a 3-D grid up to indeces Nx, Ny and Nz along the 3 cartesian directions. The
  grid is represented by three matrices of size Ny by Nx by Nz.

  Example
  ≡≡≡≡≡≡≡≡≡

  julia> Gx, Gy, Gz = indexGrid(3, 4, 5)
  ([1 2 3; 1 2 3; 1 2 3; 1 2 3]

  [1 2 3; 1 2 3; 1 2 3; 1 2 3]

  [1 2 3; 1 2 3; 1 2 3; 1 2 3]

  [1 2 3; 1 2 3; 1 2 3; 1 2 3]

  [1 2 3; 1 2 3; 1 2 3; 1 2 3], [1 1 1; 2 2 2; 3 3 3; 4 4 4]

  [1 1 1; 2 2 2; 3 3 3; 4 4 4]

  [1 1 1; 2 2 2; 3 3 3; 4 4 4]

  [1 1 1; 2 2 2; 3 3 3; 4 4 4]

  [1 1 1; 2 2 2; 3 3 3; 4 4 4], [1 1 1; 1 1 1; 1 1 1; 1 1 1]

  [2 2 2; 2 2 2; 2 2 2; 2 2 2]

  [3 3 3; 3 3 3; 3 3 3; 3 3 3]

  [4 4 4; 4 4 4; 4 4 4; 4 4 4]

  [5 5 5; 5 5 5; 5 5 5; 5 5 5])
{% endhighlight %}

Next, for each point in the 3D grid we defined, we calculate the resonance frequency given by the coordinates of that point (which would be the mode numbers). For the sake of argument, let's choose:

+ The values of $$L_x$$, $$L_y$$ and $$L_z$$ respectively equal to 5.0 meters, 4.0 meters and 3.0 meters.
+ Let's assume air in equilibrium at room temperature and ordinary pressure, making for a speed of sound of 343 meters per second.

Hence, let's do:

{% highlight julia %}
julia> F = modeFrequency.(A, B, C, 5.0, 4.0, 3.0)
{% endhighlight %}

And let's get the first 10 unique modal frequencies (so, we show one frequency only even if more than one mode have the same frequency):

{% highlight julia %}
julia> sort(unique(F))[1:10]
{% endhighlight %}

Which in my case returns:

{% highlight julia %}
10-element Array{Float64,1}:
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
{% endhighlight %}

0 is of course a special case of little interest. The other modes are interesting though, and we see that modes for a room that big start right at the lower end of the audible spectrum, so we quite don't like to have those in a listening room. Why? We will have to look at the pressure field at each of the modal frequencies to understand why, but for the time being let's visualize one more thing about the frequencies.

If you installed the plots package, let's use it:

{% highlight julia %}
julia> using Plots
{% endhighlight %}

now let's issue this:

{% highlight julia %}
julia> G = sort(F[:])

julia> histogram(G[G .< 4000], xlabel = "Frequency [Hz]", ylabel = "Number of Modes", legend = :none)
{% endhighlight %}

You should get a plot similar to this:

![Modes Histogram]({{ site.baseurl }}/res/pictures/2018-12-31-acoustic-modes-of-a-rectangular-room/figure_1.png  "Modes Histogram")

Let's commend this out, as it is telling us few interesting things.
<ol>
	<li>First of all, we just sorted all the modal frequencies we calculated before and put them in the array G. G contains each of the computed modal frequency, also the degenerate ones. So, many values actually appear in G more than one time.</li>
	<li>Then, we did an histogram of all the values of G that are smaller than 4 kHz.
<ol>
	<li>We are concentrating on the values smaller of 4 kHz as we don't really have all the possible high frequency modes, as we limited the mode numbers to 100. If we calculated with 1000, or 10000, then we would have many more high frequency modes, but our computer would most likely not really like it...</li>
	<li>Histogram means that we are making many little bins from 0 Hz to 4 kHz, and counting how many modes have frequency within the limits of each bin. The bins are ~50 Hz wide.</li>
</ol>
</li>
</ol>
We notice that the number of modal frequencies per bin gets very high very fast. The gradient seems to slow down a bit after 3500 Hz, but that is most likely due to the fact that our method is not finding all the modes between 3 kHz and 4 kHz.

This representation does not relate very well to perception. For that, instead of uniform bins, we should maybe use third octave or <a href="https://en.wikipedia.org/wiki/Equivalent_rectangular_bandwidth" target="_blank" rel="noopener">ERB</a> bins. However, it proves the point that there are far more high frequency modes than low frequency ones. This is why room modes are a problem mainly at low frequency. Why? This is another question that the shape of the modes can answer, so let's have a look at them!

You can run the example code in the repo to produce a plot and an animation in your /tmp folder, called rmode.gif:

{% highlight julia %}
julia> include("Example.jl")
{% endhighlight %}

They should look like those below:

![First Room Mode]({{ site.baseurl }}/res/pictures/2018-12-31-acoustic-modes-of-a-rectangular-room/figure_2.png  "First Room Mode")

![First Room Mode (Animated)]({{ site.baseurl }}/res/pictures/2018-12-31-acoustic-modes-of-a-rectangular-room/figure_3.gif  "First Room Mode (Animated)")

So, what these two pictures represent?

They represent the pressure field at the first modal frequency. As a reminder, the pressure field is a disturbance of pressure around static atmospheric pressure. The pressure disturbance here was scaled to be between -0.2 Pa to 0.2 Pa, so to be a 80 dB SPL pressure wave, which is what we would have if a sound source was producing a sine wave at a some appropriate power.

Also, we are not representing the pressure on the entire room, but only on a horizontal plane suspended at 1.80 meters from the floor. In essence, if we were a 1.80 meters tall person, the plots tell us the pressure field we would experience if we walked to any x-y spot on the floor.

The animation in particular shows us the wave mode action in slow motion (representing it at 34.3 Hz would just make it hard to understand). We can clearly see that the wave does not travel, but that positive and negative peaks keep on exchanging periodically as time goes on. In fact, the whole spacial pressure distribution is modulated by a cosine wave at 34.3 Hz, as we seen above.

We can see that there is a line right in the middle of the floor where we would not hear anything, as in there the disturbance is 0 Pa, no matter how loud we would crank up our sound source. This is the problem of modes in rooms. To be able to listen to music properly, we would like our room to not alter the sound coming from the speakers. However, we see now that depending on where we are in the room there are places where, if the frequency is just right, we do not hear any sound at all!

For different modes we will have different spots of deafness. These are called nodal surfaces (as they are 2D surfaces in a 3D space, as we will see at the bottom). Likewise, there are gonna be spots of maximal sound pressure, which are the blue and red areas above. Close to the east and west walls of the room we would hear our 34.3 at its loudest, for example.

If we had to plot the frequency response of the room for a choice of transmitter and receiver position (which is quite hard to do, let's save it for later) we would see massive resonances and anti-resonances, peaks and valleys. These would be much more significant where the modes are spread out, i.e. at low frequency, as the space between peaks would be far a part. However, at high frequency, many modes tend to have similar frequency so the frequency response gets much more even: where one mode would have a nodal surface, maybe one very close has not, so the peaks and valleys are way less pronounced.

Use of absorption is a common way to make the frequency response even, as it tends to make peak and valleys less sharp thanks to damping. In fact, we should not forget that we imposed rigid walls. If the walls are not rigid, maybe thanks to the use of various absorption panels, then the solution will change.

Finally, let's have a look at full blown 3D plots of a higher mode. To do so, we can install the following packages. At the Julia REPL, press ], then:

{% highlight julia %}
(v1.0) pkg> add Makie#master AbstractPlotting#master GLMakie#master
{% endhighlight %}

This might need some time. When finished, press CTRL+C. It migth be best to quit Julia with quit() and the launch it again. Let's include the room modelling functions:

{% highlight julia %}
julia> include("RectangularRoom.jl")
{% endhighlight %}

Then, let's define 3 Cartesian axes x y and z along the room edges, with a point every 0.1 meters. This can be done as in this example:

{% highlight julia %}
julia> x, y, z = roomAxes(0.1, 0.1, 0.1, 5.0, 4.0, 3.0)
{% endhighlight %}

Now, we can define an inline function which just calculates the mode shape in our room for a choice of mode numbers (4 3 2 in this example):

{% highlight julia %}
julia> f(x, y, z) = mode(x, y, z, 4, 3, 2, 5.0, 4.0, 3.0)
{% endhighlight %}

Finally, let's use our new Makie package and let's make a nice 3D contour plot:

{% highlight julia %}
julia> using Makie
julia> contour(x, y, z, f, levels = 10, alpha = 0.1)
{% endhighlight %}

![A Mode in 3D]({{ site.baseurl }}/res/pictures/2018-12-31-acoustic-modes-of-a-rectangular-room/figure_4.png  "A Mode in 3D")

A contour plot is a plot representing many 2D surfaces in a 3D volume. Each of these surface is drawn at a constant value of the pressure field, that is, on every little shell the pressure field is constant at a certain value.

The coolest thing is that you can use your mouse in the Makie window to zoom and rotate the figure. This plot clarifies that, in a room with highly reflective walls, modal pressure fields really take the shape of  "blobs" of sound separated by nodal surfaces, which in the plot appear as voids between the blobs. It is now also clear what the numbers $$ n_x$$, $$ n_y$$, $$ n_z$$ mean: they are the number of nodes encountered along each direction.

Feel free to experiment with different mode numbers. Also, you can edit Example.jl to simulate other rooms. Comments should clarify how to do it. Beware that the animation generation might not work for complex modes, so maybe comment it out if it gives some trouble.

<h1>Conclusion</h1>
We had a look at room modes and now that we understand them better, and have a way to calculate them, we can move on into using a solver to attempt replicating the results. Once we can replicate these results well, we will know that the solver is reliable. So we can change the input geometry for our solvers in order to tackle complex not-rectangular rooms.
