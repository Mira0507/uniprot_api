Using Uniprot API in R
======================

2023-06-07

Mira Sohn

As a bioinformatician, I used to encounter analyses requiring a wide variety of protein features based on [Uniprot](https://www.uniprot.org/) IDs. Today, I spent a considerable amount of time figuring out how to retrieve the protein length in terms of the number of amino acids for my proteins of interest in R. After trying a couple of conventional tools developed to retrieve data from [Ensembl](https://useast.ensembl.org/index.html), [NCBI](https://www.ncbi.nlm.nih.gov/), or Uniprot in R, I noticed that the data was not exactly the same as what is currently provided on the Uniprot webpages depending on protein. 

Therefore, I decided to take advantage of the [API provided by Uniprot](https://www.uniprot.org/help/api). Uniprot provides intructions on how to access the server in terminal or using python. Unfortunately, my analysis was performed in R script. Instead of going back and forth to do that in terminal or python, I wrote simple codes connecting to the API in R. Here's what I did.

## 1. Load libraries

This worklow is designed to use the package [`httr`](https://httr.r-lib.org/index.html) in R.

```r

# Load libraries
library(tidyverse)
library(httr)  # main tool

```

## 2. Iteratively retrieve data using a function

I wrote a function to retrieve one record at a time here:

```r

# Create a function to retreive protein length for a uniprot ID via Uniprot API
# (Take a character vector consisting of Uniprot IDs as input)
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

### a. Establishing URL

The URL includes info about what API you're connecting to, your Uniprot IDs of interest, data format to be retrieved, and data fields of interest. If you're looking for data about the Uniprot ID `"P21447"`, the URL would be `"https://rest.uniprot.org/uniprotkb/search?query=P21447&format=tsv&fields=accession,length"` where the `query`, `format`, and `fields` indicate Uniprot ID, output format, and data column to be retrieved, respectively. You can manipulate the string to retrieve different data format and fields. Visit [this instruction](https://www.uniprot.org/help/api_queries) for more details.

### b. Querying data

Once querying data, you can check the `response` about how your communication went.

```r

# Print your `response`
> response
Response [https://rest.uniprot.org/uniprotkb/search?query=P21447&format=tsv&fields=accession,length]
  Date: 2023-06-11 03:22
  Status: 200    # status 200 means it was succussful
  Content-Type: text/plain; format=tsv
  Size: 25 B
Entry   Length
P21447  1276

# Print your status info
> http_status(response)
$category
[1] "Success"
$reason
[1] "OK"
$message
[1] "Success: (200) OK"

```

You get 200 as a proof of succussful communication. Check the ["Quickstart guide"](https://cran.r-project.org/web/packages/httr/vignettes/quickstart.html) of the `httr` package for more info.

### c. Retrieving data

If your communication suceeded (`if (http_status(response)$category == "Success")`), you can extract the content (`content()`). The content returns a string, which can build a tabular format, since set your format to `tsv` (tab-separated values).

```r
# Print the `content`
> content
[1] "Entry\tLength\nP21447\t1276\n"
```

### d. Cleaning retrieved data

You can convert the string to a data frame using the function `read_tsv()` (or using any other preferred functions), as shown below:

```r
# Read the `content`
> read_tsv(content)
Rows: 1 Columns: 2
── Column specification ───────────────────────────────────────────────────────────────────────────────────────────────
Delimiter: "\t"
chr (1): Entry
dbl (1): Length

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
# A tibble: 1 × 2
  Entry  Length
  <chr>   <dbl>
1 P21447   1276

```

Instead of building a data frame, I directly pulled the column `Length`, as demonstrated here:


```r
# Save length as a numeric vector by pulling the column `Length`
Length <- read_tsv(content)$Length
```

Afterwards, every pulled length was added to the vector `length.vector`. If your API doesn't respond to a query, an `NA` is returned and added to the `length.vector`.

## 3. In practice

How do take advantage of this in real world analysis? One straightforward way would be to use R data frame. Assume you have a data frame with a column storing Uniprot IDs of interest, as shown below:

```r

# Explore your input data frame
> head(df)
  Uniprot
1  Q3UHJ0
2  P21447
3  P55096
4  Q6P542
5  Q99LR1
6  Q5SSL4

> nrow(df)
[1] 50

```

The function `read_protein_size()` returning a numeric vector for protein length could be used to add a new column for protein size corresponding to each Uniprot.

```r

# Add a column to save protein length by retrieving data using Uniprot API
df$Length <- read_protein_size(df$Uniprot)

# Explore your updated data frame
> head(df)
  Uniprot Length
1  Q3UHJ0    959
2  P21447   1276
3  P55096    659
4  Q6P542    837
5  Q99LR1    398
6  Q5SSL4    859
```

Note that it's possible to get missing values (`NA`s) in the data frame due to failed API communications.

```r
# Count the number of missing values in the `df`
> sum(is.na(df))
[1] 0
```

My analysis resulted in finding one missing value out of 2795 Uniprot IDs. It was "Q80TK0", which has been merged into "D3YUB6" according to the Uniprot. You can make a decision about how to impute this missing value, which is out of scope in this demonstration.

I've created [a demo script](https://github.com/Mira0507/uniprot_api/blob/main/api_demo.Rmd) in R along with [an input file](https://github.com/Mira0507/uniprot_api/blob/main/uniprot_input_demo.txt). Hope they're useful for someone who are interested in this demonstration.
