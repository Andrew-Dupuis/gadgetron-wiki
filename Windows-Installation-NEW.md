It is probably appropriate to start this section with a warning: Windows is not the easiest environment in which to work with the Gadgetron. As indicated in [List of Dependencies], the Gadgetron relies on multiple external libraries. Many of those libraries are not available as easy install packages and must be compiled separately. That is why we offer an option to download precompiled binaries of dependencies from our S3 bucket on AWS. If you are uncomfortable setting up development tools on Windows, or if you are just looking for a fast and easy way to get started with the Gadgetron, we recommend installing on Ubuntu Linux - possibly using a virtual machine inside Windows (see [Linux Installation]).

The following is a list of steps we have used to install the Gadgetron on a clean Microsoft Windows Server 2012 R2 Base (64-bit) machine. 

- Install Visual Studio 2012 and Update 4 for Visual Studio 2012

- Install Git

- Install cmake

- Install CUDA (optional, but required for GPU support)

    After running the installer and installing it to default location, set the envirnment variables (if they are not already set):

    ```CUDA_PATH=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v6.0```
    ```CUDA_PATH_V6_0=C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v6.0```

    Note: The setup scripts assume that version of CUDA installed is 6.0. If you decide to install different version of CUDA, update the scripts.

- Install MKL (optional)

    After running the installer, set the environment variable (assuming that install location is C:\Intel):

    MKLROOT_PATH=C:\Intel\ComposerXE2013SP1

### Copy the content of the S3 bucket to a ```C:\``` folder

### Install precompiled dependencies:

Go into ```C:\gadgetron\dependencies``` folder and do the following for each dependency (assuming that you copied content of the S3 bucket to ```C:\```)

- ACE:

    Set the environment variable: ```ACE_ROOT=C:\gadgetron\Dependencies\ACE\binaries\ACE_wrappers```

- Armadillo:

    Set the environment variable: ```ARMA_HOME=C:\gadgetron\Dependencies\Armadillo\binaries\armadillo-4.400.1\install_vc11```

- Boost:

    Set the environment variable: ```BOOST_ROOT=C:\gadgetron\Dependencies\Boost\binaries\boost_1_56_0```

- dcmtk:

    Set the environment variable: ```DCMTK_HOME=C:\gadgetron\Dependencies\dcmtk\binaries\dcmtk-3.6.0\install_vc11```

- FFTW:

    Set the environment variable: ```FFTW3_ROOT_DIR=C:\gadgetron\Dependencies\FFTW\binaries\FFTW3```

- gtest:

    Set the environment variable: ```GTEST_ROOT=C:\gadgetron\Dependencies\gtest\binaries\gtest-1.7.0```

- SWIG:

    Set the environment variable: ```SWIG_ROOT=C:\gadgetron\Dependencies\SWIG\swigwin-3.0.2\swigwin-3.0.2```

- XSLT:

    Set the environment variable: ```XSLT_ROOT=C:\gadgetron\Dependencies\XSLT\binaries```

- HDF5:

    Run the installer and install it to default location.
    Set the environment variable: ```HDF5_ROOT=C:\Program Files\HDF_Group\HDF5\1.8.13```

- Python:

    Run the installer and six executables from the Python folder.
    Add Python path (```C:\Python27```) to the 'PATH' environment variable


### Setting additional environment variables:

Create the root folder for the gadgetron, ismrmrd and siemens_to_ismrmrd projects 
(for example: ```C:\gadgetron\projects```).

Set the root folder environment variable:
 
```PROJECT_ROOT=C:\gadgetron\projects```

Set the environment variable for install folders for gadgetron and ismrmrd:

GADGETRON_HOME=C:\gadgetron\projects\install\gadgetron
ISMRMRD_HOME=C:\gadgetron\projects\install

### Cloning repositories:

ISMRMRD:

Go to C:\gadgetron\projects:
```
git clone https://github.com/ismrmrd/ismrmrd.git
cd ismrmrd
git checkout development
```
GADGETRON:

Go to C:\gadgetron\projects:
```
git clone https://github.com/gadgetron/gadgetron.git
cd gadgetron
git checkout development
```
SIEMENS_TO_ISMRMRD:

Go to C:\gadgetron\projects:
```
git clone https://github.com/nih-fmrif/siemens_to_ismrmrd.git
```
Compile ISMRMRD, GADGETRON and SIEMENS_TO_ISMRMRD projects using scripts that you downloaded from the S3 bucket:
- ISMRMRD: Run the script: compile_ismrmrd.bat
- GADGETRON: Run the script: compile_gadgetron.bat
- SIEMENS_TO_ISMRMRD: Run the script: compile_siemens_to_ismrmrd.bat

Run the script set_gadgetron_path.bat to set the path variable

Copy the xsltproc.exe from C:\gadgetron\dependencies\XSLT\binaries to the location of the siemens_to_ismrmrd.exe file (C:\gadgetron\projects\install\ismrmrd\bin)

You now have a working installation of the Gadgetron in Windows. Follow the instructions below to run a simple reconstruction example ([Gadgetron Hello World])