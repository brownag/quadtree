
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

A quadtree object is created from a raster or matrix:

``` r
library(quadtree)
library(sp)
library(raster)

data(habitat, package="quadtree") #load sample data
qt = qt_create(habitat, split_threshold=.03, split_method="sd") #create a quadtree

par(mfrow=c(1,2), mar=c(3,2,2,2))
plot(habitat, zlim=c(0,1), main="raster representation", axes=FALSE, box=FALSE)
qt_plot(qt, crop=TRUE, na_col=NULL, border_lwd=.2,xlab="", ylab="", legend=FALSE, zlim=c(0,1), main="quadtree representation", axes=FALSE)
```

<img src="man/figures/README-example_create-1.png" width="100%" />

The package allows for flexibility in the quadtree creation process.
Several functions defining how to split and aggregate cells are
provided, and custom functions can be written for both of these
processes. In addition, quadtrees can be created using other quadtrees
as “templates”, so that the new quadtree has the identical structure as
the template quadtree.

Once created, cell values can be extracted, as with a raster:

``` r
pts = cbind(c(20000,32000,5000),c(10000,30000,27000))
qt_extract(qt,pts)
#> [1] 0.9692383 0.5340000 0.1364531
```

In addition, functions are provided for calculating least-cost paths
using the quadtree as a resistance surface.

``` r
start_point = c(6989,34007)
end_point = c(33015,38162)
lcp_finder = qt_lcp_finder(qt, start_point)
lcp = qt_find_lcp(lcp_finder, end_point, use_original_end_points = TRUE)

qt_plot(qt, border_col="gray70", crop=TRUE, na_col=NULL, border_lwd=.5)
lines(lcp[,1:2])
points(rbind(start_point, end_point), col="red", pch=16)
```

<img src="man/figures/README-example_lcp-1.png" width="70%" height="70%" />

## File structure

The code here *mostly* conforms to the standard R package structure. The
exception is the ‘other\_files’ folder.

-   /R - contains the R files
-   /data - contains Rdata files (.rda) containing sample data
-   /man - contains the documentation files
-   /other\_files - contains ‘scratchwork’ scripts that I use to test
    the package functionality. This folder is ignored when the package
    is built.
-   /src - contains the C++ code

## Implementation Details

The bulk of the code is written in C++ and interfaced with R via Rcpp.

The overall design philosophy was to keep the core C++ code completely
independent from the R code (i.e. no Rcpp-related code in the core C++
files.) This results in a three-tiered organization of the code - core
C++ code, Rcpp C++ code, and R code.

### Core C++ code

This consists of the following files (only the .h files are listed to
avoid redundancy):

-   Matrix.h - Defines the Matrix class implementing basic matrix
    funcitonality
-   Node.h - Defines the Node class, which are the ‘nodes’ of the
    quadtree
-   Quadtree.h - Defines the Quadtree class, which can be seen as a
    wrapper that provides a link to the interconnected nodes that make
    up the quadtree
-   Point.h - Defines a simple Point class
-   PointUtilities.h - Defines a namespace containing functions for
    performing calculations with Point objects
-   ShortestPathFinder.h - Defines a class for finding the least-cost
    paths using a quadtree as a cost surface

As mentioned before, these files are completely independent of R and can
be built and run independently of R.

### Rcpp C++ code

These files are called ‘wrappers’ - essentially they each contain an
instance of relevant object, and provide additional Rcpp-related
functions that can be accessed from R. These essentially provide the
“bridge” that allows the functionality in the core C++ files to be
accessed from R.

-   QuadtreeWrapper.h - wrapper class for ‘Quadtree’
-   NodeWrapper.h - wrapper class for ‘Node’
-   ShortestPathFinderWrapper.h - wrapper class for ‘ShortestPathFinder’

In addition, ‘R\_interface.h’ defines a namespace that currently
contains only a single function, which converts an Rcpp matrix to the
Matrix class I created. This function is separate from the other files
because it is a general-purpose function and thus didn’t fit in any of
the wrapper classes.

### R code

This category consists of the files in the ‘R’ folder. Many of these
functions are actually unnecessary, as all of the relevant functionality
can be accessed from R via the ‘Wrapper’ classes. However, because of
the object-based representation of these objects, the syntax for
accessing this functionality differs from typical R syntax. Thus, most
of the R functions provided here are syntactic sugar designed to make
the functionality available in a format that conforms to traditional R
code syntax. The two exceptions are `qt_create.R` and `qt_plot.R` which
are more complex and are not simple wrappers for C++ functions.

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

I added most of the R code later because I wanted a way to access the
functions in a way that more closely resembles typical R syntax - this
will hopefully make it more intuitive for R users to work with.

Overall, the structure works fairly well, but I wonder if it’s overly
complicated. I’m worried my desire to keep the C++ code independent of
the R code results in a cumbersome project organization. Regardless, it
works, and has allowed me to easily make edits to the underlying C++
structure and then use those exact files for both projects I’m working
on rather than needing slightly different versions for each. There
definitely might be more sophisticated ways of achieving the same
purpose, but what I’m doing works.
