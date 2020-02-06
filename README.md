---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# osfr <a href="https://docs.ropensci.org/osfr"><img src="man/figures/logo.png" align="right" height="139" /></a>

[![Build Status](https://travis-ci.com/ropensci/osfr.svg)](https://travis-ci.com/ropensci/osfr)
[![AppVeyor build status](https://ci.appveyor.com/api/projects/status/github/ropensci/osfr?branch=master&svg=true)](https://ci.appveyor.com/project/aaronwolen/osfr)
[![Coverage status](https://codecov.io/gh/ropensci/osfr/branch/master/graph/badge.svg)](https://codecov.io/github/ropensci/osfr?branch=master)
[![](https://badges.ropensci.org/279_status.svg)](https://github.com/ropensci/software-review/issues/279)
[![](https://joss.theoj.org/papers/d5398fc36ea92794a20914143d3fcdc4/status.svg)](https://joss.theoj.org/papers/d5398fc36ea92794a20914143d3fcdc4)
[![DOI](https://zenodo.org/badge/42329785.svg)](https://zenodo.org/badge/latestdoi/42329785)

## Overview

osfr provides a suite of functions for interacting with the Open Science Framework ([OSF][osf]).

**What is OSF?**

*OSF is a free and [open source][osf-gh] project management repository designed to support researchers across their entire project lifecycle. The service includes unlimited cloud storage and file version history, providing a centralized location for all your research materials that can be kept private, shared with select collaborators, or made publicly available with citable DOIs.*

## Installation

This package is currently under development and is not available on CRAN. You can install the current version with the *remotes* package:

``` r
# install.packages("remotes")
remotes::install_github("ropensci/osfr")
```

## Usage Examples

*Note: You need to [setup an OSF personal access token (PAT)][auth] to use osfr to manage projects or upload files.*

### Accessing Open Research Materials

Many researchers use OSF to archive and share their work. You can use osfr to explore publicly accessible projects and download the associated files---all you need to get started is the project's URL or GUID (global unique identifier).

Every user, project, component, and file on OSF is assigned a GUID that is embedded in the corresponding entity's URL. For example, you can access the main OSF project for the *Cancer Reproducibility Project* at <https://osf.io/e81xl/>. The GUID for this project is `e81xl`.

We can then use osfr to *retrieve* this project and load it into R by providing the GUID:


```r
library(osfr)

cr_project <- osf_retrieve_node("e81xl")
cr_project
#> # A tibble: 1 x 3
#>   name                                    id    meta            
#>   <chr>                                   <chr> <list>          
#> 1 Reproducibility Project: Cancer Biology e81xl <named list [3]>
```

This returns an `osf_tbl` object with a single row representing the retrieved project. Let's list the files that have been uploaded to this project.


```r
osf_ls_files(cr_project)
#> # A tibble: 4 x 3
#>   name                                     id                     meta          
#>   <chr>                                    <chr>                  <list>        
#> 1 Adjustment of 50 studies to 37 studies.… 565602398c5e4a3877d72… <named list […
#> 2 papers_and_keywords.xlsx                 553e671b8c5e4a219919e… <named list […
#> 3 Full_dataset_of_papers_formatted.xls     553e671b8c5e4a219919e… <named list […
#> 4 METHOD_to_select_papers.txt              553e671b8c5e4a219919e… <named list […
```

This returns another `osf_tbl` with 1 row for each of the files and directories in the project. We can examine any of these files directly on OSF with `osf_open()`, which opens the corresponding file's view in your default browser.

This project contains 2 ***components***: *Replication Studies* and *Data collection and publishing guidelines*. We can list these components with osfr using `osf_ls_nodes()`.


```r
osf_ls_nodes(cr_project)
#> # A tibble: 2 x 3
#>   name                                      id    meta            
#>   <chr>                                     <chr> <list>          
#> 1 Replication Studies                       p7ayb <named list [3]>
#> 2 Data collection and publishing guidelines a5imq <named list [3]>
```

osfr is compatible with the [pipe operator][magrittr] and [dplyr][], providing a powerful set of tools for working with `osf_tbl`s. Here, we're listing the sub-components nested within the *Replication Studies* component, filtering for a specific study ([*Study 19*](https://osf.io/7zqxp/)) and then listing the files uploaded to that study's component.


```r
library(dplyr)

cr_project %>%
  osf_ls_nodes() %>%
  filter(name == "Replication Studies") %>%
  osf_ls_nodes(pattern = "Study 19") %>%
  osf_ls_files()
#> # A tibble: 6 x 3
#>   name                                    id                     meta           
#>   <chr>                                   <chr>                  <list>         
#> 1 Replication_Study_19.docx               57c9e8ed594d9001e7a24… <named list [3…
#> 2 Replication_Study_19.Rmd                578e2b23594d9001f4816… <named list [3…
#> 3 Replication_Study_19_track_changes.docx 581a27b76c613b0223322… <named list [3…
#> 4 Replication_Study_19_track_changes_2.d… 58714d46594d9001f801f… <named list [3…
#> 5 Response_letter_Replication_Study_19.d… 58755747b83f6901ff066… <named list [3…
#> 6 Study_19_Correction_Letter.docx         5a56569125719b000ff28… <named list [3…
```

We could continue this pattern of exploration and even download local copies of project files using `osf_download()`. Or, if you come across a publication that  directly references a file's OSF URL, you could quickly download it to your project directory by providing the URL or simply the GUID:


```r
osf_retrieve_file("https://osf.io/btgx3/") %>%
  osf_download()
#> # A tibble: 1 x 4
#>   name                id                    local_path            meta          
#>   <chr>               <chr>                 <chr>                 <list>        
#> 1 Study_19_Figure_1.… 5751d71d9ad5a1020793… ./Study_19_Figure_1.… <named list […
```


### Managing Projects

You can use osfr to [create projects][osf-create], [add sub-components][osf-create] or [directories][osf-mkdir], and [upload files][osf-upload]. See [Getting Started][getting-started] to learn more about building projects with osfr, but here is a quick example in which we:

1. Create a new project called *Motor Trend Car Road Tests*
2. Create a sub-component called *Car Data*
3. Create a directory named *rawdata*
4. Upload a file (`mtcars.csv`) to the new directory
5. Open the uploaded file on OSF


```r
# create an external data file
write.csv(mtcars, "mtcars.csv")

osf_create_project(title = "Motor Trend Car Road Tests") %>%
  osf_create_component("Car Data") %>%
  osf_mkdir("rawdata") %>%
  osf_upload("mtcars.csv") %>%
  osf_open()
```

![Screenshot of the uploaded file on OSF](man/figures/screen-shot.png)

## Details on `osf_tbls`

There are 3 main types of OSF entities that osfr can work with:

1. **nodes:** both [projects][help-proj] and [components][help-comp] (i.e., sub-projects) are referred to as nodes
2. **files:** this includes both files *and* folders stored on OSF
3. **users:** individuals with OSF accounts

osfr represents these entities within `osf_tbl`s---specialized data frames built on the tibble class that provide useful information about the entities like their `name` and unique `id` for users, and API data in the `meta` column that's necessary for osfr's internal functions. Otherwise, they're just `data.frames` and can be manipulated using standard functions from base R or dplyr.

## Acknowledgments

OSF is developed by the [Center for Open Science][cos] in Charlottesville, VA.

The original version of osfr was developed by [Chris Chartgerink][chris] and further developed by [Brian Richards][brian] and [Ryan Hafen][ryan]. The current version was developed by [Aaron Wolen][aaron] and is *heavily* inspired by [Jennifer Bryan][jenny] and [Lucy D'Agostino McGowan][lucy]'s excellent [googledrive][] package. Seriously, we borrowed a lot of great ideas from them. Other important resources include [http testing](https://books.ropensci.org/http-testing/) by Scott Chamberlain and [R Packages](http://r-pkgs.had.co.nz) by Hadley Wickham. Development was also greatly facilitated by OSF's excellent [API documentation][osf-api].

Big thanks to Rusty Speidel for designing our logo and [Tim Errington][tim] for his feedback during development.

## Contributing

Check out the [Contributing Guidelines][contrib] to get started with osfr development and note that by contributing to this project, you agree to abide by the terms outlined in the [Contributor Code of Conduct][coc].



[![ropensci_footer](https://ropensci.org/public_images/ropensci_footer.png)](https://ropensci.org)

<!-- links -->
[osf]: https://osf.io "Open Science Framework"
[cos]: https://cos.io "Center for Open Science"
[osf-gh]: https://github.com/CenterForOpenScience/osf.io "OSF's GitHub Repository"
[osf-api]: https://developer.osf.io "OSF API Documentation"
[help]: http://help.osf.io "OSF Support"
[help-proj]: https://help.osf.io/hc/en-us/articles/360019737594-Create-a-Project "OSF: Create a Project"
[help-comp]: https://help.osf.io/hc/en-us/articles/360019737614-Create-Components "OSF: Create a Component"
[magrittr]: https://magrittr.tidyverse.org
[dplyr]: https://dplyr.tidyverse.org
[googledrive]: https://googledrive.tidyverse.org
[tibble]: https://tibble.tidyverse.org

[chris]: https://github.com/chartgerink
[brian]: https://github.com/bgrich
[ryan]: https://github.com/hafen
[aaron]: https://github.com/aaronwolen
[jenny]: https://github.com/jennybc
[lucy]: https://github.com/lucymcgowan
[tim]: https://github.com/timerrington

[getting-started]: https://docs.ropensci.org/osfr/articles/getting_started
[auth]: https://docs.ropensci.org/osfr/articles/auth

[osf-create]: https://docs.ropensci.org/osfr/reference/osf_create
[osf-mkdir]: https://docs.ropensci.org/osfr/reference/osf_mkdir
[osf-upload]: https://docs.ropensci.org/osfr/reference/osf_upload

[contrib]: https://github.com/ropensci/osfr/blob/master/.github/CONTRIBUTING.md
[coc]: https://github.com/ropensci/osfr/blob/master/.github/CODE_OF_CONDUCT.md
