The following is a list of steps we have used to install the Gadgetron on a clean Microsoft Windows Server 2012 R2 Base (64-bit) machine.

As indicated in [[List of Dependencies]], the Gadgetron relies on multiple external libraries. Many of those libraries are not available as easy install packages and must be compiled separately. That is why we offer an option to download precompiled binaries of dependencies. 

If you are uncomfortable setting up development tools on Windows, or if you are just looking for a fast and easy way to get started with the Gadgetron, we recommend installing on Ubuntu Linux - possibly using a virtual machine inside Windows (see [[Linux Installation]])

## Installation instructions:

### Install required software: 

- Install Visual Studio 2013 with Update 4

- Install [Git](http://git-scm.com/downloads)

- Install [cmake](http://www.cmake.org/download/)

### Install dependencies:

- **ACE**:

    Download .zip file with precompiled binaries from the link: [ACE-6.3.0.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/ACE/zip/ACE-6.3.0.zip)

    Unzip the file to: C:\gadgetron\Dependencies\ACE.

    Set the environment variable: ACE_ROOT=C:\gadgetron\Dependencies\ACE\ACE-6.3.0\ACE_wrappers.

- **Armadillo**:

    Download .zip file with precompiled binaries from the link: [armadillo-4.600.2.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Armadillo/zip/armadillo-4.600.2.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\Armadillo.
    
    Set the environment variable: ARMA_HOME=C:\gadgetron\Dependencies\Armadillo\armadillo-4.600.2\install_vc12

- **Boost**:
    
    Download .zip file with precompiled binaries from the link: [boost_1_56_0.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Boost/zip/boost_1_56_0.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\Boost.
    
    Set the environment variable: BOOST_ROOT=C:\gadgetron\Dependencies\Boost\boost_1_56_0.

- **CUDA** (optional, but required for GPU support):

    NOTE: The lowest version of CUDA supported by VS2013 is 6.5.

    After running the installer and installing it to default location, set the environment variables (if they are not already set):

    CUDA_PATH=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v6.5
    CUDA_PATH_V6_5=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v6.5


- **MKL** (optional):

    NOTE: The lowest version of MKL supported by gadgetron is 11.2.0 

    After running the installer, set the environment variable (assuming that install location is C:\Intel):

    MKLROOT_PATH=C:\Intel\ComposerXE2015


- **dcmtk**:

    Download .zip file with precompiled binaries from the link: [dcmtk-3.6.0.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/dcmtk/zip/dcmtk-3.6.0.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\dcmtk.
    
    Set the environment variable: DCMTK_HOME=C:\gadgetron\Dependencies\dcmtk\dcmtk-3.6.0\install_vc12

- **FFTW**:

    Download .zip file with precompiled binaries from the link: [FFTW3.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/FFTW/zip/FFTW3.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\FFTW.
    
    Set the environment variable: FFTW3_ROOT_DIR=C:\gadgetron\Dependencies\FFTW\FFTW3

- **gtest**:

    Download .zip file with precompiled binaries from the link: [gtest-1.7.0.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/gtest/zip/gtest-1.7.0.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\gtest.
    
    Set the environment variable: GTEST_ROOT=C:\gadgetron\Dependencies\gtest\gtest-1.7.0

- **XSLT**:

    Download .zip file with precompiled binaries from the link: [XSLT.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/XSLT/zip/XSLT.zip)
    
    Unzip the file to: C:\gadgetron\Dependencies\XSLT.
    
    Set the environment variable: XSLT_ROOT=C:\gadgetron\Dependencies\XSLT\XSLT

- **HDF5**:

    Download .zip file from the link: [hdf5-1.8.14-win64-vs2013-shared.zip](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/HDF5/hdf5-1.8.14-win64-vs2013-shared.zip)
    
    Run the installer and install it to default location.
    
    Set the environment variable: HDF5_ROOT=C:\Program Files\HDF_Group\HDF5\1.8.14

- **Python**:

    Download installer and six executables from the following links:

    [Scipy-stack-14.5.30.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/Scipy-stack-14.5.30.win-amd64-py2.7.exe)

    [Twisted-14.0.0.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/Twisted-14.0.0.win-amd64-py2.7.exe)

    [libxml2-python-2.9.1.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/libxml2-python-2.9.1.win-amd64-py2.7.exe)

    [numpy-MKL-1.9.0b2.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/numpy-MKL-1.9.0b2.win-amd64-py2.7.exe)

    [psutil-2.1.1.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/psutil-2.1.1.win-amd64-py2.7.exe)

    [python-2.7.8.amd64.msi](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/python-2.7.8.amd64.msi)

    [zope.interface-4.1.1.win-amd64-py2.7.exe](https://s3.amazonaws.com/install-gadgetron-vs2013/Dependencies/Python/zope.interface-4.1.1.win-amd64-py2.7.exe)

    Run the installer and six executables from the Python folder.
    
    Add Python path (```C:\Python27```) to the 'PATH' environment variable


### Set additional environment variables:

Create the root folder for the gadgetron, ismrmrd and siemens_to_ismrmrd projects 
(for example: ```C:\gadgetron\projects```).

- Set the root folder environment variable:
 
    PROJECT_ROOT=C:\gadgetron\projects

- Set the environment variable for install folders for gadgetron and ismrmrd:

    GADGETRON_HOME=C:\gadgetron\projects\install\gadgetron
    ISMRMRD_HOME=C:\gadgetron\projects\install

### Clone repositories:

**ISMRMRD:**

Go to C:\gadgetron\projects:
```
git clone https://github.com/ismrmrd/ismrmrd.git
```
**GADGETRON:**

Go to C:\gadgetron\projects:
```
git clone https://github.com/gadgetron/gadgetron.git
```
**SIEMENS_TO_ISMRMRD:**

Go to C:\gadgetron\projects:
```
git clone https://github.com/nih-fmrif/siemens_to_ismrmrd.git
```

### Compile ISMRMRD, GADGETRON and SIEMENS_TO_ISMRMRD projects:

Download following scripts and save them in the same folder:

[gadgetron_windows_setup.bat](https://s3.amazonaws.com/install-gadgetron-vs2013/gadgetronScripts/gadgetron_windows_setup.bat)

[compile_ismrmrd.bat](https://s3.amazonaws.com/install-gadgetron-vs2013/gadgetronScripts/compile_ismrmrd.bat)

[compile_gadgetron.bat](https://s3.amazonaws.com/install-gadgetron-vs2013/gadgetronScripts/compile_gadgetron.bat)

[compile_siemens_to_ismrmrd.bat](https://s3.amazonaws.com/install-gadgetron-vs2013/gadgetronScripts/compile_siemens_to_ismrmrd.bat)

[set_gadgetron_path.bat](https://s3.amazonaws.com/install-gadgetron-vs2013/gadgetronScripts/set_gadgetron_path.bat)

Compile projects by running the scripts:
- **ISMRMRD:** Run the script: ```compile_ismrmrd.bat```
- **GADGETRON:** Run the script: ```compile_gadgetron.bat```
- **SIEMENS_TO_ISMRMRD:** Run the script: ```compile_siemens_to_ismrmrd.bat```

Run the script ```set_gadgetron_path.bat``` to set the path variable

Copy the ```xsltproc.exe``` from ```C:\gadgetron\dependencies\XSLT\binaries``` to the location of the ```siemens_to_ismrmrd.exe``` file (```C:\gadgetron\projects\install\bin```)


***

You now have a working installation of the Gadgetron in Windows. Follow the instructions below to run a simple reconstruction example [[Gadgetron Hello World]]