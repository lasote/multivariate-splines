##Multivariate-BSplines
Multivariate-BSplines is a curve fitting library utilizing a speedy implementation of [BSplines](http://en.wikipedia.org/wiki/B-spline) with [Eigens](http://eigen.tuxfamily.org/index.php?title=Main_Page) SparseMatrix.

###Requirements: 
* [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page)
* [CMake](http://www.cmake.org/)
* [Git](http://git-scm.com/)
* [GCC](https://gcc.gnu.org/) or an equivalent C++11 compiler


###Steps to install on UNIX
0. `sudo apt-get update && sudo apt-get install git cmake build-essential` or equivalent
1. `git clone https://github.com/bgrimstad/multivariate-bsplines.git`
2. `cd multivariate-bsplines`
3. `cmake .`
4. `make`
5. `make install`

###Steps to install on Windows
1. Clone https://github.com/bgrimstad/multivariate-bsplines
2. Download Eigen: http://bitbucket.org/eigen/eigen/get/3.2.2.zip
  1. Extract the zip-file into a new folder, and write down the location of that folder
3. Download and install CMake: http://www.cmake.org/files/v3.0/cmake-3.0.1-win32-x86.exe 
4. Download and install Qt Creator: http://qt-project.org/downloads
  1. Make sure that MinGW is marked for installation
5. Run Qt Creator, select `Open project`
  1. Navigate into the multivariate-bsplines folder, select `CMakeLists.txt`
  2. In the arguments field, write: `-DEIGEN_DIRECTORY=C:/path/to/eigen/from/step/2.1`
  3. Run CMake
6. Now you can build the library with Qt Creator, and the library files will be output to your build directory.

* You may have to add -static as a flag to your linker if you are compiling with MinGW.
* C++11 must be enabled.
* Remember to add the Eigen directory to your include path.
* If you get asked to specify where CMake is located, then you can typically find the file cmake.exe in C:\Program Files (x86)\CMake\bin.


####Options:
These options go along with step #3, and are used like this:

*     cmake . -DEIGEN_DIRECTORY=/home/me/eigen

*     cmake . -DEIGEN_DIRECTORY=/path/to/eigen -DHEADER_DIRECTORY=/home/me/c++/multivariate-bsplines/includes

The syntax is: `-D<VARIABLE_NAME>=<VARIABLE_VALUE>`. If you have any spaces in your value you must surround it with double quotes (").

| Variable name     | Default value                 | Description                               |
| ----------------- | ----------------------------- | ----------------------------------------- |
| EIGEN_DIRECTORY   | /usr/local/include/eigen3     | Path to the Eigen lib.                    |
| HEADER_DIRECTORY  | include/multivariate-bsplines | Where the headers should be installed.    |
| LIBRARY_DIRECTORY | lib/multivariate-bsplines     | Where to install the library file.        |

Note for the header and library paths:
If the path is relative (the first character is not / on UNIX or C:/ (or equivalent) on Windows), then the actual path used will be relative to [CMAKE_INSTALL_PREFIX](http://www.cmake.org/cmake/help/v2.8.12/cmake.html#variable:CMAKE_INSTALL_PREFIX).

####Troubleshooting
`fatal error: Eigen/Dense: No such file or directory`: The compiler could not find Eigen. You either need to install Eigen, and then run step #3 again (with `-DEIGEN_DIRECTORY="/path/to/eigen"` if Eigen did not install to the default directory (/usr/local/include/eigen3)).

`make: *** [install] Error 1`: You probably need elevated rights to install the library because you are trying to write to a directory you don't have permission to write to. Either change the install paths via the options, or run step #5 again like this: `sudo make install`.

###Usage
This is a simple example demonstrating the use of Multivariate-BSplines. Note that there is no restrictions to the dimension of x or y (except that they have to be >= 1, of course), nor is there any requirement that their dimensions should be equal.

Remember to compile with a c++11 compiler! That means you probably have to add a flag when compiling.

```c++
#include <iostream>
#include <sorteddatatable.h>
#include <bspline.h>
using std::cout;
using std::endl;

// Note: DenseVector is a typedef of Eigen::VectorXd (from generaldefinitions.h)
// typedef Eigen::VectorXd DenseVector;

// The function we are sampling.
// In reality this would probably be a set of sensors or other, more useful, data sources.
DenseVector func(DenseVector &x)
{
    DenseVector y(2);
    y(0) = x(0) / (1.3 * x(1) + 0.01) + 2 * x(1);
    y(1) = 10.2 * x(0) - 3 * x(1);
    return y;
}

int main()
{
    // The BSpline class needs its data sorted by x0, x1 ... xn.
    // This class makes sure that it is. It also allows / disallows duplicates
    // and checks that the grid is complete.
    SortedDataTable table;

    DenseVector x(2);
    // Record the domain of i and j so we don't evaluate outside their
    // domains in the loops following these two loops.
    double i_max = 0.0, j_max = 0.0;
    for(double i = 0.0; i <= 2.6; i += 0.11)
    {
        i_max = i;
        for(double j = 1.4; j <= 5.3; j += 0.05)
        {
            j_max = j;
            x(0) = i; x(1) = j;
            // addSample has several signatures, see the header file.
            table.addSample(x, func(x));
        }
    }

    Bspline bSpline(table, 3);

    DenseVector y, b;
    for(double i = 0.0; i <= i_max; i += 0.07)
    {
        for(double j = 1.4; j <= j_max; j += 0.09)
        {
            x(0) = i; x(1) = j;
            y = func(x);
            b = bSpline.evaluate(x);
            cout << y(0) << ", " << y(1) << " =? ";
            cout << b(0) << ", " << b(1) << endl;
        }
    }

    return 0;
}
```

###Grid
The grid, meaning all the values of x where you have sampled the function, must be complete. This means that if you have sampled the function in `x = [0 0]`, `x = [1 0]` and `x = [2 1]`, you must also sample the function in `x = [1 1]`, `x = [0 1]` and `x = [2 0]`. You must have sampled the function in all permutations of x within the possible values of x<sub>0</sub>, x<sub>1</sub> ... x<sub>n</sub>. The number of samples will then (disregarding duplicates) be num(x<sub>0</sub>) * num(x<sub>1</sub>) * ... * num(x<sub>n</sub>) where num(x) is the number of distinct values of x the the function has been sampled in. You can check if the grid is complete by calling `isGridComplete()` on your SortedDataTable.


This is an **incomplete** grid:

| x<sub>0</sub>   | x<sub>1</sub>   | y<sub>0</sub>    | y<sub>1</sub>   |
| ---- | ---- | ----- | ---- |
| 2.1    | 1   | - 7   | 1    |
| 2.3    | 3   | 10  | 3.5    |
| 2.1  | 3  | 9.3   | - 34  |


This is a **complete** grid:

| x<sub>0</sub>   | x<sub>1</sub>   | y<sub>0</sub>    | y<sub>1</sub>   |
| ---- | ---- | ----- | ---- |
| 2.1    | 1   | - 7   | 1    |
| 2.3    | 3   | 10  | 3.5    |
| 2.1  | 3  | 9.3   | - 34  |
| 2.3   | 1  | 0   | 2.3   |

Please note that whether the grid is complete or not only depends on the values of x, not those of y.
