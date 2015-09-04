# Installing ISMRMRD, Gadgetron, Siemens converter, and GE tools on Mac OS X.

This is the Homebrew version of the installation:

- `/usr/local` is owned by the main user
- `root` (and `sudo`) are not used
- Homebrew's python 2.7 is installed
- Using Matlab R2015b
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

    # MATLAB, ISMRMRD, GADGETRON
    export MATLAB_HOME=/Applications/MATLAB_R2015b.app
    export ISMRMRD_HOME=/usr/local
    export GADGETRON_HOME=/usr/local

    export PATH=$MATLAB_HOME/bin:$PATH
    export PATH=$ISMRMRD_HOME/bin:$PATH
    export PATH=$GADGETRON_HOME/bin:$PATH

    export DYLD_FALLBACK_LIBRARY_PATH=$ISMRMRD_HOME/lib:$DYLD_FALLBACK_LIBRARY_PATH
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

        pip install h5py PyXB

0.  Install ISMRMRD Python API:

        git clone https://github.com/ismrmrd/ismrmrd-python
        cd ismrmrd-python
        python setup.py install

0. Install ISMRMRD Python Tools:

        git clone https://github.com/ismrmrd/ismrmrd-python-tools
        cd ismrmrd-python-tools
        python setup.py install

0.  Install GTest

    ```
    wget https://github.com/google/googletest/archive/release-1.7.0.tar.gz
    tar xzf release-1.7.0.tar.gz
    cd googletest-release-1.7.0
    cp -r include/gtest /usr/local/include/
    mkdir build
    cd build
    cmake ..
    make
    cp libgtest* /usr/local/lib/
    ```

0. **Future Step**: Install CUDA and CULA

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
    cmake -D CMAKE_INSTALL_PREFIX=$GADGETRON_HOME -D PYTHON_LIBRARY=`python-config --prefix`/Python -D PYTHON_INCLUDE_DIR=`python-config --prefix`/Headers ../
    make
    make install

### Siemens converter (needed for Gadgetron integration tests)

    git clone https://github.com/ismrmrd/siemens_to_ismrmrd
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

0. Copy `$GADGETRON_HOME/share/gadgetron/config/gadgetron.xml.example` to `$GADGETRON_HOME/share/gadgetron/config/gadgetron.xml`

0. Fetch Gadgetron test data: run `python get_data.py` in `test/integration` in the Gadgetron source tree.

0. Run integration tests: run `python run_all_tests.py $ISMRMRD_HOME $GADGETRON_HOME test_cases.txt` in `test/integration` in the Gadgetron source tree.