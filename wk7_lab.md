pm566_wk7_lab
================
Yiping Li
2022-10-07

Question 1: How many sars-cov-2 papers? Build an automatic counter of
sars-cov-2 papers using PubMed. You will need to apply XPath as we did
during the lecture to extract the number of results returned by PubMed
in the following web address:
<https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2>

``` r
# Downloading the website
website <- xml2::read_html("https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]/div[1]")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
stringr::str_extract(counts, "[0-9,+]")
```

    ## [1] "1"

``` r
stringr::str_extract(counts, "[0-9,]+")
```

    ## [1] "179,200"

Question 2: Academic publications on COVID19 and Hawaii You need to
query the following The parameters passed to the query are documented
here.

Use the function httr::GET() to make the following query:

Baseline URL:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi>

Query parameters:

db: pubmed term: covid19 hawaii retmax: 1000

``` r
library(httr)
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(db='pubmed',
               term = 'covid19 hawaii',
               retmax= 1000)
)

# Extracting the content of the response of GET
ids <- httr::content(query_ids)
```

Question 3: Get details about the articles The Ids are wrapped around
text in the following way: <Id>… id number …</Id>. we can use a regular
expression that extract that information. Fill out the following lines
of code:

``` r
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[[:digit:]]</Id")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "</?Id>")
```

With the ids in hand, we can now try to get the abstracts of the papers.
As before, we will need to coerce the contents (results) to a list
using:

Baseline url:
<https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi>

Query parameters:

db: pubmed id: A character with all the ids separated by comma, e.g.,
“1232131,546464,13131” retmax: 1000 rettype: abstract Pro-tip: If you
want GET() to take some element literal, wrap it around I() (as you
would do in a formula in R). For example, the text “123,456” is replaced
with “123%2C456”. If you don’t want that behavior, you would need to do
the following I(“123,456”).

``` r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db='pubmed',
    id=paste(ids,collapse =','),
    retmax=1000,
    rettype='abstract'
    )
)

# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```
