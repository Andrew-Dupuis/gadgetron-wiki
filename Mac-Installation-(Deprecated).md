The following instructions assume that you are starting on a Mac with OS X 10.8.5 (Mountain Lion) installed. Additionally it assumes that you have Xcode (3.2.6) installed. If you you are on a different release, you may need to make some adjustments.

We use MacPorts (<http://www.macports.org/>) to install the required dependencies. You may use a different package management system or prefer to install packages manually. In that case, please look at the [List of Dependencies] and install the required dependencies for the components you would like to use.

If you are on OS X 10.9 (Mavericks), you can follow the instructions at the [Mac 10.9 Installation] page, which use Homebrew instead of macports.

MacPorts is not the fastest way to install packages as several of them are are compiled locally. We use this method here nonetheless to make it easier to follow the instructions. Please be patient when running the `port` commands.

-   Install MacPorts.

    Download `MacPorts-2.2.1-10.8-MountainLion` from <http://www.macports.org/>.

    Run `sudo port -v selfupdate` to make sure you are up to date.

-   Get your Python installation up to date. Mac OS X ships with Python installed, but it is not a complete distribution. You need to update it if you would like to do Python development with the Gadgetron. If you already have numpy and SciPy installed, you may be able to skip this step. If you do not wish to use Python, you can also skip this step:
    
        sudo port install python27 py27-numpy py27-scipy py27-libxml2

    This should install Python 2.7. Now select Python 2.7 as as the active Python installation:
    
        sudo port select python python27

    To make sure the build system finds the right version of Python we need to edit a couple of symbolic links manually:

        cd /System/Library/Frameworks/Python.framework/Versions
        sudo ln -s /opt/local/Library/Frameworks/Python.framework/Versions/2.7
        sudo rm Current
        sudo ln -s 2.7 Current

-   Install Boost. Boost gets special treatment here. Depending on whether you would like to do Python development, you need to install Boost with or without boost\_python. If you would like Python:

        sudo port install boost +python27

    If you don't need Python support:

        sudo port install boost

-   Now we can install the rest of the packages:

        sudo port install git-core cmake libACE fftw-3-single fftw-3 qt4-mac-devel hdf5-18 libxml2 xercesc3 armadillo

    This may take quite a long time (hours).

-   Download, compile, and install ISMRMRD. Detailed instructions can be found at <http://ismrmrd.github.io>:

        git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
        cd ismrmrd-code/
        mkdir build
        cd build/
        cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_COMPILER=g++ -DCMAKE_C_COMPILER=gcc -DPYTHON_LIBRARY=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/lib/libpython2.7.dylib     -DPYTHON_INCLUDE_DIR=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/Headers     -DPYTHON_NUMPY_INCLUDE_DIR=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/lib/python2.7/site-packages/numpy/core/include -DJAVA_INCLUDE_DIR=/System/Library/Frameworks/JavaVM.framework/Headers ../
        make
        sudo make install

    Last command will install the library in `/usr/local/ismrmrd`.

    Make sure that `/usr/local/ismrmrd/lib` is in your `DYLD_LIBRARY_PATH` environment variable (see below).

-   To visualize HDF5 files you may also want to install HDFView from <http://www.hdfgroup.org/ftp/HDF5/hdf-java/hdfview/hdfview_install_macosx_intel64.zip>

-   Install CUDA and CULA. If you would like to use the GPU components, you need to install the following:

    -   The Nvidia development driver (`cuda_5.5.20_mac_64.pkg`) from <https://developer.nvidia.com/cuda-downloads>.

    -   The CULA Dense Libraries (`cula_dense_free_R16-osx.dmg`) from <http://www.culatools.com/downloads/dense>.

-   Compiling the Gadgetron:
 
        cd gadgetron
        mkdir build
        cd build
        cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_CXX_COMPILER=g++ -DCMAKE_C_COMPILER=gcc     -DPYTHON_LIBRARY=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/lib/libpython2.7.dylib     -DPYTHON_INCLUDE_DIR=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/Headers     -DPYTHON_NUMPY_INCLUDE_DIR=/opt/local/Library/Frameworks/Python.Framework/Versions/Current/lib/python2.7/site-packages/numpy/core/include     -DJAVA_INCLUDE_DIR=/System/Library/Frameworks/JavaVM.framework/Headers ../
        make
        sudo make install 

 
-   Set environment variables:

        export GADGETRON_HOME=/usr/local/gadgetron
        export ISMRMRD_HOME=/usr/local/ismrmrd
        export PATH=$PATH:$GADGETRON_HOME/bin
        export PATH=$PATH:$ISMRMRD_HOME/bin
        export DYLD_FALLBACK_LIBRARY_PATH=$GADGETRON_HOME/lib:$DYLD_FALLBACK_LIBRARY_PATH
        export DYLD_FALLBACK_LIBRARY_PATH=$ISMRMRD_HOME/lib:$DYLD_FALLBACK_LIBRARY_PATH

    You may wish to add these lines to `~/.profile`, You may also want to add paths to CUDA and CULA libraries if you are using those:

 -   After compiling and installing, please rename the file
    `GADGETRON_HOME/config/gadgetron.xml.example` to
    `GADGETRON_HOME/config/gadgetron.xml`

-   Test your Gadgetron by following the instructions in [Gadgetron Hello World].
