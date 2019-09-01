---
layout: post
title:  "What is Acoustic Modelling"
date:   2018-11-30 20:42:56 +0100
categories: science physics opensource
---

Welcome to the first actual episode of the series about acoustic modelling with open source software. We will first try to understand what modelling acoustics means. In reality it doesn't mean just one thing, as many phenomena of acoustic wave production and propagation can be modeled and simulated in various different ways, with higher or lower degree of accuracy. However, the core of the modelling problem resides in partial differential equations. This post will be a very, <span style="text-decoration: underline;">very</span>, brief, intuitive and not rigorous introduction to the topic, mainly to give context to those that are not accustomed to the concept. If you are experienced about physics and acoustics, you can completely skip this episode.

{% include mathjax.html %}

<h1>Partial Differential Equations</h1>
An equation is a statement of the relationship between various quantities of interest. For example, look at this equation:
<p style="text-align: center;">$$ F=m\cdot a$$</p>
It is the famous Second Law of Motion by Isaac Newton, which tells us that the total force acting on a body is equal to its acceleration multiplied by its mass. Acceleration and force are actually 3D vectors, but let's keep the problem 1D, for simplicity, so that $$ F$$ and $$ a$$ are real numbers, just like the mass $$ m$$. The cool thing about equations is that they allow us to solve for unknown things. For example, let's assume we know the mass and force acting on a body, somehow. Then, it is easy to to get its acceleration:
<p style="text-align: center;">$$ a=\frac{F}{m}$$</p>
We just have to manipulate the equation, by using the rules of mathematics, so to get the result we are searching for expressed in terms of what we know.

So, we can see equations in two ways: a statement of the relationship between properties but also a way to <em>solve</em> for unknown properties, given those we know. This should start ringing some bells, as in the first introductory episode we seen that many programs for physics simulation are called <em>solvers</em>.

Now, it turns out that physical systems can generally be described by partial differential equations. What that means? Well, let's first talk about differential equations. Let's look at the Second Law of Motion above. A body will have a position $$ x$$ in our hypothetical 1D universe, right? So, what would be its velocity? By definition, the velocity is the rate by which a body change its position in time. So, first of all the position of our body changes in time -<em>the position is a function of time</em>-:
<p style="text-align: center;">$$ x=x\left(t\right)$$</p>
where $$ t$$ denotes the variable time. Then, how we understand the rate by which the position varies in time? Well, let's imagine that this is the case:
<p style="text-align: center;">$$ x\left(t\right)=c\cdot t^{2}$$</p>
Where $$ c$$ is a constant of value 1 meters per second squared. Now, we will plot the first 5 seconds of this function:

<img class="alignnone size-full wp-image-3376" src="{{ site.baseurl }}/res/pictures/2018-11-30-what-is-acoustic-modelling/figure_1.png" alt="Figure_1" width="600" height="400" />

So, this means that, when the body started moving at the time of 0 s (we count the time from the start of motion) the position, with respect that at 0 s, changed during time as represented above. Remember that we are doing 1D physics, so the body traveled along a line until, after 5 seconds, it was a full 25 meters away along that line.

However, we see also few more things. At the very beginning it took more than 2 seconds to travel 5 meters. But towards the end the body traveled the same distance in half a second. So, the amount of distance made in a unit of time, the velocity, must have varied. Indeed, velocity is a function of time, too. Now, look at the plot below.

<img class="alignnone size-full wp-image-3378" src="{{ site.baseurl }}/res/pictures/2018-11-30-what-is-acoustic-modelling/figure_2.png" alt="Figure_2" width="600" height="400" />

We just added a line, the orange line, which is tangent to the blue line at the instant 4 s. Tangent means that it touches only at that point. Well, this line happens to be, close to that point, a nice approximation of the blue line. Indeed, at the point itself, they have the same value, and the blue and orange line can be barely told apart in a small region nearby. If the orange value was the position function, the rate by which position varies in time, the velocity, would be easy: it would be its slope, constant along the line. In fact, the slope tells us the amount of space traveled per unit time. But our position function is the blue line. Now, the orange line, being tangent, represents well the local trend of the blue line. Hence, we conclude that the velocity of our body, at the instant 4 s only, is given by the slope of the orange line.

We can repeat this for every time instant: the velocity at one instant is the slope of the tangent line to the position function at that instant. It turns out that it is possible to build another function, the so called <em>derivative function</em>, that does just that: at each instant it returns the slope of the original function at that instant. Hence:
<p style="text-align: center;">$$ v\left(t\right)=\frac{dx\left(t\right)}{dt}$$</p>
where with $$ \frac{dx\left(t\right)}{dt}$$ we mean the function that takes as input $$ t$$ and returns as output the slope of $$ x\left(t\right)$$ at the instant $$ t$$: the derivative function we talked about. Without going into details, there are standard ways to compute the derivative of a known function.

And now, what is the acceleration? Well, it is the rate of change of velocity in time. Hence:
<p style="text-align: center;">$$ a\left(t\right)=\frac{dv\left(t\right)}{dt}$$</p>
Or:
<p style="text-align: center;">$$ a\left(t\right)=\frac{d^{2}x\left(t\right)}{dt^{2}}$$</p>
Where we mean that to get the acceleration we differentiate the position two times. So, we can rewrite the Second Law of Motion as follows:
<p style="text-align: center;">$$ F=m\frac{d^{2}x\left(t\right)}{dt^{2}}$$</p>
Now, if we knew $$ x\left(t\right)$$ we would know $$ F$$ easily, but what if $$ x\left(t\right)$$ is the unknown? Now the equation turned in a so called<em> differential equation</em>, where the unknown is not a simple number, but a full blown function, on which operations of differentiation (taking derivatives) were carried out. Of course, these equations are much harder to solve, as we are not dealing with simple numbers (as if equations involving only number were easy at all...). Also, consider that the force might be varying with time, too. And the mass, even, if we think about a vehicle burning part of is internal mass for propulsion (such as a car, or a rocket).

To solve a differential equation of this kind, one has to know the initial conditions. In this case, it means knowing the position and velocity at 0 s. This because infinite functions can satisfy the equation, so to get our one we need to be selective. For example, if we were interested in what would happen if our body had to start from rest, we would impose:
<p style="text-align: center;">$$ x\left(0\right)=0$$</p>
<p style="text-align: center;">$$ \frac{dx\left(0\right)}{dt}=0$$</p>
Now, let's make a huge leap and think about acoustic. What we want to calculate is the so called pressure field. What that means?

Well, sound is waves through air (or any other gas or fluid). Air in equilibrium at a certain temperature, as every gas, has a certain pressure. A pressure wave is a variation on this static equilibrium pressure. If we parametrize the 3D space with three coordinates $$ x$$, $$ y$$ and $$ z$$, and time with $$ t$$, then our pressure wave, or disturbance, is written as:
<p style="text-align: center;">$$ p=p\left(x,y,z,t\right)$$</p>
That because, of course, the pressure disturbance exist all over the gas, hence at all coordinates, and evolves with time. Now, there are many different differential equations that can supply a solution for a pressure field. Many because they attempt to model the phenomenon with various degrees of accuracy.  These equations are often derived from first principles, that is, basic physical laws. You can rest assured that our initial example, the Second Law of Motion, is lurking within all of them, in some form or another... We will look at the simplest one, the D'Alembert equation:
<p style="text-align: center;">$$ \frac{\partial^{2}p}{\partial x^{2}}+\frac{\partial^{2}p}{\partial y^{2}}+\frac{\partial^{2}p}{\partial z^{2}}=\frac{1}{c^{2}}\frac{\partial^{2}p}{\partial t^{2}}$$</p>
This equation relates the partial derivatives of the pressure disturbance to one another. The partial derivatives are just like the derivatives we seen so far, but operated with respect just one of the many variables on which the pressure depend. And we use $$ \partial$$ instead of $$ d$$ to denote them. $$ c$$ is the speed of sound in the gas, which turns out to depend on the thermodynamic properties of the gas as well as its thermodynamic conditions. For air at room temperature it is 343 meters per second.

So, this equation is meant to model pressure waves propagating in a fluid in thermodynamic equilibrium. Not only that, but we are assuming that the pressure wave is not able to change the thermodynamic equilibrium, maybe because very small in amplitude (don't worry, there are other more complex equations to account for that).

To solve this equation, we will also need boundary conditions. That is, we have to specify what happens to the perturbation right at the edges of the volume that contains the air, other than specify the shape of that volume too. In fact, fluids are contained within enclosures. For example, our room loosely constrains some air. The walls of our rooms, and any other item, affect how pressure waves propagate in their proximity, as well as dictating how they get reflected (bounced off) and diffused (scattered all over). This is why sound treatment in rooms is done by hanging panels of various kinds into the walls: what happens at the borders dictates the whole of the properties of the pressure waves, even within the volume.

So, this is what simulating acoustics means in its broadest sense. It means studying a body of air (or any fluid) contained by walls of various shapes and properties. Maybe there could be field sources too, such as a speakers or ideal point sources (for simplicity). Maybe the sources are vibrating bodies, that we will need to simulate with other partial differential equations. In general, we will select (or create, if we are <em>that</em> badass) the best partial differential equation to model our system of interest. Best is defined in terms of our objectives: we will topically select the simplest equation among those that can predict the phenomenon we are studying up to the accuracy we require. And since that equation is normally too hard to solve, we will use a numerical solver to get an approximate solution.

So, the takeaway for now is:
<ul>
	<li>Physical phenomena are modeled by partial differential equations, and acoustics is no exception.</li>
	<li>There are many partial differential equations, more or less complex, to model phenomena to higher or lower degrees of accuracy.</li>
	<li>Partial differential equations define how, in our model, the rates of change of our function of interest (with respect the various variables they are defined with) are related to one another. This is important, as in everyday life we often forget that the rate of change of a quantity is more important than the quantity itself, as that determines its evolution.</li>
	<li>Finding our function of interest, the <em>field</em>, means providing initial and/or boundary conditions to the problem, as the whole field is defined by what happens at the beginning and/or at the borders of its volume of definition.</li>
	<li>These equations are very complex to solve, even if their solution is known to exist, so we often use numerical solvers.</li>
</ul>
In the next episodes we will start looking at how to solve some equations for simple benchmark systems.
<h1>Further Reading</h1>
If you are interested in this field, and have a good grip on mathematical analysis and algebra, as well as mathematical physics, I would recommend the following books:
<ul>
	<li><a href="https://www.wiley.com/en-gb/Fundamentals+of+Acoustics,+4th+Edition-p-9780471847892" target="_blank" rel="noopener">Fundamentals of Acoustics</a> is perhaps the one with the most gentle introduction (which I know of, there are for sure more basic texts around).</li>
	<li><a href="https://www.crcpress.com/Room-Acoustics-Sixth-Edition/Kuttruff/p/book/9781482260434" target="_blank" rel="noopener">Room Acoustics</a> contains some very good knowledge mainly about indoor acoustics.</li>
	<li><a href="https://press.princeton.edu/titles/4523.html" target="_blank" rel="noopener">Theoretical Acoustics</a> is a book not for the faint of heart. It covers most complex phenomena in acoustics, but it requires a very significant body of previous knowledge to make the most of it.</li>
</ul>
