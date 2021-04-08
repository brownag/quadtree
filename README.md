
<!-- README.md is generated from README.Rmd. Please edit that file -->

# quadtree

This package provides functionality for working with raster-like
quadtrees.

## Installation

The package can be installed with the following R command:

``` r
devtools::install_gitlab("dafriend/quadtree")
```

For documentation of all the functions, see the PDF file stored at the
root directory.

## Example

A quadtree object is created from a ‘raster’ object:

``` r
library(quadtree)
library(raster)
#> Loading required package: sp
#create raster of random values
nrow = 32
ncol = 32
set.seed(1)
rast = raster(matrix(runif(nrow*ncol), nrow=nrow, ncol=ncol), xmn=0, xmx=ncol, ymn=0, ymx=nrow)

#create quadtree using the 'expand' method
qt = qt_create(rast, range_limit = .9) 
qt_plot(qt) #plot the quadtree
```

<img src="man/figures/README-example-1.png" width="100%" />

Cell values can be extracted:

``` r
qt_extract(qt,cbind(c(1,10,30),c(15,3,17)))
#> [1] 0.7836425 0.5221430 0.3244501
```

Least cost paths can be calculated:

``` r
start_point = c(1,1)
end_point = c(31,31)
lcp_finder = qt_lcp_finder(qt, start_point)
lcp = qt_find_lcp(lcp_finder, end_point, use_original_end_points = TRUE)
qt_plot(qt, border_col="gray60")
lines(lcp)
points(lcp, pch=16, cex=.5)
points(rbind(start_point, end_point), col="red", pch=16)
```

<img src="man/figures/README-unnamed-chunk-3-1.png" width="100%" />

## File structure

The code here *mostly* conforms to the standard R package structure. The
exception is the ‘other\_files’ folder.

  - /R - contains the R files
  - /man - contains the documentation files
  - /other\_files - contains ‘scratchwork’ scripts that I use to test
    the package functionality. This folder is ignored when the package
    is built.
  - /src - contains the C++ code

## Implementation Details

The bulk of the code is written in C++ and interfaced with R via Rcpp.

The overall design philosophy was to keep the core C++ code completely
independent from the R code (i.e. no Rcpp-related code in the core C++
files.) This results in a three-tiered organization of the code - core
C++ code, Rcpp C++ code, and R code.

### Core C++ code

This consists of the following files (only the .h files are listed to
avoid redundancy):

  - Matrix.h - Defines the Matrix class implementing basic matrix
    funcitonality
  - Node.h - Defines the Node class, which are the ‘nodes’ of the
    quadtree
  - Quadtree.h - Defines the Quadtree class, which can be seen as a
    wrapper that provides a link to the interconnected nodes that make
    up the quadtree
  - Point.h - Defines a simple Point class
  - PointUtilities.h - Defines a namespace containing functions for
    performing calculations with Point objects
  - ShortestPathFinder.h - Defines a class for finding the least-cost
    paths using a quadtree as a cost surface

As mentioned before, these files are completely independent of R and can
be built and run independently of R.

### Rcpp C++ code

These files are called ‘wrappers’ - essentially they each contain an
instance of relevant object, and provide additional Rcpp-related
functions that can be accessed from R.

  - QuadtreeWrapper.h - wrapper class for ‘Quadtree’
  - NodeWrapper.h - wrapper class for ‘Node’
  - ShortestPathFinderWrapper.h - wrapper class for ‘ShortestPathFinder’

In addition, ‘R\_interface.h’ defines a namespace that currently
contains only a single function, which converts an Rcpp matrix to the
Matrix class I created. This function is separate from the other files
because it is a general-purpose function and thus didn’t fit in any of
the wrapper classes.

### R code

This category consists of the files in the ‘R’ folder. These functions
are actually unnecessary, as all of the relevant functionality can be
accessed from R via the ‘Wrapper’ classes. However, because of the
object-based representation of these objects, the syntax for accessing
accessing this functionality differs from typical R syntax. Thus, the R
functions provided here are syntactic sugar designed to make the
functionality available in a format that conforms to traditional R code
syntax.

## Notes on package structure

The 3-tiered structure described here may be needlessly complicated. I
designed it this way because I am using the functionality implemented in
the C++ files for a pure-C++ implementation of an agent-based model.
Thus, to avoid having to unnecessarily include R interface code in those
files, I wanted a way to be able to use the original files for both
projects (this package and the C++ ABM) without needing to have slightly
different implementations to fit their respective purposes. Thus, I
settled on using the ‘wrapper class’ approach as a way to augment the
original classes with the necessary Rcpp functionality without having to
make any changes to the underlying C++ code.

I added the R code later because I wanted a way to access the functions
in a way that more closely resembles typical R syntax - this will
hopefully make it more intuitive for R users to work with.

Overall, the structure works fairly well, but I wonder if it’s overly
complicated. I’m worried my desire to keep the C++ code independent of
the R code results in a cumbersome project organization. Regardless, it
works, and has allowed me to easily make edits to the underlying C++
structure and then use those exact files for both projects I’m working
on rather than needing slightly different versions for each. There
definitely might be more sophisticated ways of achieving the same
purpose, but what I’m doing works.
