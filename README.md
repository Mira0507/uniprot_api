Using Uniprot API in R
======================

2023-06-07

Mira Sohn

As a bioinformatician, I used to encounter analyses that required me to retrieve a wide variety of protein features based on [Uniprot](https://www.uniprot.org/) IDs. Today, I spent a considerable amount of time figuring out how to retrieve the protein length in terms of the number of amino acids for my proteins of interest in R. After trying a couple of conventional tools specialized for bioinformatics developed in R, I came to the conclusion that protein length data provided by [Ensembl](https://useast.ensembl.org/index.html) or [NCBI](https://www.ncbi.nlm.nih.gov/) is not always identical to that from Uniprot. 

Therefore, I decided to take advantage of the [API provided by Uniprot](https://www.uniprot.org/help/api). Uniprot provides intructions on how to access to access the server in terminal or using python. Unfortunately, I needed it in the middle of my R script. Therefore, I decided to write simple codes connecting to the API in R. Here's my recap of how it was solved.

1. Load libraries

This worklow will be taking advantage of [`httr`](https://httr.r-lib.org/index.html).

```r

# Load libraries
library(tidyverse)
library(httr)  # main tool

```

2. Build a function to interatively retrieve data

I'll be demonstrating one record at a time using the function below:

```r

# Create a function to retreive protein length for a uniprot ID via Uniprot API
read_protein_size <- function(id.vector) { 
    
    length.vector <- c()
    for (id in id.vector) {
        print(paste(id, "being retrieved..."))
        # Assign URL
        search_url <- paste0("https://rest.uniprot.org/uniprotkb/search?query=",
                             id,
                             "&format=tsv&fields=accession,length")
        # Send a GET request to the API
        response <- GET(search_url)

        # Check if the request was successful
        if (http_status(response)$category == "Success") {
            # Parse the response body
            content <- content(response, "text", encoding="UTF-8")

            # Read the TSV data into a data frame
            Length <- read_tsv(content)$Length

            # Print the protein length
            print(paste("The length of the protein with UniProt ID", id, "is", Length, "amino acids."))
        } else {
            print("Failed to retrieve data")
            Length <- NA
        }
        length.vector <- c(length.vector, Length)
    }
    return(length.vector)
}

```

The key steps were

- establishing URL (`search_url`)
- querying data (`GET(search_url)`)
- retrieving data (`content(response, "text", encoding="UTF-8")`)





