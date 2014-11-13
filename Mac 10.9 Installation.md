DANGER! THIS PAGE IS A WORK IN PROGRESS

Installing ISMRMRD, Gadgetron, GE-Tools, Siemens-Tools on OSX 10.9 Mavericks


This is the Homebrew version of the installation:
  /usr/local is owned by the main user
  root (and sudo) are not used
  Homebrew's python is installed

Using Matlab R20013b and Java 7 from Oracle

GPU features are NOT supported

Tested by S. Inati on a mid-2013 MacBook Air with 10.9.1 on 2014-02-03

Prerequisites
-------------
0. Install XCode and its command line tools

1. Install Homebrew and check that it's working
  ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)"
  brew doctor

2. Set some environment variables in ~/.bash_profile

# Put homebrew paths first
export PATH=/usr/local/bin:/usr/local/sbin:$PATH

# Python
export PYTHONHOME=/usr/local/Frameworks/Python.Framework/Versions/2.7
export PYTHONPAT=$PYTHONHOME/lib/python2.7/site-packages
export PYTHONPATH=/usr/local/opt/libxml2/lib/python2.7/site-packages:$PYTHONPATH

# MATLAB, ISMRMRD, GADGETRON
export MATLAB_HOME=/Applications/MATLAB_R2013b.app
export ISMRMRD_HOME=/usr/local/ismrmrd
export GADGETRON_HOME=/usr/local/gadgetron

export PATH=$MATLAB_HOME/bin:$PATH
export PATH=$ISMRMRD_HOME/bin:$PATH
export PATH=$GADGETRON_HOME/bin:$PATH

export DYLD_FALLBACK_LIBRARY_PATH=$ISMRMRD_HOME/lib
export DYLD_FALLBACK_LIBRARY_PATH=$GADGETRON_HOME/lib:$DYLD_FALLBACK_LIBRARY_PATH
export DYLD_FALLBACK_LIBRARY_PATH=$DYLD_FALLBACK_LIBRARY_PATH:$MATLAB_HOME/bin/maci64

2. Add the brew science formulae and the python formulae
  brew tap homebrew/science
  brew tap Homebrew/python

3. Install ISMRMRD and Gadgetron dependencies
  brew install wget
  brew install cmake
  brew install qt
  brew install hdf5 --enable-cxx
  brew install fftw
  brew install swig
  brew install glew
  brew install xerces-c
  brew install ace
  brew install armadillo
  brew install dcmtk
  brew install doxygen
  brew install docbook-xsl
  brew install fop

  # The python stuff needs to be installed before some other things
  # to make sure that we link against the brew python
  brew install python
    (double check that /usr/local/bin/python and
     /usr/local/bin/pip are first in your path)
  brew install boost --build-from-source
  brew install numpy
  brew install scipy
  brew install matplotlib --with-pygtk --with-pyqt
  brew install libxml2 --with-python
  pip install pyxb

4. Install the binary distribution of CodeSynthesis XSD
  wget http://www.codesynthesis.com/download/xsd/3.3/macosx/i686/xsd-3.3.0-i686-macosx.tar.bz2
  tar -xzf xsd-3.3.0-i686-macosx.tar.bz2
  cd xsd-3.3.0-i686-macosx
  cp bin/xsd /usr/local/bin/
  cp -r libxsd/xsd /usr/local/include/

  fix the bug in /usr/local/include/xsd/cxx/zc-istream.txx. 
  Change line 35 from:
    setg (b, b, e);
  to:
    std::streambuf::setg (b, b, e);

5. Install GTest
  wget https://googletest.googlecode.com/files/gtest-1.7.0.zip
  unzip gtest-1.7.0.zip
  cp -r gtest-1.7.0/include/gtest /usr/local/include/.
  cd gtest-1.7.0
  mkdir build
  cd build
  cmake ..
  make
  cp libgtest* /usr/local/lib/.

6. Install CUDA and CULA
  NOTE: THIS PART IS STILL BROKEN - DON'T DO IT!
  CUDA 5.5
  CULA cula_dense_free_R16a-osx

7. Make sure you are using the same JDK as the Matlab Java version.
  start matlab
  version -java
  For Matlab R2013b I used http://download.oracle.com/otn-pub/java/jdk/7u51-b13/jdk-7u51-macosx-x64.dmg

ISMRMRD
-------
git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
cd ismrmrd-code
mkdir build
cd build/
cmake ../
make
make install

Gadgetron
----------
git clone git://git.code.sf.net/p/gadgetron/gadgetron gadgetron-code
cd gadgetron-code
mkdir build
cd build/
cmake ../
make
make install

Post-install Config and Testing
-------------------------------
3. Set the environment variables