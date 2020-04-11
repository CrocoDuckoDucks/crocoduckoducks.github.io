---
layout: post
title:  "Intro to ParaView"
date:   2020-04-11 20:00:00 +0100
categories: science physics opensource
---

In the [previous episodes](https://crocoduckoducks.github.io/modelling-acoustics-with-open-source-software/) we often made use of the [ParaView](https://www.paraview.org/) postprocessor to visualise our solution field from the [ElmerFEM](http://www.elmerfem.org/blog/) solver. ParaView can do all sorts of cool visualisations and animations, as well as providing the way of doing quantitative analysis. It is by far the best option to visualise and postprocess results from ElmerFEM. It also allows to export data in various formats, such as CSV, that allow us to do any additional kind of postprocessing of verification, by either using [Julia](https://duckduckgo.com/?t=ffab&q=julia+lan&ia=web), [Python](https://www.python.org/) or any other language, or even spreadsheet software if you fancy that (for whatever reason...). ParaView functionalities are so many, and so advanced, that it is impossible to cover them all in a blog post. For that, you should instead refer to [this page](https://www.paraview.org/resources/). In this page I will merely collect a few useful tips and tricks that I found useful when working on my studies. As I find more tips and tricks as this series goes on, I will update this page.

# Basic ParaView Usage

Normally, ElmerFEM will provide us with _*.vtu_ files. These files can be imported into ParaView. Upon boot, ParaView will look something like this:

![ParaView Opening Screen]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_1.png  "ParaView Opening Screen")

In order to load  a _vtu_ file, we can act as follows:

* From the top menu, click _File_.
    * Select _Open_.
    * Navigate to your file, and select it.
    * Click on the _OK_ button.
* On the left, in the _Properties_ browser, you will see that the _Apply_ button is highlighted. If you click you will load all the data of the _vtu_ file into ParaView.
    
For example, below I loaded the ElmerFEM file for the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) episode.

![ParaView Loaded with Data]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_2.png  "ParaView Loaded with Data")

We can see that ParaView opened a default view on our data, looking at it from above, and that our domain does not seem very interestingly coloured. With your mouse, you can:

* Rotate your point of view by holding the left button and panning around with the mouse.
* Zoom in and out by using the mouse wheel or by holding the right mouse button and panning around.
* Pan your view by holding the mouse middle button and panning around.

We will not discuss the fundamentals of using ParaView.

## Data Sets

Your _vtu_ file probably contains more than one single dataset. For example, the file I loaded above contains two arrays of data, _pressure wave 1_ and _pressure wave 2_, together with _GeometryIds_ which normally we do not directly use. As you remember from the previous episodes, _pressure wave 1_ and _pressure wave 2_ are the real and imaginary part, respectively, of the complex steady state pressure field output by ElmerFEM Helmholtz solver.

A simple way to colour your domain by any of these datasets is by selecting it in the menu in the toolbar as follows:

![Colouring by pressure wave 1]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_3.png  "Colouring by pressure wave 1")

![Result]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_4.png  "Result")

You will probably remember from the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) episode that when there are many files called in a similar fashion to _case0001.vtu_, _case0002.vtu_, _case0003.vtu_ ... ParaView will group them all together when selecting which file to open, and you will be able to load them all at once. In this case, whenever ParaView recognises a series, it interprets it as time series. On the top of the window you can see some _play/stop/rewind_ style of controls. These allow you to cycle through all the case files. Not always the files are grouped by time. For example, in the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) episode they were just different steady state solutions at different frequencies, but ParaView will still cycle through them, treating them as frames of an animation.

On the left of the menu we used to select the field by which to colour our domain, you can see various buttons to set the scale and the colour map. It is possible to rescale according to the current frame data, a custom range, all data, or the visible data.

You should feel free to explore all controls.

## Filters

ParaView can manipulate the fields in many different ways to allow us to see things more clearly. This is achieved by filters. A selection of common filters is available as buttons right above the _Pipeline Browser_. In order to use them, a dataset has to be selected from the _Pipeline Browser_ to act as the input of the filter. Certain filters can take as input multiple datasets.

For example, let's try to apply the _Contour_ filter to our dataset:

* Select the dataset from the _Pipeline Browser_.
* Click the _Contour_ button right above the _Pipeline Browser_.

After you do so, you will see a view like the one below.

![Setting the Contour Filter]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_5.png  "Setting the Contour Filter")

Now we need to configure the filter, and this is done by editing the properties below the _Pipeline Browser_.

* In _Contour By_, select the field you want to contour by (in this case _pressure wave 1_.
* Under _Isosurfaces_, in _Value Range_, you can enter any values of configure a range by acting on the buttons on on the side.
* Once you are done, you can click on _Apply_ on the top of the _Properties_ section.

Below, my results for 10 equally spaced _Isosurfaces_ values.

![Results]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_6.png  "Results")

This procedure is typical of any filter:
* Select the input dataset.
* Select the filter, either from the toolbar or by the _Filters_ menu.
* Configure the filter.
* Apply.

Note that the filter will create an output dataset. In the _Pipeline Browser_, you can click on the eyeball to enable view many different datasets at the same time. In this case, you could make use of the _Opacity_ control, as shown below.

![Multiple Datasets in View]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_7.png  "Multiple Datasets in View")

You can also edit the background and ad _Data Axes Grid_, and many more things, as splitting the render view with the buttons on the top left of the render view to see datasets side by side, or by crating many more _Layouts_.

This guide cannot be exhaustive, so now that you get the basics of ParaView, let me list some tricks I commonly use. For more information, you should refer to the the [ParaView Resources page](https://www.paraview.org/resources/).

# Tricks

## Linked 3D Views

* Click the _Split Horizontal_ or _Split Vertical_ button on the top left of the _RenderView_.

![Linked 3D Views 1]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_8.png  "Linked 3D Views 1")

* Then, click the _Render View_ button. Right click anywhere on the view you just opened and select _Link Camera..._.

![Linked 3D Views 2]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_9.png  "Linked 3D Views 2")

* Now just click on the other 3D view (the one on the left in the current example). This will link the cameras of the two views.

Now you can click on any view. The _Pipeline Browser_ will update to the data being displayed in the selected view. So, you can have two linked views showing different things. For example, below we look at the raw _pressure wave 1_ and its _Isosurfaces_:

![Linked 3D Views 3]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_10.png  "Linked 3D Views 3")

## Calculator Filter 

Select your data and add the calculator filter. This filter allows to compute quantities from the fields in the dataset in a simple way. For example, this expression produces the dB SPL value from the field:

```text
20 * log10(sqrt(pressure wave 1^2 + pressure wave 2^2) / 20e-6)
```

![Calculator Filter]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_11.png  "Calculator Filter")

## Python Calculator Filter 

The Python calculator filter allows you to do more complex things with respect the calculator filter. Select your data and add the calculator filter. Then, put your expression in the _Properties_ section.

The inputs are of the calculator are served as arrays of classes with different attributes. These attributes are dictionaries at which we can access the field.

For example, this expression normalises the real part of the pressure field between 1 and -1:

```python
inputs[0].PointData['pressure wave 1'] / max(inputs[0].PointData['pressure wave 1'])
```

Our filter had one input, so we get into `inputs[0]`. Then, we are interested into the `PointData`, as that are is the kind of data our field is stored as. Finally, we select the field of interested, `pressure wave 1`. For more information, refer to [this page](https://www.paraview.org/Wiki/Python_calculator_and_programmable_filter).

![Python Calculator Filter]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_12.png  "Python Calculator Filter")

## Python Annotation

Creating annotations is pretty easy (for example, use the _Text_ filter), but what if we want a different annotation for every time frame? This could be a bit tricky, actually.

My favourite way is to create many CSV files with the stuff I want to use for the annotation. For example, look at [this code](https://github.com/CrocoDuckoDucks/Acoustic-Models/blob/65e6d86a240d6ef39f9cddea554237bdda647aad/2020-03-28-rigid-walled-room/validate.jl#L23). The code is from the [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) episode. The code will create 10 different CSV files, each one containing a single value of frequency, and suffixed with _1, _2, ... _10. Below the contents of the first file:

f_1.csv:
```text
34.3
```

Then, we can go ahead on ParaView to open the CSV files. Select _File_, than _Open_, and navigate to the repository `frequencies/`, where the script saved the CSV files (or anywhere else you put similar files). As ParaView did with our ElmerFEM vtu _vtu_ files, it will show them all collapsed as below. Select them and click _OK_.

![Importing CSV Files for Python Annotation 1]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_13.png  "Importing CSV Files for Python Annotation 1")

Once you click _OK_ the _Open Data With..._ window will appear. Select _CSV Reader_ and click _OK_. Then, in the _Properties_ section on the left un-tick _Have Headers_, as we do not have any. Now you can click _Apply_ at the top of the _Properties_ section.

![Importing CSV Files for Python Annotation 2]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_14.png  "Importing CSV Files for Python Annotation 2")

This will open a new _SpreadSheetView_ which you can dismiss by clicking the _x_ button on its top right. Notice, however, how the _Attribute_ listbox in the _SpreadSheetView_ says _Row Data_. According to what kind of CSV you might cycle through different _Attributes_ until you find your data, but generally every spreadsheet you make yourself will just have _Row Data_.

Now, from the _Pipeline Browser_, select the newly imported table. Then click on _Filters_, _Alphabetical_ and _Python Annotation_ (you can also use _Search_).

With the Python annotation we can compose a string by using attributes from the input fields. From _Array Association_, select the the correct attribute (in our case _Row Data_). Then, we can just index into it when we composes our _Expression_. For example:

```python
"{0:.2f} Hz".format(input.RowData[0][0])
```

from the `input` variable, we select the `RowData` attribute. Since we have only one input, we select the first with `[0]`. Finally, since we have only one element in the row, we select the first with another `[0]`. This will produce the result below.

![Python Annotation]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_15.png  "Python Annotation")

Now, the annotation will change for each frame according to the values in our original CSV files. This is how the annotations in the _gif_ files in [Rigid Walled Room](https://crocoduckoducks.github.io/science/physics/opensource/2020/03/28/rigid-walled-room.html) were made.

## Save Data to CSV

If after your calculation, you want to save the data to a CSV file(s), I suggest you proceed as follows:

* Select the dataset you want to export to CSV from the _Pipeline Browser_
* Select _File_, and then _Save Data_.
* Navigate to where you want to save files and input a file name. Then click _OK_.
* Configure now the _CSVWriter_, as its window will popup.

You might want to do different things. The screenshot below shows how to configure the writer to write all timesteps and all _Point Data_ arrays. Using _30_ as _Precision_ is overkill, but ensures all the digits are there. This will create a CSV where each row is for a mesh node point. The _x_, _y_, and _z_ coordinates of the point will be reported, and then all the values of the field at the node, in different columns. When ready, just click _OK_. This data can be used for validation or additional post-processing.

![CSVWriter Configuration]({{ site.baseurl }}/res/pictures/2020-04-11-intro-to-paraview/Figure_16.png  "CSVWriter Configuration")

## Exporting Pictures, Videos and State

My favourite way to export a picture is by using _File_ > _Save Screenshot_. All controls are self explanatory. If you have multiple views, and you want all of them exported, remember to tick _Save All Views_ in the _Save Screenshot Options_ window.

To export the animation, go to _File_ > _Save Animation_. My favourite format is _avi_. If you have multiple views, and you want all of them exported, remember to tick _Save All Views_ in the _Save Animation Options_ window. You can set the _Frame Rate_ to your liking.

To convert the _avi_ files to _gif_, I like to use `ffmpeg`:

```bash
ffmpeg -i avi_anim.avi -r rate gif_anim.gif
```
Where you will have to replace `avi_anim.avi` to your actual _avi_ file name, `rate` to the desired frame rate and `gif_anim.gif` to the desired output file name. Note that `ffmpeg` is also able to do many clever things, like change the final size, or do some optimisation.

Finally, you can save the ParaView state by going to _File_ > _Save State_. This will save a _pvsm_ file containing all of your work. If you want to load it into another ParaView session, go to _File_ > _Load State_. If the location of your dataset files is changed, you will be given option to find them again when loading.

# Conclusion

This clearly was not an exhaustive guide on how to use ParaView, but it contains most of the things I find myself doing most of the time. I will update it with new tricks here and there. Insofar as the series in concerned, this post can serve as a quick reference for common ParaView hacks and links to the documentation.

And now, back to modelling...
