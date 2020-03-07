# Visualizing Trends and Relationships





We now have a language for creating graphics. Next we must build the intuition of which plots to build and when. We will cover the most basic of visualizations starting with univariate followed by bivariate plots. We will then discuss ways to extend visualizations beyond two variables and some simple principles of design.

In most cases a data analysis will start with a visualization. And that visualization will be dictated by the characteristics of the data available to us. In intro to statistics you probably learned about the four types of data which are: nominal and ordinal, together referred to as _categorical_;  interval and ratio, together referred to as _numerical_ We're going to contextualize these in R terms where _categorical_ is `character` and _numerical_ is `numeric`.

Categorical and numeric have different are treated differently and thus lead to different kinds of visualizations. When we refer to categorical or character, we are often thinking of groups or a label. In the case where we don't have a quantifiable numeric value, we often count those variables.

  
## Univariate visualizations

There is always a strong urge to begin creating epic graphs with facets, shapes, colors, and hell, maybe even three dimensions. But we must resist that urge! We _must_ understand the distributions of our data before we start visualizing them and drawing conlcusions. Who knows, we may find anomalies or errors in the data cleaning process or even collection process. We should always begin by studying the indivual variable characteristics with univariate visualizations. 


> Note that univariate visualizations are for numeric variables

There a couple of things that we are looking for in a univariate visualization. In the broadest sense, we're looking at characterizations of central tendency, and spread. When we create these visualizations we're trying to answer the following questions:


- Where is the middle? 
- Is there more than one middle? 
- How close together are our points? 
- Are there any points very far from the middle?
- Is the distribution flat? Is it steep?

In exploring these questions we will rely on three types of plots:

1. Histogram
2. Density 
3. Box plot

Each type of plot above serves a different purpose.  

### Histogram

We have already created a number of histograms already. But it is always good to revisit the subject. Histograms puts our data into `n` buckets (or bins, or groups, or whatever your stats professor called them), counts the number of values that fall into each bucket, and use that frequency count as the height of each bar. 

The true benefit of the histogram is that it is the easiest chart to consume by the layperson. But the downside is that merely by changing the number of bins, the distribution can be rather distorted and it is on you, the researcher and analyst, to ensure that there is no miscommunication of data.

When we wish to create a histogram, we use the `geom_histogram(bins = n)` geom layer. Since it is a univariate visualization, we only specify one aesthetic mapping—in this case it is `x`. 

Let's look at the distribution of the `med_yr_moved_inraw` column for an example.


* Create a histogram of `med_yr_moved_inraw` with 10 bins. 


```r
ggplot(acs, aes(med_yr_moved_inraw)) +
  geom_histogram(bins = 10)
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-2-1.png" width="672" />

This histogram is rather informative! We can see that shortly after 2000, there was a steep increase in people moving in. Right after 2005 we can see that number tapering off—presumably due to the housing crises that begat the Great Recession.

Now, if we do not specify the number of bins, we get a very different histogram. 


```r
ggplot(acs, aes(med_yr_moved_inraw)) +
  geom_histogram()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-3-1.png" width="672" />

The above histogram shows gaps in between buckets of the histogram. On a first glance, we would assume that there may be missing data or some phenemonon in the data recording process that led to some sort of missingness. But that isn't the case! If we count the number of observations per year manually, the story becomes apparent.

> Note: I am using the base R function `table()` to produce counts. This produces a class `table` object which is less friendly to work with. Using `table()` rather than count serves two purposes: 1) you get to learn another function and 2) the printing method is more friendly for a bookdown document. 


```r
(moved_counts <- table(acs$med_yr_moved_inraw))
## 
## 1991 1995 1996 1997 1998 1999 2000 2001 2002 2003 2004 2005 2006 2007 2008 
##    1    2    5    7   14   31   56   77  108  121  141  109  113   84   73 
## 2009 2010 2011 2012 2013 
##   67  125  140   29    8

glue::glue("There are {length(moved_counts)} unique values")
## There are 20 unique values
```

> The glue function provides a way to create strings by combining R expressions and plain text. More in the appendix.

This above tells us something really important and explains why our histogram is all wonky. Our histogram looks the way it does because we have specified more bins than there are unique values! The moral of the story is that when creating a histogram, be thoughtful and considerate of the number of bins your are using—it changes the whole story.


### Density Function plot


Histograms are a fairly straight-forward chart that provides illustrates the distribution of a sample space. The histogram does not provide a fine grain picture of what the underlying distribution looks like. When we are concerned with understanding the underlying shape of a distribution we should use a **kernel density plot** (aka density plot). 

The density plot represents a variable over a continuous space and by doing so creates a much better visual representation of the underlying distribution shape with all of its curves. 

Like a histogram, we only provide a single variable to the aesthetic mappings. The geom layer for a density distribution is `geom_density()`.

* Create a density plot of `med_yr_moved_inraw`


```r
ggplot(acs, aes(med_yr_moved_inraw)) +
  geom_density()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-5-1.png" width="672" />


Now compare the histogram to the density plot. 

* Which do you feel does a better job illustrating the shape of the distribution?
* Which do you think is more interpretable?

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-6-1.png" width="672" />




### Boxplot

The boxplot is the third univariate visualization we will cover. Unlike histograms and density plot, the box plot's power comes from being able to effectively illustrate outliers and the general spread of a variable. 

There are five elements that make the boxplot:

1. Minimum
2. First quartile (25th percentile)
3. Median (50th percentile)
3. Third quartile (75th percentile)
3. Maximum 


When creating a boxplot, the definition of minimum and maximum change a little bit. We are defining the minimums and maximums _without_ the outliers. And in the context of a boxplot the outliers are determined by the **IQR** (inner quartile range). The IQR is different between the third and first quartile. We then take the IQR and _add_ it to the third quartile to find the upper bound and then subtract the IQR from the first quartile to find the lower bound.

$IQR = Q3 - Q1$

$Minimum = Q1 - IQR$

$Maximum = Q3 + IQR$


> Note that this is a naive approach to defining an outlier. This is not a hard and fast rule of what is considered an outlier. There are many considerations that should go into defining an outlier other than arbitrary statistical heuristics. Be sure to have a deep think before calling anything an outlier. 

Any points that fall outside of that the minimum and maximum are plotted individually to give you an idea of any _potential_ outliers. 

To create a boxplot we use the `geom_boxplot()` function.

* Create a boxplot of `med_house_income`



```r
ggplot(acs, aes(med_house_income)) +
  geom_boxplot() 
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-7-1.png" width="672" />

From this boxplot, what can we tell about median household income in Massachusetts? 

## Bivariate visualizations

We are ready to introduce a second variable into the analysis. With bivariate relationships (two-variables) we are often looking to answer, in general, if one variable changes with another. But the way we approach these relationships is dependent upon the type of variables we have to work with. We can can either be looking at the bivariate relationship of 

* 2 numeric variables,
* 1 numeric variable and 1 categorical,
* or 2 categorical variables. 

### Two Numeric Variables

#### Scatter plot

When confronted with two numeric variables, your first stop should be the scatter plot. A scatter plot positions takes two continuous variables and plots each point at their (x, y) coordinate. This type of plot illustrates how the two variables change with each other—if at all. It is exceptionally useful for pinpointing linearity, clusters, points that may be disproportionately distorting a relationship, etc. 

Scatter plots are useful for asking questions such as "when x increases how does y change?" Because of this natural formulation of statistical questions—i.e. we are always interested in how the x affects the y—we plot the variable of interest vertically along the y axis and the independent variable along the horizontal x axis. 

Take for example the question "how does the proportion of individuals under the age of eighteen increase with the number of family households?"

Using a scatter plot, we can begin to answer this question! Notice how in the formulation of our question we are asking how does y change with x. In this formulation we should plot the `fam_house_per` against the `age_u18` column.

> Note: when plotting _against_ something. We are plotting x _against_ y.

Recall that to plot a scatter plot we use the `geom_point()` layer with an x and y aesthetic mapped.


```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_point()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-8-1.png" width="672" />

The above scatter plot is useful, but there is one downside we should be aware of and that is the number of points that we are plotting. Since there are over 1,400 points—as is often the case with big data—they will likely stack on top of each other hiding other points and leading to a dark uninterpretable mass! We want to be able to decipher the concentration of points as well as the shape.

> When there are too many points to be interpretable this is called overplotting

To improve the visualization we have a few options. We can make each point more transparent so as they stack on top of eachother they become darker. Or, we can make the points very small so that as they cluster they become a bigger and darker mass.

To implement these stylistic enhancements we need to set some aesthetic arguments inside of the geom layer. In order to change the transparency of the layer we will change the `alpha` argument. `alpha` takes a value from 0 to 1 where 0 is entirely transparent and 1 is completely opaque. Try a few values and see what floats your boat!


```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_point(alpha = 1/4)
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-9-1.png" width="672" />

Alternatively, we can change the size (or even a combination of both) of our points. To do this, change the `size` argument inside of `geom_point()`. There is not a finite range of values that you can specify so experimentation is encouraged! 


```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_point(size = 1/3)
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-10-1.png" width="672" />

> Remember when deciding the `alpha` and `size` parameters your are implementing stylistic changes and as such there are no _correct_ solution. Only marginally better solutions. 


#### Hexagonal heatmap 

Scatter plots do not scale very well with hundreds or thousand of points. When the scatter plot becomes a gross mass of points, we need to find a better way to display those data. One solution to this is to create a heatmap of our points. You can think of a heatmap as the two-dimension equivalent to the histogram. 

The heatmap "divides the plane into rectangles [of equal size], counts the number of cases in each rectangle", and then that count is then used to color the rectangle[^bin2d]. An alternative to the rectangular heatmap is the hexagonal heatmap. The hexagonal heatmap has a few minor visual benefits over the rectangular heatmap. But choosing which one is better suited to the task it up to you! 

The geoms to create these heatmaps are 

* `geom_bin2d()` for creating the rectangular heatmap and
* `geom_hex()` for a hexagonal heatmap.


* Convert the above scatter plot into a heat map using both above geoms. 


```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_bin2d()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-11-1.png" width="672" />



```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_hex()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-12-1.png" width="672" />


Just like a histogram we can determine the number of bins that are used for aggragating the data. By adjusting the `bins` argument to `geom_hex()` or `geom_bin2d()` we can alter the size of each hexagon or rectangle. Again, the decision of how many bins to include is a trade-off between interpretability and accurate representation of the underlying data. 

* Set the number of `bins` to 20 


```r
ggplot(acs, aes(fam_house_per, age_u18)) +
  geom_hex(bins = 20)
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-13-1.png" width="672" />


### One numeric and one categorical

#### ridgelines 

### boxplot (1 continuous 1 categorical)

  - we can use boxplots to compare groups
  - for this, we set the categorical variable to the x aesthetic
  

```r
ggplot(acs, aes(county, age_u18)) +
  geom_boxplot() +
  coord_flip()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-14-1.png" width="672" />


### barplot (1 categorical 1 continuous / discrete)

- the `geom_bar()` will count the number observations of the specified categorical variables


```r
ggplot(acs, aes(county)) +
  geom_bar() +
  coord_flip()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-15-1.png" width="672" />

### lollipop chart

- barplot's more fun cousin, the lollipop chart
- the package `ggalt` makes a `geom` for us so we don't have to create it manually
- we do, however, have to count the observations ourself. 
  - we use the function `count()` to do this. 
  - the arguments are `x`, or the tibble, and `...` these are the columns we want to count
    - important note: in the tidyverse, the first argument is almost always the tibble we are working with
    - this will become very useful at a later point
- remember, to install packages, navigate to the console and use the function `install.packages("pkg-name")`.


```r
library(ggalt)
## Registered S3 methods overwritten by 'ggalt':
##   method                  from   
##   grid.draw.absoluteGrob  ggplot2
##   grobHeight.absoluteGrob ggplot2
##   grobWidth.absoluteGrob  ggplot2
##   grobX.absoluteGrob      ggplot2
##   grobY.absoluteGrob      ggplot2

acs_counties <- count(acs, county)

ggplot(acs_counties, aes(county, n)) +
  geom_lollipop() +
  coord_flip()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-16-1.png" width="672" />


- ridgelines (1 continuous 1 categorical)
- line chart (1 continuous 1 time), this is a unique case

## Expanding bivariate visualizations to trivariate & other tri-variate

- we can visualize other vcariables by setting further aesthetics.
  - can set the color or fill, size, and shape
- we alreay did this previously when we set the color, let's do that here. 
  - lets see how commuting by walking changes with the family house and under 18 pop
    - set the color argument of the `aes()` function as `color = by_walk`
      - it's important you do this within the aesthetics function 
    

```r
ggplot(acs, aes(fam_house_per, age_u18, color = by_auto)) +
  geom_point()
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-17-1.png" width="672" />

- we can add size to this as well by setting the `size` aesthetic
  - lets see if the more female headed house holds there are affects commuting by car as minors increases


```r
ggplot(acs, aes(fam_house_per, age_u18, color = by_auto, size = fem_head_per)) +
  geom_point(alpha = .2)
```

<img src="04b-visualization-strategies_files/figure-html/unnamed-chunk-18-1.png" width="672" />

- from this chart we can see quite a few things:
  - as `fam_house_per` increases so does the under 18 pop,
  - as both `age_u18` and `fam_house_per` increase so does the rate of communiting by car
  - as both `age_u18` and `fam_house_per` so does female headed houses, but to a lesser degree
  - this gives us a good idea of some relationships that we can test with our data at a later point


```r
minors_lm <- lm(age_u18 ~ fam_house_per + by_auto + fem_head_per, data = acs)

huxtable::huxreg(minors_lm)
## Registered S3 methods overwritten by 'broom.mixed':
##   method         from 
##   augment.lme    broom
##   augment.merMod broom
##   glance.lme     broom
##   glance.merMod  broom
##   glance.stanreg broom
##   tidy.brmsfit   broom
##   tidy.gamlss    broom
##   tidy.lme       broom
##   tidy.merMod    broom
##   tidy.rjags     broom
##   tidy.stanfit   broom
##   tidy.stanreg   broom
```

<!--html_preserve--><table class="huxtable" style="border-collapse: collapse; margin-bottom: 2em; margin-top: 2em; width: 50%; margin-left: auto; margin-right: auto;  ">
<col><col><tr>
<td style="vertical-align: top; text-align: center; white-space: nowrap; border-style: solid solid solid solid; border-width: 0.8pt 0pt 0pt 0pt; padding: 4pt 4pt 4pt 4pt;"></td>
<td style="vertical-align: top; text-align: center; white-space: nowrap; border-style: solid solid solid solid; border-width: 0.8pt 0pt 0.4pt 0pt; padding: 4pt 4pt 4pt 4pt;">(1)</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">(Intercept)</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">0.000&nbsp;&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;"></td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">(0.006)&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">fam_house_per</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">0.245 ***</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;"></td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">(0.009)&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">by_auto</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">0.016 *&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;"></td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">(0.007)&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">fem_head_per</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">0.257 ***</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;"></td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; border-style: solid solid solid solid; border-width: 0pt 0pt 0.4pt 0pt; padding: 4pt 4pt 4pt 4pt;">(0.012)&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">N</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">1311&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">R2</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">0.564&nbsp;&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">logLik</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; padding: 4pt 4pt 4pt 4pt;">2444.648&nbsp;&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td style="vertical-align: top; text-align: left; white-space: nowrap; border-style: solid solid solid solid; border-width: 0pt 0pt 0.8pt 0pt; padding: 4pt 4pt 4pt 4pt;">AIC</td>
<td style="vertical-align: top; text-align: right; white-space: nowrap; border-style: solid solid solid solid; border-width: 0pt 0pt 0.8pt 0pt; padding: 4pt 4pt 4pt 4pt;">-4879.296&nbsp;&nbsp;&nbsp;&nbsp;</td>
</tr>
<tr>
<td colspan="2" style="vertical-align: top; text-align: left; white-space: normal; padding: 4pt 4pt 4pt 4pt;"> *** p &lt; 0.001;  ** p &lt; 0.01;  * p &lt; 0.05.</td>
</tr>
</table>
<!--/html_preserve-->



Trivariate:

- grouped / stacked bar charts
- heatmaps 


Color Ramps:

- diverging when there is a true middle
- dark is low bright is high


- most data analyses start with a visualization. 
- the data we have will dictate the type of visualizations we create
- there are many many different ways in which data can be represented
- generally these can be bucketed into a few major categories
  - numeric 
    - integer
    - double
  - character 
    - think groups, factors, nominal, anything that doesn't have a numeric value that makes sense to count, aggregate, etc.
  - time / order 
  
  
twitter thread on this:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr"><a href="https://twitter.com/hashtag/rstats?src=hash&amp;ref_src=twsrc%5Etfw">#rstats</a> / general <a href="https://twitter.com/hashtag/datascience?src=hash&amp;ref_src=twsrc%5Etfw">#datascience</a> tweeps:<br><br>what types of visualizations do you think new folks should learn and embrace? i.e. hists, scatters, etc.</p>&mdash; Josiah 👨🏻
💻 (@JosiahParry) <a href="https://twitter.com/JosiahParry/status/1216059573548638208?ref_src=twsrc%5Etfw">January 11, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>




-----------

**Notes**

- we specify data and mappings, nothing is displayed at that time
- from the defaults we can see that the scales and coordinates are intuitively handled
- the default coordinate system is the cartesian coordinate system which has two axes x (horizontal), and y (vertical)
- in order to see the data, we needed to add geometry to our base layer
  - when we add Geoms to the graphic the data and aesthetic mappings are inherited from the defaults
- we do not necessarily need to default data if we do not feel the need to do so. 
- we can specify the data and mappings inside of the geom layer itself
  - this will be handy when we work with geospatial data
- stat refers to statistical transformations. 
  - often you will not need to apply transformations to your data as their _identity_ (assigned values) are sufficient.

with bi-variate relationships we're looking to answer, in general, if one variable affects the other. we usually will be comparing two numeric variables or one numeric and one categorical variable
in the former situtation we're looking to see if there is a related trend, i.e. when one goes up does the other go down or vice versa
in the latter scenario, we want to know if the distribution of the data changes for different groups


[^wickham]: https://vita.had.co.nz/papers/layered-grammar.pdf
[^bin2d]: https://ggplot2.tidyverse.org/reference/geom_bin2d.html
https://cfss.uchicago.edu/notes/grammar-of-graphics/

recommended reading: [A Layered Grammar of Graphics](https://vita.had.co.nz/papers/layered-grammar.pdf)

- in R we will use a package called ggplot2 to do the visualizaiton of our data 
- the `gg` in ggplot stands for "grammar of graphics".
- once we can internalize the grammar, creating plots becomes rather easy
- we specify our aesthetics
- we **add** layers (hence the plus sign). these take values from the specified aesthetics
- can add multiple layers
- add aesthetics other than x and y. helps us visualize more dimensions of the data. we can use shape, color, and size



### revisiting the cartesian plane

- x and y coordinates
- generally two numeric values on the x and y. think of the standards scatterplot (below)
- we also can place groups on one axis
  - i.e. barchart (below)
- the y is usually the variable of interest
  - as we move along the x axis (to the right) we can see how the y changes in response