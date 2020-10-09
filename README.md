
<!-- README.md is generated from README.Rmd. Please edit that file -->

# horroR - scraping imdb top 250 horror films

For fun and organisation scrape some horror films using functional R.

### the functions

Creates a function factory to pull out different css selectors on the
page.

``` r
scrape_factory <- function(css) {
  force(css)
  function(p) {
    p %>% 
      html_nodes(css) %>% 
      html_text() %>% 
      str_trim() 
  }
}

# css selectors as a list of functions
selectors <- list(
  ranking = ".text-primary",
  title = ".lister-item-content .col-title a",
  year = ".text-muted",
  imdb_rating = ".col-imdb-rating"
) %>% 
  map(scrape_factory)

# this will get the first 250 films starting from the base URL
pagination <- function(base_url, start = seq(1, 201, by = 50)) {
  ifelse(start == 1, 
         base_url,
         paste0(base_url, ",desc&start=", start, "&ref_=adv_nxt")
  )
} 
```

### do the work

``` r
base_url <- "https://www.imdb.com/search/title/?title_type=feature&num_votes=25000,&genres=horror&sort=user_rating,desc&view=simple&sort=user_rating"

page <- map(pagination(base_url), ~scrape(bow(.)))

# for each page call the selector functions and collapse to a table
# then row bind the results and tidy some of the columns 
top250_horror <- map_dfr(page, ~map_dfc(selectors, exec, .)) %>% 
  mutate(
    ranking = as.integer(ranking),
    imdb_rating = as.numeric(imdb_rating),
    has_casey_seen_it = NA # the most prolific watcher will curate this column
  )

top250_horror
#> # A tibble: 250 x 5
#>    ranking title                            year   imdb_rating has_casey_seen_it
#>      <int> <chr>                            <chr>        <dbl> <lgl>            
#>  1       1 Psycho                           (1960)         8.5 NA               
#>  2       2 The Shining                      (1980)         8.4 NA               
#>  3       3 Alien                            (1979)         8.4 NA               
#>  4       4 The Thing                        (1982)         8.1 NA               
#>  5       5 What Ever Happened to Baby Jane? (1962)         8.1 NA               
#>  6       6 The Cabinet of Dr. Caligari      (1920)         8.1 NA               
#>  7       7 The Exorcist                     (1973)         8   NA               
#>  8       8 Rosemary's Baby                  (1968)         8   NA               
#>  9       9 Les diaboliques                  (1955)         8   NA               
#> 10      10 Let the Right One In             (2008)         7.9 NA               
#> # … with 240 more rows
```

### upload to google sheets

``` r
gs_sheet_url <- "your-sheet-url"

sheet <- gs4_get(gs_sheet_url)

sheet_write(top250_horror, sheet, sheet = "imdb-top")
```
