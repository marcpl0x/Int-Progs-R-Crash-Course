# R Crash Course - 02 Data Wrangling, Simple Analyses and Visualisation
Marc Koh
2026-03-23

# Hello again!

This is is the second part to the R Crash Course. In this Markdown,
we’ll tackle:

1.  Some basic data wrangling using `tidyverse`

2.  Some basic visualisation techniques

3.  Basic statistical analyses

This assumes that you have already completed the previous Markdown or
have prior knowledge of R set up, package installation and activation,
and data import.

Set yourself up with your working directory (where all your R files and
data is) and make sure you have a script called something sensible open
to type your code into.

> [!NOTE]
>
> ### Note:
>
> This Markdown also assumes that you have some knowledge of statistical
> tests and lingo. Although there will be some explanation, we will NOT
> go into detail of statistical theory, as it is assumed that your home
> university would have taught you some of this in greater detail.
>
> If you are having troubles some of the concepts or want to know more,
> please consult your tutors or academic.

# t-tests (Continous vs. 2 Categories)

## Context

Let’s imagine that you have been out to Moreton Bay Research Station
(MBRS) and you are trying to see if the crabs at 2 different locations,
Polka Point and Adams Beach. You’ve collected 15 crabs from each
location at random and weighed them, logging your data into 2 columns in
a long format:

- `location`: where the crab was collected from

- `crab_mass_g`: the mass of the crab in grams

## Data Wrangling

“Data wrangling” is the term used to describe altering certain
characteristics about your dataset (**NOT YOUR DATA**) so that R is able
to read it as you intended to carry out analyses.

We know that what we essentially have here is **continuous data from 2
different groups**.

However, R is not aware of this. Let me show you…

### Packages

First and foremost, get into the habit of calling your packages first so
that R knows that you are using functions from them:

``` r
# Packages ------
library("tidyverse")
library("readxl")
library("here")
```

> [!NOTE]
>
> ### A Note on Warnings and Conflicts
>
> If you are not on the latest version of R, you may get warning
> messages that look like this:
>
> ``` r
> Warning: package ‘readxl’ was built under R version 4.5.2
> ```
>
> In the words of one of my lecturers:
>
> *“… and like all good statisticians, we will ignore R’s warnings”*
>
> You may also need to take note of Conflicts. This is what happens when
> you have 2 packages that have the same function names. For example,
> both `dplyr`, which you have just called, and `stats` (a built in
> package that activates on booting R) have a function called
> `filter()`.
>
> This means that you should be extra careful when calling functions,
> making sure to call the correct one by referencing the function:
>
> ``` r
> dplyr::filter()
> # OR
> stats::filter()
> ```

### Import your data

Just as we did previously, import the `dummy_univar.xlsx` data:

``` r
fp <- here("data", "dummy_univar.xlsx") # generate file path

crab <- read_xlsx(fp, sheet = "t test") # import excel sheet

glimpse(crab) # check
```

    Rows: 29
    Columns: 2
    $ location    <chr> "Polka Point", "Polka Point", "Polka Point", "Polka Point"…
    $ crab_mass_g <dbl> 470, 672, 505, 576, 228, 191, 675, 182, 33, 533, 194, 416,…

Now, have a look at the structure of the data set using the `str()`
function:

``` r
str(crab)
```

    tibble [29 × 2] (S3: tbl_df/tbl/data.frame)
     $ location   : chr [1:29] "Polka Point" "Polka Point" "Polka Point" "Polka Point" ...
     $ crab_mass_g: num [1:29] 470 672 505 576 228 191 675 182 33 533 ...

So here is what you can tell from the structure of the data frame:

1.  You have a `crab_mass_g` column that has been read in as a `numeric`
    variable

2.  You have a `location` variable that is classified as `character`

Although not entirely an issue, we ideally want R to recognise
`location` as a **categorical** variable instead of a character.

Using `tidyverse`, you can do this using `mutate()` and `as_factor()`:

``` r
crab <- crab %>% # take crab and reassign it to crab
  mutate(location = as.factor(location)) # mutate `location` into a factor and assign it back to `location`

glimpse(crab) # check
```

    Rows: 29
    Columns: 2
    $ location    <fct> Polka Point, Polka Point, Polka Point, Polka Point, Polka …
    $ crab_mass_g <dbl> 470, 672, 505, 576, 228, 191, 675, 182, 33, 533, 194, 416,…

> [!NOTE]
>
> ### Pipes
>
> The `%>%` seen in the above chunk is called a pipe. Pipes are used to
> string together functions for a better workflow.
>
> Think of a pipe as telling R “do this, AND THEN, do that”
>
> `%>%` is the `tidyverse` pipe and **WILL NOT work if** `tidyverse`
> **or** `dplyr` **is not loaded**. The default hot key for this is:
>
> - Windows:
>   - `Ctrl`+`Shift`+`M`
> - MacOS:
>   - `Cmd`+`Shift`+`M`
>
> Alternatively, you can use the universal pipe `|>` (`|`+`>`), which
> does not have a hot key.

In the above chunk, you’ve taken the column `location`, which was
initially recognised as `character`, and explicitly told R that you want
it to be read in as a `factor` column instead. This is analogous with
categorical.

Just to be sure now, we should probably check to see if there are only 2
locations. You can do this using either:

``` r
levels(crab$location) # this can only be done if you have already factorised the column
```

    [1] "Adams Beach" "polka Point" "Polka Point"

OR

``` r
unique(crab$location) # more general use
```

    [1] Polka Point polka Point Adams Beach
    Levels: Adams Beach polka Point Polka Point

OR

``` r
plot(crab)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/plot%20check-1.png)

You may have already spotted this in the `str()` chunk, but both the
`unique()` and `levels()` functions tell us that there is someone
(probably one of your incapable group mates, it could never be you!) has
accidentally entered the location as “polka Point” instead of “Polka
Point” like you had previously agreed upon!

Let’s fix this:

``` r
crab <- crab %>% 
  mutate(location = fct_recode(location,
                               `Polka Point` = "polka Point")) # new level = old level

str(crab) # check "Factor w/ 2 levels"
```

    tibble [29 × 2] (S3: tbl_df/tbl/data.frame)
     $ location   : Factor w/ 2 levels "Adams Beach",..: 2 2 2 2 2 2 2 2 2 2 ...
     $ crab_mass_g: num [1:29] 470 672 505 576 228 191 675 182 33 533 ...

> [!TIP]
>
> ### Will it work??
>
> If you are unsure if a function will make the intended changes to your
> data frame, you could always assign it to a different object then
> overwrite the old one:
>
> ``` r
> crab1 <- crab %>% # name it crab1 instead
>   mutate(location = fct_recode(location,
>                                `Polka Point` = "polka Point")) # new level = old level
>
> str(crab1) # check "Factor w/ 2 levels"
>
> crab <- crab1 # assign it back
>
> rm(crab1) # remove from environment to keep it clean
> ```

## Statistical Analysis

Now that you have your dataset arranged appropriately, it’s time to see
if there is a difference in the means of the mass of crabs at Polka
Point and Adams Beach.

Because you are comparing 2 groups here, the most appropriate
statistical test is a two tailed t-test.

If you like hypotheses:

$H_0$: There is no difference between the mass of crabs at Polka Point
and Adams Beach

$H_1$: There is a difference between the mass of crabs at Polka Point
and Adams Beach

> [!WARNING]
>
> ### There are difference kinds of t-tests!!!
>
> There are 3 basic types of t-tests depending on your data and
> hypotheses: One Tailed, Two Tailed, and Paired. If unsure which to
> use, try to look it up and ask your tutor for guidance.

Here is how you do a t test in R:

``` r
m1 <- t.test(crab_mass_g ~ location, # y ~ x
             data = crab) # define the dataset to be used

m1 # print result
```


        Welch Two Sample t-test

    data:  crab_mass_g by location
    t = -3.0804, df = 21.85, p-value = 0.005499
    alternative hypothesis: true difference in means between group Adams Beach and group Polka Point is not equal to 0
    95 percent confidence interval:
     -329.25183  -64.23388
    sample estimates:
    mean in group Adams Beach mean in group Polka Point 
                     208.4000                  405.1429 

``` r
# OR

m2 <- t.test(crab$crab_mass_g ~ crab$location) # this notation represents dataset$column.name
m2
```


        Welch Two Sample t-test

    data:  crab$crab_mass_g by crab$location
    t = -3.0804, df = 21.85, p-value = 0.005499
    alternative hypothesis: true difference in means between group Adams Beach and group Polka Point is not equal to 0
    95 percent confidence interval:
     -329.25183  -64.23388
    sample estimates:
    mean in group Adams Beach mean in group Polka Point 
                     208.4000                  405.1429 

🎉Congratulations🎉! You’ve done your first(?) t-test!

The next step of this is to report your statistics in your assessment
piece. Have a think about this and read a few journal articles or author
guidelines from the journal citation style you have to use.

> [!NOTE]
>
> ### Bitta trivia
>
> The t-test was formulated to make beer taste better (real)

## Plots

For the crab data, the most appropriate scientific plot would be a
boxplot. Although we have done a rough one in the the Data Wrangling
Section, it is not what we consider “publication ready”.

Generally in marine science, publication plots made in R are usually
made using a package that is part of `tidyverse` called `ggplot`.

`ggplot` is pretty intuitive as it works in layers (like an onion or
cake). You start with the base `ggplot()` layer, then add layers of
points (`geom_point` or `geom_jiter`), bars (`geom_bar`), boxplots
(`geom_boxplot`), etc. You can even customize labels and colours in the
same string of code using functions such as `labs()` and
`scale_colour_manual()`. Almost like the tidyverse pipe `%>%`, you can
use `+` to string these layers together.

If you haven’t already activated `tidyverse`, you will have run
`library(ggplot)` before running the code below.

``` r
( # putting the code in brackets like this immediately prints the plot out
  crab_p1 <- ggplot(data = crab, #define data as crab
                  aes(x = location, # define x aesthetic as location
                      y = crab_mass_g))+ # define y aesthetic as mass
  geom_boxplot()+ # add boxplot
  labs(y = "Mass of Crab (g)", # relabel y axis
       x = "Location")+ # relabel x axis
  #geom_jitter()+ # add optional jitter of data points
  theme_classic( # publication theme
    base_size = 14 # set font size to 14
    ) 
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/crab%20boxplot-1.png)

Once you’re happy with the customisation of your plot, you can export it
using code as well:

``` r
ggsave(filename = "crab_boxplot.jpg", # name the image and define file type
       plot = crab_p1, # the plot you want to export
       dpi = 600, # resolution
       height = 7,
       width = 10)
```

# Some Plots Are More Equal Than Others

Ideally, a good figure, along with a good figure caption, should allow
the reader to almost grasp the full result of your study without
actually having to read your entire results section.

It is important to choose the correct plots to represent your results
and data so that you can achieve this too!!

## General Rules with Scientific Plots

Generally in the scientific community, there are certain things to avoid
if possible.

### Examples of plots to avoid

*“Scientists are never 100% sure, that’s why we use p \< 0.05.”*

One thing to always note is that **the beauty of science is that we
ALWAYS leave room for error because we are always refining our
understanding of the world**. We therefore have to reflect this as well
in our plots and should avoid using plots that present only 1 value.

Bad plots include things like:

- Bar graphs (specifically without error bars)

  - The only show 1 value, usually a mean, without any error.

  - Allows you to hide how many data points are actually present in your
    dataset. If you use a bar plot, people automatically assume you do
    not have enough data points

- Pie Charts

  - Unless you work in finance, these are pretty dogshit.

  - Relies on the reader to proportion your whole pie into different
    angles and convert that into your unit of measurement. Why leave the
    interpretation up to your reader? So they can make a TikTok about it
    and use it to spread misinformation?

- Anything 3D if there are only 2 dimensions of data

  - Special place in hell for people who do 3D pie charts

### Only add colour when absolutely necessary

In the t-test example above, we used a box plot with no colour. The
reason for this is that the plot communicates everything necessary for
the reader without any bells and whistles.

Take this plot for example:

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/bad%20plot-1.png)

> [!NOTE]
>
> ### What’s wrong with this plot?
>
> The distinction of colour here is not needed and adds **no informative
> value to the plot**. The x-axis already tells you that the 2 locations
> are distinct, the colour does not add to or take away from that
> distinction, it just requires you to add on a legend that does not add
> any new information to the plot.

## Colour: When and How

Colour in plots is necessary if you are **distinguishing between more
than 2 groups, or there are subgroups** are involved.

When you do choose to use colours, you can create your own palette. Just
make sure you have the correct number of colours for your number of
groups:

``` r
# If I had 4 groups

my_pal <- c("black", "pink", "blue", "green")
```

If you are familiar with hexcodes, you can use those too:

``` r
hex_pal <- c("#FF00CC", "#729891", "#875812", "#0000FF")
```

If you need help to see what names match which of R’s 657 built in
colours, run this in your console:

``` r
demo("colors") # American spelling here
```

## What Colours should I choose?

Colours that you pick should be contrasting, but at the same time, you
should take extra care to make your palettes accessible. This means that
you should try to pick colours that are colour vision deficient
friendly.

Apart from asking one of your colourblind mates, you can use packages
such as the `colorblindchecker()` to ensure that your palettes are
accessible.

``` r
#install.packages(colorblindcheck)
library(colorblindcheck)

colorblindcheck::palette_plot(my_pal)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/colour%20blind%20check-1.png)

``` r
colorblindcheck::palette_plot(hex_pal)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/colour%20blind%20check-2.png)

### Palette Packages

Not artistic or don’t have the time for it? The beauty of R is that
because it is open source, people go out of their way to create work
around via packages to improve their workflow and willingly share it
with the community. Some really cool palettes are available online that
are also already colour vision deficient friendly. Here are some of my
favourites:

#### `MetBrewer`

[MetBrewer](https://www.blakerobertmills.com/my-work/met-brewer) is made
by old mate Blake Robert Mills. He essentially went to the Metropolitan
Museum of Art (where they hold the Met Gala i think) nd colour matched
various works of art. He based it on the work of [Professor Cynthia
Brewer](https://scholar.google.com/citations?user=i2yTpDcAAAAJ&hl=en).

Here is `MetBrewer` in action:

``` r
library(MetBrewer)

MetBrewer::display_all() # all the palettes in MetBrewer
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/metbrewer%20demo-1.png)

``` r
MetBrewer:::display_all(colorblind_only = T) # Colourblind friendly palettes
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/metbrewer%20demo-2.png)

My personal favourite for plotting is “Kandinksy”, inspired by [Wassily
Kandinksy’s Kleine Welten
IV](https://www.moma.org/collection/works/67212).

#### `wesanderson`

Introduced to me by one of my favourite lecturers at UQ, [Daniel
Harris](https://about.uq.edu.au/experts/16758) (very cool guy). The
`wesanderson` package is exactly what it sounds like; a package of
colour palettes that were used in the making of his movies (Fantastic
Mr. Fox, The Grand Budapest Hotel, The Darjeeling Limited, etc.).

Read more [here](https://github.com/karthik/wesanderson) on Karthik
Ram’s (UC Berkley) GitHub…

``` r
library(wesanderson)
```

    Registered S3 method overwritten by 'wesanderson':
      method        from     
      print.palette MetBrewer

``` r
# from ?wes_palette

wes_palette("Royal1")
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/wesanderson%20demo-1.png)

``` r
wes_palette("GrandBudapest1")
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/wesanderson%20demo-2.png)

``` r
wes_palette("Cavalcanti1")
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/wesanderson%20demo-3.png)

``` r
wes_palette("Cavalcanti1", 3)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/wesanderson%20demo-4.png)

``` r
# If you need more colours than normally found in a palette, you
# can use a continuous palette to interpolate between existing
# colours
pal <- wes_palette(21, name = "Zissou1", type = "continuous")
image(volcano, col = pal)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/wesanderson%20demo-5.png)

#### `viridis`

`viridis` is the most commonly used colour palette for continuous data.
All the pallettes in it are colour blind friendly.

There is an awesome intro to it on
[CRAN](https://cran.r-project.org/web/packages/viridis/vignettes/intro-to-viridis.html)

``` r
library(viridis)
```

    Loading required package: viridisLite

``` r
#create dummy heatmap data
x <- LETTERS[1:20]
y <- paste0("var", seq(1,20))
data <- expand.grid(X=x, Y=y)
data$Z <- runif(400, 0, 5)
 
# Heatmap 
ggplot(data, aes(X, Y, fill= Z)) + 
  geom_tile()+
  scale_fill_viridis(discrete = F) # fill with viridis palette
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/viridis%20demo-1.png)

## Plotting Resources

It would be very advantageous for your scientific career if you
familiarise yourself with plotting in `ggplot`. Here’s a
[cheatsheet](https://posit.co/wp-content/uploads/2022/10/data-visualization-1.pdf)
that might be super useful.

A word of note, it is possible to visualise your data in base R using
basic functions like `plot()` and `hist()`, but it is generally only
used for exploration of data to see how we should go about analysiing
it.

Another resource I cannot recommend enough it the [R Graph
Gallery](https://r-graph-gallery.com/). I personally use this a lot
because it shows you example code to generate all sorts of plots that
you can refer to for your own data!!!

# ANOVA (Continuous vs. 3 or more Categories)

Analysis of Variance (ANOVA) is used in the context of testing for
differences between 3 or more groups or subgroups. The theory behind it
is that it alleviates the problem of committing Type I errors (false
positives, or rejecting the null hypothesis when it is actually true)
that comes from doing multiple t-tests.

ANOVAs produce an F-value which is the ratio of the variation *between*
groups to the variation *within* groups using Mean Squares:

$$F = \frac{MSB}{MSW} = \frac{Var_{Between}}{Var_{Within}}$$

Essentially, the higher the F value, the more variation there is
*between* samples than *within* groups, showing that the groups are
distinct.

ANOVAs also require you to do a post-hoc test called Tukey’s Test for
Honest Significant Difference. This is to show which groups are
different, and how exactly they differ.

## Context

You and your mates have collected data on coral sizes from 3 different
sites on Heron Reef, Great Barrier Reef and want to see if there is a
difference in sizes of different types of corals at different locations.

$H_0$: There is no difference between different types of corals at
different locations on Heron Reef

$H_1$: There is a difference between different types of corals at
different locations on Heron Reef

Your dataset has the following columns:

- `location`: Three study sites

- `coral morphospecies`: Three different types of coral

- `size_mm` diameter of individual coral measured in $mm$

## Data Import and Wrangling

Import your dataset and make the necessary columns factors:

``` r
coral <- read_xlsx(fp, # from t-test section
                   sheet = 2) # you can also reference sheet numbers if youa re sure it is the right sheet

glimpse(coral)
```

    Rows: 45
    Columns: 3
    $ location              <chr> "Heron Bommie", "Heron Bommie", "Heron Bommie", …
    $ `coral morphospecies` <chr> "Branching", "Branching", "Branching", "Branchin…
    $ size_mm               <dbl> 1246, 434, 428, 655, 1717, 822, 904, 486, 1462, …

``` r
coral <- coral %>% 
  mutate(location = as_factor(location),
         coral_morphospecies = as_factor(`coral morphospecies`)) %>%  # here, I created a new column of coral_morphospecies because we don't like spaces!!!
         select(!(`coral morphospecies`)) # remove the old column

str(coral) # check
```

    tibble [45 × 3] (S3: tbl_df/tbl/data.frame)
     $ location           : Factor w/ 3 levels "Heron Bommie",..: 1 1 1 1 1 1 1 1 1 1 ...
     $ size_mm            : num [1:45] 1246 434 428 655 1717 ...
     $ coral_morphospecies: Factor w/ 3 levels "Branching","Soft",..: 1 1 1 1 1 2 2 2 2 2 ...

## Statistical Analysis

As discussed above, there are 2 steps to ANOVAs:

### The ANOVA itself:

ANOVAs in R are pretty straightforward, the function used is `aov()`.

> [!TIP]
>
> Just like t-tests, there are different types of ANOVAs: One Way, or
> Two Way. This depends on your hypothesis. Ask your tutor if unsure!!!

#### Formulating your ANOVA

There are several ways to perform ANOVAs, depending on our hypotheses.

For example, if we were purely looking for differences in sizes of
different `coral_morphospecies` **OR** `location`, the formula of your
ANOVA would look like:

$$size\_mm\sim{coral\_morphospecies}$$ $$Or$$

$$size\_mm\sim{location}$$ If you looking at looking at the effect that
`location` **and** `coral morphospecies` individually in the same model:

$$size\_mm\sim{location\,+\,coral\_morphospecies}$$ In the context of
our study here, we are looking at how both of these variables affect
size **together**, or **how size is affected when the coral
morphospecies and location interact with one another**.

This calls for a formula where the $+$ is replaced with $*$
(multiplication):

$$size\_mm\sim{location\,*\,coral\_morphospecies}$$

Or in long form:

$$size\_mm\sim{location\,+\,coral\_morphospecies\,+\,location:coral\_morphospecies}$$

In the above formula, the interaction term considered
is`location:coral_morphospecies`. This essentially considers variation
in `size_mm` explained by `location`, `coral_morphospecies`, and the
interaction of both terms (`location:coral_morphospecies`).

> [!NOTE]
>
> ### To interact or not to interact
>
> Interaction of terms depends on the research question you are trying
> to answer. Do you want to consider each variable on its own? Or do you
> want to explain combined effects they may have on your response
> variable?

``` r
m2 <- aov(size_mm ~ location*coral_morphospecies, # interaction of terms here
          data = coral)

summary(m2) # summary(your.anova) is needed here to print the full results you need
```

                                 Df    Sum Sq  Mean Sq F value   Pr(>F)    
    location                      2  80409017 40204508  21.030 8.91e-07 ***
    coral_morphospecies           2 107478818 53739409  28.110 4.43e-08 ***
    location:coral_morphospecies  4  27601022  6900256   3.609   0.0142 *  
    Residuals                    36  68822716  1911742                     
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

> [!TIP]
>
> ### Stars align…
>
> In certain analyses functions, you may see the `Signif. codes` legend
> pop up at the bottom of your `summary()`. The astericks (`*`)
> essentially reflect the p-value of your variable.
>
> The more `*`s you get, the lower your p-value.

Hooray! ANOVA done!

What you have above essentially shows that:

- `location` has a significant effect on size ($p < 0.001$)
- `coral morphospecies` has a significant effect on size ($p < 0.001$)
- `location:coral morphospecies` has a significant effect on size
  ($p < 0.05$)

Ok so we know that there is a difference between these, but are they all
different?

Find out next episode….

### Tukey’s Test for Honest Significant Difference

To figure out which groups are exactly different, we’ll have to do a
Tukey Test.

Let’s look at `location` first:

``` r
m2_ph_location <- TukeyHSD(m2, # your anova
                           which = "location") # select the column you are interested in
m2_ph_location # print
```

      Tukey multiple comparisons of means
        95% family-wise confidence level

    Fit: aov(formula = size_mm ~ location * coral_morphospecies, data = coral)

    $location
                                    diff       lwr      upr     p adj
    Coral Gardens-Heron Bommie  271.7333 -962.3316 1505.798 0.8530788
    Blue Pools-Heron Bommie    2961.7333 1727.6684 4195.798 0.0000031
    Blue Pools-Coral Gardens   2690.0000 1455.9351 3924.065 0.0000161

If you are a visual person, you can also `plot()` the results:

``` r
plot(m2_ph_location)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/coral%20tukey%201%20plot-1.png)

> [!NOTE]
>
> In the `plot()` chunk, what you are looking for is each whisker plot
> to **not cross the dotted vertical line at 0**.
>
> This means that 0 is not included in the confidence interval of
> possible differences. i.e., we are confident that there is an non-zero
> difference between both groups.

Now for `coral_morphospecies`:

``` r
m2_ph_morph <- TukeyHSD(m2, which = "coral_morphospecies")
m2_ph_morph
```

      Tukey multiple comparisons of means
        95% family-wise confidence level

    Fit: aov(formula = size_mm ~ location * coral_morphospecies, data = coral)

    $coral_morphospecies
                           diff       lwr      upr     p adj
    Soft-Branching     650.2667 -583.7983 1884.332 0.4110203
    Massive-Branching 3554.8000 2320.7351 4788.865 0.0000001
    Massive-Soft      2904.5333 1670.4684 4138.598 0.0000044

``` r
plot(m2_ph_morph)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/coral%20tukey%202-1.png)

Lastly, the interaction term:

``` r
m2_ph_int <- TukeyHSD(m2, which = "location:coral_morphospecies")
m2_ph_int
```

      Tukey multiple comparisons of means
        95% family-wise confidence level

    Fit: aov(formula = size_mm ~ location * coral_morphospecies, data = coral)

    $`location:coral_morphospecies`
                                                      diff         lwr       upr
    Coral Gardens:Branching-Heron Bommie:Branching   426.2 -2457.00448  3309.404
    Blue Pools:Branching-Heron Bommie:Branching      868.6 -2014.60448  3751.804
    Heron Bommie:Soft-Heron Bommie:Branching         132.8 -2750.40448  3016.004
    Coral Gardens:Soft-Heron Bommie:Branching       -138.8 -3022.00448  2744.404
    Blue Pools:Soft-Heron Bommie:Branching          3251.6   368.39552  6134.804
    Heron Bommie:Massive-Heron Bommie:Branching     2133.6  -749.60448  5016.804
    Coral Gardens:Massive-Heron Bommie:Branching    2794.2   -89.00448  5677.404
    Blue Pools:Massive-Heron Bommie:Branching       7031.4  4148.19552  9914.604
    Blue Pools:Branching-Coral Gardens:Branching     442.4 -2440.80448  3325.604
    Heron Bommie:Soft-Coral Gardens:Branching       -293.4 -3176.60448  2589.804
    Coral Gardens:Soft-Coral Gardens:Branching      -565.0 -3448.20448  2318.204
    Blue Pools:Soft-Coral Gardens:Branching         2825.4   -57.80448  5708.604
    Heron Bommie:Massive-Coral Gardens:Branching    1707.4 -1175.80448  4590.604
    Coral Gardens:Massive-Coral Gardens:Branching   2368.0  -515.20448  5251.204
    Blue Pools:Massive-Coral Gardens:Branching      6605.2  3721.99552  9488.404
    Heron Bommie:Soft-Blue Pools:Branching          -735.8 -3619.00448  2147.404
    Coral Gardens:Soft-Blue Pools:Branching        -1007.4 -3890.60448  1875.804
    Blue Pools:Soft-Blue Pools:Branching            2383.0  -500.20448  5266.204
    Heron Bommie:Massive-Blue Pools:Branching       1265.0 -1618.20448  4148.204
    Coral Gardens:Massive-Blue Pools:Branching      1925.6  -957.60448  4808.804
    Blue Pools:Massive-Blue Pools:Branching         6162.8  3279.59552  9046.004
    Coral Gardens:Soft-Heron Bommie:Soft            -271.6 -3154.80448  2611.604
    Blue Pools:Soft-Heron Bommie:Soft               3118.8   235.59552  6002.004
    Heron Bommie:Massive-Heron Bommie:Soft          2000.8  -882.40448  4884.004
    Coral Gardens:Massive-Heron Bommie:Soft         2661.4  -221.80448  5544.604
    Blue Pools:Massive-Heron Bommie:Soft            6898.6  4015.39552  9781.804
    Blue Pools:Soft-Coral Gardens:Soft              3390.4   507.19552  6273.604
    Heron Bommie:Massive-Coral Gardens:Soft         2272.4  -610.80448  5155.604
    Coral Gardens:Massive-Coral Gardens:Soft        2933.0    49.79552  5816.204
    Blue Pools:Massive-Coral Gardens:Soft           7170.2  4286.99552 10053.404
    Heron Bommie:Massive-Blue Pools:Soft           -1118.0 -4001.20448  1765.204
    Coral Gardens:Massive-Blue Pools:Soft           -457.4 -3340.60448  2425.804
    Blue Pools:Massive-Blue Pools:Soft              3779.8   896.59552  6663.004
    Coral Gardens:Massive-Heron Bommie:Massive       660.6 -2222.60448  3543.804
    Blue Pools:Massive-Heron Bommie:Massive         4897.8  2014.59552  7781.004
    Blue Pools:Massive-Coral Gardens:Massive        4237.2  1353.99552  7120.404
                                                       p adj
    Coral Gardens:Branching-Heron Bommie:Branching 0.9998937
    Blue Pools:Branching-Heron Bommie:Branching    0.9841593
    Heron Bommie:Soft-Heron Bommie:Branching       1.0000000
    Coral Gardens:Soft-Heron Bommie:Branching      1.0000000
    Blue Pools:Soft-Heron Bommie:Branching         0.0173660
    Heron Bommie:Massive-Heron Bommie:Branching    0.2944102
    Coral Gardens:Massive-Heron Bommie:Branching   0.0635734
    Blue Pools:Massive-Heron Bommie:Branching      0.0000001
    Blue Pools:Branching-Coral Gardens:Branching   0.9998593
    Heron Bommie:Soft-Coral Gardens:Branching      0.9999939
    Coral Gardens:Soft-Coral Gardens:Branching     0.9991464
    Blue Pools:Soft-Coral Gardens:Branching        0.0584846
    Heron Bommie:Massive-Coral Gardens:Branching   0.5840508
    Coral Gardens:Massive-Coral Gardens:Branching  0.1809095
    Blue Pools:Massive-Coral Gardens:Branching     0.0000002
    Heron Bommie:Soft-Blue Pools:Branching         0.9946070
    Coral Gardens:Soft-Blue Pools:Branching        0.9613692
    Blue Pools:Soft-Blue Pools:Branching           0.1749501
    Heron Bommie:Massive-Blue Pools:Branching      0.8717567
    Coral Gardens:Massive-Blue Pools:Branching     0.4258441
    Blue Pools:Massive-Blue Pools:Branching        0.0000010
    Coral Gardens:Soft-Heron Bommie:Soft           0.9999967
    Blue Pools:Soft-Heron Bommie:Soft              0.0256998
    Heron Bommie:Massive-Heron Bommie:Soft         0.3753842
    Coral Gardens:Massive-Heron Bommie:Soft        0.0898010
    Blue Pools:Massive-Heron Bommie:Soft           0.0000001
    Blue Pools:Soft-Coral Gardens:Soft             0.0114003
    Heron Bommie:Massive-Coral Gardens:Soft        0.2225350
    Coral Gardens:Massive-Coral Gardens:Soft       0.0435899
    Blue Pools:Massive-Coral Gardens:Soft          0.0000000
    Heron Bommie:Massive-Blue Pools:Soft           0.9311943
    Coral Gardens:Massive-Blue Pools:Soft          0.9998194
    Blue Pools:Massive-Blue Pools:Soft             0.0033290
    Coral Gardens:Massive-Heron Bommie:Massive     0.9974171
    Blue Pools:Massive-Heron Bommie:Massive        0.0000763
    Blue Pools:Massive-Coral Gardens:Massive       0.0007320

As you can see, it is a very long list…

Let’s try a bit of wrangling to filter out the ones that do not show
significant statistical difference:

``` r
str(m2_ph_int) # check the structure to see if we can change it to a data frame
```

    List of 1
     $ location:coral_morphospecies: num [1:36, 1:4] 426 869 133 -139 3252 ...
      ..- attr(*, "dimnames")=List of 2
      .. ..$ : chr [1:36] "Coral Gardens:Branching-Heron Bommie:Branching" "Blue Pools:Branching-Heron Bommie:Branching" "Heron Bommie:Soft-Heron Bommie:Branching" "Coral Gardens:Soft-Heron Bommie:Branching" ...
      .. ..$ : chr [1:4] "diff" "lwr" "upr" "p adj"
     - attr(*, "class")= chr [1:2] "TukeyHSD" "multicomp"
     - attr(*, "orig.call")= language aov(formula = size_mm ~ location * coral_morphospecies, data = coral)
     - attr(*, "conf.level")= num 0.95
     - attr(*, "ordered")= logi FALSE

``` r
m2_ph_int_filtered <- m2_ph_int[["location:coral_morphospecies"]] %>% # take the table from the TukeyHSD
  as.data.frame() %>% # convert it to a dataframe from a list
  dplyr::filter(`p adj`< 0.05) # filter for p < 0.05

# note that you cannot plot this the same way since you converted it into a different thing!!!

m2_ph_int_filtered
```

                                                 diff        lwr       upr
    Blue Pools:Soft-Heron Bommie:Branching     3251.6  368.39552  6134.804
    Blue Pools:Massive-Heron Bommie:Branching  7031.4 4148.19552  9914.604
    Blue Pools:Massive-Coral Gardens:Branching 6605.2 3721.99552  9488.404
    Blue Pools:Massive-Blue Pools:Branching    6162.8 3279.59552  9046.004
    Blue Pools:Soft-Heron Bommie:Soft          3118.8  235.59552  6002.004
    Blue Pools:Massive-Heron Bommie:Soft       6898.6 4015.39552  9781.804
    Blue Pools:Soft-Coral Gardens:Soft         3390.4  507.19552  6273.604
    Coral Gardens:Massive-Coral Gardens:Soft   2933.0   49.79552  5816.204
    Blue Pools:Massive-Coral Gardens:Soft      7170.2 4286.99552 10053.404
    Blue Pools:Massive-Blue Pools:Soft         3779.8  896.59552  6663.004
    Blue Pools:Massive-Heron Bommie:Massive    4897.8 2014.59552  7781.004
    Blue Pools:Massive-Coral Gardens:Massive   4237.2 1353.99552  7120.404
                                                      p adj
    Blue Pools:Soft-Heron Bommie:Branching     1.736601e-02
    Blue Pools:Massive-Heron Bommie:Branching  5.133279e-08
    Blue Pools:Massive-Coral Gardens:Branching 2.139664e-07
    Blue Pools:Massive-Blue Pools:Branching    9.634315e-07
    Blue Pools:Soft-Heron Bommie:Soft          2.569979e-02
    Blue Pools:Massive-Heron Bommie:Soft       7.988238e-08
    Blue Pools:Soft-Coral Gardens:Soft         1.140029e-02
    Coral Gardens:Massive-Coral Gardens:Soft   4.358987e-02
    Blue Pools:Massive-Coral Gardens:Soft      3.241923e-08
    Blue Pools:Massive-Blue Pools:Soft         3.329037e-03
    Blue Pools:Massive-Heron Bommie:Massive    7.634107e-05
    Blue Pools:Massive-Coral Gardens:Massive   7.319847e-04

## Plots

Based on our data, the best thing to use to plot is the boxplot.

### `location` Boxplot

Similar to what you did with the t-test, so I will not comment on the
code…

``` r
(coral_location_p <- ggplot(data = coral,
                           aes(x = location,
                               y = size_mm))+
  geom_boxplot()+
  labs(x = "Location",
       y = "Size (mm)")+
  theme_classic(base_size = 14)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/location%20boxplot-1.png)

### `coral_morphospecies` Boxplot

Again, easy stuff…

``` r
(coral_morph_p <- ggplot(data = coral,
                           aes(x = coral_morphospecies,
                               y = size_mm))+
  geom_boxplot()+
  labs(x = "Coral Morphospecies",
       y = "Size (mm)")+
  theme_classic(base_size = 14)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/morph%20boxplot-1.png)

### `location:coral_morphospecies` Boxplot

So the interacting term is a little more complicated. There are 2 ways
to do this.

#### Use Colour to Differentiate

This is an appropriate time to use colour!

Because you have 2 levels of differentiation (subgroups) due to the
interaction term, we can use colour to differentiate these groups within
groups.

``` r
(coral_int1 <- ggplot(data = coral,
                      aes(x = location,
                          y = size_mm,
                          fill = coral_morphospecies))+ # this adds a fill to the plot to differentiate the different coral morphospecies
  geom_boxplot()+
  labs(x = "Location",
       y = "Size (mm)")+
  scale_fill_manual("Coral Morphospecies", # 
                    values = met.brewer("Kandinsky") # use MetBrewer palette
                    )+
  theme_classic(base_size = 14)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/int%20boxplot%201-1.png)

#### Use Facets

Alternatively, use a facet (subdivide a plot):

``` r
(coral_int2 <- ggplot(data = coral,
                      aes(x = location,
                          y = size_mm))+ 
  geom_boxplot()+
  labs(x = "Location",
       y = "Size (mm)")+
  theme_classic(base_size = 14)+
    facet_wrap(~coral_morphospecies, # facet by coral_species for easier comparison 
               scales = "free")+ # scale the y-axis for each morphospecies
  theme(axis.text.x = element_text(angle = 45,# angle text 45 degrees
                                   hjust = 1)) # horizontal adjust
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/facet%20eg-1.png)

``` r
# the text adjustment must come at the end in this case to get the angle changed after the facet!!
```

# Linear Regression (Continuous vs Continuous)

Linear regressions are exactly what they sound like: looking for linear
correlations between 2 continuous variables.

In a sense, it is a play on the equation of a linear line:

$$y=mx+c$$

However, unlike regular math formulae, we are fitting a line to data
that does not always sit on the said line, unlike a mathematical truth
like $y=mx+c$.

We therefore have to account for the fact that **there will be a
difference in what is recorded vs what we predict**. The difference in
what is predicted known as the **error or residual** in linear
regression.

This therefore approximates our $y$ to

$$\hat{y}\sim{\beta{_0}+\beta{_1}x}$$ $$or$$

$$y\sim{\beta{_0}+\beta{_1}x+\epsilon}$$

Where $\epsilon$ is the residual/error.

This error is important for linear regression graphs as it helps us to
eventually calculate our confidence intervals.

It also allows for the calculation of the Coefficient of Determination
($R^2$). $R^2$ is an important measure as it essentially explains **how
much of the variation of your response variable is explained by your
model**. The closer this value is to 1, the better!

> [!WARNING]
>
> ### $R^2$ vs $r$
>
> A key thing that people always mix up is correlation ($r$) and the
> Coefficient of Determination ($R^2$). They are not the same thing!!!
>
> - Correlation is the measure of how variables change with one another.
>   Specifically, $r$ is **directional**, which means $-1<r<1$. The
>   formula for correlation (Pearson’s) is:
>
> $$r = \frac{\sum{xy}-n\bar{x}\bar{y}}{(n-1)s_{x}s_{y}}$$ Where: - $n$
> = sample size - $s_x$ = Standard Deviation of $x$ - $s_y$ = Standard
> Deviation of $y$ - $\bar{x}$ = mean of $x$ - $\bar{y}$ = mean of $y$
>
> - $R^2$ explained above, is more of a measure of the “health” of your
>   model overall and how much of the variation of $y$ is explained by
>   the model you have fitted. It is important to note that $0<R^2<1$.
>   **It can never be less than 0**.
>
> $$R^{2} =1- \frac{\sum_{i=1}^n({y_{i}}-\hat{y_i})^2}{\sum_{i=1}^n({y_{i}}-\bar{y_i})^2} = 1- \frac{Sum\,of\,Squares_{Residual}}{Sum\,of\,Squares_{Total}} $$
> Where: - $n$ = sample size - $y_i$ = actual value of $y$ - $\hat{y_i}$
> = predicted value of $y$ - $\bar{y_i}$ = mean of $y$
>
> While in the specific case of 1 continuous vs 1 continuous models you
> can square the $r$ to get the $R^2$, **you may not be able to do so
> for more complex models, so I suggest you wipe your brain of that
> information**!!!!!
>
> In R, you will get 2 $R^2$ values:
>
> 1.  The Multiple $R^2$ value
>
> - Accounts for variance explained by model, even with irrelevant or
>   insignificant variables
>
> 2.  The Adjusted $R^2$ value
>
> - Adjusts the $R^2$ to penalise for insignificant or junk variables

## Context

For this set of dummy data, you have somehow gotten the permits to
collect data on Bull Sharks in Quandamooka for your MBRS project! Your
group is trying to see if you can predict the length of a bull shark
based on its girth.

*For legal reasons this is a totally fictional dataset that I have
generated and in no way were any sharks actually used in the generation
of this data.*

$H_0$: Girth of bull sharks *cannot* be used to predict length if bull
sharks in Quandamooka.

$H_1$: Girth of bull sharks *can* be used to predict length if bull
sharks in Quandamooka.

## Import and Check Data

Let’s import the dataset:

``` r
shark <- read_xlsx(fp, sheet = "linear model")

glimpse(shark)
```

    Rows: 30
    Columns: 2
    $ length_cm <dbl> 315.02, 148.34, 81.70, 92.00, 129.35, 78.63, 328.60, 280.59,…
    $ girth_cm  <dbl> 78.76, 111.25, 61.28, 46.00, 97.01, 58.97, 82.15, 70.15, 146…

> [!NOTE]
>
> ### Assumption of Normality
>
> One of the most important underlying assumptions of statistical tests
> that use means is the normality of the data (if the data is normally
> distributed or not).
>
> We have not tested this in the last few and I will explain why in a
> little bit.
>
> There are several ways to test for normality. Most commonly we use:
>
> 1.  Histograms
>
> - What you are looking for is a big hump in the middle and sides
>   (tails) that taper out nicely
>
> ``` r
> hist(shark$length_cm)
> ```
>
> ![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20hist-1.png)
>
> ``` r
> hist(shark$girth_cm)
> ```
>
> ![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20hist-2.png)
>
> 2.  Shapiro-Wilk Test
>
> - The Shapiro-Wilk Test essentially lines up your data points against
>   a theoretical normal disruptions and tests the hypotheses:
>
> $H_0$: Data is normally distributed
>
> $H_1$: Data is not normally distributed
>
> ``` r
> shapiro.test(shark$length_cm)
> ```
>
>
>         Shapiro-Wilk normality test
>
>     data:  shark$length_cm
>     W = 0.9315, p-value = 0.05384
>
> ``` r
> shapiro.test(shark$girth_cm)
> ```
>
>
>         Shapiro-Wilk normality test
>
>     data:  shark$girth_cm
>     W = 0.94016, p-value = 0.09186
>
> 3.  Quantile-Quantile (Q-Q) Plot
>
> - This essentialy is the Shapiro-Wilk Test visualised. Ideally, it
>   should look like a nice straight diagonal line of points.
>
> ``` r
> qqnorm(shark$length_cm)
> ```
>
> ![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20qq-1.png)
>
> ``` r
> qqnorm(shark$girth_cm)
> ```
>
> ![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20qq-2.png)
>
> So you can see that from eyeballing it, the Histograms are not exactly
> normal, but the Shapiro-Wilk Tests suggest borderline normality.
>
> In practice, it is always good to use different methods to test for
> normality and then decide what to do from there. That is why I have
> shown you both so that you can make a call on whether or not you
> should try transforming the data.
>
> I will choose not to for now in this case.
>
> ### Why didn’t we test normality before?
>
> After a certain amount of samples, we generally assume that the data
> meets an approximately normal distribution because of the Central
> Limit Theorem, which states that *the sampling distribution of a mean
> approaches normal after a certain number of samples, regardless of the
> population size or distribution*.
>
> Generally, we accept the minimum sample size to be 30.

## Analysis

Let’s do a quick linear regression of the data using the `lm()`
function:

``` r
m3 <- lm(length_cm~girth_cm,
         data = shark)

summary(m3)
```


    Call:
    lm(formula = length_cm ~ girth_cm, data = shark)

    Residuals:
       Min     1Q Median     3Q    Max 
    -65.10 -56.91 -30.41  28.93 157.21 

    Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
    (Intercept)  68.0755    24.3840   2.792  0.00934 ** 
    girth_cm      1.2576     0.2445   5.144 1.87e-05 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 69.18 on 28 degrees of freedom
    Multiple R-squared:  0.4858,    Adjusted R-squared:  0.4675 
    F-statistic: 26.46 on 1 and 28 DF,  p-value: 1.873e-05

### Check Residuals

An important assumption for Linear Regressions is to check the
residuals.

We can do this by plotting your model using `plot()`

``` r
par(mfrow = c(2,2)) # split your plotting area into 2 x 2

plot(m3)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20resid%20check-1.png)

``` r
graphics.off() # reset your plotting area
```

Of the 4 plots shown to you, we are going to ignore the bottom 2
(Scale-Location and Residuals vs Leverage) for now and focus on the top
2.

#### Residuals vs Fitted

This plot essentially shows you how far your predicted points are from
the fitted linear line. Ideally, you want to have them look totally
random with no discernible pattern. Analogies I have been told are that
they should look like:

- Stars in the night sky
- Someone threw a bucket of pennies all over the ground

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20resid%20p1-1.png)

The main issue with our residuals now is that they are doing what we
call *“trumpeting”*, where our residuals are concentrated on the left
side and get wider as we move right across the graph.

We will see if we can fix this in a bit

#### Q-Q Residuals

Just like the Q-Q Plot mentioned above, this essentially fits your
residuals to a theoretical normal distribution to provide a visual cue
of how well it fits a normal distribution.

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20resid%20p2-1.png)

Pretty terrible!

### Re-fitting Your Regression

A lot of times, you will end up having to re-fit regressions because
assumptions are not initially met. All part of the process…

In the case of Linear Regressions, you can do this by transforming the
data.

Here is a table of suggestions for transformations you can apply to your
data to potentially help you fit a better model:

| Type of Transformation     | Function  | When To Use It        |
|----------------------------|:---------:|-----------------------|
| Log                        | `log(x)`  | Data is skewed right  |
| Square Root                | `sqrt(x)` | If Log is too strong  |
| Reciprocal ($\frac{1}{x}$) |   `1/x`   | If Log is not enough  |
| Square                     |   `x^2`   | For left skew         |
| Exponential                | `exp(x)`  | For extreme left skew |

Potential types of transformations for your data.

Some times, it takes a while for you to finally correct the model to
your liking. It does take a bit of guessing and checking if you lack
experience in this.

> [!NOTE]
>
> ### Skew
>
> Remember that skew is relative to the tail of your data!!
>
> If your data is right skewed, that means that the tail is on the right
> side of the graph. Left means the tail is on the left side of the
> mean.

### Transformed Regression

So again, the current issue we are facing is that, as girth and length
increases, so does the margin of error. i this case, it is generally
best to transform your `y` (response) variable instead of your `x`.

Let’s create a new column in the shark dataset with a a few
transformation for funsies and see if it improves `m3`:

``` r
shark <- shark %>% 
  mutate(log_len = log(length_cm), # add an log length column
         sqrt_len = sqrt(length_cm)) # sqrt length
```

Try 2 new models:

``` r
par(mfrow = c(2,2)) # split plots into 2 x 2

m3.log <- lm(log_len~girth_cm,
              data = shark)

summary(m3.log)
```


    Call:
    lm(formula = log_len ~ girth_cm, data = shark)

    Residuals:
        Min      1Q  Median      3Q     Max 
    -0.9518 -0.2818 -0.1042  0.1818  0.8342 

    Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
    (Intercept) 4.229923   0.164402  25.729  < 2e-16 ***
    girth_cm    0.008894   0.001649   5.395 9.42e-06 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 0.4664 on 28 degrees of freedom
    Multiple R-squared:  0.5097,    Adjusted R-squared:  0.4922 
    F-statistic: 29.11 on 1 and 28 DF,  p-value: 9.418e-06

``` r
plot(m3.log)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20new%20models-1.png)

``` r
m3.sqrt <- lm(sqrt_len~girth_cm,
              data = shark)

summary(m3.sqrt)
```


    Call:
    lm(formula = sqrt_len ~ girth_cm, data = shark)

    Residuals:
       Min     1Q Median     3Q    Max 
    -3.607 -1.928 -1.051  1.319  5.577 

    Coefficients:
                Estimate Std. Error t value Pr(>|t|)    
    (Intercept) 8.339482   0.948489   8.792 1.52e-09 ***
    girth_cm    0.051255   0.009511   5.389 9.58e-06 ***
    ---
    Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

    Residual standard error: 2.691 on 28 degrees of freedom
    Multiple R-squared:  0.5091,    Adjusted R-squared:  0.4916 
    F-statistic: 29.04 on 1 and 28 DF,  p-value: 9.582e-06

``` r
plot(m3.sqrt)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20new%20models-2.png)

``` r
graphics.off() # reset graphics
```

The $R^2$ increased from 0.4675 to 0.4922 (log model) and 0.4916 (square
root model). In either case, the improvement is not by much (only about
3% improvement) but the residuals have greatly improved.

In this case, it is entirely up to you which model you choose to go with
since the difference is so small, but based on $R^2$, I will use
`m3.log` as our final model.

## Visualisation

Depending on which model you chose in the end, the visualisation would
be different. Let’s go through a few options.

### Raw plot

Here is a visualisation of the raw values of girth and length:

``` r
(
  shark_raw_p <- ggplot(data = shark,
                        aes(x = girth_cm,
                            y = length_cm))+
    geom_point()+ # add points
    geom_smooth(method = "lm")+ # add smoothed linear trend
    labs(x = "Girth (cm)",
         y = "Length (cm)")+
    theme_classic(base_size = 14)
)
```

    `geom_smooth()` using formula = 'y ~ x'

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20raw%20plot-1.png)

### Transformed Plot

All we have to do here is change the `y =` argument in `aes()`

``` r
(
  shark_raw_p <- ggplot(data = shark,
                        aes(x = girth_cm,
                            y = log_len))+
    geom_point()+ # add points
    geom_smooth(method = "lm")+ # add smoothed linear trend
    labs(x = "Girth (cm)",
         y = "log(Length) (cm)")+ # make sure you change the label to reflect this
    theme_classic(base_size = 14)
)
```

    `geom_smooth()` using formula = 'y ~ x'

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/shark%20trf%20plot-1.png)

### `visreg`

`visreg` is another really cool package built to visualise regressions
(who would have guessed…). You can use it for Linear Regressions (this
tutorial) or more advanced models including Generalised Linear Models
(GLMs) and Generalised Linear Mixed Models (GLMMs). You can find out
more [here](https://pbreheny.github.io/visreg/index.html)

While it uses base R to plot, you can output it as a `gg` object to make
it compatible with `ggplot` functions.

``` r
library(visreg)

m3.log_visreg <- visreg(m3.log,
                        xvar = "girth_cm", # note quotation here 
                        scale = "response",# can be either "response" (log_len) or "linear" ("length_cm")
                        xlab = "Girth (cm)",
                        ylab = "log(Length) (cm)",
                        partial = TRUE, # force points
                        #points = list(cex = 1.5, # change point size
                        #              pch = 2), # change point shape
                        gg = TRUE) # make a gg object

(
  m3.log_visreg <- m3.log_visreg+
  theme_classic(base_size = 14) # add theme
  )
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/visreg%20shark-1.png)

> [!WARNING]
>
> ### Confidence Bands
>
> When plotting linear trends lines, always make sure to include
> confidence bands of some sort. In `geom_smooth()` and `visreg()`, the
> default confidence band that is shown in grey is the 95% Confidence
> Interval. Be sure to check this if using a new function!!

# Chi Square Test (Categorical vs. Categorical)

Chi (pronounced “kai”) Square ($\chi^2$) Tests are useful for comparing
categorical (usually count) data.

They come in 2 different forms:

1.  Chi Square Test for Goodness of Fit

- Tests to see if there is an equal chance of the same outcome between
  each group.

2.  Chi Square Test for Independence

- Tests to see if groups are independent of one another (if there are
  real differences in the counts)

Chi Square Tests are a pretty uncommon form of statistical analysis,
because they are very situational, usually for count data if you have
nothing else to examine, and marine science as moved on to more complex
analyses.

Effectively, what you care calculating is:

$$\chi^2 =\sum{\frac{(Observed-Expected)^2}{Expected}}$$

They are relatively easy to understand and interpret.

## Chi Sq Test for Goodness of Fit

### Context

Let’s say we tried to do a project on trying to figure out if Giant
Clams on Research Beach on Heron Island have a preference for
settlement. Do they prefer to face a certain compass direction?

$H_0$ Giant Clams have no settlement preference

$H_1$ Giant Clams have a preference of a certain compass orientation

### Data Import and Wrangling

Chi Square test are the only analysis in R where the wide method of data
entry is preferred over long.

``` r
clam <- read_xlsx(fp, sheet = 4)

glimpse(clam)
```

    Rows: 1
    Columns: 4
    $ North <dbl> 4
    $ South <dbl> 20
    $ East  <dbl> 8
    $ West  <dbl> 15

### Analysis

It is a super simple test:

``` r
chisq.test(clam) # no formula for this, just chuck the dataset in
```


        Chi-squared test for given probabilities

    data:  clam
    X-squared = 13, df = 3, p-value = 0.004637

How a Chi Square Test works is, unless told otherwise, it assumes that
there is an expected equal probability for a sample to belong to each
group that you have presented ($H_0$). In this case, it is telling you
that there is statistical power to conclude that there is an actual
difference in counts between each group.

> [!NOTE]
>
> Unlike an ANOVA, there is no required post-hoc test for most cases. It
> is sufficient to say that the proportions are different and present
> the propotions accordingly.

### Visualisation

This is one of the rare moments where something like a bar plot would be
an appropriate plot.

We will, however, need to do some pivoting of the table into long format
for the sake of `ggplot`:

``` r
clam_long <- clam %>% 
  pivot_longer( # this makes the table long from wide
    cols = 1:4, # columns 1 through to 4
    names_to = "direction", # put compass directions into a column
    values_to = "count") %>% # move the counts into their own column
  mutate(direction = as_factor(direction)) # convert direction to a categorical variable

str(clam_long)
```

    tibble [4 × 2] (S3: tbl_df/tbl/data.frame)
     $ direction: Factor w/ 4 levels "North","South",..: 1 2 3 4
     $ count    : num [1:4] 4 20 8 15

``` r
(
  clam_p <- ggplot(data = clam_long,
                 aes(x = direction,
                     y = count))+
  geom_bar(stat="identity")+
  labs(x = "Compass Direction",
       y = "No. of Clams")+
  theme_classic(base_size = 14)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/clam%20plot-1.png)

## Chi Sq Test for Independence

This is usually used to compare counts from 2 categorical variables to
see if they are independent of one another

### Context

Here, we are trying to see if mangroves and sea grass are utilised as
nursery habitats by the same groups of organisms. Your group has left
Go-Pros out in the mangroves and sea grass to and counted all the
individual organisms who swam into frame.

$H_0$ The same organisms utilise both mangroves and sea grass as nursery
habitats

$H_1$ Different organisms utilise mangroves and sea grass as nursery
habitats

### Data Import and Wrangling

This time, The data is in long format, and we shall read it in that way:

``` r
nursery <- read_xlsx(fp, sheet = 5)

glimpse(nursery)
```

    Rows: 8
    Columns: 3
    $ Habitat <chr> "Mangrove", "Mangrove", "Mangrove", "Mangrove", "Seagrass", "S…
    $ Species <chr> "Snapper", "Whiting", "Prawns", "Stingrays", "Snapper", "Whiti…
    $ Count   <dbl> 35, 50, 13, 1, 3, 21, 147, 7

For analysis, we will have have to make the dataset wide using
`pivot_wider()` and convert it to a contingecy table:

``` r
nurse_cont <- nursery %>% 
  pivot_wider(names_from = "Species", # take names from "Species"
              values_from = "Count") %>%  # take numbers from "Count"
  dplyr::select(!Habitat) %>% # remove the first column here, chisq.test() will only accept numbers
  as.matrix() %>% # convert to a matrix
  as.table() # convert to a table

nurse_cont
```

      Snapper Whiting Prawns Stingrays
    A      35      50     13         1
    B       3      21    147         7

### Analysis

Conduct the Chi Square Test on the contingency table:

``` r
chisq.test(nurse_cont)
```

    Warning in chisq.test(nurse_cont): Chi-squared approximation may be incorrect


        Pearson's Chi-squared test

    data:  nurse_cont
    X-squared = 144.76, df = 3, p-value < 2.2e-16

This shows that there is a difference in the organisms that utilise each
habitat!

### Visualisation

We will take the original data in long format and plot it. This is one
of the times where it is appropriate to use colour!

Firest, wrangle the data slightly:

``` r
nursery <- nursery %>% 
  mutate(Habitat = as_factor(Habitat),
         Species = as_factor(Species))

str(nursery)
```

    tibble [8 × 3] (S3: tbl_df/tbl/data.frame)
     $ Habitat: Factor w/ 2 levels "Mangrove","Seagrass": 1 1 1 1 2 2 2 2
     $ Species: Factor w/ 4 levels "Snapper","Whiting",..: 1 2 3 4 1 2 3 4
     $ Count  : num [1:8] 35 50 13 1 3 21 147 7

You can either facet the plots to show the different make ups, or plot
them together and use colour:

``` r
(
  nurse_p1 <- ggplot(data = nursery,
                 aes(x = Habitat,
                     y = Count,
                     fill = Species))+
  geom_bar(position = "fill", # this makes it proportional
           stat="identity")+
    scale_fill_manual("Species", 
                      values = wes_palette("Darjeeling2"))+
  labs(x = "Habitat",
       y = "Propotion Composition")+
  theme_classic(base_size = 14)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/nursery%20plots-1.png)

``` r
(
  nurse_p2 <- ggplot(data = nursery,
                 aes(x = Species,
                     y = Count))+
  geom_bar(stat="identity")+
  labs(x = "Species",
       y = "Count")+
  theme_classic(base_size = 14)+
    facet_wrap(~Habitat)
)
```

![](02_Data_Wrangling_Simple-Analyses_Visualisation_files/figure-commonmark/nursery%20plots-2.png)

# Parting Words

Well done! you’ve made it to the end of this tutorial! Good luck with
your projects and remember to ask for help from your tutors or academics
if you need it!
