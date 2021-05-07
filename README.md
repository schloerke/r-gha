<!-- badges: start -->
[![R-CMD-check](https://github.com/schloerke/r-gha/workflows/R-CMD-check/badge.svg)](https://github.com/schloerke/r-gha/actions)
<!-- badges: end -->

# R + GitHub Actions - _A love story_

> This repo is not maintained after May 11, 2021 and may become out of date.

## Usage

Assuming you have an R package in your repo...

```r
## Typical repos
# usethis::use_github_action_check_standard()

# This repo uses (and all future repos should use)
usethis::use_github_action("check-pak")
usethis::use_github_action("pkgdown-pak")
```

# Definitions

* GitHub Actions
  * GitHub's answer to CI
  * > Automate, customize, and execute your software development workflows right in your repository with GitHub Actions. You can discover, create, and share actions to perform any job you'd like, including CI/CD, and combine actions in a completely customized workflow.
  * Links:
    * Main: https://docs.github.com/en/actions
    * Intro: https://docs.github.com/en/actions/learn-github-actions/introduction-to-github-actions
    * Syntax: https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions
* CRAN
  * Unified location to submit and share packages
  * Requires a person to manually inspect / approve submitted packages
    * This process is slow and tedious due to many back and forth requests.
* `pak`
  * > A fresh approach to package installation
  * Website: https://pak.r-lib.org/
  * GitHub: https://github.com/r-lib/pak
# Story

When submitting packages to CRAN, it helps to let them know where you've tested R packages and on which platforms.  The more platforms and R versions that you can test your package on, the better!

#### History

Before GitHub Actions, developers used [Travis-CI](https://travis-ci.com/github/rstudio) (and some packages still do!). However, Travis was limited in that it could only test using a single type linux machine.  You could test on multiple R versions.

Given authors develop on a mac, this required package authors to [use `r-hub`](https://github.com/r-hub/rhub) (lead by Gabor) to test on different flavors of windows machines.  This service was MUCH better the praying and hoping your code worked, but was still not very robust.

R-hub may still be a good place to check if you have compiled `./src` code. However, this is a small number of R packages.

#### Need

As RStudio has become more popular and usage demands are becoming more spread out, packages [like `plumber`](https://github.com/rstudio/plumber/blob/e846cb3edf980cfea3e37325c54d69c814db2194/.github/workflows/R-CMD-check.yaml#L23-L41) need to be sure they work on multiple operating systems and multiple versions of R.

To feel comfortable submitting a heavily used R package, I like to test on using:

* Operating Systems
  * macOS,
  * windows,
  * ubuntu 16, and
  * ubuntu 20
* R versions
  * devel
  * release
  * oldrelease
  * 3.4 ([As there is no name called `reallyoldrelease`](https://github.com/r-lib/actions/issues/26#issuecomment-567524058))

These combinations create a large test matrix and is not fun to manually execute.

#### Solution

GitHub Actions (GHA) for R was initially explored by [Max Held](https://github.com/maxheld83) during his summer internship at RStudio. Since then, GHA created a complete different interface and Max's work was repurposed in https://github.com/r-lib/actions by Jim Hester.

Using `r-lib/actions` allows package developers to use consistent routines, yet customize the CI execution.

Two useful steps:
* `r-lib/actions/setup-r@v1`: Set up the R process. [Example](https://github.com/schloerke/r-gha/blob/aad7c06173e7a7236a46fe1e15583ff5859d9138/.github/workflows/R-CMD-check.yaml#L46-L50)
* `r-lib/actions/setup-pandoc@v1`: Set up pandoc. [Example](https://github.com/schloerke/r-gha/blob/aad7c06173e7a7236a46fe1e15583ff5859d9138/.github/workflows/R-CMD-check.yaml#L52)

This repo uses `pak` to [install system requirements](https://github.com/schloerke/r-gha/blob/aad7c06173e7a7236a46fe1e15583ff5859d9138/.github/workflows/R-CMD-check.yaml#L69-L74) and [install all package dependencies](https://github.com/schloerke/r-gha/blob/aad7c06173e7a7236a46fe1e15583ff5859d9138/.github/workflows/R-CMD-check.yaml#L76-L80). To me, any general usage of `remotes` should be replaced with `pak`. It is better for many reasons. Two being:
* wholistic installation (consistency): It knows everything that is going to be installed before installing the first package.
* parallel installation (speed): If two packages are not dependent on each other, they may be install in parallel.


# Real Life Example

* `r-lib/actions`
  * Examples: https://github.com/r-lib/actions/tree/master/examples

* `plumber` R package
  * `R CMD check`: [.github/workflows/R-CMD-check.yaml](https://github.com/rstudio/plumber/blob/master/.github/workflows/R-CMD-check.yaml)
    * Checks package on many different combinations of OS and R version
  * Build and deploy a `Docker` file: [.github/workflows/docker.yaml](https://github.com/rstudio/plumber/blob/master/.github/workflows/docker.yaml)
    * Builds and deploys `plumber`'s Docker file to [dockerhub](https://hub.docker.com/r/rstudio/plumber)
  * Build and deploy a `pkgdown` website: [.github/workflows/pkgdown.yaml](https://github.com/rstudio/plumber/blob/master/.github/workflows/pkgdown.yaml)
    * Automatically pushes latest changes to [`gh-pages` branch](https://github.com/rstudio/plumber/tree/gh-pages).
  * Test coverage: [.github/workflows/test-coverage.yaml](https://github.com/rstudio/plumber/blob/master/.github/workflows/test-coverage.yaml)
    * Execute test coverage (basically testing all code) on a single operating system and R version. This workflow [uploads test coverage to `codecov`](https://app.codecov.io/gh/rstudio/plumber).
  * PR commands: [.github/workflows/pr-commands.yaml](https://github.com/rstudio/plumber/blob/master/.github/workflows/pr-commands.yaml)
    * Listens to comments and conditionally adds the results of `devtools::document()` to the same PR.

Test coverage and PR commands could be combined into a single workflow, such as `rstudio/shiny`'s Rituals.
* `shiny` R package
  * `Rituals`: [.github/workflows/rituals.yaml](https://github.com/rstudio/shiny/blob/master/.github/workflows/rituals.yaml)
    * Performs many _rituals_ within a single GHA workflow
      * [Pulls from _push_ branch or _PR_ branch](https://github.com/rstudio/shiny/blob/9c80d7a4ec3bcb1e4f358d11b7a67f77e240e6c8/.github/workflows/rituals.yaml#L31-L38)
      * [Conditionally checks for bad urls (on release only)](https://github.com/rstudio/shiny/blob/2360bde13efac1fe501efee447a8f3dde0136722/.github/workflows/rituals.yaml#L84-L99)
      * [`devtools::document()`](https://github.com/rstudio/shiny/blob/2360bde13efac1fe501efee447a8f3dde0136722/.github/workflows/rituals.yaml#L101-L105)
      * [Builds all JavaScript code](https://github.com/rstudio/shiny/blob/2360bde13efac1fe501efee447a8f3dde0136722/.github/workflows/rituals.yaml#L111-L132)
      * [Pushes back to the _push_ branch or _PR_ branch](https://github.com/rstudio/shiny/blob/2360bde13efac1fe501efee447a8f3dde0136722/.github/workflows/rituals.yaml#L139-L147)
* `bslib` R package
  * `Rituals`
    * [Build Readme.md](https://github.com/rstudio/bslib/blob/ba6a80d14eb2bb04aea5977e27714740daa57965/.github/workflows/rituals.yaml#L86-L90)
