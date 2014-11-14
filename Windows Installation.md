It is probably appropriate to start this section with a warning: Windows is not the easiest environment in which to work with the Gadgetron. As indicated in [List of Dependencies], the Gadgetron relies on multiple external libraries. Many of those libraries are not available as easy install packages and must be compiled separately. If you are uncomfortable setting up development tools on Windows, or if you are just looking for a fast and easy way to get started with the Gadgetron, we recommend installing on Ubuntu Linux - possibly using a virtual machine inside Windows (see [Linux Installation]).

The following is a list of steps we have used to install the Gadgetron on a clean Windows 7 (64-bit) machine. 

For the 32 bit windows: we do not test the Gadgetron on 32bit windows anymore, since 32bit windows is obsolete in most cases.

### Install Visual Studio 2010 (with Service Pack 1)

### Install CUDA/CULA (optional, but required for GPU support).

Download Cuda drivers/toolkit from from <http://developer.nvidia.com/cuda/cuda-downloads>.
Install CUDA 5.5 (e.g. cuda_5.5.20_winvista_win7_win8_general_64.exe)
Install `cula_dense_free_R16a-windows.exe` from <http://www.culatools.com/downloads/dense>.
Assuming CULA was installed in `C:\Program Files\CULA\R16`, add `C:\Program Files\CULA\R16\bin64` to your `PATH` environment variable.

### Create a folder for external libraries, say `C:\Libraries`.

### Install FFTW3 (<http://www.fftw.org/install/windows.html>)

Copy FFTW3 binaries to ``C:\Libraries\FFTW3``.

Create .lib files, on the command line type: 

    c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3f-3.def
    c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3-3.def
    c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3l-3.def

Add `C:\Libraries\FFTW3` to your PATH environment variable.

On 32 bit Windows remember to remove the /machine:x64 argument, the default is 32 bit.

### Install ACE (<http://download.dre.vanderbilt.edu/>)

Unpack ACE into `C:\\Libraries\\ACE-6.2.0\\ACE_wrappers`

Add `config.h` in `ACE_ROOT/ace/` with the following content:

    //We are on Windows
    #include "ace/config-win32.h" 

    //This ensured that INLINE settings 
    //do not vary between Debug and Release modes
    #define ACE_NO_INLINE 

Open the VS 2010 project in the source code archive

Set build type to Release/x64

Build (this takes a while)

Add to `PATH` environment variable: `C:\Libraries\ACE-6.2.0\ACE_wrappers\lib`

### Install Python (optional).

Regrettably, the off-the-shelf Python header files cannot be compiled in debug mode on Windows. This enforces the Gadgetron framework to be compiled in release mode only if you enable the Python components.

Install python-2.7.6.amd64.msi (<http://www.python.org>)

Add install folder (e.g. `C:\Python27`) to PATH environment variable

Add `PYTHON_ROOT` environment variable

From <http://www.lfd.uci.edu/~gohlke/pythonlibs/> download and

Install the following (+ additional libraries that you may need for your Python development):
- `numpy-MKL-1.7.2.win-amd64-py2.7`
- `scipy-0.13.2.win-amd64-py2.7`
- `libxml2-python-2.9.1.win-amd64-py2.7`

## Optionally, install MKL <http://software.intel.com/en-us/intel-mkl>

Purchase and install MKL to 'C:\Program Files (x86)\Intel\Composer XE 2013 SP1'

Add

    `C:\Program Files (x86)\Intel\Composer XE 2013 SP1\redist\intel64\mkl;C:\Program Files (x86)\Intel\Composer XE 2013 SP1\redist\intel64\compiler`
    
to your `PATH` environment variable.

## Optionally, install ACML (BLAS and LAPACK)

Download `acml4.4.0-win64.exe` from: <http://developer.amd.com/downloads/acml4.4.0-win64.exe>

Install Library in say `C:\Libraries\acml4.4.0`

Add

    `C:\Libraries\acml4.4.0\win64\lib;C:\Libraries\acml4.4.0\win64_mp\lib`
    
to your `PATH` environment variable.

Notice. Newer versions of the ACML-library are available (version 5.2.0 at the time of preparing this manual) - however, these libraries are distributed without required dependencies and will not work out of the box. We recommend sticking to the earlier version 4.4.0 until these issues have been resolved.

##Install the newest Boost release (<http://www.boost.org>)

We recommend using the precompiled binaries from BoostPro (e.g <http://boostpro.com/download/x64/boost_1_51_setup.exe>)

Just install everything, you might need other components later.

Alternatively, if you want to compile the latest boost <http://www.boost.org/users/history/version_1_55_0.html> 

Download boost_1_55_0.zip to 'C:\Libraries\boost' and unzip this package to 'C:\Libraries\boost\boost_1_55_0'.

Open an windows command window and 

    cd C:\Libraries\boost\boost_1_55_0
    bootstrap
    b2 address-model=64 --build-type=complete stage

The boost will be compiled and staged at 'C:\Libraries\boost\boost_1_55_0\stage\lib'.

Add `C:\Libraries\boost\boost_1_55_0\stage\lib' to `your PATH environment variable.`

##Install git (if you are using source code management):

Run the newest installation package named something like
Git-\*-preview\*.exe from: <http://code.google.com/p/msysgit/>

Use run in git bash only option

Use checkout Windows LF and commit Unix Line feeds

##Install CMake (<http://www.cmake.org/cmake/resources/software.html>)

Install the latest release (e.g. cmake-2.8.12.1-win32-x86.exe)

##Install HDF5:

You will need the HDFView application to view data files used by the Gadgetron, it can be downloaded rom <http://www.hdfgroup.org/HDF5/> install HDFView : `hdfview_install_win64.exe`

The precompiled binaries (e.g. hdf5-1.8.12-win64-vs10shared.zip <http://www.hdfgroup.org/HDF5/release/obtain5.html>) should work fine with the Gadgetron. Remember to add the path (e.g. `C:\Program Files\HDF Group\HDF5\1.8.12\bin`) to the HDF libraries to your `PATH` environment library.

##Install CodeSynthesis XSD (<http://www.codesynthesis.com/download/xsd/3.3/windows/i686/xsd-3.3.msi>).

Remember to add the path to the XSD binaries to your `PATH` environment variable. E.g. `C:\Program Files (x86)\CodeSynthesis XSD 3.3\bin\` and `C:\Program Files (x86)\CodeSynthesis XSD 3.3\bin64\.`

##Download, compile, and install the ISMRM Raw Data format. 

Detailed instructions are available at (<http://ismrmrd.github.io>).    

From a git bash shell:

    cd C:\Libraries
    git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
    cd ismrmrd-code/
    mkdir build
    cd build/
    cmake-gui.exe

Last command will open CMake's graphical user interface. Hit the configure button and deal with the dependencies that CMake is unable to find. Hit configure again and repeat the process until CMake has enough information to configure. Once the configuration is complete, you can hit generate to generate a Visual Studio project, which you can open and use to build ISMRMRD.

A few CMAKE variables are needed:

    BOOST_ROOT=C:/Libraries/boost/boost_1_55_0
    XSD_EXECUTABLE=C:/Program Files (x86)/CodeSynthesis XSD 3.3/bin/xsd.exe
    XSD_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include
    HDF5_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/cmake/hdf5
    HDF5_CXX_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5_cppdll.lib
    HDF5_C_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5dll.lib
    HDF5_C_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include
    HDF5_CXX_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include
    FFTW3F_LIBRARY=C:/Libraries/FFTW3/libfftw3f-3.lib
    FFTW3_INCLUDE_DIR=C:/Libraries/FFTW3
    XERCESC_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XS D3.3/include/xercesc
    XERCESC_LIBRARIES=C:/Program Files (x86)/CodeSynthesis XSD 3.3/lib64/vc-10.0/xerces-c_3.lib


Note: use the '/' instead of '\'

The directory to install ISMRMRD is 'C:/Libraries/install/ismrmrd'.

Remember to add the path to the ISMRMRD to your `PATH` environment variable. E.g. `C:/Libraries/install/ismrmrd/bin` and `C:/Libraries/install/ismrmrd/lib'.

If your working folder is the same as 'C:/Libraries', the following CMake command line can be used to generate the visual studio solution:

    cmake "-G" "Visual Studio 10 Win64" "-DBOOST_ROOT=C:/Libraries/boost/boost_1_55_0" "-DHDF5_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/cmake/hdf5"  "-DHDF5_CXX_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5_cpp.lib" "-DHDF5_C_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5.lib" "-DHDF5_C_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include" "-DHDF5_CXX_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include" "-DFFTW3_INCLUDE_DIR=C:/Libraries/FFTW3" "-DFFTW3F_LIBRARY=C:/Libraries/FFTW3/libfftw3f-3.lib" "-DXERCESC_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include/xercesc" "-DXERCESC_LIBRARIES=C:/Program Files (x86)/CodeSynthesis XSD 3.3/lib64/vc-10.0/xerces-c_3.lib" "-DXSD_EXECUTABLE=C:/Program Files (x86)/CodeSynthesis XSD 3.3/bin/xsd.exe" "-DXSD_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include" "-DCMAKE_INSTALL_PREFIX=C:/Libraries/install" "C:/Libraries/ismrmrd-code"

##Download and compile the Google Test if you want to use the unit test framework. <https://code.google.com/p/googletest/downloads/detail?name=gtest-1.7.0.zip>

Unzip the gtest-1.7.0.zip to C:/Libraries/gtest-1.7.0
    
    cd C:/Libraries/gtest-1.7.0
    mkdir vc10
    cd vc10
    cmake-gui.exe

set the 'Where is the source code' as 'C:/Libraries/gtest-1.7.0'
set the 'Where to build the binaries' as 'C:/Libraries/gtest-1.7.0/vc10'

Hit configure
Check cmake option 'BUILD_SHARED_LIBS', 'gtest_force_shared_crt', 'gtest_disable_pthreads'
Hit configure
Hit Generate

Then in the C:/Libraries/gtest-1.7.0/vc10, open the visual studio solution 'gtest.sln', compile both Debug and Release version.

##Download, compile and install the Armadillo C++ linear algebra library <http://arma.sourceforge.net/download.html>.

unzip the armadillo-4.000.2.tar.gz to C:/Libraries/armadillo-4.000.2

    cd C:/Libraries/armadillo-4.000.2
    mkdir vc10
    cd vc10
    cmake-gui.exe

Set the 'Where is the source code' as 'C:/Libraries/armadillo-4.000.2'
Set the 'Where to build the binaries' as 'C:/Libraries/armadillo-4.000.2/vc10'
In the C:/Libraries/armadillo-4.000.2/include/armadillo_bits, find the config.hpp file and uncomment

    //#define ARMA_USE_LAPACK
    //#define ARMA_USE_BLAS
    //#define ARMA_USE_WRAPPER
    //#define ARMA_BLAS_LONG_LONG

Set CMAKE_INSTALL_PREFIX=C:/Libraries/armadillo-4.000.2/install
Hit configure
If the Intel MKL is installed, set:


    mkl_LIBRARY=C:/Program Files (x86)/Intel/Composer XE 2013 SP1/mkl/lib/intel64/mkl_core.lib
    mkl_core_LIBRARY=C:/Program Files (x86)/Intel/Composer XE 2013 SP1/mkl/lib/intel64/mkl_core.lib
    mkl_intel_lp64_LIBRARY=C:/Program Files (x86)/Intel/Composer XE 2013 SP1/mkl/lib/intel64/mkl_intel_ilp64.lib
    mkl_intel_thread_LIBRARY=C:/Program Files (x86)/Intel/Composer XE 2013 SP1/mkl/lib/intel64/mkl_intel_thread.lib
    mkl_lapack_LIBRARY=C:/Program Files (x86)/Intel/Composer XE 2013 SP1/mkl/lib/intel64/mkl_lapack95_ilp64.lib


Since the 64bit windows is assumed to be used here, to handle the array size bigger than 4GB, the ilp64 interface of MKL should be used. For more information, please check <http://software.intel.com/sites/products/documentation/hpc/mkl/lin/MKL_UG_structure/Support_for_ILP64_Programming.htm>

Hit configure
Hit Generate

Then in the C:/Libraries/armadillo-4.000.2/vc10, open the visual studio solution 'armadillo.sln', compile and install the Release version.

In the C:/Libraries/armadillo-4.000.2/install/include/armadillo_bits/config.hpp, uncomment #define ARMA_BLAS_LONG_LONG again. It is due to that the default config.hpp.cmake coming with the armadillo package disables the 64bit interface for BLAS and LAPACK.

Remember to add the path to your `PATH` environment variable. E.g. `C:/Libraries/armadillo-4.000.2/install/lib'.

##xsltproc
    
Download zlib-1.2.5.win32.zip, iconv-1.9.2.win32.zip, libxml2-2.7.8.win32.zip, libxslt-1.1.26.win32.zip from <http://xmlsoft.org/sources/win32/> 

Unzip all these four packages and create C:/Libraries/xslt

Copy following files from the unzipped packages to C:/Libraries/xslt

    iconv.dll
    libexslt.dll
    libxml2.dll
    libxslt.dll
    xsltproc.exe
    zlib1.dll

Add the `C:/Libraries/xslt' to your `PATH` environment variable.

##Gadgetron

Download and unpack Gadgetron source code

Assume the gadgetron is put into C:/Libraries/gadgetron

Create Visual Studio project (your process may vary):

Start cmake-gui

Select source (C:/Libraries/gadgetron) and target directories (C:/Libraries/gadgetron/build`)

Hit configure (first time) -- "ok" the dialogue box.

Add PATH variable BOOST_ROOT to point to BOOST folder (use GUI button "+Add Entry" to do this)

In our example, BOOST_ROOT=C:/Libraries/boost/boost_1_55_0    

Hit configure (again)

Specify location of FFTW and FFTWf libraries
  
    FFTW3F_LIBRARY=C:/Libraries/FFTW3/libfftw3f-3.lib
    FFTW3_INCLUDE_DIR=C:/Libraries/FFTW3
    FFTW3_LIBRARY=C:/Libraries/FFTW3/libfftw3-3.lib

Specify location of CULA

    CULA_CORE_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_core.lib
    CULA_INCLUDE_DIR=C:/Program Files/CULA/R16/include
    CULA_LAPACK_BASIC_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_lapack.lib
    CULA_LAPACK_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_lapack.lib
    CULA_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_core.lib

Specify location of ACE

    ACE_DEBUG_LIBRARY=C:/Libraries/ACE-6.2.0/ACE_wrappers/lib/ACEd.lib
    ACE_INCLUDE_DIR=C:/Libraries/ACE-6.2.0/ACE_wrappers
    ACE_LIBRARY=C:/Libraries/ACE-6.2.0/ACE_wrappers/lib/ACE.lib

Specify location of Armadillo

    ARMADILLO_INCLUDE_DIR=C:/Libraries/armadillo-4.000.2/install/include
    ARMADILLO_LIBRARY=C:/Libraries/armadillo-4.000.2/install/lib/armadillo.lib

Specify location of hdf5

    HDF5_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/cmake/hdf5
    HDF5_CXX_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5_cppdll.lib
    HDF5_C_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5dll.lib
    HDF5_C_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include
    HDF5_CXX_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include

Specify location of xsd and xercesc

    XSD_EXECUTABLE=C:/Program Files (x86)/CodeSynthesis XSD 3.3/bin/xsd.exe
    XSD_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include
    XERCESC_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include/xercesc
    XERCESC_LIBRARIES=C:/Program Files (x86)/CodeSynthesis XSD 3.3/lib64/vc-10.0/xerces-c_3.lib

Specify location of ismrmrd

    ISMRMRD_INCLUDE_DIR=C:/Program Files/ISMRMRD/ismrmrd/include
    ISMRMRD_LIBRARIES=C:/Program Files/ISMRMRD/ismrmrd/lib/ismrmrd.lib
    ISMRMRD_SCHEMA_DIR=C:/Program Files/ISMRMRD/ismrmrd/schema
    ISMRMRD_XSD_INCLUDE_DIR=C:/Libraries/ismrmrd-code/build/src/xsd
    ISMRMRD_XSD_SOURCE=C:/Libraries/ismrmrd-code/build/src/xsd/ismrmrd.cxx

Specify location of Google Test

    GTEST_INCLUDE_DIRS:PATH=C:/Libraries/gtest-1.7.0/include
    GTEST_LIBRARY=C:/Libraries/gtest-1.7.0/vc10/Release/gtest.lib
    GTEST_LIBRARY_DEBUG=C:/Libraries/gtest-1.7.0/vc10/Debug/gtest.lib
    GTEST_MAIN_LIBRARY=C:/Libraries/gtest-1.7.0/vc10/Release/gtest_main.lib
    GTEST_MAIN_LIBRARY_DEBUG=C:/Libraries/gtest-1.7.0/vc10/Debug/gtest_main.lib

Specify location of Matlab  

If you have matlab installed, the matlab wrapper mex files can be compiled:
e.g., if the matlab is installed at C:\MATLAB\R2012b,
    
    MATLAB_ENG_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libeng.lib
    MATLAB_INCLUDE_DIR=C:/MATLAB/R2012b/extern/include
    MATLAB_MAT_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmat.lib
    MATLAB_MEX_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmex.lib
    MATLAB_MX_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmx.lib
    MATLAB_UT_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libut.lib

Specify location of MKL

    MKLROOT_PATH=C:/Program Files (x86)/Intel/Composer XE 2013 SP1   
 
The FindMKL script shall find this path automatically. 
If your MKL is installed somewhere else, then specify it accordingly.


Hit configure (again)

Specify `NUMPY_INCLUDE_DIRS` = `C:/Python27/Lib/site-packages/numpy/core/include`

Hit configure (again)

Specify the following CMAKEFILEPATH variables if acml is used:
    
    BLAS_acml_LIBRARY= \
      C:/Libraries/acml4.4.0/win64/lib/libacml_dll.lib
    
    BLAS_acml_mp_LIBRARY= \
      C:/Libraries/acml4.4.0/win64_mp/lib/libacml_mp_dll.lib

Hit configure

Make sure that the HDF5_C_LIBRARY and HDF5_CXX_LIBRARY FILEPATH variables are set correctly (we have observed that they might be incorrectly set to point to the dll instead of the lib files by
default).

Hit configure

cmake cached variables:

USE_OPENMP : if checked, the gadgetron will be compiled with OpenMP enabled

BUILD_TOOLBOX_STATIC: if checked, variant toolboxes in gadgetron will be compiled as static libraries

Hit generate

You should now have a visual studio project that you can open and build (try Release/x64 mode and try the install target). If you are lacking sufficient write permission to install in the default location, run Visual Studio as Administrator or change CMAKE_INSTALL_PREFIX to a folder to which you have write permissions. Notice that /gadgetron is automatically appended to the path you specify.

If your working folder is the same as 'C:/Libraries', the following CMake command line can be used to generate the visual studio solution:

    cmake "-G" "Visual Studio 10 Win64" "-DBOOST_ROOT=C:/Libraries/boost/boost_1_55_0" "-DHDF5_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/cmake/hdf5"  "-DHDF5_CXX_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5_cpp.lib" "-DHDF5_C_LIBRARY=C:/Program Files/HDF Group/HDF5/1.8.12/lib/hdf5.lib" "-DHDF5_C_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include" "-DHDF5_CXX_INCLUDE_DIR=C:/Program Files/HDF Group/HDF5/1.8.12/include" "-DFFTW3_INCLUDE_DIR=C:/Libraries/FFTW3" "-DFFTW3F_LIBRARY=C:/Libraries/FFTW3/libfftw3f-3.lib"  "-DFFTW3_LIBRARY=C:/Libraries/FFTW3/libfftw3-3.lib"  "-DCULA_CORE_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_core.lib" "-DCULA_INCLUDE_DIR=C:/Program Files/CULA/R16/include" "-DCULA_LAPACK_BASIC_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_lapack.lib" "-DCULA_LAPACK_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_lapack.lib" "-DCULA_LIBRARY=C:/Program Files/CULA/R16/lib64/cula_core.lib" "-DACE_DEBUG_LIBRARY=C:/Libraries/ACE-6.2.0/ACE_wrappers/lib/ACEd.lib" "-DACE_INCLUDE_DIR=C:/Libraries/ACE-6.2.0/ACE_wrappers" "-DACE_LIBRARY=C:/Libraries/ACE-6.2.0/ACE_wrappers/lib/ACE.lib"  "-DARMADILLO_INCLUDE_DIR=C:/Libraries/armadillo-4.000.2/install/include" "-DARMADILLO_LIBRARY=C:/Libraries/armadillo-4.000.2/install/lib/armadillo.lib" "-DISMRMRD_INCLUDE_DIR=C:/Libraries/install/ismrmrd/include" "-DISMRMRD_LIBRARIES=C:/Libraries/install/ismrmrd/lib/ismrmrd.lib" "-DISMRMRD_SCHEMA_DIR=C:/Libraries/install/ismrmrd/schema" "-DISMRMRD_XSD_INCLUDE_DIR=C:/Libraries/ismrmrd-code/build/src/xsd" "-DISMRMRD_XSD_SOURCE=C:/Libraries/ismrmrd-code/build/src/xsd/ismrmrd.cxx" "-DGTEST_INCLUDE_DIRS:PATH=C:/Libraries/gtest-1.7.0/include" "-DGTEST_LIBRARY=C:/Libraries/gtest-1.7.0/vc10/Release/gtest.lib" "-DGTEST_LIBRARY_DEBUG=C:/Libraries/gtest-1.7.0/vc10/Debug/gtest.lib" "-DGTEST_MAIN_LIBRARY=C:/Libraries/gtest-1.7.0/vc10/Release/gtest_main.lib" "-DGTEST_MAIN_LIBRARY_DEBUG=C:/Libraries/gtest-1.7.0/vc10/Debug/gtest_main.lib" "-DMATLAB_ENG_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libeng.lib" "-DMATLAB_INCLUDE_DIR=C:/MATLAB/R2012b/extern/include" "-DMATLAB_MAT_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmat.lib" "-DMATLAB_MEX_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmex.lib" "-DMATLAB_MX_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libmx.lib" "-DMATLAB_UT_LIBRARY=C:/MATLAB/R2012b/extern/lib/win64/microsoft/libut.lib" "-DMKLROOT_PATH=C:/Program Files (x86)/Intel/Composer XE 2013 SP1"  "-DXERCESC_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include/xercesc" "-DXERCESC_LIBRARIES=C:/Program Files (x86)/CodeSynthesis XSD 3.3/lib64/vc-10.0/xerces-c_3.lib" "-DXSD_EXECUTABLE=C:/Program Files (x86)/CodeSynthesis XSD 3.3/bin/xsd.exe" "-DXSD_INCLUDE_DIR=C:/Program Files (x86)/CodeSynthesis XSD 3.3/include" "-DSWIG_EXECUTABLE=C:/Libraries/swigwin-2.0.11/swig.exe" "-DCMAKE_INSTALL_PREFIX=C:/Libraries/install"  "C:/Libraries/gadgetron"    

Because the gadgetron is installed at 'C:/Libraries/install/gadgetron', GADGETRON_HOME therefore is 'C:/Libraries/install/gadgetron' in this example. 

After compiling and installing, please rename the file `GADGETRON_HOME/config/gadgetron.xml.example` to `GADGETRON_HOME/config/gadgetron.xml`

Before attempting to run any reconstructions, please set the environment variable `GADGETRON_HOME` to point to the installation folder of your Gadgetron installation and make sure that the paths of all dependencies are in your `PATH` environment variable.

If your working directory is not 'C:/Libraries/', the compilation process should be changed accordingly.

You now have a working installation of the Gadgetron in Windows. Follow the instructions below to run a simple reconstruction example ([Gadgetron Hello World])