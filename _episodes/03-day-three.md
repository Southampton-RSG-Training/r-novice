---
title: 'Day 3: Visualising Data'
slug: r-novice-visualising-data
teaching: 150
exorcises: 30
layout: episode
objectives:
- "Produce scatter plots, boxplots, and time series plots using ggplot."
- "Set universal plot settings."
- "Describe what faceting is and apply faceting in ggplot."
- "Modify the aesthetics of an existing ggplot plot (including axis labels and color)."
- "Build complex and customized plots from data in a data frame."
keypoints:
- "ggplot2 is a plotting package that makes it simple to create complex plots
from data in a data frame."
- "Define an aesthetic mapping (using the aes function), by selecting the
variables to be plotted and specifying how to present them in the graph."
- "Add ‘geoms’ – graphical representations of the data in the plot using
geom_point() for a scatter plot, geom_boxplot() for a boxplot, and
geom_line() for a line plot."
- "Faceting splits one plot into multiple plots based on a factor from the
dataset."
- "Every single component of a ggplot graph can be customized using the
generic theme() function.  However, there are pre-loaded themes available
that change the overall appearance of the graph without much effort."
- "The gridExtra package allows us to combine separate ggplots into a single
figure using grid.arrange()."
- "Use ggsave() to save a plot and edit the arguments (height, width, dpi) to
change the dimension and resolution."
---

## If you didn’t finish the previous day (or are having problems with the data) run this to create the needed envronment.

``` r
library("tidyverse")
dir.create(file.path("data_raw"), showWarnings = FALSE)
dir.create(file.path("data"), showWarnings = FALSE)
download.file(url = "https://ndownloader.figshare.com/files/2292169", destfile = "data_raw/portal_data_joined.csv")
surveys <- read_csv("data_raw/portal_data_joined.csv")
surveys <- surveys |>
  filter(!is.na(weight)) |>
  mutate(weight_kg = weight / 1000)
surveys_complete <- surveys |>
  filter(!is.na(weight),
         !is.na(hindfoot_length),
         !is.na(sex))
species_counts <- surveys_complete |>
  count(species_id) |>
  filter(n >= 50)
surveys_complete <- surveys_complete |>
  filter(species_id %in% species_counts$species_id)
write_csv(surveys_complete, "data/surveys_complete.csv")
```

We start by loading the required packages. **`ggplot2`** is included in
the **`tidyverse`** package.

``` r
library("tidyverse")
```

If not still in the workspace, load the data we saved in the previous
lesson.

``` r
surveys_complete <- read_csv("data/surveys_complete.csv")
```

## Plotting with **`ggplot2`**

**`ggplot2`** is a plotting package that makes it simple to create
complex plots from data in a data frame. It provides a more programmatic
interface for specifying what variables to plot, how they are displayed,
and general visual properties. Therefore, we only need minimal changes
if the underlying data change or if we decide to change from a bar plot
to a scatterplot. This helps in creating publication quality plots with
minimal amounts of adjustments and tweaking.

**`ggplot2`** functions like data in the ‘long’ format, i.e., a column
for every dimension, and a row for every observation. Well-structured
data will save you lots of time when making figures with **`ggplot2`**

ggplot graphics are built step by step by adding new elements. Adding
layers in this fashion allows for extensive flexibility and
customization of plots.

To build a ggplot, we will use the following basic template that can be
used for different types of plots:

    ggplot(data = <DATA>, mapping = aes(<MAPPINGS>)) +  <GEOM_FUNCTION>()

  - use the `ggplot()` function and bind the plot to a specific data
    frame using the `data` argument

<!-- end list -->

``` r
ggplot(data = surveys_complete)
```

![](fig/blank-graph-1.png)<!-- -->

  - define an aesthetic mapping (using the aesthetic (`aes`) function),
    by selecting the variables to be plotted and specifying how to
    present them in the graph, e.g. as x/y positions or characteristics
    such as size, shape, color, etc.

<!-- end list -->

``` r
ggplot(data = surveys_complete, mapping = aes(x = weight, y = hindfoot_length))
```

![](fig/with-axis-1.png)<!-- -->

  - add ‘geoms’ – graphical representations of the data in the plot
    (points, lines, bars). **`ggplot2`** offers many different geoms; we
    will use some common ones today, including:
    
      - `geom_point()` for scatter plots, dot plots, etc.
      - `geom_boxplot()` for, well, boxplots\!
      - `geom_line()` for trend lines, time series, etc.

To add a geom to the plot use `+` operator. Because we have two
continuous variables, let’s use `geom_point()` first:

``` r
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
  geom_point()
```

![](fig/first-ggplot-1.png)<!-- -->

The `+` in the **`ggplot2`** package is particularly useful because it
allows you to modify existing `ggplot` objects. This means you can
easily set up plot “templates” and conveniently explore different types
of plots, so the above plot can also be generated with code like this:

``` r
# Assign plot to a variable
surveys_plot <- ggplot(data = surveys_complete,
                       mapping = aes(x = weight, y = hindfoot_length))

# Draw the plot
surveys_plot +
    geom_point()
```

![](fig/first-ggplot-with-plus-1.png)<!-- -->

**Notes**

  - Anything you put in the `ggplot()` function can be seen by any geom
    layers that you add (i.e., these are universal plot settings). This
    includes the x- and y-axis you set up in `aes()`.
  - You can also specify aesthetics for a given geom independently of
    the aesthetics defined globally in the `ggplot()` function.
  - The `+` sign used to add layers must be placed at the end of each
    line containing a layer. If, instead, the `+` sign is added in the
    line before the other layer, **`ggplot2`** will not add the new
    layer and will return an error message.
  - Including `data = ` and `mapping = ` is optional, we include it here 
    for clarity but it is often omitted.

<!-- end list -->

``` r
# This is the correct syntax for adding layers
surveys_plot +
  geom_point()

# This will not add the new layer and will return an error message
surveys_plot
  + geom_point()
```

> ## Challenge
> 
> Scatter plots can be useful exploratory tools for small datasets. For
> data sets with large numbers of observations, such as the
> `surveys_complete` data set, overplotting of points can be a
> limitation of scatter plots. One strategy for handling such settings
> is to use hexagonal binning of observations. The plot space is
> tessellated into hexagons. Each hexagon is assigned a color based on
> the number of observations that fall within its boundaries. To use
> hexagonal binning with **`ggplot2`**, first install the R package
> `hexbin` from CRAN:
> 
> ``` r
> install.packages("hexbin")
> library("hexbin")
> ```
> 
> Then use the `geom_hex()` function:
> 
> ``` r
> surveys_plot +
>  geom_hex()
> ```
> 
>   - What are the relative strengths and weaknesses of a hexagonal bin
>     plot compared to a scatter plot? Examine the above scatter plot
>     and compare it with the hexagonal bin plot that you created.
> 
> 
{: .challenge}


> ## Stretch Challenge (Intermediate - 15 mins)
> 
> Add a smoothing line to the plot so that it is easier to interpret.
> You could also investigate whether a contour plot or heatmap improves
> clarity of interpretation. Hint: Have a look at the [**`ggplot2`**
> cheat sheet](https://ggplot2.tidyverse.org/) 
{: .challenge}


## Building your plots iteratively

Building plots with **`ggplot2`** is typically an iterative process. We
start by defining the dataset we’ll use, lay out the axes, and choose a
geom:

``` r
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point()
```

![](fig/create-ggplot-object-1.png)<!-- -->

Then, we start modifying this plot to extract more information from it.
For instance, we can add transparency (`alpha`) to avoid overplotting:

``` r
ggplot(data = surveys_complete, aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1)
```

![](fig/adding-transparency-1.png)<!-- -->

We can also add colors for all the points:

``` r
ggplot(data = surveys_complete, mapping = aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1, color = "blue")
```

![](fig/adding-colors-1.png)<!-- -->

Or to color each species in the plot differently, you could use a vector
(which must have data-type factor) as an input to the argument
**color**. **`ggplot2`** will provide a different color corresponding to
different values in the vector. Here is an example where we color with
**`species_id`**:

``` r
ggplot(data = surveys_complete, mapping = aes(x = weight, y = hindfoot_length)) +
    geom_point(alpha = 0.1, aes(color = species_id))
```

![](fig/color-by-species-1-1.png)<!-- -->

> ## Challenge
> 
> Use what you just learned to create a scatter plot of `weight` over
> `species_id` with the plot types showing in different colors. Is this
> a good way to show this type of data?
> 
> > ## Solution
> > 
> > ```r
> > ggplot(data = surveys_complete, mapping = aes(x = species_id, y = weight)) +
> >   geom_point(aes(color = plot_type))
> > ```
> > 
> > ![](fig/unnamed-chunk-4-1.png)<!-- --> 
> {: .solution}
>  
{: .challenge}

## Boxplot

We can use boxplots to visualize the distribution of weight within each
species:

``` r
ggplot(data = surveys_complete, mapping = aes(x = species_id, y = weight)) +
    geom_boxplot()
```

![](fig/boxplot-1.png)<!-- -->

By adding points to the boxplot, we can have a better idea of the number
of measurements and of their distribution:

``` r
ggplot(data = surveys_complete, mapping = aes(x = species_id, y = weight)) +
    geom_boxplot(alpha = 0) +
    geom_jitter(alpha = 0.3, color = "tomato")
```

![](fig/boxplot-with-points-1.png)<!-- -->

Notice how the boxplot layer is behind the jitter layer? What do you
need to change in the code to put the boxplot in front of the points
such that it’s not hidden?

> ## Challenges
> 
> Boxplots are useful summaries, but hide the *shape* of the
> distribution. For example, if there is a bimodal distribution, it
> would not be observed with a boxplot. An alternative to the boxplot is
> the violin plot (sometimes known as a beanplot), where the shape (of
> the density of points) is drawn.
> 
>   - Replace the box plot with a violin plot; see `geom_violin()`.
> 
> In many types of data, it is important to consider the *scale* of the
> observations. For example, it may be worth changing the scale of the
> axis to better distribute the observations in the space of the plot.
> Changing the scale of the axes is done similarly to adding/modifying
> other components (i.e., by incrementally adding commands). Try making
> these modifications:
> 
>   - Represent weight on the log<sub>10</sub> scale; see
>     `scale_y_log10()`.
> 
> So far, we’ve looked at the distribution of weight within species. Try
> making a new plot to explore the distribution of another variable
> within each species.
> 
>   - Create boxplot for `hindfoot_length`. Overlay the boxplot layer on
>     a jitter layer to show actual measurements.
> 
>   - Add color to the data points on your boxplot according to the plot
>     from which the sample was taken (`plot_id`).
> 
> Hint: Check the class for `plot_id`. Consider changing the class of
> `plot_id` from integer to factor. Why does this change how R makes the
> graph? 
{: .challenge}


> ## Stretch Challenge (Intermediate - 20 mins)
> 
>   - Add even more information to your boxplot by adding another
>     variable as the colour or fill in the aesthetic mappings.
> 
>   - Try different ways of visualising error on your boxplot e.g. 95%
>     error bars, pointrange, deciles, quintiles 
{: .challenge}


## Plotting time series data

Let’s calculate number of counts per year for each genus. First we need
to group the data and count records within each group:

``` r
yearly_counts <- surveys_complete |>
  count(year, genus)
```

Timelapse data can be visualized as a line plot with years on the x-axis
and counts on the y-axis:

``` r
ggplot(data = yearly_counts, aes(x = year, y = n)) +
     geom_line()
```

![](fig/first-time-series-1.png)<!-- -->

Unfortunately, this does not work because we plotted data for all the
genera together. We need to tell ggplot to draw a line for each genus by
modifying the aesthetic function to include `group = genus`:

``` r
ggplot(data = yearly_counts, aes(x = year, y = n, group = genus)) +
    geom_line()
```

![](fig/time-series-by-species-1.png)<!-- -->

We will be able to distinguish species in the plot if we add colors
(using `color` also automatically groups the data):

``` r
ggplot(data = yearly_counts, aes(x = year, y = n, color = genus)) +
    geom_line()
```

![](fig/time-series-with-colors-1.png)<!-- -->

## Integrating the pipe operator with ggplot2

In the previous lesson, we saw how to use the pipe operator `|>` to use
different functions in a sequence and create a coherent workflow. We can
also use the pipe operator to pass the `data` argument to the `ggplot()`
function. The hard part is to remember that to build your ggplot, you
need to use `+` and not `|>`.

``` r
yearly_counts |>
    ggplot(mapping = aes(x = year, y = n, color = genus)) +
    geom_line()
```

![](fig/integrating-the-pipe-1.png)<!-- -->

The pipe operator can also be used to link data manipulation with
consequent data visualization.

``` r
yearly_counts_graph <- surveys_complete |>
    count(year, genus) |>
    ggplot(mapping = aes(x = year, y = n, color = genus)) +
    geom_line()

yearly_counts_graph
```

![](fig/pipes-and-manipulation-1.png)<!-- -->

## Faceting

`ggplot` has a special technique called *faceting* that allows the user
to split one plot into multiple plots based on a factor included in the
dataset. In the following `vars` is used to analyse the factors on which
the plot will be split. We will use it to make a time series plot for
each species:

``` r
ggplot(data = yearly_counts, aes(x = year, y = n)) +
    geom_line() +
    facet_wrap(facets = vars(genus))
```

![](fig/first-facet-1.png)<!-- -->

Now we would like to split the line in each plot by the sex of each
individual measured. To do that we need to make counts in the data frame
grouped by `year`, `genus`, and `sex`:

``` r
 yearly_sex_counts <- surveys_complete |>
                        count(year, genus, sex)
```

We can now make the faceted plot by splitting further by sex using
`color` (within a single plot):

``` r
ggplot(data = yearly_sex_counts, mapping = aes(x = year, y = n, color = sex)) +
  geom_line() +
  facet_wrap(facets =  vars(genus))
```

![](fig/facet-colour-1.png)<!-- -->

We can use facet\_grid() to facet our plots by multiple variables, while
specifying which variable we want displayed by row and which by column:

``` r
ggplot(data = yearly_sex_counts,
  mapping = aes(x = year, y = n, color = sex)) +
  geom_line() +
  facet_grid(rows = vars(sex), cols =  vars(genus))
```

![](fig/facet-remove-grid-1.png)<!-- -->

You can also organise the panels only by rows (or only by columns):

``` r
# One column, facet by rows
ggplot(data = yearly_sex_counts,
  mapping = aes(x = year, y = n, color = sex)) +
  geom_line() +
  facet_grid(rows = vars(genus))
```

![](fig/facet-rows-1.png)<!-- -->

``` r
# One row, facet by column
ggplot(data = yearly_sex_counts,
  mapping = aes(x = year, y = n, color = sex)) +
  geom_line() +
  facet_grid(cols = vars(genus))
```

![](fig/facet-column-1.png)<!-- -->

**Note:** `ggplot2` before version 3.0.0 used formulas to specify how
plots are faceted. If you encounter `facet_grid`/`wrap(...)` code
containing `~`, please read
<https://ggplot2.tidyverse.org/news/#tidy-evaluation>.

## **`ggplot2`** themes

Usually plots with white background look more readable when printed.
Every single component of a `ggplot` graph can be customized using the
generic `theme()` function, as we will see below. However, there are
pre-loaded themes available that change the overall appearance of the
graph without much effort.

For example, we can change our previous graph to have a simpler white
background using the `theme_bw()` function:

``` r
 ggplot(data = yearly_sex_counts,
        mapping = aes(x = year, y = n, color = sex)) +
     geom_line() +
     facet_wrap(vars(genus)) +
     theme_bw()
```

![](fig/facet-by-species-and-sex-white-bg-1.png)<!-- -->

In addition to `theme_bw()`, which changes the plot background to white,
**`ggplot2`** comes with several other themes which can be useful to
quickly change the look of your visualization. The complete list of
themes is available at
<https://ggplot2.tidyverse.org/reference/ggtheme.html>.
`theme_minimal()` and `theme_light()` are popular, and `theme_void()`
can be useful as a starting point to create a new hand-crafted theme.

The [ggthemes](https://jrnold.github.io/ggthemes/reference/index.html)
package provides a wide variety of options.

> ## Challenge
> 
> Use what you just learned to create a plot that depicts how the
> average weight of each species changes through the years.
> 
> > ## Solution
> > 
> > ```r
> > yearly_weight <- surveys_complete |>
> >   group_by(year, species_id) |>
> >   summarize(avg_weight = mean(weight))
> > 
> > ggplot(data = yearly_weight, mapping = aes(x=year, y=avg_weight))+
> >   geom_line() +
> >   facet_wrap(vars(species_id)) +
> >   theme_bw()
> > ```
> >
> > ![](fig/challange-theme-bw-1.png)<!-- --> 
> {: .solution}
> 
> 
> 
{: .challenge}


## Customization

Take a look at the [**`ggplot2`** cheat
sheet](https://ggplot2.tidyverse.org/), and think of ways you could
improve the plot.

Now, let’s change names of axes to something more informative than
‘year’ and ‘n’ and add a title to the figure:

``` r
ggplot(data = yearly_sex_counts, aes(x = year, y = n, color = sex)) +
    geom_line() +
    facet_wrap(vars(genus)) +
    labs(title = "Observed genera through time",
         x = "Year of observation",
         y = "Number of individuals") +
    theme_bw()
```

![](fig/number-species-year-with-right-labels-1.png)<!-- -->

The axes have more informative names, but their readability can be
improved by increasing the font size. This can be done with the generic
`theme()` function:

``` r
ggplot(data = yearly_sex_counts, mapping = aes(x = year, y = n, color = sex)) +
    geom_line() +
    facet_wrap(vars(genus)) +
    labs(title = "Observed genera through time",
        x = "Year of observation",
        y = "Number of individuals") +
    theme_bw() +
    theme(text=element_text(size = 16))
```

![](fig/number-species-year-with-right-labels-xfont-size-1.png)<!-- -->

Note that it is also possible to change the fonts of your plots. If you
are on Windows, you may have to install the [**`extrafont`**
package](https://github.com/wch/extrafont), and follow the instructions
included in the README for this package.

After our manipulations, you may notice that the values on the x-axis
are still not properly readable. Let’s change the orientation of the
labels and adjust them vertically and horizontally so they don’t
overlap. You can use a 90 degree angle, or experiment to find the
appropriate angle for diagonally oriented labels:

``` r
ggplot(data = yearly_sex_counts, mapping = aes(x = year, y = n, color = sex)) +
    geom_line() +
    facet_wrap(vars(genus)) +
    labs(title = "Observed genera through time",
        x = "Year of observation",
        y = "Number of individuals") +
    theme_bw() +
    theme(axis.text.x = element_text(colour = "grey20", size = 12, angle = 90,
                                     hjust = 0.5, vjust = 0.5),
          axis.text.y = element_text(colour = "grey20", size = 12),
          text = element_text(size = 16))
```

![](fig/number-species-year-with-theme-1.png)<!-- -->

If you like the changes you created better than the default theme, you
can save them as an object to be able to easily apply them to other
plots you may create:

``` r
grey_theme <- theme(axis.text.x = element_text(colour="grey20", size = 12,
                                               angle = 90, hjust = 0.5,
                                               vjust = 0.5),
                    axis.text.y = element_text(colour = "grey20", size = 12),
                    text=element_text(size = 16))

ggplot(surveys_complete, aes(x = species_id, y = hindfoot_length)) +
    geom_boxplot() +
    grey_theme
```

![](fig/number-species-year-with-right-labels-xfont-orientation-1.png)<!-- -->

> ## Challenge
> 
> With all of this information in hand, please take another five minutes
> to either improve one of the plots generated in this exercise or
> create a beautiful graph of your own. Use the RStudio [**`ggplot2`**
> cheat sheet](https://ggplot2.tidyverse.org/) for inspiration.
> 
> Here are some ideas:
>
>   - Try using a different color palette using [RColorBrewer](https://www.datanovia.com/en/blog/the-a-z-of-rcolorbrewer-palette/).
>   - See if you can change the thickness of the lines.
>   - Can you find a way to change the name of the legend? What about
>     its labels? 
> 
> 
{: .challenge}


## Arranging and exporting plots

Faceting is a great tool for splitting one plot into multiple plots, but
sometimes you may want to produce a single figure that contains multiple
plots using different variables or even different data frames. The
**`gridExtra`** package allows us to combine separate ggplots into a
single figure using `grid.arrange()`:

``` r
library(gridExtra)

spp_weight_boxplot <- ggplot(data = surveys_complete,
                             aes(x = species_id, y = weight)) +
  geom_boxplot() +
  labs(x = "Species",
       y = expression(log[10](Weight))) +
  scale_y_log10() +
  labs()

spp_count_plot <- ggplot(data = yearly_counts,
                         aes(x = year, y = n, color = genus)) +
  geom_line() +
  labs(x = "Year", y = "Abundance")

grid.arrange(spp_weight_boxplot, spp_count_plot, ncol = 2, widths = c(4, 6))
```

![](fig/gridarrange-example-1.png)<!-- -->

In addition to the `ncol` and `nrow` arguments, used to make simple
arrangements, there are tools for [constucting more complex
layouts](https://cran.r-project.org/web/packages/gridExtra/vignettes/arrangeGrob.html).

After creating your plot, you can save it to a file in your favorite
format. The Export tab in the **Plot** pane in RStudio will save your
plots at low resolution, which will not be accepted by many journals and
will not scale well for posters.

Instead, use the `ggsave()` function, which allows you easily change the
dimension and resolution of your plot by adjusting the appropriate
arguments (`width`, `height` and `dpi`):

``` r
my_plot <- ggplot(data = yearly_sex_counts,
                  aes(x = year, y = n, color = sex)) +
    geom_line() +
    facet_wrap(vars(genus)) +
    labs(title = "Observed genera through time",
        x = "Year of observation",
        y = "Number of individuals") +
    theme_bw() +
    theme(axis.text.x = element_text(colour = "grey20", size = 12, angle = 90,
                                     hjust = 0.5, vjust = 0.5),
          axis.text.y = element_text(colour = "grey20", size = 12),
          text = element_text(size = 16))

ggsave("fig/name_of_file.png", my_plot, width = 15, height = 10)

## This also works for grid.arrange() plots
combo_plot <- grid.arrange(spp_weight_boxplot, spp_count_plot, ncol = 2,
                           widths = c(4, 6))
```

![](fig/ggsave-example-1.png)<!-- -->

``` r
ggsave("fig/combo_plot_abun_weight.png", combo_plot, width = 10, height = 6, dpi = 300)
```

Note: The parameters `width` and `height` also determine the font size
in the saved plot.

<p style="text-align: right; font-size: small;">

Page built on: 📆 2023-01-18 ‒ 🕢 09:26:39

</p>
