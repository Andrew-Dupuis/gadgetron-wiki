# Installing ISMRMRD, Gadgetron, Siemens converter, and GE tools on Mac OS X.

This is the Homebrew version of the installation:

- `/usr/local` is owned by the main user
- `root` (and `sudo`) are not used
- Homebrew's python 2.7 is installed
- Using Matlab R2013b and Java 7 from Oracle
- GPU features are **NOT** supported

## Prerequisites

0.  Install [Xcode](https://developer.apple.com/xcode/downloads/)
0.  Install OS X Command Line Tools on the terminal:
    
    ```xcode-select --install```

0. Install [Homebrew](http://brew.sh/) and check that it's working
0. Set environment variables in `~/.bash_profile` or its equivalent

    ```
    # Put homebrew paths first
    export PATH=/usr/local/bin:/usr/local/sbin:$PATH

    # Python
    export PYTHONHOME=/usr/local/Frameworks/Python.framework/Versions/2.7
    export PYTHONPATH=$PYTHONHOME/lib/python2.7/site-packages
    export PYTHONPATH=/usr/local/opt/libxml2/lib/python2.7/site-packages:$PYTHONPATH

    # MATLAB, ISMRMRD, GADGETRON
    export MATLAB_HOME=/Applications/MATLAB_R2013b.app
    export ISMRMRD_HOME=/usr/local
    export GADGETRON_HOME=/usr/local

    export PATH=$MATLAB_HOME/bin:$PATH
    export PATH=$ISMRMRD_HOME/bin:$PATH
    export PATH=$GADGETRON_HOME/bin:$PATH

    export DYLD_FALLBACK_LIBRARY_PATH=$ISMRMRD_HOME/lib
    export DYLD_FALLBACK_LIBRARY_PATH=$GADGETRON_HOME/lib:$DYLD_FALLBACK_LIBRARY_PATH
    export DYLD_FALLBACK_LIBRARY_PATH=$DYLD_FALLBACK_LIBRARY_PATH:$MATLAB_HOME/bin/maci64
    ```

0. Add the brew science formulae and the python formulae

        brew tap homebrew/science
        brew tap Homebrew/python

0. Install ISMRMRD and Gadgetron dependencies

        brew install wget cmake boost qt fftw swig glew ace armadillo dcmtk doxygen docbook-xsl fop
        brew install hdf5 --enable-cxx
  
        brew install python boost-python numpy scipy
        brew install matplotlib --with-pygtk --with-pyqt
        brew install libxml2 --with-python
        brew install libiconv

        pip install h5py pyxb Cython ismrmrd

0.  Install GTest

    ```
    wget https://googletest.googlecode.com/files/gtest-1.7.0.zip
    unzip gtest-1.7.0.zip
    cp -r gtest-1.7.0/include/gtest /usr/local/include/.
    cd gtest-1.7.0
    mkdir build
    cd build
    cmake ..
    make
    cp libgtest* /usr/local/lib/.
    ```

0. **Future Step**: Install CUDA and CULA

0.  Make sure you are using the same JDK as the Matlab Java version.
    In theory you can determine your version like so:

    - `javac -version` on the command line
    - `version -java` in Matlab

    For Matlab R2013b I used http://download.oracle.com/otn-pub/java/jdk/7u51-b13/jdk-7u51-macosx-x64.dmg

## Installation from Sources

### ISMRMRD

    git clone https://github.com/ismrmrd/ismrmrd
    cd ismrmrd
    mkdir build
    cd build/
    cmake -D CMAKE_INSTALL_PREFIX=$ISMRMRD_HOME ../
    make
    make install

### Gadgetron

    git clone https://github.com/gadgetron/gadgetron
    cd gadgetron
    mkdir build
    cd build/
    cmake -DCMAKE_INSTALL_PREFIX=$GADGETRON_HOME -DPYTHON_LIBRARY=`python-config --prefix`/Python -DPYTHON_INCLUDE_DIR=`python-config --prefix`/Headers ../
    make
    make install

### Siemens converter (needed for Gadgetron integration tests)

    git clone https://github.com/nih-fmrif/siemens_to_ismrmrd
    cd siemens_to_ismrmrd
    mkdir build
    cd build/
    cmake -D CMAKE_INSTALL_PREFIX=$ISMRMRD_HOME ../
    make
    make install

### GE tools (optional)

    git clone https://github.com/nih-fmrif/ge-tools
    cd ge-tools
    mkdir build
    cd build/
    cmake -D CMAKE_INSTALL_PREFIX=$ISMRMRD_HOME ../
    make
    make install

## Post-install Config and Testing

0. Copy `$GADGETRON_HOME/config/gadgetron.xml.example` to `$GADGETRON_HOME/share/gadgetron/config/gadgetron.xml`

0. Fetch Gadgetron test data: run `python get_data.py` in `test/integration` in the Gadgetron source tree.

0. Run integration tests: run `python run_all_tests.py $ISMRMRD_HOME $GADGETRON_HOME test_cases.txt` in `test/integration` in the Gadgetron source tree.