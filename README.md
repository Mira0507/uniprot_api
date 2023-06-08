Using Uniprot API in R
======================

2023-06-07

Mira Sohn

As a bioinformatician, I used to encounter analyses that required me to retrieve a wide variety of protein features based on [Uniprot](https://www.uniprot.org/) IDs. Today, I spent considerable amount of time to figure out how to retrieve protein length in the number of amino acids for my proteins of interest in R. After trying a couple of conventional tools specialized for bioinformatics developed in R, I reached a conclusion that they don't always give identical data to Uniprot, even though my input was Uniprot IDs. I'm assuming that such tools retrieve data from [ensembl](https://useast.ensembl.org/index.html) or [NCBI](https://www.ncbi.nlm.nih.gov/).

Therefore, I decided to take advantage of [API provided by Uniprot](https://www.uniprot.org/help/api). Uniprot provides intruction about how to access to access to the server in terminal or python. Unfortunately, I was in need of it in the middle of my R script. In particular, I had Uniprot IDs of interest, which I wanted to retrieve corresponding protein size.


```r

# Load libraries
library(tidyverse)
library(httr)
library(jsonlite)

```

