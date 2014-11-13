The Gadgetron depends on a number of libraries that can either be downloaded for free or that may already be part of the installation on your workstation. If you are working on a Linux platform you should be able to install all dependencies without compiling anything. The following is a list of the components that you will need. Some are optional.

To install these components please follow the platform specific installation instructions provided in [Linux Installation] [Mac Installation] [Windows Installation].

#### Required libraries

* CMake. Available from <http://www.cmake.org/cmake/resources/software.html>.
* Boost. Available from <http://www.boost.org>. The components required are _system_ and _thread_ for all platforms. Windows users will additionally need components _date_time_ and _chrono_.
* FFT3W. Available from <http://www.fftw.org>.

#### Optional libraries

* ADAPTIVE Computing Environment (ACE). Available from <http://www.cs.wustl.edu/~schmidt/ACE.html>. The Gadgetron's streaming infrastructure depends on ACE, so without it you can only compile the toolboxes and standalone applications.
* ISMRM Raw Data format (ISMRMRD). This is the MRI raw data format used in the streaming framework. Without this library installed you will not be able to reconstruct the provided MRI examples. The toolboxes can however still be used. Available from <http://ismrmrd.sourceforge.net>.
* CodeSynthesis XSD. Available from <http://www.codesynthesis.com/products/xsd>. Required if you install the ISMRMRD.
* HDF5. ISMRMRD uses the HDF5 file format for storing raw data and images. Available from <http://www.hdfgroup.org/HDF5>.
* Armadillo. Available from <http://arma.sourceforge.net/>. A library providing linear algebra routines (BLAS/LAPACK). It is strongly recommended as without it much toolbox and gadget functionality will be unavailable. For Windows users we recommend to use the precompiled binaries we provide.
* Intel Math Kernel Library (MKL). This is a very efficient library (on Intel processors) containing parallel algorithm implmentations of linear algebra subroutines. It is a commercial library and a license is required. The Gadgetron will work without this library, but some advanced features can be either slower or unavailable. We do strive to provide Armadillo based fallback routines for MKL optimized code paths.
* CUDA. For GPU support you need to install CUDA from Nvidia. You will need a CUDA driver for your graphics card too. Available from <http://developer.nvidia.com/cuda-downloads>.
* CULA. We use CULA for LAPACK routines on the GPU. This is the only dependency which is not Open Source. You can however download a free (registration required) version of CULA. Available from <http://www.culatools.com/downloads/dense>.
* QT4. A few standalone applications use QT for creating user interfaces. Available from <http://qt.nokia.com>.
* Doxygen. Required if you would like to build the API documentation. Available from <http://www.stack.nl/~dimitri/doxygen>.
* Git. We use git to manage our source code archives. You can use any source code management system you prefer (or none at all), but if you would like to stay in line with the Gadgetron team, use git. Available from <http://git-scm.com>.
