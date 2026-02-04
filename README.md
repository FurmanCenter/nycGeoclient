
<!-- README.md is generated from README.Rmd. Please edit that file -->

# nycGeoclient

R interface for NYC’s Geoclient API

The code is copied from this
[repo](https://github.com/austensen/geoclient). This repo was copied
over, so that changes can be made by the Furman Center (e.g., change API endpoint)

This packages uses NYC’s Geoclient API but is neither endorsed nor
supported by the the City of New York.

For information about the Geoclient API visit [NYC’s Developers
Portal](https://developer.cityofnewyork.us/api/geoclient-api).

### Installation

Install from Github with [remotes](https://github.com/r-lib/remotes):

``` r
# install.packages("remotes")
remotes::install_github("FurmanCenter/nycGeoclient")
```

### Set up *Geoclient* API keys

You can acquire your Geoclient app ID and Key by first registering with
the [NYC’s Developer
Portal](https://developer.cityofnewyork.us/user/register?destination=api)
at, then [create a new
project](https://developer.cityofnewyork.us/create/project), selecting
“Geoclient v1” from available APIs.

To avoid having to provide the ID and Key with each function call you
can use `geoclient_api_keys()` to add your Geoclient app ID and Key to
your `.Renviron` file so they can be called securely without being
stored in your code.

### Basic Usage

There are 6 main location types that can be set with *Geoclient*:
Address, BBL (Borough-Block-Lot), BIN (Building Identification Number),
Blockface, Intersection, and Place (“well-known NYC place name”). All of
these functions return the results of the *Geoclient* API call as a
dataframe, with additional columns for the arguments provided to the
function.

``` r
geo_address(
  house_number = "139", 
  street = "MacDougal St", 
  borough = "MN",
  zip = "10012"
)
#> # A tibble: 1 × 198
#>   input_house_number input_street input_borough input_zip no_results
#>   <chr>              <chr>        <chr>         <chr>     <lgl>     
#> 1 139                MacDougal St manhattan     10012     FALSE     
#> # ℹ 193 more variables: alleyCrossStreetsFlag <chr>, assemblyDistrict <chr>,
#> #   atomicPolygon <chr>, bbl <chr>, bblBoroughCode <chr>, …
```

You can also pull out just a single column if that is all you need.

``` r
df <- tibble::tribble(
  ~num,  ~st,                ~boro,         ~zip,
  "139", "MacDougal St",     "manhattan",   "11231",
  "295", "Lafayette street", NA,            "10012-2722",
  "40",  "WASHINGTON SQ S",  "MN",          NA
)

dplyr::mutate(df, bbl = geo_address(num, st, boro, zip)[["bbl"]])
#> [==============================================>------------------------]
#> 67%[=======================================================================]
#> 100%
#> # A tibble: 3 × 5
#>   num   st               boro      zip        bbl       
#>   <chr> <chr>            <chr>     <chr>      <chr>     
#> 1 139   MacDougal St     manhattan 11231      1005430053
#> 2 295   Lafayette street <NA>      10012-2722 1005107502
#> 3 40    WASHINGTON SQ S  MN        <NA>       1005410001
```

For each of these location types there are two functions in this package
that allow the arguments to be supplied either as individual vector, or
with a dataframe and bare column names.

``` r
geo_address_data(df, num, st, boro, zip)
#> [=======================================================================] 100%
#> # A tibble: 3 × 287
#>   input_house_number input_street     input_borough input_zip  no_results
#>   <chr>              <chr>            <chr>         <chr>      <lgl>     
#> 1 139                MacDougal St     manhattan     11231      FALSE     
#> 2 295                Lafayette street <NA>          10012-2722 FALSE     
#> 3 40                 WASHINGTON SQ S  manhattan     <NA>       FALSE     
#> # ℹ 282 more variables: alleyCrossStreetsFlag <chr>, assemblyDistrict <chr>,
#> #   atomicPolygon <chr>, bbl <chr>, bblBoroughCode <chr>, …
```

The return dataframe will always be the same length and in the same
order, so you can easily add all the return columns to your existing
dataframe.

``` r
dplyr::bind_cols(df, geo_address_data(df, num, st, boro, zip))
#> [=======================================================================] 100%
#> # A tibble: 3 × 291
#>   num   st             boro  zip   input_house_number input_street input_borough
#>   <chr> <chr>          <chr> <chr> <chr>              <chr>        <chr>        
#> 1 139   MacDougal St   manh… 11231 139                MacDougal St manhattan    
#> 2 295   Lafayette str… <NA>  1001… 295                Lafayette s… <NA>         
#> 3 40    WASHINGTON SQ… MN    <NA>  40                 WASHINGTON … manhattan    
#> # ℹ 284 more variables: input_zip <chr>, no_results <lgl>,
#> #   alleyCrossStreetsFlag <chr>, assemblyDistrict <chr>, atomicPolygon <chr>, …
```

In addition to the 6 location types, *Geoclient* also provides a
single-field search option, which will guess the location type. This can
be particularly helpful when you have address data that is not easily
separated for use with `geo_address()`.

``` r
df <- tibble::tribble(
  ~address,
  "139 MacDougal St manhattan, 11231",
  "295 Lafayette street, 10012-2722",
  "40 WASHINGTON SQ S MN"
)

geo_search_data(df, address)
#> [=======================================================================] 100%
#> # A tibble: 3 × 278
#>   input_location no_results alleyCrossStreetsFlag assemblyDistrict atomicPolygon
#>   <chr>          <lgl>      <chr>                 <chr>            <chr>        
#> 1 139 MacDougal… FALSE      X                     55               302          
#> 2 295 Lafayette… FALSE      X                     66               106          
#> 3 40 WASHINGTON… FALSE      <NA>                  66               105          
#> # ℹ 273 more variables: bbl <chr>, bblBoroughCode <chr>, bblTaxBlock <chr>,
#> #   bblTaxLot <chr>, blockfaceId <chr>, …
```
