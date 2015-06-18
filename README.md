<!-- README.md is generated from README.Rmd. Please edit that file -->
ddpcr: Analysis and visualization of Digital Droplet PCR data in R and on the web
=================================================================================

This package provides an interface to explore, analyze, and visualize droplet digital PCR (ddPCR) data in R. An interactive tool was also created and is available online to facilitate this analysis for anyone who is not comfortable with using R.

Background
----------

Droplet Digital PCR (ddPCR) is a technology provided by Bio-Rad for performing digital PCR. The basic workflow of ddPCR involves three main steps: partitioning sample DNA into 20,000 droplets, PCR amplifying the nucleic acid in each droplet, and finally passing the droplets through a reader that detects flurescent intensities in two different wavelengths corresponding to FAM and HEX dyes. As a result, the data obtained from a ddPCR experiment can be visualized as a 2D scatterplot (one dimension is FAM intensity and the other dimension is HEX intensity) with 20,000 points (each droplet represents a point).

ddPCR experiments can be defined as singleplex, duplex, or multiplex depending on the number of dyes used (one, two, and more than two, respectively). A duplex experiment typically uses one FAM dye and one HEX dye, and consequently the droplets will be grouped into one of four clusters: double-negative (not emitting fluorescence from either dye), HEX-positive, FAM-positive, or double-positive. When plotting the droplets, each quadrant of the plot corresponds to a cluster; for example, the droplets in the lower-left quadrant are the double-negative droplets.

After running a ddPCR experiment, a key step in the analysis is gating the droplets to determine how many droplets belong to each cluster. Bio-Rad provides an analysis software called QuantaSoft which can be used to perform gating. QuantaSoft can either do the gating automatically or allow the user to set the gates manually. Most ddPCR users currently gate their data manually because QuantaSoft's automatic gating often does a poor job and **there are no other tools available for gating ddPCR data**.

Overview
--------

The `ddpcr` package allows you to upload ddPCR data, perform some basic analysis, explore characteristic of the data, and create customizable figures of the data.

### Main features

The main features include:

-   **Identify failed wells** - determining which wells in the plate seemed to have failed the ddPCR experiment, and thus these wells will be excluded from all downstream analysis. No template control (NTC) will be deemed as failures by this tool.
-   **Identify outlier droplets** - sometimes a few droplets can have an extremely high fluorescent intensity value that is probably erroneous, perhaps as a result of an error with the fluorescent reader. These droplets are identified and removed from the downstream analysis.
-   **Identify empty droplets** - droplets with very low fluorescent emissions are considered empty and are removed from the downstream analysis. Removing these droplets is beneficial for two reasons: 1. the size of the data is greatly reduced, which means the computations will be faster on the remaining droplets, and 2. the real signal of interest is in the non-empty droplets, and empty droplets can be regarded as noise.
-   **Calculating template concentration** - after knowing how many empty droplets are in each well, the template concentration in each well can be calculated.
-   **Gating droplets** - if your experiment matches some criteria (more on that soon), then automatic gating can take place; otherwise, you can gate the data manually just like on QuantaSoft.
-   **Explore results** - the results from each well (\# of drops, \# of outliers, \# of empty drops, concentration) can be explored as a histogram or boxplot to see the distribution of all wells in the plate.
-   **Plot** - you can plot the data in the plate with many customizable features.

### Supported experiment types

While this tool was originally developed to automatically gate data for a particular ddPCR assay (the paper for that experiment is in progress), any assay with similar characteristics can also use this tool to automatically gate the droplets. In order to benefit from the full automatic analysis, your ddPCR experiment needs to have these characteristics:

-   The experiment is a duplex ddPCR experiment
-   The majority of droplets are empty (double-negative)
-   The majority of non-empty droplets are double-positive
-   There can be a third cluster of either FAM+ or HEX+ droplets

In other words, the built-in automatic gating will work when there are three clusters of droplets: (1) double-negative, (2) double-positive, and (3) either FAM+ or HEX+. These types of experiments will be referred to as **(FAM+)/(FAM+HEX+)** or **(HEX+)/(FAM+HEX+)**. Both of these experiment types fall under the name of **PNPP experiments**; PNPP is short for PositiveNegative/PositivePositive, which is a reflection of the droplet clusters. Here is what a typical well from a PNPP experiment looks like:

[![Supported experiment types](vignettes/figures/supported-exp-types.png)](vignettes/figures/supported-exp-types.png)

If your experiment matches the criteria for a **PNPP** experiment (either a **(FAM+)/(FAM+HEX+)** or a **(HEX+)/(FAM+HEX+)** experiment), then after calculating empty droplets the program will analyze the rest of the droplets and assign each droplet one of the following three clustes: FAM+ (or HEX+), FAM+HEX+, or rain. Here is the result of analyzing a single well from a **(FAM+)/(FAM+HEX+)** experiment:

[![Analyze result](vignettes/figures/ppnp-simple-result.png)](vignettes/figures/ppnp-simple-result.png)

If your ddPCR experiment is not a **PNPPP** type, you can still use this tool for the rest of the analysis, exploration, and plotting, but it will not benefit from the automatic gating. However, `ddpcr` is built to be easily extensible, which means that you can add your own experiment "type". Custom experiment types need to define their own method for gating the droplets in a well, and then they can be used in the same way as the built-in experiment types.

Analysis using the interactive tool
-----------------------------------

If you're not comfortable using R and would like to use a visual tool that requires no programming, you can [use the tool online](TODO). If you do know how to run R, using the interactive tool (built with [shiny](http://shiny.rstudio.com/))

Analysis using R
----------------

Enough talking, time for action!

First, install `ddpcr`

    devtools::install_github("daattali/ddpcr")

### Quick start

Here are two basic examples of how to use `ddpcr` to analyze and plot your ddPCR data. One example shows an analysis where the gating thresholds are manually set, and the other example uses the automated analysis. Note how `ddpcr` is designed to play nicely with the [magrittr pipe](https://github.com/smbache/magrittr) `%>%` for easier pipeline workflows. Explanation will follow, these are just here as a spoiler.

``` r
library(ddpcr)
dir <- system.file("sample_data", "small", package = "ddpcr")

# example 1: manually set thresholds
new_plate(dir, type = CROSSHAIR_THRESHOLDS) %>%
  subset("B01,B06") %>%
  set_thresholds(c(5000, 7500)) %>%
  analyze %>%
  plot(show_grid_labels = TRUE, title = "Ex 1 - manually set gating thresholds")

# example 2: automatic gating
new_plate(dir, type = FAM_POSITIVE_PPNP) %>%
  subset("B01:B06") %>%
  analyze %>%
  plot(show_mutant_freq = FALSE, show_grid_labels = TRUE, title = "Ex 2 - automatic gating")
```

<img src="vignettes/README-unnamed-chunk-2-1.png" title="" alt="" width="50%" /><img src="vignettes/README-unnamed-chunk-2-2.png" title="" alt="" width="50%" />

### Loading ddPCR data

The first step is to get the ddPCR data into R. `ddpcr` uses the data files that are exported by QuantaSoft as its input. You need to have all the well files for the wells you want to analyze (one file per well), and you can optionally add the results file from QuantaSoft. If you loaded an experiment named *2015-05-20\_mouse* with 50 wells to QuantaSoft, then QuantaSoft will export the following files:

-   50 data files (well files): each well will have its own file with the name ending in \*\_Amplitude.csv". For example, the droplets in well A01 will be saved in *2015-05-20\_mouse\_A01\_Aamplitude.csv*
-   1 results file: a small file named *2015-05-20\_mouse.csv* will be generated with some information about the plate, including the name of the sample in each well (assuming you named the samples previously)

The well files are the only required input to `ddpcr`, and since ddPCR plates contain 96 wells, you can upload anywhere from 1 to 96 well files. The results file is not mandatory, but if you don't provide it then the wells will not have sample names attached to them.

`ddpcr` contains a sample dataset called *small* that has 5 wells. We use the `new_plate()` function to initialize a new ddPCR plate object. If given a directory, it will automatically find all the valid well files in the directory and attempt to find a matching results file.

``` r
library(ddpcr)
dir <- system.file("sample_data", "small", package = "ddpcr")
plate <- new_plate(dir)
#> Reading data files into plate... DONE (0 seconds)
#> Initializing plate of type `ddpcr_plate`... DONE (0 seconds)
```

You will see some messages appear - every time `ddpcr` runs an analysis step (initializing the plate is part of the analysis), it will output a message decribing what it's doing.

### Explore the data

We can explore the data we loaded even before doing any analysis

``` r
plate
#> ddpcr plate
#> -----------
#> Dataset name: small
#> Plate type: ddpcr_plate
#> Data summary: 5 wells; 76,143 drops
#> Completed analysis steps: INITIALIZE
#> Remaining analysis steps: REMOVE_FAILURES, REMOVE_OUTLIERS, REMOVE_EMPTY
```

Among other things, this tells us how many wells and total droplets we have in the data, and what step of the analysis we are at. , so if we want to find out what wells wells are in the data we can use

``` r
plate %>% wells_used
#> [1] "B01" "B06" "C01" "C06" "C09"
```

Or to see all the data and the results so far, we can use the `plate_data()` and `plate_meta()` functions

``` r
plate %>% plate_data
#> Source: local data frame [76,143 x 4]
#> 
#>    well  HEX  FAM cluster
#> 1   B01 1374 1013       1
#> 2   B01 1411 1018       1
#> 3   B01 1428 1024       1
#> 4   B01 1313 1026       1
#> 5   B01 1362 1027       1
#> 6   B01 1290 1028       1
#> 7   B01 1319 1030       1
#> 8   B01 1492 1032       1
#> 9   B01 1312 1036       1
#> 10  B01 1294 1037       1
#> ..  ...  ...  ...     ...
plate %>% plate_meta(only_used = TRUE)
#>   well sample row col used drops
#> 1  B01     #1   B   1 TRUE 17458
#> 2  B06     #9   B   6 TRUE 13655
#> 3  C01     #3   C   1 TRUE 15279
#> 4  C06    #12   C   6 TRUE 14513
#> 5  C09    #30   C   9 TRUE 15238
```

Notice that *meta* (short for *metadata*) is synonymous with *results* for our purposes.

### Subset the plate

If you aren't interested in all the wells, you can use the `subset()` function to retain only certain wells. Alternatively, you can use the `data_files` argument of the `new_plate()` function to only load certain well files instead of a full directory.

subset clusters type
