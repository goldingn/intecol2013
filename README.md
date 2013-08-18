INTECOL 2013
===========

Here is some material for my talk and my workshop presentation at the [INTECOL 2013 conference](http://www.intecol2013.org/ "INTECOL 2013 website").

Talk
---
On Tuesday 20th August at 16.00 (Capital Suite 4) I'll be giving a talk on my new species distribution modelling approach GRaF. The abstract etc. can be found [here](http://eventmobi.com/INTECOL2013/#!/session/183611/ "GRaF talk abstract").
The slides for my talk can be downloaded from [figshare](http://figshare.com/articles/GRaF_Fast_flexible_Bayesian_species_distribution_modelling_using_Gaussian_random_fields/775351)


Workshop
---

On Tuesday 20th August at 13.30 (Capital Suite room 7) I'll be showcasing the [GRaF R package](http://cran.r-project.org/web/packages/GRaF/index.html "GRaF package") as part of a workshop on [New Computational Methods in R](http://www.intecol2013.org/70_NewComputationalMethodsinRworkshop.html "Workshop").

The workshop is being put together by [Matthew Smith] (http://research.microsoft.com/en-us/people/mattsmi/) from the BES's [computational ecology special interest group](http://www.britishecologicalsociety.org/getting-involved/special-interest-groups/computational-ecology/ "computational ecology SIG") which you should join if any of this stuff interests you!

The workshop starts at 12.15, so come along then to see presentations on other new R packages, including Mark Stevenson's work on the geographic profiling.

### Tutorial
During the workshop, I'll be stepping how to fit models in GRaF and how the approach differs from other SDMs approaches. I'll basically be going through this [tutorial](https://rawgithub.com/goldingn/intecol2013/master/tutorial/graf_workshop.html). I'll try and update this with any extra stuff that comes up in the meeting.

If you want to follow along, you can download the r code on its own [here](https://github.com/goldingn/intecol2013/blob/master/tutorial/graf_workshop.R)

### GRF links
Here are  that I found useful for getting to grips with Gaussian random fields (AKA Gaussian processes).

[Rasmussen & Williams' Gaussian Processes book](http://www.gaussianprocess.org/gpml) is free to download and gives a comprehensive overview of GPs. They come from a machine learning perspective and the maths is pretty dense, so it isn't very easy reading for non-technical.

[This blog post](http://www.jameskeirstead.ca/blog/gaussian-process-regression-with-r/) shows how to code up your own Gaussian process regressions in R and is a bit more accessible.


### Building R packages
Part of the aim of the workshop is to discuss and share the process of creating R packages.
This is becoming increasingly easy, especially with utilities such as Hadley Wickham's [devtools package](http://cran.r-project.org/web/packages/devtools/index.html) and [functions available in RStudio](http://www.rstudio.com/ide/docs/packages/overview).

[Friedrisch Leisch's tutorial](http://cran.r-project.org/doc/contrib/Leisch-CreatingPackages.pdf) is comprehensive and well worth a read, as is [this walk through](http://biostat.mc.vanderbilt.edu/wiki/Main/HowToCreateAnRPackage).


Thesis
---
You can download [my DPhil thesis over at figshare](http://figshare.com/articles/PhD_thesis_Mapping_and_understanding_the_distributions_of_potential_vector_mosquitoes_in_the_UK_New_methods_and_applications/767289 "Nick's DPhil thesis").

Chapter 5 describes the GRaF species distribution modelling approach which is the subject of the workshop and talk.
