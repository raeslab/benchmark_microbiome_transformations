
# Benchmarking microbiome transformations

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://github.com/vllorens/microbiota_simulations/blob/master/LICENSE)
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4719508.svg)](https://doi.org/10.5281/zenodo.4719508)

This repository contains the scripts to generate and analyze the simulations from the paper "Benchmarking microbiome transformations favors experimental quantitative approaches to address compositionality and sampling depth biases". 


## How to use this repository

Clone or download this repository to your local computer. 

The different folders and files contained here are:

* **scripts**: contains the scripts to generate the simulated data and run the analyses included in the paper
* **data/raw**: contains the data object including the raw simulations of the three types of communities, from which, the original matrices are generated
* **R**: contains dditional helper functions, called from the scripts
* **README.md**: this file
* **microbiota_simultions.Rproj**: the R project from which to run all the analyses (for easier portability)
* **.gitignore**: all the files not to include here


## Required software and installation

* **R** (code was developed and tested using R v3.5.1)
* The following **R packages**:

```
## Check for package availability and install

## In R:

packages_cran = c("tidyverse", "ggpubr", "ggtext",
             "rstatix", "ggsignif", "gdata",
             "BiocManager", "RColorBrewer", "devtools", "inlmisc")

packages_bioconductor <- c("phyloseq", "DESeq2", "ALDEx2",
                         "edgeR", "vegan", "metagenomeSeq")
                         
packages_github <- c("ggloor/CoDaSeq/CoDaSeq")

## Install packages

lapply(packages_cran, FUN=function(x){
    if (!requireNamespace(x, quietly = TRUE)){
        install.packages(x)
    }
})

lapply(packages_bioconductor, FUN=function(x){
    if (!requireNamespace(x, quietly = TRUE)){
        BiocManager::install(x, update = F, ask=F)
    }
})

lapply(packages_github, FUN=function(x){
    pkg_name <- strsplit(x, split="/")[[1]]
    pkg_name <- pkg_name[length(pkg_name)]
    if(!requireNamespace(pkg_name, quietly = TRUE)){
        devtools::install_github(x)
    }
})

```


## Steps to reproduce the analyses of the paper

Each section contains the scripts to generate the tables and panels of the figures from the manuscript, as well as additional tables and figures for further exploration.

It is recommended to run the scripts in R by executing the project file *.Rproj


### 1. Generate data

Run the script **generate_data.R**. In this script, first we obtain individual simulated taxonomy matrices. Three types of populations are simulated:

* Healthy, where there is an ecological succession in which higher microbial load correlates to higher richness.
* Dysbiosis, where in a fraction of the samples there is a drop in microbial load. Most taxa decrease or dissapear, some remain stable and others increase with lower cell counts.
* Blooming, where a single taxon blooms and overtakes the community.

From the data object with all matrices, we write txt files with the original matries and we also produce the sequencing output, the RMP matrices, and the QMP matrices. Finally, we also generate metadata for all the original taxonomy matrices, and calculate some basic statistics about the generated matrices.

### 2. Calculate alpha diversity on the simulated matrices

Run the script **alpha_diversity.R**. In this script, we calculate different alpha diversity metrics on the original matrices, as well as on the matrices transformed by the different methods tested.

### 3. Calculate taxon-counts, taxon-taxon and taxon-metadata correlations

Run the script **correlations.R**. In this script, we use different data transformation methods to determine taxon-total count, taxon-taxon and taxon-metadata associations in all our populations. For comparisons, we will only use Spearman correlations for taxon-taxon and either Spearman correlation or Kruskal-wallis test for taxon-metadata associations (Spearman for numeric and Kruskal-Wallis for factors in the metadata)

The following transformations are benchmarked:

Sequencing:
* *Seq*: Sequencing data, none other transformation applied.

Relative transformations:
* *Rel*: Relative data, notmalized scaling to the total sequencing counts.
* *AST*: Sequencing data is normalized using total sum scaling + arcsine squared-root transformation.
* *RMP*: Sequencing data is downsized to achieve even sequencing depth.

Compositional transformations:
* *CLR*: Data normalized using Centered Log Ratio transformation (CoDaSeq implementation)
* *CSS*: Data normalized using cumulative sum scaling (MetagenomeSeq implemetnation).
* *GMPR*: Size factors for library-size based normalization are calculated using the Geometric Mean of Pairwise Ratios (implementation using GMPR package).
* *TMM*: Normalization using trimmed mean of M-values (EdgeR implementation).
* *RLE*: Normalization using relative log-expression (EdgeR implementation).
* *UQ*: Normalization using upper quartile (EdgeR implementation).
* *VST*: Variance stabilizing transformation from DESeq2.

Quantitative (experimentally-derived) transformations:
* *QMP*: Normalized by rarefying to even sampling depth and scaling to total microbial loads (measured experimentally).
* *ACS*: Normalized by scaling to total microbial loads (measured experimentally).

Additionally, we calculate these correlations in the original simulated matrices (*REAL*) for reference.

In all of these methods, we will only evaluate those taxa with prevalence > 50%. We will use the sequencing matrix to determine this threshold on the zeros.


### 4. Comparison QMP vs ACS

Run the script **qmp_vs_acs_dysbiosis.R**. In this script we compare the performance of QMP and ACS in detecting taxa associated to a disease condition that is characterized by a decrease in total microbial loads. 


### 5. Test the effect of the number of samples and sequencing depth

Run the following scripts: 

* **effect_number_samples.R**
* **effect_sequencing_depth.R**

These two scripts repeat the correlation analysis from **section 3**, but screening throughout the different parameters to evaluate their influence on the different transformations. These scripts require generating the data and calculating all of the correlations several times, so they may take a long time. You may consider running them in the command line (on a server).

