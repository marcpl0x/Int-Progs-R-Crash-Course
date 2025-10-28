03_Multivariate
================
Marc Koh
2025-10-14

# Multivariate Analyses

This Markdown will give you a quick tutorial to follow for multivariate
analyses.

# Introduction

So far, you have done some basic data analyses in the form of:

- t-tests
- ANOVAs
- Linear Models
- Generalised Linear Models

These are known as *univariate* analyses, where you have one response
variable and one or several independent variables:

$$Y \sim X_{1} + X_{2} * X_{3}+ \cdots+ X_{n}$$

As you probably know by now, this is usually done to address hypotheses
or research questions where you are looking at how a specific singular
variable changes with the rest of your predictors.

Ecological data is a complex thing, where in certain scenarios, you want
to tease out more information, or look at slightly more complex
relationships.

For example, using something like a Simpsons Diversity Index is a pretty
common way to compare species diversities between locations.

Although a totally acceptable way to assess diversity, there are better
alternatives to assess this now that the average computer has much more
more processing power.

## What Is Multivariate Statistics?

If univariate statistics has 1 response variable (uni = 1, duh),
multivariate statistics has **multiple** response variables:

$$Y_{1} + Y_{2} + Y_{3}+ \cdots+ Y_{n}\sim X_{1} + X_{2} * X_{3}$$ In
the example of species diversity, $Y_{n}$ represents species that were
recorded in your dataset, where you are looking at how certain
environmental variables could affect changes in species abundance:

$$Sp_{1}+Sp_{2}+Sp_{3}+....Sp_{n} \sim Site + Salinity +Temperature$$

## Where do I Begin? (Workflow)

If you’re not familiar with data analysis, my suggested general workflow
is that you:

<div class="alert alert-info">

1.  Clean and wrangle your data

2.  Visualise your data

3.  Conduct analyses

4.  Interpret your analyses

</div>

This workflow is what I generally apply for most data analyses.

The next section will give you an example of this using dummy
foraminifera data collected on Heron Island.

# Multivariate Analyses Example

This example will run through the data analysis from importing, cleaning
and wrangling, visualisation.

More specifically, we will be doing a **Non-Metric Multidimensional
Scaling (NMDS)** and a **Permutational Multivariate Analysis of Variance
(PERMANOVA)**.

If you have done a Principle Component Analysis (PCA) before, an NMDS is
a similar technique that uses distances from a dissimilarity calculation
instead of eigenvalues which PCAs use.

The research question that is trying to be answered here is:

<div class="alert alert-info">

<strong>What is the distribution of Forams on Heron Reef?</strong>

</div>

Specifically, this group tried to look for differences in 3 locations,
the Reef Crest, Lagoon, and Flat, and at several different sediment size
classes to see if Forams had a settlement preferences in terms of
location or sediment size.

For the sake of this example, we will just look at how
<p style="color:red">

location affects species abundance
</p>

Let’s begin…

## Packages

Packages you will specifically need to follow along with this are:

``` r
library(readxl) # for reading excel into R
library(tidyverse) # for wrangling and cleaning
library(vegan) # package designed specifically for community ecology
library(ggordiplots) # ggplot version for multivariate visualisation
library(MetBrewer) # MET colour palette
library(pairwiseAdonis) # for pairwise testing
```

## Load Data In

Let’s load this dummy dataset in to see what it looks like:

``` r
foram <- read_xlsx("Dummy_foram.xlsx",
                   sheet = 1) # you can choose which sheet to import
```

Have a look at what it looks like:

``` r
head(foram)
```

    ## # A tibble: 6 × 14
    ##   Location `Sample number` `Sediment Size` Latitude Longitude `No Forams`  Star
    ##   <chr>              <dbl>           <dbl>    <dbl>     <dbl>       <dbl> <dbl>
    ## 1 Crest                  1             125    -23.5      152.         196    NA
    ## 2 Crest                  1             250    -23.5      152.         191     4
    ## 3 Crest                  1             500    -23.5      152.         167    29
    ## 4 Crest                  2             125    -23.5      152.         197    NA
    ## 5 Crest                  2             250    -23.5      152.         197     3
    ## 6 Crest                  2             500    -23.5      152.         153    47
    ## # ℹ 7 more variables: `smooth spiral` <dbl>, `rough spiral` <dbl>,
    ## #   `rice-grain` <dbl>, `rounded flat` <dbl>, bell <dbl>, bumpy <dbl>,
    ## #   round <dbl>

Some things to note here:

1.  The first 6 columns are abiotic variables, everything after is the
    abundance of **morphospecies**

2.  Some of these columns are categorical variables, which we will have
    to make sure we tell R later.

3.  There are a lot of NAs in the dataset, which probably should have
    been filled with 0s.

## Data Wrangling

Let’s get rid of the NAs and make some of the environmental variables
categoridcal:

``` r
foram <- foram %>% replace(is.na(.), 0) # replace NAs with 0s

foram <- foram %>% mutate(Location = as_factor(Location),
                          `Sample number` = as_factor(`Sample number`),
                          `Sediment Size` = as_factor(`Sediment Size`))

str(foram) # check structure
```

    ## tibble [45 × 14] (S3: tbl_df/tbl/data.frame)
    ##  $ Location     : Factor w/ 3 levels "Crest","Lagoon",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ Sample number: Factor w/ 5 levels "1","2","3","4",..: 1 1 1 2 2 2 3 3 3 4 ...
    ##  $ Sediment Size: Factor w/ 3 levels "125","250","500": 1 2 3 1 2 3 1 2 3 1 ...
    ##  $ Latitude     : num [1:45] -23.5 -23.5 -23.5 -23.5 -23.5 ...
    ##  $ Longitude    : num [1:45] 152 152 152 152 152 ...
    ##  $ No Forams    : num [1:45] 196 191 167 197 197 153 195 190 166 192 ...
    ##  $ Star         : num [1:45] 0 4 29 0 3 47 0 2 33 1 ...
    ##  $ smooth spiral: num [1:45] 1 2 0 0 0 0 0 4 0 0 ...
    ##  $ rough spiral : num [1:45] 0 0 1 0 0 0 0 0 0 0 ...
    ##  $ rice-grain   : num [1:45] 0 2 0 3 0 0 4 1 0 7 ...
    ##  $ rounded flat : num [1:45] 1 1 2 0 0 0 0 3 0 0 ...
    ##  $ bell         : num [1:45] 1 0 1 0 0 0 1 0 0 0 ...
    ##  $ bumpy        : num [1:45] 1 0 0 0 0 0 0 0 1 0 ...
    ##  $ round        : num [1:45] 0 0 0 0 0 0 0 0 0 0 ...

All sorted!

## Visualisation (NMDS)

Recall the sample equation above:

$$Sp_{1}+Sp_{2}+Sp_{3}+....Sp_{n} \sim Site + Salinity +Temperature$$

If we re-purpose it for our data and research question, it would look
something like this: $$Sp_{1}+Sp_{2}+Sp_{3}+....Sp_{n} \sim Location$$

However, since we have multiple rows of data, a more accurate way to
visualise this is using a table, or matrix:

Where, each observation in the matrix is an abundance count per species
per collection.

Therefore, to get this matrix, we need to pull out the species abundance
data into what we call a **community matrix**

### Create Community Matrix

This is fairly simple, all you have to do is pull out the data from
column 7 onwards:

``` r
for_com <- foram %>% 
  select(`No Forams`:ncol(foram)) #

str(for_com)
```

    ## tibble [45 × 9] (S3: tbl_df/tbl/data.frame)
    ##  $ No Forams    : num [1:45] 196 191 167 197 197 153 195 190 166 192 ...
    ##  $ Star         : num [1:45] 0 4 29 0 3 47 0 2 33 1 ...
    ##  $ smooth spiral: num [1:45] 1 2 0 0 0 0 0 4 0 0 ...
    ##  $ rough spiral : num [1:45] 0 0 1 0 0 0 0 0 0 0 ...
    ##  $ rice-grain   : num [1:45] 0 2 0 3 0 0 4 1 0 7 ...
    ##  $ rounded flat : num [1:45] 1 1 2 0 0 0 0 3 0 0 ...
    ##  $ bell         : num [1:45] 1 0 1 0 0 0 1 0 0 0 ...
    ##  $ bumpy        : num [1:45] 1 0 0 0 0 0 0 0 1 0 ...
    ##  $ round        : num [1:45] 0 0 0 0 0 0 0 0 0 0 ...

Next, we have to check the range of values to make sure that we don’t an
over representation of species (e.g. a data point where Sp1 is 10000000
while everything is between 1 and 10)

``` r
range(for_com) # Check range
```

    ## [1]   0 200

**It is important that you check the difference in orders of magnitude
here because an over-representation of a species may mask any
differences between data points caused by under-represented species.**

Although this is not too bad, I will apply a square root to the
community matrix to show you how to reduce the orders of magnitude:

``` r
for_com <- sqrt(for_com)
range(for_com) # check again
```

    ## [1]  0.00000 14.14214

Depending on the orders of magnitude of difference between your
abundances, you may end up having to apply more aggressive
transformations like:

1.  Log-transformation `log()`

2.  Fourth-root transformations `sqrt(sqrt())`

<div class="alert alert-info">

<strong>Warning!</strong> There is a trade off between over-transforming
your data and something we call stress. I will explain this in a bit…

</div>

### NMDS

An NMDS is a visualisation technique used to visualise the community
matrix.

In simpler terms, a matrix allows for us to look at data from different
angles to find similarities between each data point. This means that
there are several different dimensions (or angles) in which we can
interpret the matrix. The NMDS uses what is called distance-based
ordination to find the best 2 to 3 dimensions in which the data can be
best visualised.

<figure>
<img src="Kamp_etal2020_nmds.jpg"
alt="An example of an NMDS plot from Kamp et al. 2020 with 3 Dimensions" />
<figcaption aria-hidden="true">An example of an NMDS plot from Kamp et
al. 2020 with 3 Dimensions</figcaption>
</figure>

Each row of data is represented as one point in these plots.

#### Stress

One of the things you will have to report is stress.

Stress is a measure of goodness-of-fit of the matrix to a 2 dimensional
or 3 dimensional space. According to [original paper proposing
NMDS](https://link.springer.com/article/10.1007/BF02289565) by Kruskal,
the ideal level of stress should be **\< 0.1** and **anything above 0.2
should be interpreted with caution**. A few reasons why your stress may
be above 0.2 include:

- Extremely large datasets

- Over-transforming your data

Although 0.2 is what most people agree on, there may be times in future
it is impossible for you to reduce it below 0.2. There are other
techniques around this including producing a histogram of Permutation
Based Ordination of your data to show that it already is the best case
scenario.

Let’s try to get an NMDS from the foram data:

``` r
for_com.nmds <- metaMDS(for_com,
                        autotransform = F,
                        distance = "bray") # Bray Curtis Dissmiliarity Index
```

    ## Run 0 stress 0.1290251 
    ## Run 1 stress 0.1280097 
    ## ... New best solution
    ## ... Procrustes: rmse 0.03253447  max resid 0.1949358 
    ## Run 2 stress 0.1405792 
    ## Run 3 stress 0.131517 
    ## Run 4 stress 0.130007 
    ## Run 5 stress 0.1373431 
    ## Run 6 stress 0.1352423 
    ## Run 7 stress 0.1341896 
    ## Run 8 stress 0.1319916 
    ## Run 9 stress 0.1347564 
    ## Run 10 stress 0.1280584 
    ## ... Procrustes: rmse 0.04590416  max resid 0.1991473 
    ## Run 11 stress 0.1374323 
    ## Run 12 stress 0.129178 
    ## Run 13 stress 0.1322126 
    ## Run 14 stress 0.1268539 
    ## ... New best solution
    ## ... Procrustes: rmse 0.05569676  max resid 0.1959758 
    ## Run 15 stress 0.1364703 
    ## Run 16 stress 0.1280592 
    ## Run 17 stress 0.1322071 
    ## Run 18 stress 0.1374322 
    ## Run 19 stress 0.1446407 
    ## Run 20 stress 0.1360949 
    ## *** Best solution was not repeated -- monoMDS stopping criteria:
    ##      1: no. of iterations >= maxit
    ##     19: stress ratio > sratmax

``` r
signif(for_com.nmds$stress, digits = 4) # find stress
```

    ## [1] 0.1269

We have a stress value of $0.1269$, which is not too bad!

Now, lets try to visualise this in a 2D space:

``` r
theme_set(theme_bw())

for_com.loc.plot <- gg_ordiplot(for_com.nmds,
                                 groups = foram$Location,
                                 ellipse = T, 
                                 kind = "se", # show ellipses for standard error of mean
                                 plot = F)

for_com.loc.plot <- for_com.loc.plot$plot+
  geom_point(data = for_com.loc.plot$df_mean.ord,
             aes(x = x, y = y,  colour = Group),
             shape = 25,
             size = 3)+ 
#  geom_text(data = for_com.loc.plot$df_mean.ord,
#            aes(x =x, y = y, label = Group),
#            size = 3)+
  scale_colour_manual("Location", # manual colour palette change
                      values = met.brewer("Kandinsky"), 
                      breaks = c("Crest", "Lagoon", "Flat"))+
  annotate("label", x = 0.5, y = -0.15, 
           label = bquote("stress = 0.1269")) # you may have to move this in your own plot

for_com.loc.plot
```

![](03_Multivariate_files/figure-gfm/nmds%20Plot-1.png)<!-- -->

Well done, you’ve made your first NMDS!

You can make it better by adding Species Scores to this:

``` r
data.scores <- as_tibble(scores(for_com.nmds$species))
species.scores <- as.data.frame(scores(for_com.nmds, "species"))
species.scores$species <- rownames(species.scores)
species.scores <- species.scores[1:8,] # remove round, too far away

for_com.loc.plot + 
  geom_text(data = species.scores,
            aes(x = NMDS1,y = NMDS2,label = species), 
            alpha=1) +  # add the species labels
  coord_equal()
```

    ## Coordinate system already present. Adding new coordinate system, which will
    ## replace the existing one.

![](03_Multivariate_files/figure-gfm/species%20scores-1.png)<!-- -->

What the Species Scores does is it imposes the species onto the NMDS
plot, meaning that the closer a data point is to the species name, the
higher the amount of that species in that sample.

## Statistical Analysis

Recall that an NMDS is not really an analysis, and more of a
visualisation. Although the NMDS above shows you the potential
differences between locations, you still haven’t really tested it with a
statisical test yet.

This is where the PERMANOVA comes in.

A PERMANOVA is essentially an ANOVA with permutations for the
multivariate data, meaning that is repeated several times based on the
ordination (from mutliple dimensions) and the best result is presented.
This is one in action:

``` r
set.seed(6971)

for_permanova <- adonis2(for_com ~ Location,
                         data = foram) 

for_permanova
```

    ## Permutation test for adonis under reduced model
    ## Terms added sequentially (first to last)
    ## Permutation: free
    ## Number of permutations: 999
    ## 
    ## adonis2(formula = for_com ~ Location, data = foram)
    ##          Df SumOfSqs      R2      F Pr(>F)    
    ## Location  2  0.16833 0.20416 5.3871  0.001 ***
    ## Residual 42  0.65619 0.79584                  
    ## Total    44  0.82452 1.00000                  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1

The PERMANOVA here essentially shows that about 20.4% of variation can
of the mean species composition can be attributed to the variable of
`Location`.

Remember that for an ANOVA, you will have to do a Tukey Test for Honest
Significant Difference. PERMANOVAs are similiar, in that you can do a
pairwise test to show this using the `pairwise.adnois` package:

``` r
# For if you have a second level:
#foram_df <- as.data.frame(foram) %>% 
#  rename(sed_size = `Sediment Size`)

for_pair <- pairwise.adonis2(for_com ~ Location, 
                             #strata = 'sed_size', # second level
                             data = foram)

for_pair
```

    ## $parent_call
    ## [1] "for_com ~ Location , strata = Null , permutations 999"
    ## 
    ## $Crest_vs_Lagoon
    ##          Df SumOfSqs      R2      F Pr(>F)    
    ## Location  1  0.14657 0.23857 8.7728  0.001 ***
    ## Residual 28  0.46781 0.76143                  
    ## Total    29  0.61438 1.00000                  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## $Crest_vs_Flat
    ##          Df SumOfSqs      R2      F Pr(>F)
    ## Location  1  0.03359 0.05594 1.6593  0.178
    ## Residual 28  0.56689 0.94406              
    ## Total    29  0.60049 1.00000              
    ## 
    ## $Lagoon_vs_Flat
    ##          Df SumOfSqs      R2      F Pr(>F)    
    ## Location  1  0.07233 0.20666 7.2937  0.001 ***
    ## Residual 28  0.27768 0.79334                  
    ## Total    29  0.35002 1.00000                  
    ## ---
    ## Signif. codes:  0 '***' 0.001 '**' 0.01 '*' 0.05 '.' 0.1 ' ' 1
    ## 
    ## attr(,"class")
    ## [1] "pwadstrata" "list"

What you are seeing here is essentially there is a real difference in
species composition between the Flat and the Lagoon, as well as the
Crest and the Lagoon, while there is no real difference between the
Crest and Flat.

This means that the Lagoon has a unique composition of species compared
to the other 2 locations!

<div class="alert alert-info">

<strong>Warning!</strong> Because this method is permutation based, the
results may not always be the same every time you run it. This is why
you set a seed using the `set.seed()` function so that your results can
be replicated.

</div>
