Gadgetron Users Guide

A Medical Image Reconstruction Framework

Dr. Michael Schacht Hansen

<michael.hansen@nih.gov>
Dr. Thomas Sangild Sørensen

<sangild@cs.au.dk>
Revision 1.1

<>
Table of Contents

WARNING
1. Introduction
1.1. What is the Gadgetron
1.2. Revision History
1.2.1. Version 1.1
1.2.2. Version 1.0
1.3. Obtaining Gadgetron
1.3.1. Dependencies
1.4. Compiling and Installing Gadgetron
1.4.1. Linux Installation Instructions
1.4.2. Mac OS X Installation Instructions
1.4.3. Windows Installation Instructions
1.5. Hello Gadgetron: Your First Image Reconstruction
2. Framework Overview
2.1. Gadgetron Streaming Architecture
2.1.1. Gadgets
2.1.2. Readers and Writers
2.1.3. Stream Configuration
2.1.4. Communication Sequence
2.1.5. File Organization
2.2. Gadgetron Toolboxes
2.2.1. NDArray
2.2.2. vector_td
2.2.3. complext
2.2.4. Fourier Transforms
2.2.5. Linear (Matrix) Operators
2.2.6. Linear Solvers
2.2.7. Non-linear Solvers
2.3. Gadgetron Gadgets
2.3.1. MRI Gadgets
2.3.2. Python Gadgets
2.3.3. Making a new Gadget Library
2.4. Gadgetron Clients
2.4.1. Available Clients
2.4.2. Making a new Client
3. Gadgetron Applications
3.1. Basic 2D FFT MRI
3.2. Cartesian 2D Parallel MRI (GRAPPA)
3.3. Non-Cartesian 2D Parallel MRI (SENSE)
4. Standalone Applications
4.1. Image Denoising
4.2. Image Deblurring
4.3. Non-Cartesian FFT
4.4. Non-Cartesian parallel MRI (SENSE)
5. Frequently Asked Questions (FAQ)
A. Simple Array File Format
B. HDF5 Files
C. Future Features
Bibliography
List of Figures

2.1. Gadgetron Architecture
2.2. Gadget
2.3. Overview of Python Prototyping
2.4. Result from ThresholdGadget experiment
3.1. Simple 2D FT Reconstruction Chain
3.2. GRAPPA Reconstruction Chain
3.3. GRAPPA Reconstruction Results
3.4. Gadgetron Chain for Non-Cartesian Sense
3.5. Non-Cartesian Sense Reconstruction Results
4.1. A noisy version of the Shepp-Logan phantom
4.2. Result after total variation denoising
4.3. Blurry Shepp-Logan phantom
4.4. Deblurred phantom from the Conjugate Gradient solver
4.5. Deblurred phantom from the constrained Split Bregman solver
A.1. Simple Array File Format
B.1. Examining Data with HDFView
B.2. Viewing Images in HDF5 Files
B.3. Setting for viewing HDF5 output images.
WARNING

This manual is currently in revision from version 1.0 to 1.1. Not all sections are updated yet.

Chapter 1. Introduction

Table of Contents

1.1. What is the Gadgetron
1.2. Revision History
1.2.1. Version 1.1
1.2.2. Version 1.0
1.3. Obtaining Gadgetron
1.3.1. Dependencies
1.4. Compiling and Installing Gadgetron
1.4.1. Linux Installation Instructions
1.4.2. Mac OS X Installation Instructions
1.4.3. Windows Installation Instructions
1.5. Hello Gadgetron: Your First Image Reconstruction
1.1. What is the Gadgetron

The Gadgetron is a streaming data processing framework for medical image reconstruction. It has been developed to make it easier to prototype, test, and deploy new image reconstruction algorithms.

The framework features a number of reconstruction applications that can be employed directly. Moreover, it contains a wide range of toolboxes with common data structures and algorithms designed for a much broader use. These toolboxes can be used within the streaming framework to create new dedicated reconstruction components or used as shared libraries in standalone (or third party) applications.

This document serves as an introduction to the Gadgetron framework and provides some "getting started" examples of using it. A scientific paper is also available [HANSEN12].

Although the Gadgetron is a generic, multi-modality image reconstruction framework, it was initially developed to support the work of the authors in the field of advanced MRI reconstruction. Specifically to support work on fast image reconstruction, not only on traditional CPU architectures, but also using commodity graphics hardware (GPUs). Some examples that are made publicly available through the Gadgetron framework include fast (re)gridding on the GPU [SANGILD08], Cartesian parallel imaging on the GPU [HANSEN08], and non-Cartesian parallel imaging on the GPU [SANGILD09].

1.2. Revision History

1.2.1. Version 1.1

Version 1.1 contains multip bug fixes and optimizations and some major structural changes (especially in the MRI specific Gadgets). Most notably, the Gadgetron now uses the proposed ISMRM Raw Data format (http://ismrmrd.github.io) throught the MRI specific Gadgets. A non-exhaustive list of changes can be found below:

The GadgetMessageAcquisition, GadgetMessageImage, etc. used to describe raw data and MRI images have been replaced with the corresponding classes from the ISMRMRD library.

There is now a Gadgetron configuration file (gadgetron.xml) used to control the port number of the Gadgetron when starting. That makes it easier to maintain the same port for a given installation without supplying it on the command line.

The dependency on TinyXML has been almost entirely removed. We are now using a class representations of headers and configuration generated with CodeSynthesis XSD (http://www.codesynthesis.com/products/xsd/).

All XML representations now have schema definitions to make it easier to validate configuration files etc.

Toolbox updates and bug fixes.

1.2.2. Version 1.0

First release of the Gadgetron

1.3. Obtaining Gadgetron

The Gadgetron is made available as a cross-platform source code distribution, which compiles and has been tested to run on Linux, Mac OS X, and Windows 7. Compilation instructions for these platforms are provided below.

Generally speaking, the Gadgetron is easiest set up on Linux since all dependencies are readily available. If you want to get started quickly with the Gadgetron and happen to not be using Linux, it is easy to install Ubuntu (our preferred Linux distribution) in a virtual machine (e.g. VirtualBox, https://www.virtualbox.org/) and follow the Linux compilation instructions below.

The Gadgetron is available from the project Sourceforge website:

http://sourceforge.net/projects/gadgetron

This manual is available in HTML form at:

http://gadgetron.sourceforge.net/latest/manual/gadgetron_manual.html

Or in PDF form at:

http://gadgetron.sourceforge.net/latest/manual/gadgetron_manual.pdf

API documentation (generated with Doxygen) is available from:

https://gadgetron.github.io/api_master//

1.3.1. Dependencies

The Gadgetron depends on a number of libraries that can either be downloaded for free or that may already be part of the installation on your workstation. If you are working on a Linux platform you should be able to install all dependencies without compiling anything. The following is a list of the components that you will need. Some are optional.

To install these components please follow the platform specific installation instructions provided below (Section 1.4, “Compiling and Installing Gadgetron”).

CMake. The Gadgetron uses cmake for cross platform building. Available from http://www.cmake.org/cmake/resources/software.html.

ADAPTIVE Computing Environment (ACE). Available from http://www.cs.wustl.edu/~schmidt/ACE.html.

Boost C++ Libraries. Available from http://www.boost.org/.

FFT3W Library for Fast Fourier Transforms. Available from http://www.fftw.org/.

ISMRM Raw Data Library (http://ismrmrd.github.io).

CodeSynthesis XSD (http://www.codesynthesis.com/products/xsd/).

BLAS and LAPACK (optional). Most Linux distributions come with these libraries and they are included on Mac OS X as well, but the vendor depends on your distribution and platform. See specific instructions below for Windows.

HDF5 (optional). Some of the the Gadgetron clients use the HDF5 file format for storing raw data and images. The libraries are available with most Linux distributions, and can be downloaded for Windows and Mac OS X from http://www.hdfgroup.org/HDF5/.

CUDA (optional). For GPU support, you need CUDA from Nvidia, which can de downloaded from http://developer.nvidia.com/cuda-downloads. You will need a CUDA driver for your graphics card too, which is available from the same website.

CULA (optional). We use CULA for LAPACK routines on the GPU. This is the only dependency which is not Open Source. You can however download a free version of CULA from http://www.culatools.com/downloads/dense/. Registration is required.

QT4 (optional). A few standalone and Gadgetron client example applications use QT for creating user interfaces. It comes packaged with most Linux distributions, but can otherwise be obtained from http://qt.nokia.com/ for all platforms.

Doxygen (optional). If you would like to build the API documentation you need Doxygen. It is available from http://www.stack.nl/~dimitri/doxygen/.

Docbook (optional). If you would like to build the manual (this document) you need Docbook. A number of tools are needed such as xsltproc and fop (for the PDF version of the library). Additionally you need the Docbook stylesheets, available at http://docbook.sourceforge.net/.

Git (optional). We use git to manage our source code archives. You can use any source code management system you prefer (or none at all), but if you would like to stay in line with the Gadgetron team, use git. Available from http://git-scm.com/.

1.4. Compiling and Installing Gadgetron

1.4.1. Linux Installation Instructions

Linux is the preferred operating system to get started using the Gadgetron. All of the required dependencies are included in most major Linux distributions and can be installed easily and without having to compile anything. In the following sections we walk you through the required steps to set up a full Gadgetron installation. We assume that you are starting with a freshly installed Ubuntu 12.04 available from the Ubuntu website (http://www.ubuntu.com). If you don't have a machine available for installing Ubuntu, you can always try it out in a virtual machine using virtualization software such as VirtualBox (https://www.virtualbox.org/).

If you would like to use the GPU components included in the Gadgetron and you have an Nvidia GPU available on your system, please complete the CUDA/CULA installations as described in Section 1.4.1.1, “Installing GPU components (CUDA and CULA) on Linux”.

First install all dependencies for Gadgetron. The following will install everything you need:

user@mycomputer:~$ sudo apt-get install doxygen cmake \
 libqt4-dev libglew1.6-dev \
 docbook5-xml docbook-xsl-doc-pdf \
 docbook-xsl-doc-html docbook-xsl-ns xsltproc \
 fop git-core libboost-dev libboost-python-dev \
 libfftw3-dev libace-dev python-dev python-numpy \
 freeglut3-dev libxi-dev liblapack-dev build-essential \
 libhdf5-serial-dev h5utils hdf5-tools hdfview \
 libboost-system-dev libboost-thread-dev xsdcxx \
 libxerces-c-dev   
First download, compile, and install ISMRMRD (there are more detailed instructions on the http://ismrmrd.github.io website):

  git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
  cd ismrmrd-code/
  mkdir build
  cd build
  cmake ../
  make
  sudo make install
Last command will install the library in /usr/local/ismrmrd.

Now download the Gadgetron archive and compile it. If you have access to a git repository, you can get the code with:

  git clone git://git.code.sf.net/p/gadgetron/gadgetron gadgetron
Configure and build the Gadgetron:

  cd gadgetron/
  mkdir build
  cd build
  cmake ../
  make  
Install (default location is /usr/local/gadgetron):

user@mycomputer:~/gadgetron/build$ sudo make install 
The final step is to add/modify a few environment variables in your ~/.bashrc file.

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/gadgetron/lib:/usr/local/ismrmrd/lib
export PATH=$PATH:/usr/local/gadgetron/bin:/usr/local/ismrmrd/bin
export GADGETRON_HOME=/usr/local/gadgetron     
Rename the example configuration file GADGETRON_HOME/config/gadgetron.xml.example to GADGETRON_HOME/config/gadgetron.xml

You are now set up to run a simple example reconstruction as outlined in Section 1.5, “Hello Gadgetron: Your First Image Reconstruction”.

1.4.1.1. Installing GPU components (CUDA and CULA) on Linux

First install the Nvidia driver. The Ubuntu distribution comes with a driver that will work with CUDA in some instances, but we recommend that you install the latest developer driver from the Nvidia website.

Download (to the current directory) devdriver_4.2_linux_64_295.41.run from http://developer.nvidia.com/cuda/cuda-downloads and install the driver:

sudo sh ./devdriv(er_4.2_linux_64_295.41.run
The process of getting this driver installed may vary from installation to installation. Specifically, you may need to remove any existing Nvidia driver before installing and you will have to shut down the display manager before installing.

The display manager can be shut down with:

sudo service lightdm stop
Important notice: Unfortunately we have experienced on several Ubuntu installations that the machine hangs at the splash screen after installation of the Nvidia graphics driver. If you experience this problem, or if you just want to be on the safe side before rebooting, open a terminal (boot in recovery mode) and edit /etc/default/grub. Locate the line defining GRUB_CMDLINE_LINUX_DEFAULT and change splash to nosplash (or add nosplash if splash is not present). Furthermore add nomodeset. E.g. GRUB_CMDLINE_LINUX_DEFAULT="quiet nosplash nomodeset". Finally update the boot manager with the new settings: sudo update-grub

Next we need to install gcc 4.4 since Ubuntu comes preconfigured with gcc 4.6, which is not compatible with the current versions of the CUDA nvcc compiler.

sudo apt-get install gcc-4.4 g++-4.4 build-essential   
Set up alternative systems to allow easy switching between the two versions of gcc/g++

sudo update-alternatives --install /usr/bin/gcc gcc \
 /usr/bin/gcc-4.6 40 --slave /usr/bin/g++ g++ /usr/bin/g++-4.6

sudo update-alternatives --install /usr/bin/gcc \
 gcc /usr/bin/gcc-4.4 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.4    
Check your gcc compiler (should now be version 4.4.7):

gcc -v      
When you want to switch between the two compiler versions:

sudo update-alternatives --config gcc      
The final step is to actually install CUDA and CULA. Download the following files:

cudatoolkit_4.2.9_linux_64_ubuntu11.04.run from http://developer.nvidia.com/cuda/cuda-downloads

cula_dense_free_R15-linux64.run from http://www.culatools.com/downloads/dense/ (free registration requited)

Go to the folder where the files were downloaded and type:

sudo sh ./cudatoolkit_4.2.9_linux_64_ubuntu11.04.run
sudo sh ./cula_dense_free_R15-linux64.run
Follow the instructions. When you are done with the installation you may want to add the following to your ~/.bashrc file.

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib
export CULA_ROOT="/usr/local/cula"
export CULA_INC_PATH="$CULA_ROOT/include"
export CULA_BIN_PATH_32="$CULA_ROOT/bin"
export CULA_BIN_PATH_64="$CULA_ROOT/bin64"
export CULA_LIB_PATH_32="$CULA_ROOT/lib"
export CULA_LIB_PATH_64="$CULA_ROOT/lib64"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CULA_LIB_PATH_64    
You are now ready to compile and run CUDA (and CULA) applications. You may want to download the CUDA SDK from Nvidia to validate your installation but this is not required.

1.4.2. Mac OS X Installation Instructions

The following instructions assume that you are starting on a Mac with OS X 10.6.8 (Snow Leopard) Installed. Additionally it assumes that you have Xcode (3.2.6) installed. If you have upgraded to Lion or are on an older release, you should still be able to make it all compile, but you may have to make some adjustments.

We use MacPorts (http://www.macports.org/) to install the required dependencies. You may use a different package management system or prefer to install packages manually. In that case, please look at the list of dependencies (Section 1.3.1, “Dependencies”) and install the required dependencies for the components you would like to use.

MacPorts is not the fastest way to install packages as they are compiled locally. We use this method here nonetheless to make it easier to follow the instructions. Please be patient when running the port commands.

Install MacPorts.

Download MacPorts-2.1.2.pkg from http://www.macports.org/.

Run sudo port -v selfupdate to make sure you up to date.

Get your Python installation up to date. Mac OS X ships with Python installed, but it is not a complete distribution. You need to update it if you would like to do Python development with the Gadgetron. If you already have numpy and SciPy installed, you may be able to skip this step. If you do not wish to use Python, you can also skip this step.

sudo port install python27 py27-numpy py27-scipy py27-libxml2
This should install Python 2.7. Now select Python 2.7 as as the active Python installation:

sudo port select python python27
To make sure the build system finds the right version of Python we need to edit a couple of symbolic links manually:

cd /System/Library/Frameworks/Python.framework/Versions
sudo ln -s /opt/local/Library/Frameworks/Python.framework/Versions/2.7
sudo rm Current
sudo ln -s 2.7 Current
Install Boost. Boost gets special treatment here. Depending on whether you would like to do Python development, you need to install Boost with or without boost_python. If you would like Python:

sudo port install boost +python27
If you don't need Python support:

sudo port install boost
Now we can install the rest of the packages:

sudo port install git-core cmake libACE \
fftw-3-single fftw-3 qt4-mac-devel hdf5-18 \
libxml2 xercesc3
This may take quite a long time (hours).

Install CodeSynthesis XSD (needed to compile ISMRM Raw Data Format)

wget http://www.codesynthesis.com/download/xsd/3.3/macosx/i686/xsd-3.3.0-i686-macosx.tar.bz2
tar -xzf xsd-3.3.0-i686-macosx.tar.bz2
cd xsd-3.3.0-i686-macosx
sudo cp bin/xsd /usr/local/bin/
sudo cp -r libxsd/xsd /usr/local/include/
More detailed instructions at http://ismrmrd.github.io.

Download, compile, and install ISMRMRD. Detailed instructions can be found at http://ismrmrd.github.io.

git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
cd ismrmrd-code/
mkdir build
cd build/
cmake ../
make
sudo make install
Last command will install the library in /usr/local/ismrmrd.

Make sure that /usr/local/ismrmrd/lib is in your DYLD_LIBRARY_PATH environment variable (see below).

To visualize HDF5 files you may also want to install HDFView from http://www.hdfgroup.org/ftp/HDF5/hdf-java/hdfview/hdfview_install_macosx_intel64.zip

Install CUDA and CULA. If you would like to use the GPU components, you need to install the following:

The Nvidia development driver (devdriver_4.2.10_macos.dmg) from http://developer.nvidia.com/cuda/cuda-downloads/.

The CUDA Toolkit (cudatoolkit_4.2.9_macos.pkg) from http://developer.nvidia.com/cuda/cuda-downloads.

The CULA Dense Libraries (cula_dense_free_R15-osx.dmg) from http://www.culatools.com/downloads/dense/.

Compiling the Gadgetron:

$ cd gadgetron
$ mkdir build
$ cd build
$ cmake -DPYTHON_NUMPY_INCLUDE_DIR= \
  /opt/local/Library/Frameworks/Python.framework \
  /Versions/2.7/lib/python2.7/site-packages/numpy/core/include ../
$ make
$ sudo make install
The long path for the numpy header files is only needed if you want Python support. You can avoid this by creating a symbolic link:

$ cd /opt/local/Library/Frameworks/Python.framework
$ cd Versions/2.7/include/python2.7
$ sudo ln -s ../../lib/python2.7/site-packages/numpy/core/include/numpy
After creating this link you should be able to compile with the following:

$ cd gadgetron
$ mkdir build
$ cd build
$ cmake ../
$ make
$ sudo make install
Set environment variables:

$ export GADGETRON_HOME=/usr/local/gadgetron
$ export PATH=$PATH:/usr/local/gadgetron/bin:/usr/local/ismrmrd/bin
$ export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/usr/local/gadgetron/lib:/usr/local/ismrmrd/lib
You may wish to add these lines to ~/.bash_profile, You may also want to add paths to CUDA and CULA libraries if you are using those:

$ export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/usr/local/cula/lib64
$ export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/usr/local/cuda/lib
After compiling and installing, please rename the file GADGETRON_HOME/config/gadgetron.xml.example to GADGETRON_HOME/config/gadgetron.xml

Test your Gadgetron by following the instructions in Section 1.5, “Hello Gadgetron: Your First Image Reconstruction”.

1.4.3. Windows Installation Instructions

It is probably appropriate to start this section with a warning: Windows is not the easiest environment in which to work with the Gadgetron. As indicated in Section 1.3.1, “Dependencies”, the Gadgetron relies on multiple external libraries. Many of those libraries are not available as easy install packages and must be compiled separately. If you are uncomfortable setting up development tools on Windows, or if you are just looking for a fast and easy way to get started with the Gadgetron, we recommend installing on Ubuntu Linux - possibly using a virtual machine inside Windows (see Section 1.4.1, “Linux Installation Instructions”).

The following is a list of steps we have used to install the Gadgetron on a clean Windows 7 (64-bit) machine. It has also been tested sucessfully on a 32 bit machine (but in this case you choose 32 bit packages/configuration where appropriate).

The Gadgetron distribution also includes a Windows Powershell Script in doc/windows_installation/GadgetronWindowsInstallation.ps1, which describes the command line steps for installing the dependencies. You cannot complete the installation by simply running the script. Most likely your Windows machine will not allow you to run the script the directly without changing security settings and the download of some of the dependencies cannot be automated since you have to log in (or provide an email address) when downloading. The script can serve as a guide and we recommend (if you would like to use the script) to open it in the Powershell ISE and execute it line-by-line (or section by section). The general installation steps are written out here too, you may need to make some adjustements for your particular setup.

Install Visual Studio 2010 (with Service Pack 1)

Install CUDA/CULA (optional, but required for GPU support).

Download Cuda drivers/toolkit from from

http://developer.nvidia.com/cuda/cuda-downloads/:

Install Nvidia Developer Driver (Version 301.32)

Install Nvdia Toolkit (4.2)

Install gpucomputingsdk

Install cula_dense_free_R15-win64.exe from

http://www.culatools.com/downloads/dense/.

Assuming CULA was installed in C:\Program Files\CULA\R145, add

C:\Program Files\CULA\R15\bin64 to your PATH environment variable.

Create a folder for external libraries, say C:\Libraries.

Install FFTW3 (http://www.fftw.org/install/windows.html)

Copy FFTW3 binaries to C:\Libraries\FFTW3

Create *.lib files, on the command line type:

c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3f-3.def
c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3-3.def
c:\Libraries\FFTW3>lib /machine:x64 /def:libfftw3l-3.def
Add C:\Libraries\FFTW3 to your PATH environment variable.

On 32 bit Windows remember to remove the /machine:x64 argument, the default is 32 bit.

Install ACE (http://download.dre.vanderbilt.edu/)

Unpack ACE into C:\Libraries\ACE-6.1.0\ACE_wrappers

Add config.h in ACE_ROOT/ace/ with the following content:

//We are on Windows
#include "ace/config-win32.h" 

//This ensured that INLINE settings 
//do not vary between Debug and Release modes
#define ACE_NO_INLINE 
Open the VS 2010 project in the source code archive

Set build type to Release/x64

Build (this takes a while)

Add to PATH envoronment variable: C:\Libraries\ACE-6.0.5\ACE_wrappers\lib

Install Python (optional).

Regrettably, the off-the-shelf Python header files cannot be compiled in debug mode on Windows. This enforces the Gadgetron framework to be compiled in release mode only if you enable the Python components.

Install python-2.7.3.amd64 (http://www.python.org)

Add install folder (e.g. C:\Python27) to PATH environment variable

Add PYTHON_ROOT environment variable

From http://www.lfd.uci.edu/~gohlke/pythonlibs/ download and install the following (+ additional libraries that you may need for your Python development):

numpy-MKL-1.6.2.win-amd64-py2.7

scipy-0.10.1.win-amd64-py2.7

libxml2-python-2.7.8.win-amd64-py2.7

Install ACML (BLAS and LAPACK)

Download acml4.4.0-win64.exe from: http://developer.amd.com/downloads/acml4.4.0-win64.exe

Install Library in say C:\Libraries\acml4.4.0

Add C:\Libraries\acml4.4.0\win64\lib;C:\Libraries\acml4.4.0\win64_mp\lib to your PATH environment variable.

Notice. Newer versions of the ACML-library are available (version 5.2.0 at the time of preparing this manual) - however, these libraries are distributed without required dependencies and will not work out of the box. We recommend sticking to the earlier version 4.4.0 until these issues have been resolved.

Install the newest Boost release (http://www.boost.org)

We recommend using the precompiled binaries from BoostPro (e.g. http://boostpro.com/download/x64/boost_1_51_setup.exe)

Just install everything, you might need other components later.

Install git (if you are using source code management):

Run the newest installation package named something like Git-*-preview*.exe from: http://code.google.com/p/msysgit/

Use run in git bash only option

Use checkout Windows LF and commit Unix Line feeds

Install CMake (http://www.cmake.org/cmake/resources/software.html)

Install the latest release (e.g. http://www.cmake.org/files/v2.8/cmake-2.8.9-win32-x86.exe)

Install HDF5:

You will need the HDFView application to view data files used by the Gadgetron, it can be downloaded rom http://www.hdfgroup.org/HDF5/ install HDFView : hdfview_install_win64.exe

The precompiled binaries (e.g. http://www.hdfgroup.org/ftp/HDF5/current/bin/windows/HDF5189-win64-vs10-shared.zip) should work fine with the Gadgetron. Remember to add the path (e.g. C:\Program Files\HDF Group\HDF5\1.8.9\bin) to the HDF libraries to your PATH environment library.

Install CodeSynthesis XSD (http://www.codesynthesis.com/download/xsd/3.3/windows/i686/xsd-3.3.msi).

Remember to add the path to the XSD binaries to your PATH environment variable. E.g. C:\Program Files (x86)\CodeSynthesis XSD 3.3\bin\ and C:\Program Files (x86)\CodeSynthesis XSD 3.3\bin64\.

Download, compile, and install the ISMRM Raw Data format. Detailed instructions are available at (http://ismrmrd.github.io).

From a git bash shell:

git clone git://git.code.sf.net/p/ismrmrd/code ismrmrd-code
cd ismrmrd-code/
mkdir build
cd build/
cmake-gui.exe
Last command will open CMake's graphical user interface. Hit the configure button and deal with the dependencies that CMake is unable to find. Hit configure again and repeat the process until CMake has enough information to configure. Once the configuration is complete, you can hit generate to generate a Visual Studio project, which you can open and use to build ISMRMRD.

Download and unpack Gadgetron source code.

Create Visual Studio project (your process may vary):

Start cmake-gui

Select source ($GADGETRON_HOME) and target directories ($GADGETRON_HOME/build)

Hit configure (first time) -- "ok" the dialogue box.

Add PATH variable BOOST_ROOT to point to BOOST folder (use GUI button "+Add Entry" to do this)

Hit configure (again)

Specify location of FFTW and FFTWf libraries

Hit configure (again)

Specify the locations of GLEW and GLUT:

Header files in C:\ProgramData\NVIDIA Corporation\NVIDIA GPU COMPUTING SDK 4.1\shared\inc

Library files in C:\ProgramData\NVIDIA Corporation\NVIDIA GPU COMPUTING SDK 4.1\lib/x64

Specify location of CULA (include path and core/lapack library filepaths)

Specify location of ACE include dir

Hit configure (again)

Specify PYTHON_NUMPY_INCLUDE_DIR = C:/Python27/Lib/site-packages/numpy/core/include

Hit configure (again)

Specify the following CMAKE FILEPATH variables:

BLAS_acml_LIBRARY= \
  C:/Libraries/acml4.4.0/win64/lib/libacml_dll.lib

BLAS_acml_mp_LIBRARY= \
  C:/Libraries/acml4.4.0/win64_mp/lib/libacml_mp_dll.lib

Hit configure

Make sure that the HDF5_C_LIBRARY and HDF5_CXX_LIBRARY FILEPATH variables are set correctly (we have observed that they might be incorrectly set to point to the dll instead of the lib files by default).

Hit configure

Hit generate

You should now have a visual studio project that you can open and build (try Release/x64 mode and try the install target). If you are lacking sufficient write permission to install in the default location, run Visual Studio as Administrator or change CMAKE_INSTALL_PREFIX to a folder to which you have write permissions. Notice that /gadgetron is automatically appended to the path you specify.

After compiling and installing, please rename the file GADGETRON_HOME/config/gadgetron.xml.example to GADGETRON_HOME/config/gadgetron.xml

Before attempting to run any reconstructions, please set the environment variable GADGETRON_HOME to point to the installation folder of your Gadgetron installation and make sure that the paths of all dependencies are in your PATH environment variable.

You now have a working installation of the Gadgetron in Windows. Follow the instructions below to run a simple reconstruction example (Section 1.5, “Hello Gadgetron: Your First Image Reconstruction”).

1.5. Hello Gadgetron: Your First Image Reconstruction

Some basic sample datasets are available from the Sourceforge website:

http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/testdata/

You will generally encounter two types of data in this manual: a) Simple array format described in Appendix A, Simple Array File Format and b) ISMRMRD HDF5 files which are described in more detail at http://ismrmrd.github.io. It is beyond the scope of this manual to explain the HDF5 file format, but we have added a small introductory section in the appendix (Appendix B, HDF5 Files).

Download the file simple_gre.h5 from the website (on Linux simply type):

wget http://sourceforge.net/projects/gadgetron/files/testdata/ismrmrd/simple_gre.h5
Open two terminal windows to observe both client and Gadgetron communication output. In the Gadgetron terminal window simply type:

user@mycomputer:~/temp/gadgetron_out$ gadgetron  
In the client window (in the folder where you just downloaded the data) type:

user@mycomputer:~/temp/test_data$ mriclient \
    -d simple_gre.h5 \
    -c default.xml
You should now see some logging information both in the Gadgetron window and in the client window. Specifically, you should see that a connection is being made and when the reconstruction is done the client should shut down:

user@mycomputer:~/temp/test_data$ mriclient \
    -d simple_gre.h5 \  
    -c default.xml

Gadgetron MRI Data Sender
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      gadgetron_testdata.h5
  -- hdf5 group in   :      simple_gre
  -- conf            :      default.xml
  -- loop            :      1
  -- hdf5 file out   :      ./out.h5
  -- hdf5 group out  :      2012-05-11 12:52:14
(31540|140170355443520) Connection from 127.0.0.1:9002
31540, 81, GadgetronConnector, Close Message received
(31540|140170283570944) Handling close...
(31540|140170283570944) svc done...
(31540|140170283570944) Handling close...
The images are saved in the folder in which you started the mriclient. The client appends the result to an HDF5 file called out.h5 (if no other file name is specified). A group is created with the current time and data and the images are stored in that group. If you run multiple reconstructions one after another, the results will be added to the same file, but a new group is created for each run. That makes it easy to compare results from different reconstructions. The images are stored in a single precision format as specified by the default.xml configuration file. Please see Appendix B, HDF5 Files for details on how to read the output file. Briefly you could read and display the data in Matlab with:

images = h5read('out.h5','/<INSERT CORRECT DATE HERE>/image_0.img');
imagesc(images(:,:,1,1));colormap(gray);
Chapter 2. Framework Overview

Table of Contents

2.1. Gadgetron Streaming Architecture
2.1.1. Gadgets
2.1.2. Readers and Writers
2.1.3. Stream Configuration
2.1.4. Communication Sequence
2.1.5. File Organization
2.2. Gadgetron Toolboxes
2.2.1. NDArray
2.2.2. vector_td
2.2.3. complext
2.2.4. Fourier Transforms
2.2.5. Linear (Matrix) Operators
2.2.6. Linear Solvers
2.2.7. Non-linear Solvers
2.3. Gadgetron Gadgets
2.3.1. MRI Gadgets
2.3.2. Python Gadgets
2.3.3. Making a new Gadget Library
2.4. Gadgetron Clients
2.4.1. Available Clients
2.4.2. Making a new Client
2.1. Gadgetron Streaming Architecture

The Gadgetron consists of a streaming processing architecture and a set of toolboxes. The toolboxes are used within the streaming components but come as individual shared libraries and can thus also be used in standalone applications. The architecture is outlined in Figure 2.1, “Gadgetron Architecture”.

Figure 2.1. Gadgetron Architecture

Gadgetron Architecture

The Gadgetron receives connections from clients through a TCP/IP connection. A client can be any application from which you can open a TCP/IP socket and send data. Once a connection to a client has been established (see Section 2.1.4, “Communication Sequence”), the Gadgetron will read data from the socket and pass it on down a chain of processing steps. The responsibility of reading and writing packages on the socket is dispatched to a set of Readers and Writers (see Section 2.1.2, “Readers and Writers”). Each step in the processing chain is implemented in a module or Gadget (see Section 2.1.1, “Gadgets”). A reconstruction process is defined by defining a chain of Gadgets. The assembly of Gadgets is done dynamically at run-time (see Section 2.1.3, “Stream Configuration”).

2.1.1. Gadgets

A Gadget is the functional unit of the Gadgetron. You can think of the Gadget as a device with an input and output. Data passes through the device and is modified and/or transformed between input and output. By wiring multiple Gadgets together you create a reconstruction program. A schematic outline of a Gadget is seen in Figure 2.2, “Gadget”

Figure 2.2. Gadget

Gadget

The Gadget is an active object based on the ACE_Task from the ACE library. It has its own thread (or threads) of execution and an input queue where data is placed for processing by either the Gadgetron framework or an upstream Gadget.

The active thread(s) in the Gadget will pick up a data package from the queue, and then pass it on to a virtual process. An abbreviated version of the header Gadget.h is seen below:

class Gadget : public ACE_Task<ACE_MT_SYNCH>
{

public:
   virtual int svc(void)
   {
      //Pick up package from queue
     
      //Call process
      if (this->process(m) == -1) {
         //Handle error
      }
      return 0;
   }

   //More function (left out for simplicity)

protected:
   virtual int process(ACE_Message_Block * m) = 0;

   virtual int process_config(ACE_Message_Block * m) {
      return 0;
   }

};
The data package used by the ACE_Task is the ACE_Message_Block, which is a very basic block of data (essentially just a byte array). To allow the Gadgets to check if the data blocks on the message queue are of the expected type, the Gadgetron uses a modified ACE_Message_Block called GadgetContainerMessage, which can contain any class with a no-argument constructor. It is possible to check if the GadgetContainerMessage contains a specific type of data, and if so, access that object. Suppose we want to store a class named MyClass:

GadgetContainerMessage<MyClass>* m = 
  new GadgetContainerMessage<MyClass>();

MyClass* mc = m->getObjectPtr();

//Do something with mc

m->release(); //Delete the message block and containing data
When a function receives an ACE_Message_Block it is possible to check if it is of a certain type:

int process(ACE_Message_Block* mb)
{
  
  GadgetContainerMessage<MyClass>* m = 
    AsContainerMessage<MyClass>(mb);

  if (m) {
    MyClass* mc = m->getObjectPtr();
    
    //Do something with mc

  } else {
    //Something went wrong, deal with error
    return -1;
  }

  mb->release();

  return 0;
}
It is possible to chain more than one ACE_Message_Block together using the cont function. This effectively provides a way to pass multiple arguments into a Gadget and checking if they have the appropriate types:

int process(ACE_Message_Block* mb)
{
  
  GadgetContainerMessage<MyClass>* m1 = 
    AsContainerMessage<MyClass>(mb);

  GadgetContainerMessage<MyOtherClass>* m2 = 
    AsContainerMessage<MyOtherClass>(mb->cont());

  if (m1 && m2) {
    MyClass* mc = m1->getObjectPtr();
    MyOtherClass* moc = m2->getObjectPtr();
    
    //Do something with mc

  } else {
    //Something went wrong, deal with error
    return -1;
  }

  mb->release(); //This deletes both message blocks

  return 0;
}
It gets a bit tedious and error prone to repeat code like the above in every Gadget. To overcome this, the Gadgetron comes with a set of templated classes to automate the steps. Say we would like to make a Gadget which takes a single input argument, we would inherit from Gadget1. If you need two arguments, you inherit from Gadget2:

template <class P1, class P2> class Gadget2 : public Gadget
{
protected:
   int process(ACE_Message_Block* mb)
   {
     //Do type checking 
   }

   virtual int process(GadgetContainerMessage<P1>* m1, 
     GadgetContainerMessage<P2>* m2) = 0;
};
The base class performs the type checking for you and only when the arguments have been verified, it will call the virtual process above. So, all you need to do in order to implement a Gadget that takes two arguments is to implement this function. As an example, let's look at a very simple Gadget, which receives an image header (in ISMRM Raw Data format) and some image data and does a Fourier transform of the first 3 dimensions. First the header file FFTGadget.h

#include "gadgetroncore_export.h"
#include "Gadget.h"
#include "ismrmrd.h"
#include "hoNDArray.h"
#include <complex>

class EXPORTGADGETSCORE FFTGadget : 
public Gadget2<ISMRMRD::ImageHeader, hoNDArray< std::complex<float> > >
{
 public:
  GADGET_DECLARE(FFTGadget)

 protected:
  virtual int process( 
     GadgetContainerMessage< ISMRMRD::ImageHeader >* m1,
     GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2);

};
Let us walk through the code step by step. The Gadget takes two arguments: 1) GadgetMessageImage, which is just a struct with some image header information (it is defined in GadgetMRIHeaders.h), and 2) a hoNDArray, which is a multidimensional array (see Section 2.2.1, “NDArray”) storage container. In this case the hoNDArray contains complex floating point data.

There are a couple of other things to notice. One is the EXPORTGADGETSCORE macro in the class definition. This is needed to make things work properly on Windows. It is defined in gadgetroncore_export.h and is used (on Windows) to indicate if the class is being imported or exported from a DLL. It translates into __declspec(dllexport) or __declspec(dllimport) in Windows and is empty in Linux/OSX. It is beyond the scope of this manual to go into why such a declaration is needed, but keep this in mind when you start creating your own Gadgets. Each shared library (DLL) has its own export declaration macro.

The other thing to notice is the GADGET_DECLARE(FFTGadget) macro. This macro is required for Windows to correctly handle shared libraries and is needed whenever you create a new Gadget to make things work properly on Windows.

The actual implementation looks like this:

#include "FFTGadget.h"
#include "FFT.h"

int FFTGadget::process( 
   GadgetContainerMessage< ISMRMRD::ImageHeader >* m1,
   GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
{
  FFT<float>::instance()->ifft(m2->getObjectPtr(),0);
  FFT<float>::instance()->ifft(m2->getObjectPtr(),1);
  FFT<float>::instance()->ifft(m2->getObjectPtr(),2);

  if (this->next()->putq(m1) < 0) {
     return GADGET_FAIL;
  }

  return GADGET_OK;
}

GADGET_FACTORY_DECLARE(FFTGadget)
Once we are inside the process function, the data has already been converted to the appropriate container messages and we can start processing the data. This function uses an FFT toolbox (more on toolboxes in Section 2.2, “Gadgetron Toolboxes”). After the data has been Fourier transformed along the first 3 dimensions it is placed on the next Gadgets queue. Remember the two GadgetContainerMessage objects were originally picked up from the message queue as a chain of ACE_Message_Block objects. They are still chained together, i.e. when passing m1 on to the next Gadget we are effectively passing on both arguments.

Another couple of macros to notice are the GADGET_OK and GADGET_FAIL. They are defined as 0 and -1 respectively. The convention in the Gadgetron is to return 0 when a function succeeds and < 0 when it fails - unless the function returns a pointer.

Last thing to notice is the GADGET_FACTORY_DECLARE(FFTGadget) statement. This is a macro which declares functions for loading a Gadget of this type out of a shared library and destroying it again when we are done. It ensures that we can load the Gadget on all platforms. When you create your own gadgets you should use this macro to declare the factory function for the Gadget.

For a tutorial on how to make your own Gadget library see Section 2.3.3, “Making a new Gadget Library”.

2.1.1.1. Gadget XML Configuration

In addition to defining a Gadget's behavior in response to a data package, it is also possible for the Gadgets to receive configuration information or parameters. The user can define the Gadgets behavior in response to configuration information by implementing the process_config function in the Gadget header file. The configuration information or parameters is typically transmitted in the beginning of the reconstruction process from the client (see Section 2.1.4, “Communication Sequence”). The configuration information can in principle be in any format (a given application can use a binary format or a text format defined for the specific purpose), but conventionally the parameters are transmitted in XML format and for the MRI Gadgets, the XML configuration is the XML header from the ISMRM Raw Data file. More details on this format and how to easily parse it with the included C++ XML data binding classes can be found at http://ismrmrd.github.io.

An example of a parameter XML file for an MRI data sert is shown here:

<?xml version="1.0"?>

<ismrmrdHeader xmlns="http://www.ismrm.org/ISMRMRD" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
  xmlns:xs="http://www.w3.org/2001/XMLSchema" 
  xsi:schemaLocation="http://www.ismrm.org/ISMRMRD ismrmrd.xsd">

  <subjectInformation>
    <patientName>phantom</patientName>
    <patientWeight_kg>72.5748</patientWeight_kg>
  </subjectInformation>
  <acquisitionSystemInformation>
    <systemVendor>SIEMENS</systemVendor>
    <systemModel>Avanto</systemModel>
    <systemFieldStrength_T>1.494</systemFieldStrength_T>
    <receiverChannels>32</receiverChannels>
    <relativeReceiverNoiseBandwidth>0.79</relativeReceiverNoiseBandwidth>
  </acquisitionSystemInformation>
  <experimentalConditions>
    <H1resonanceFrequency_Hz>63620740</H1resonanceFrequency_Hz>
  </experimentalConditions>
  <encoding>
    <trajectory>cartesian</trajectory>
    <encodedSpace>
      <matrixSize>
        <x>256</x>
        <y>128</y>
        <z>1</z>
      </matrixSize>
      <fieldOfView_mm>
        <x>600</x>
        <y>300</y>
        <z>5</z>
      </fieldOfView_mm>
    </encodedSpace>
    <reconSpace>
      <matrixSize>
        <x>128</x>
        <y>128</y>
        <z>1</z>
      </matrixSize>
      <fieldOfView_mm>
        <x>300</x>
        <y>300</y>
        <z>5</z>
      </fieldOfView_mm>
    </reconSpace>
    <encodingLimits>
      <kspace_encoding_step_1>
        <minimum>0</minimum>
        <maximum>127</maximum>
        <center>64</center>
      </kspace_encoding_step_1>
      <kspace_encoding_step_2>
        <minimum>0</minimum>
        <maximum>0</maximum>
        <center>0</center>
      </kspace_encoding_step_2>
      <slice>
        <minimum>0</minimum>
        <maximum>0</maximum>
        <center>0</center>
      </slice>
      <set>
        <minimum>0</minimum>
        <maximum>0</maximum>
        <center>0</center>
      </set>
    </encodingLimits>
  </encoding>
  <sequenceTiming>
    <TR>5.86</TR>
    <TE>2.96</TE>
  </sequenceTiming>
</ismrmrdHeader>
The user/developer can use any XML parsing technique to extract parameters from this XML header, but we encourage developers to use the C++ XML Data Binding classes that are included with the ISMRM Raw Data C++ library. For example, to parse encoding limits (example from AccumulatorGadget.cpp):

int AccumulatorGadget::process_config(ACE_Message_Block* mb)
{
 
 //Calling parsing convenience function found in GadgetIsmrmrdReadWrite.cpp
 boost::shared_ptr<ISMRMRD::ismrmrdHeader> cfg = parseIsmrmrdXMLHeader(std::string(mb->rd_ptr()));

 ISMRMRD::ismrmrdHeader::encoding_sequence e_seq = cfg->encoding(); 
 if (e_seq.size() != 1) {
  GADGET_DEBUG2("Number of encoding spaces: %d\n", e_seq.size());
  GADGET_DEBUG1("Only supports one encoding space supported\n");
  return GADGET_FAIL;
 }

 ISMRMRD::encodingSpaceType e_space = (*e_seq.begin()).encodedSpace();
 ISMRMRD::encodingSpaceType r_space = (*e_seq.begin()).reconSpace();
 ISMRMRD::encodingLimitsType e_limits = (*e_seq.begin()).encodingLimits();

 GADGET_DEBUG2("Matrix size: %d, %d, %d\n", 
   e_space.matrixSize().x(), 
   e_space.matrixSize().y(), 
   e_space.matrixSize().z());

 dimensions_.push_back(e_space.matrixSize().x());
 dimensions_.push_back(e_space.matrixSize().y());
 dimensions_.push_back(e_space.matrixSize().z());

 slices_ = e_limits.slice().present() ? 
   e_limits.slice().get().maximum()+1 : 1;

  return GADGET_OK;
}
2.1.2. Readers and Writers

As illustrated in Figure 2.1, “Gadgetron Architecture” the Gadgetron uses a set of Readers and Writers to deal with the incoming communication on the TCP/IP socket. Readers are responsible for deserialization of packages and Writers are responsible for serialization of packages. All packages that arrive on the socket will start with a message ID. Based on this ID, the Gadgetron delegates the responsibility of reading the package of the socket to a particular instance of a GadgetMessageReader defined by the following abstract class:

class GadgetMessageReader
{
 public:
  virtual ACE_Message_Block* read(ACE_SOCK_Stream* stream) = 0;
};
In order to be able to read a specific type of data, the read function must be implemented for that data type. As an example here is the GadgetIsmrmrdAcquisitionMessageReader, which reads an MRI data acquisition from the socket.

class GadgetIsmrmrdAcquisitionMessageReader 
: public GadgetMessageReader
{
 public:
  GADGETRON_READER_DECLARE(GadgetIsmrmrdAcquisitionMessageReader);
  virtual ACE_Message_Block* read(ACE_SOCK_Stream* socket);
};
Note the GADGETRON_READER_DECLARE(GadgetIsmrmrdAcquisitionMessageReader) declaration. This is equivalent to the declaration needed for the Gadgets (see Section 2.1.1, “Gadgets”) in order to make them load properly from shared libraries.

The implementation of this particular reader is as follows (this is an abbreviated version without error checking, etc.):

ACE_Message_Block* GadgetIsmrmrdAcquisitionMessageReader::read(ACE_SOCK_Stream* sock)
{
 GadgetContainerMessage<ISMRMRD::AcquisitionHeader>* m1 =
   new GadgetContainerMessage<ISMRMRD::AcquisitionHeader>();

 GadgetContainerMessage<hoNDArray< std::complex<float> > >* m2 =
   new GadgetContainerMessage< hoNDArray< std::complex<float> > >();

 m1->cont(m2);

 ssize_t recv_count = 0;

 if ((recv_count = stream->recv_n(m1->getObjectPtr(), sizeof(ISMRMRD::AcquisitionHeader))) <= 0) {
  m1->release();
  return 0;
 }

 if (m1->getObjectPtr()->trajectory_dimensions) {
  GadgetContainerMessage<hoNDArray< float > >* m3 =
    new GadgetContainerMessage< hoNDArray< float > >();

 m2->cont(m3);

 std::vector<unsigned int> tdims;
 tdims.push_back(m1->getObjectPtr()->trajectory_dimensions);
 tdims.push_back(m1->getObjectPtr()->number_of_samples);

 if (!m3->getObjectPtr()->create(&tdims)) {
  m1->release();
  return 0;
 }

 if ((recv_count =
   stream->recv_n
    (m3->getObjectPtr()->get_data_ptr(),
     sizeof(float)*tdims[0]*tdims[1])) <= 0) {

     m1->release();

   return 0;
 }

 std::vector<unsigned int> adims;
 adims.push_back(m1->getObjectPtr()->number_of_samples);
 adims.push_back(m1->getObjectPtr()->active_channels);

 if (!m2->getObjectPtr()->create(&adims)) {
   m1->release();
   return 0;
 }

 if ((recv_count =
      stream->recv_n
      (m2->getObjectPtr()->get_data_ptr(),
      sizeof(std::complex<float>)*adims[0]*adims[1])) <= 0) {

    m1->release();

    return 0;
 }

return m1;
}

GADGETRON_READER_FACTORY_DECLARE(GadgetIsmrmrdAcquisitionMessageReader)
The Reader allocates two GadgetContainerMessage data blocks to contain the incoming data. First an MRI acquisition header (defined in GadgetMRIHeaders.h) is read. Based hereon the length of each acquisition (number of samples) and the number of acquisition channels are determined. A hoNDArray is allocated to store the data read from the socket. Notice that the two GadgetContainerMessage are chained together using the cont function.

A final important statement to notice is:

GADGETRON_READER_FACTORY_DECLARE(GadgetIsmrmrdAcquisitionMessageReader)
This macro declares create and destroy functions to load the reader from a shared library on all platforms supported.

Whereas the Readers are responsible for deserialization, the GadgetMessageWriter is responsible for the opposite operation (serialization). In practice, Gadgets that produce an output for the client application can hand that data back to the Gadgetron framework where it is placed on the output queue along with a message ID. This is for instance done in this (abbreviated) code from an ImageFinishGadget:

template <typename T>
int ImageFinishGadget<T>
::process(GadgetContainerMessage<ISMRMRD::ImageHeader>* m1,
   GadgetContainerMessage< hoNDArray< T > >* m2)
{
  if (!this->controller_) {
    return -1;
  }

  GadgetContainerMessage<GadgetMessageIdentifier>* mb =
    new GadgetContainerMessage<GadgetMessageIdentifier>();

  switch (sizeof(T)) {
  case 2: //Unsigned short
   mb->getObjectPtr()->id = 
      GADGET_MESSAGE_IMAGE_REAL_USHORT;
   break;
  case 4: //Float
   mb->getObjectPtr()->id = 
      GADGET_MESSAGE_IMAGE_REAL_FLOAT;
   break;
  case 8: //Complex float
   mb->getObjectPtr()->id = 
      GADGET_MESSAGE_IMAGE_CPLX_FLOAT;
   break;
  default:
   GADGET_DEBUG2("Wrong data size detected: %d\n", sizeof(T));
   mb->release();
   m1->release();
   return GADGET_FAIL;
  }

  mb->cont(m1);

  int ret =  this->controller_->output_ready(mb);

  if ( (ret < 0) ) {
   GADGET_DEBUG1("Failed to return massage to controller\n");
   return GADGET_FAIL;
  }

  return GADGET_OK;
}
Notice that the Gadget has a reference to the Gadgetron framework through the controller_ member variable, which is set during initialization.

In the framework (more specifically in the GadgetStreamController) there is an active thread responsible for writing messages that are put on to the output queue. This is done by investigating the message ID and then picking the GadgetMessageWriter associated with this ID. A Writer must implement the following abstract class:

class GadgetMessageWriter
{
 public:
  virtual int write(ACE_SOCK_Stream* stream, 
                    ACE_Message_Block* mb) = 0;
};
The Writer is handed control of the socket along with the message block. A Writer declaration could look like:

class MRIImageWriter 
  : public GadgetMessageWriter
{

public:
   GADGETRON_WRITER_DECLARE(MRIImageWriter);
   virtual int write(ACE_SOCK_Stream* sock, 
                     ACE_Message_Block* mb);
};
Notice again the GADGETRON_WRITER_DECLARE(MRIImageWriter) which ensures proper run-time linking behavior. The implementation could look like (abbreviated with no error checking, etc.):

int MRIImageWriter
     ::write(ACE_SOCK_Stream* sock, 
             ACE_Message_Block* mb)
{

   GadgetContainerMessage<ISMRMRD::ImageHeader>* imagemb = 
      AsContainerMessage<ISMRMRD::ImageHeader>(mb);
  
   GadgetContainerMessage< hoNDArray< float > >* datamb =
      AsContainerMessage< hoNDArray< float > >(imagemb->cont());
  
   if (!datamb || !imagemb) {
      //Deal with errors
   }
   
   GadgetMessageIdentifier id;
   //Example for real flow image.
   id.id = GADGET_MESSAGE_ISMRMRD_IMAGE_REAL_FLOAT; 
 
   sock->send_n (&id, sizeof(GadgetMessageIdentifier));

   sock->send_n (imagemb->getObjectPtr(), sizeof(ISMRMRD::ImageHeader));

   sock->send_n (datamb->getObjectPtr()->get_data_ptr(), 
      sizeof(float)*datamb->getObjectPtr()->get_number_of_elements());

   return 0;
}

GADGETRON_WRITER_FACTORY_DECLARE(MRIImageWriter)
Once again notice the required GADGETRON_WRITER_FACTORY_DECLARE(MRIImageWriter) macro. Also notice that the message ID is transmitted to the client. The client is expected to follow the same communication model as the Reader, but it is determined entirely by the Writer implementation how the message is transmitted.

Readers and Writers are loaded dynamically at run-time along with the Gadgets (see Section 2.1.3, “Stream Configuration”). The input and output behaviour can be adapted by manipulating which Readers and Writers are associated with which message IDs.

2.1.3. Stream Configuration

A Gadgetron reconstruction is made up of modules, i.e. Readers, Writers, and Gadgets. New reconstruction programs can be created by simply assembling existing components in a new way. The configuration of the Gadgetron stream is done at run-time and new configuration chains can be created without recompiling any of the underlying Gadgets. More specifically, the configuration is specified in an XML file that the Gadgetron will read before receiving data. The best way to explain the format is by looking at a (simplified) example:

<?xml version="1.0" encoding="UTF-8"?>
<gadgetronStreamConfiguration 
  xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
  xmlns="http://gadgetron.sf.net/gadgetron"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        
    <reader>
      <slot>1008</slot>
      <dll>gadgetroncore</dll>
      <classname>GadgetIsmrmrdAcquisitionMessageReader</classname>
    </reader>
  
    <writer>
      <slot>1004</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterCPLX</classname>
    </writer>
    <writer>
      <slot>1005</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterFLOAT</classname>
    </writer>
    <writer>
      <slot>1006</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterUSHORT</classname>
    </writer>
  
    <gadget>
      <name>Acc</name>
      <dll>gadgetroncore</dll>
      <classname>AccumulatorGadget</classname>
    </gadget>
    <gadget>
      <name>FFT</name>
      <dll>gadgetroncore</dll>
      <classname>FFTGadget</classname>
    </gadget>
    <gadget>
      <name>Extract</name>
      <dll>gadgetroncore</dll>
      <classname>ExtractGadget</classname>
    </gadget>  
    <gadget>
      <name>ImageFinishFLOAT</name>
      <dll>gadgetroncore</dll>
      <classname>ImageFinishGadgetFLOAT</classname>
    </gadget>

</gadgetronStreamConfiguration>
The stream configuration XML layout is defined in the GADGETRON_HOME/schema/gadgetron.xsd. A stream configuration must conform to this schema definition or an error will be generated when the Gadgetron attempts to load the configuration.

The configuration file format contains 3 sections: 1) Readers, 2) Writers, 3) Stream (with Gadgets) corresponding to the 3 different types of components that can be assembled in the Gadgetron.

In the example above, the Readers section contains only one reader, which is the GadgetIsmrmrdAcquisitionMessageReader mentioned previously. The message ID associated with this Reader is 1008. Every time a message with ID 1008 arrives on the socket, responsibility for reading the message will be delegated to the GadgetIsmrmrdAcquisitionMessageReader. When the Gadgetron configuration is loaded, the framework will load the GadgetIsmrmrdAcquisitionMessageReader from the DLL (shared library) gadgetroncore. On the Linux platform this would be a shared library called libgadgetroncore.so and on the Windows platform it would be called gadgetroncore.dll.

The Gadgetron framework knows how to load the components from the DLLs assuming that they have been declared properly as described in Section 2.1.2, “Readers and Writers” and Section 2.1.1, “Gadgets”.

The example Gadgetron configuration has two Writers, i.e. it is capable of outputting two different types of data. Again the declarations cause the Gadgetron framework to load specific instances of GadgetMessageWriter and associate them with specific ID numbers.

There are certain built-in Readers and Writers in addition to those specified in the configuration file. As an example, there are Readers for receiving configurations to be used by the Gadgetron and for receiving the parameters that will be passed to all Gadgets (see Section 2.1.4, “Communication Sequence”). If the Gadgetron receives a message with an ID for which there is no associated Reader or encounters a message on the output queue for which there is no associated Writer an error will be generated, the Gadgetron stream shuts down, and the connection to the client will be closed.

In the example above, we have 4 Gadgets in the reconstruction chain. The first Gadget is an AccumulatorGadget, which collects individual lines and inserts them in k-space. When the k-space image is complete it is sent to the next Gadget in the chain, the FFTGadget, which is responsible for Fourier transforming the data into image space. The next Gadget (ExtractGadget) will extract the magnitude of the complex image. Finally the last Gadget in the chain (ImageFinishGadgetFLOAT) sends the reconstructed image back to the Gadgetron framework where it is added to the output queue.

It is also possible to send configuration parameters to Gadgets using the XML file. For example, to set a parameter in a Gadget, one could write:

  <gadget>
   <name>Accumulator</name>
   <dll>gadgetroncore</dll>
   <classname>AccumulatorGadget</classname>
   <property><name>MyTestProperty</name>
   <value>Blah Blah</value></property>
   <property><name>MyTestProperty2</name>
   <value>98776.862187</value></property>
  </gadget>
The two properties will now be accessible inside the Gadget using the parameter access functions defined in Gadget.h:

class Gadget : public ACE_Task<ACE_MT_SYNCH>
{

//Other definitions

int get_bool_value(const char* name);
int get_int_value(const char* name);
double get_double_value(const char* name);

};
Additionally it is also possible to specify how many active threads there should be in a Gadget. This is specified with:

  <gadget>
   <name>Accumulator</name>
   <dll>gadgetroncore</dll>
   <classname>AccumulatorGadget</classname>
   <property><name>threads</name><value>5</value></property>
  </gadget>
Which would make the AccumulatorGadget have 5 threads.

2.1.4. Communication Sequence

Communication between a client and the Gadgetron follows a straightforward communication protocol. When the Gadgetron is started it will be expecting a connection on a specific port (port 9002 is the default). The communication sequence is as follows:

The client makes connection

The Gadgetron accepts the connection and creates a new instance of a GadgetStreamController (see Figure 2.1, “Gadgetron Architecture”). After creating the GadgetStreamController the Gadgetron returns to accept connections on the socket such that multiple clients can be connected simultaneously.

The GadgetStreamController takes control of the socket and expects to read a specific type of message, which either contains the filename of a specific stream configuration (see Section 2.1.3, “Stream Configuration”) or alternatively it can receive the actual XML stream specification directly on the socket. These two types of messages are read with Readers that are always registered for the Gadgetron (see Section 2.1.2, “Readers and Writers”). If the Gadgetron receives the filename of a Gadget stream it expects to be able to find that configuration file in the gadegtron/config folder (see Section 2.1.5, “File Organization”).

The GadgetStreamController is then expecting to receive parameters that will be transmitted to each individual Gadget. In principle the "parameters" is just a raw buffer of characters that will be transmitted as such to each individual Gadget. It is the convention however to send the parameters in an XML format. It is up to each individual Gadget to interpret the parameters. The user can implement any behavior in response to the parameters by implementing the process_config function (see Section 2.1.1, “Gadgets”). The client can send parameters at any time during a reconstruction and they will always be transmitted to all Gadgets through the process_config function.

The client then starts transmitting data packages that the Gadgetron processes. Images are returned to the client.

When the client has no more data it will send a closure package. This package causes all Gadgets (in order) to process all remaining data on their input queue and then shut down.

Once the final Gadget has shut down, the connection with the client is terminated.

To make it easier to create a new client, the Gadgetron comes with a GadgetronConnector class:

class GadgetronConnector: 
  public ACE_Svc_Handler<ACE_SOCK_STREAM, ACE_MT_SYNCH> {

public:

 int open (std::string hostname, std::string port);   
 int putq  (ACE_Message_Block * mb ,  
     ACE_Time_Value *  timeout = 0);

 int register_reader(unsigned int slot, 
     GadgetMessageReader* reader);

 int register_writer(unsigned int slot, 
     GadgetMessageWriter* writer);

 int send_gadgetron_configuration_file(std::string config_xml_name);   
 int send_gadgetron_configuration_script(std::string config_xml_name);
 int send_gadgetron_parameters(std::string xml_string);
};
This class can be used to create simple clients that open a connection with the Gadgetron using the open function and then communicate with the Gadgetron through the Readers and Writers registered with the connector. See the mriclient example application (gagetron/apps/clients/mriclient in the source code archive) for a simple example of how to build a Gadgetron client.

2.1.5. File Organization

This section provides a brief overview of the file organization in the Gadgetron installation. Once you have compiled the Gadgetron and installed it (see Section 1.4, “Compiling and Installing Gadgetron”), it will reside in its designated installation folder (GADGETRON_HOME). For the purposes of this description, we will assume that the Gadgetron was installed in /usr/local/gadgetron.

In GADGETRON_HOME you should find the following folders:

bin: Contains all executables from the Gadgetron framework including the gadgetron executable itself and all clients and standalone applications.

config: Contains Gadgetron XML configuration files (see Section 2.1.3, “Stream Configuration”). This is where the Gadgetron searches for the configurations requested by the clients during initialization of the Gadget chain (see Section 2.1.4, “Communication Sequence”). It also contains the global gadgetron.xml configuration file, which is used to set global configuration parameters such as the port number that the Gadgetron.

lib: Contains all shared libraries (Gadgets and toolboxes). Additionally, this is the default path where Python Gadgets look for Python modules.

include: Contains all header files for the Gadgets and Toolboxes in order that they can be linked into external applications and Gadget libraries compiled outside the Gadgetron source tree.

schema: Contains all the XML schema definitions used by the Gadgetron (e.g. gadgetron.xsd) and also serves as a container for schema files used by client applications and copied to this folder during installation.

cmake: Contains a set of helpful CMake scripts that can be used if you wish to build applications or Gadget libraries outside the Gadgetron source tree. Among other things it contains a FindGadgetron.cmake script, which can be used to localize and set paths for the Gadgetron using CMake.

2.2. Gadgetron Toolboxes

The core reconstruction data structures and algorithms are made available through a set of toolboxes in shared libraries. The toolboxes implement the functionality of the various Gadgets, but they can also be used in standalone applications. A non-exhaustive overview of key functionality is covered in the following sections.

2.2.1. NDArray

Most image processing operations involve multi-dimensional arrays. Although the Gadgetron framework does not impose any specific array structure on the user, it does come with an abstract multi-dimensional array used throughout: the NDArray. It has a specific implementation for the CPU (hoNDArray) and GPU (cuNDArray). The abstract class definition looks like (abbreviated version):

template <class T> class NDArray
{
 public:
  
  NDArray ();

  virtual ~NDArray();
  
  virtual T* create(std::vector<unsigned int> *dimensions); 

  virtual T* create(std::vector<unsigned int> *dimensions, 
                    T* data, bool delete_data_on_destruct = false);

  virtual int permute(std::vector<unsigned int> *dim_order,
                      NDArray<T> *out = 0, int shift_mode = 0);
  
  inline unsigned int get_number_of_dimensions() const {
    return dimensions_->size();
  }

  unsigned int get_size(unsigned int dimension);

  boost::shared_ptr< std::vector<unsigned int> > get_dimensions();
  
  inline T* get_data_ptr() const { 
    return data_; 
  }
  
  inline unsigned long int get_number_of_elements() const {
    return elements_;
  }

  //Other public functions...

protected:

  virtual int allocate_memory() = 0;
  virtual int deallocate_memory() = 0;

  //Other private functions
  
};
The CPU (host) definition would look like (abbreviated):

template <class T> class hoNDArray : public NDArray<T>
{

public:
   //Public functions...

protected:
   virtual int allocate_memory();
   virtual int deallocate_memory();
};
As is seen from the NDArray header file, this class has a no-argument constructor, which makes it suited for encapsulating in the GadgetContainerMessage mentioned in Section 2.1.1, “Gadgets”. The procedure for creating an array with complex float values would look something like this:

#include <hoNDArray.h>
#include <complex>

hoNDArray< std::complex<float> > myArray;

std::vector<unsigned int> dimensions;
dimensions.push_back(128);
dimensions.push_back(128);

if(!myArray.create(&dimensions)) {
   //Deal with errors
}

//process data
To create an NDArray contained in a GadgetContainerMessage would look something like this:

GadgetContainerMessage< hoNDArray< std::complex<float> > >* m = 
  new GadgetContainerMessage< hoNDArray< std::complex<float> > >();

std::vector<unsigned int> dimensions;
dimensions.push_back(128);
dimensions.push_back(128);

if(!m->getObjectPtr()->create(&dimensions)) {
   //Deal with errors
}

//Process data or pass on to other Gadget, etc. 

m->release(); //Delete the message block and containing data
As mentioned in Section 2.1.1, “Gadgets”, the GadgetContainerMessage is a specialized version of the ACE_Message_Block class from the ACE framework. Data is passed between Gadgets in the form of ACE_Message_Blocks and Gadgets have access to utility functions that allow them to test if a given ACE_Message_Block is in fact a partcular type of GagetContainerMessage.

2.2.1.1. GPU Support

The NDArray data structure also has a GPU implementation (abbreviated version of header below):

template <class T> class cuNDArray : public NDArray<T>
{
 public:
  cuNDArray();

  cuNDArray(const cuNDArray<T>& a);

  // Constructor from hoNDArray
  cuNDArray(hoNDArray<T> *a);

  // Assignment operator
  cuNDArray& operator=(const cuNDArray<T>& rhs);
  
  virtual ~cuNDArray();

  virtual T* create(std::vector<unsigned int> *dimensions);

  virtual T* create(std::vector<unsigned int> *dimensions, 
                    int device_no);

  virtual T* create(std::vector<unsigned int> *dimensions, 
                    T* data, bool delete_data_on_destruct = false);

  virtual boost::shared_ptr< hoNDArray<T> > to_host() const;
  
  virtual int set_device(int device_no);
  inline int get_device() { return device_; }
  
 protected:
  
  int device_; 
  virtual int allocate_memory();
  virtual int deallocate_memory();
  
};
It has a few extra create functions compared to the host (CPU) version of this array. Specifically, it is possible to provide the array with the device number that the array should be allocated on. This is important when working on systems with multiple GPU processors. The default is to allocate it on the current device (device 0 unless specifically set otherwise). It is possible to query on which device the data is allocated and to effectively move the data from one device to another through operators. Similarly, one copy constructor takes a hoNDArray and transparently copies the host data to the GPU.

2.2.2. vector_td

The class vector_td provides a basic representation of one-, two-, three-, or four-dimensional vectors (positions). It is templetized with the datatype T and dimensionality D. For convenience we provide a set of typedefs to commonly encountered instances. A subset of the definitions provided in vector_td.h is provided here (users should check the actual file e.g. for additional often used constructors):

template< class T, unsigned int D > class vector_td
{
public:

  T vec[D];

  __inline__ __host__ __device__ T& operator[](const int i) 
  {
    return vec[i];
  }

  __inline__ __host__ __device__ const T& operator[](const int i) const
  { 
    return vec[i];
  }
};


// Some typedefs for convenience

template< class REAL, unsigned int D > struct reald{
  typedef vector_td< REAL, D > Type;
};

template< unsigned int D > struct intd{
  typedef vector_td< int, D > Type;
};

template< unsigned int D > struct uintd{
  typedef vector_td< unsigned int, D > Type;
};

template< unsigned int D > struct floatd{
 typedef typename reald< float, D >::Type Type;
};

template< unsigned int D > struct doubled{
  typedef typename reald< double, D >::Type Type;
};

template< class T > struct complext{
  typedef vector_td< T, 2 > Type;
};

A number of arithmetic and conditional operators on the vector_td are defined in vector_td_operators.h. Similarly, the header vector_td_utilities.h wraps common math functionality for the vector_td class. Many common operations that take one of more cuNDArray instances with element type vector_td are defined in ndarray_vector_utilities.h. We encourage the reader to explore these utilities on his own.

The vector_td can be used in both host and device code. As an example of use it is contained in the interface of the non-Cartesian FFT described in Section 2.2.4.2, “Non-Cartesian FFT”.

2.2.3. complext

A complex number class that can be used in both host and device code is found in complext.h. It contains a substantial set of useful operators and functions.

2.2.4. Fourier Transforms

2.2.4.1. Cartesian FFT

2.2.4.1.1. FFT of a hoNDArray

The Gadgetron uses the FFTW library for Fourier transform of hoNDArray structures. Users can call the FFTW directly from their code, but to make things a little easier, we provide a simple wrapper class defined in toolboxes/ndarray/FFT.h. Here is an abbreviated version:

template <typename T> class EXPORTNDARRAY FFT
{

public:
 static FFT<T>* instance(); 

 void fft(hoNDArray< std::complex<T> >* input, 
          unsigned int dim_to_transform);

 void ifft(hoNDArray< std::complex<T> >* input, 
          unsigned int dim_to_transform);

 void fft(hoNDArray< std::complex<T> >* input);

 void ifft(hoNDArray< std::complex<T> >* input);

protected:
 FFT();
 virtual ~FFT();
};
The FFT class provides simple wrapper functionality to perform FFTs of hoNDArrays along a specific dimension or along all dimensions. It performs in place FFTs and works on complex arrays of single or double precision.

An important feature of this class is that it is a process wide singleton for the Gadgetron. As outlined in the definition above, the constructor and destructor are protected and it is not possible to allocate a new FFT object. The way to use the class is through the instance function:

#include "FFT.h"

FFT<float>::instance()->fft(...);
The reason for this is that the FFTW planning routines are not thread safe. Multiple Gadgets (that each have their own thread of execution) may need to use FFTs and consequently the planning routines need to be protected with a mutex. All of this is handled inside the FFT class and since it is a singleton only one thread can run the planning routines at any given time.

As mentioned it is possible for the users to call FFTW routines directly, and there may be some performance reasons for doing so (as opposed to using this wrapper), but please be aware of this thread safety issue when you design your Gadgets. If you want to be on the safe side, use the wrapper.

2.2.4.1.2. FFT of a cuNDArray

Cartesian Fast Fourier Transform on the GPU is supported by wrapping Cuda's FFT routines as defined in cuNDFFT.h.

template<class T> class EXPORTGPUCORE cuNDFFT
{
 public:

  cuNDFFT() {}
  virtual ~cuNDFFT() {}

  int fft ( cuNDArray<T> *input, 
    std::vector<unsigned int> *dims_to_transform );

  int ifft( cuNDArray<T> *input, 
    std::vector<unsigned int> *dims_to_transform, 
    bool do_scale = true );

  int fft ( cuNDArray<T> *input, 
    unsigned int dim_to_transform);

  int ifft( cuNDArray<T> *input, 
    unsigned int dim_to_transform, 
    bool do_scale = true );

  int fft ( cuNDArray<T> *input );
  int ifft( cuNDArray<T> *input, 
    bool do_scale = true );

 protected:
  int fft_int( cuNDArray<T> *input, 
    std::vector<unsigned int> *dims_to_transform, 
    int direction, 
    bool do_scale = true );
};
 
The interface defines forwards and inverse transforms of a single array dimension, all dimensions of the array, or a subset of dimensions.

2.2.4.2. Non-Cartesian FFT

A dedicated GPU-implementation of the NUFFT - often referred to a gridding - is provided. The interface is defined in NFFT.h provided below in abbreviated form

template< class REAL, unsigned int D > 
class EXPORTGPUNFFT NFFT_plan
{
 public: // Main interface
    
  // Constructors
  NFFT_plan();
  NFFT_plan( typename uintd<D>::Type matrix_size, 
             typename uintd<D>::Type matrix_size_os, 
             REAL W, int device = -1 );

  // Destructor
  virtual ~NFFT_plan();

  // Clear internal storage in plan
  enum NFFT_wipe_mode { NFFT_WIPE_ALL, NFFT_WIPE_PREPROCESSING };
  bool wipe( NFFT_wipe_mode mode );

  // Replan 
  bool setup( typename uintd<D>::Type matrix_size, 
              typename uintd<D>::Type matrix_size_os, 
              REAL W, int device = -1 );
    
  // Preproces trajectory 
  // Cartesian to non-Cartesian / non-Cartesian to Cartesian / both
  enum NFFT_prep_mode { NFFT_PREP_C2NC, 
                        NFFT_PREP_NC2C, 
                        NFFT_PREP_ALL };
  bool preprocess
    ( cuNDArray<typename reald<REAL,D>::Type> *trajectory, 
      NFFT_prep_mode mode );
    
  // Execute NFFT 
  // ( Cartesian to non-Cartesian or non-Cartesian to Cartesian)  
  enum NFFT_comp_mode { NFFT_FORWARDS_C2NC, 
                        NFFT_FORWARDS_NC2C, 
                        NFFT_BACKWARDS_C2NC, 
                        NFFT_BACKWARDS_NC2C };
  bool compute( cuNDArray<complext<REAL> > *in, 
                cuNDArray<complext<REAL> > *out, 
                cuNDArray<REAL> *dcw, NFFT_comp_mode mode );

  // Execute NFFT iteration 
  // (Cartesian to non-Cartesian and back to Cartesian space)
  bool mult_MH_M( cuNDArray<complext<REAL> > *in, 
                  cuNDArray<complext<REAL> > *out, 
                  cuNDArray<REAL> *dcw, 
                  std::vector<unsigned int> halfway_dims );
  
 public: // Utilities
  
  // NFFT convolution 
  // (Cartesian to non-Cartesian or non-Cartesian to Cartesian)
  enum NFFT_conv_mode { NFFT_CONV_C2NC, NFFT_CONV_NC2C };
  bool convolve( cuNDArray<complext<REAL> > *in, 
                 cuNDArray<complext<REAL> > *out, 
                 cuNDArray<REAL> *dcw, 
                 NFFT_conv_mode mode, bool accumulate = false );
    
  // NFFT FFT
  enum NFFT_fft_mode { NFFT_FORWARDS, NFFT_BACKWARDS };
  bool fft( cuNDArray<complext<REAL> > *data, 
            NFFT_fft_mode mode, bool do_scale = true );
  
  // NFFT deapodization
  bool deapodize( cuNDArray<complext<REAL> > *image );

 public: // Setup queries

  typename uintd<D>::Type get_matrix_size();
  typename uintd<D>::Type get_matrix_size_os();
  REAL get_W();
  unsigned int get_device();
  
...
};
After a NFFT_plan is constructed the preprocess function should be called with the desired trajectory. In the special case of radial sampling the header radial_utilities.h defines some convienient functions to compute radial trajectories and corresponding density compensation weights. After preprocessing the NFFT can be executed through the compute function. The individial building blocks of the NFFT - convolution, FFT, and deapodization - are exposed in the public interface and hence available for use in custom algorithms.

It is often required to perform the NFFT on a number of different inputs. Particularly in 1D and 2D the best performance is obtained if many transforms are executed concurrently in order to keep the device fully occupied. Two strategies can be combined:

The trajectory passed to the preprocess method is normally a one-dimension cuNDArray containing normalized (to the range [-0.5;0.5]) non-Cartesian positions as reald<REAL,D> elements of precision REAL and dimensionality D. However, if the cuNDArray is two-dimensional, the latter dimension specifies that we wish to transform a number of frames with different trajectories concurrently.

If a number of transformations with identical trajectories are to be transformed, the input and output arrays to the compute methods can be any multiplum of the Cartesian and non-Cartesian dimensions configured from the setup and preprocess methods. The images provided are consequently batch transformed.

Please note. The NFFT performs significantly better on GPUs supporting Cuda's shader model 2.0 or newer compared to devices supporting only shader models 1.x. The reason being that we rely on the inherent caching of global memory - available only on hardware supporting at least shader model 2.0.

2.2.5. Linear (Matrix) Operators

A fundamental building block of most image reconstruction algorithms is the abstract class linearOperator. A range of linear imaging and regularization operators are inherited from this pure virtual base class (abbreviated):

template < class REAL, class ARRAY_TYPE > class linearOperator
{
 public:

  linearOperator() { weight_ = REAL(1); }

  virtual ~linearOperator() {}

  virtual void set_weight( REAL weight ){ weight_ = weight; }
  virtual REAL get_weight(){ return weight_; }

  virtual bool set_domain_dimensions
    ( std::vector<unsigned int> *dims ) { ... }
  virtual bool set_codomain_dimensions
    ( std::vector<unsigned int> *dims ) { ... }

  virtual boost::shared_ptr< std::vector<unsigned int> > 
    get_domain_dimensions() { ... }

  virtual boost::shared_ptr< std::vector<unsigned int> > 
    get_codomain_dimensions() { ... }

  virtual int mult_M( ARRAY_TYPE* in, ARRAY_TYPE* out, 
                      bool accumulate = false) = 0;
  virtual int mult_MH( ARRAY_TYPE* in, ARRAY_TYPE* out, 
                       bool accumulate = false) = 0;
  virtual int mult_MH_M( ARRAY_TYPE* in, ARRAY_TYPE* out, 
                       bool accumulate = false) = 0;
  
  virtual boost::shared_ptr< 
    linearOperator< REAL, ARRAY_TYPE > > clone() = 0;

  ...
};
The linearOperator is templated by two arguments: 1) the basic precision REAL (e.g. float or double) and 2) the ARRAY_TYPE (e.g. NDArray<T>, hoNDArray<T>, or cuNDArray<T>) representing the expected vector format for the matrix-vector multiplication the operator implements.

Every MatrixOperator has an associated weight that is used to balance multiple matrix terms when added to a cost function (see Section 2.2.6, “Linear Solvers”).

The main functionality is provided in the three pure virtual functions, mult_M, mult_MH, and mult_MH_M, denoting multiplication with the matrix operator (M), multiplication with the adjoint (i.e. conjugate transpose) of the matrix operator (MH), and an "iteration" of the two respectively (MHM).

The clone method is required by some solvers to make a clone (copy) of a given linearOperator. Similarly, some solvers require knowledge of the domain and codomain dimensions on which the operator can be applied. The mult_M method converts the input vector of domain_size to one of codomain_size - and vice versa for mult_MH.

The linearOperator is used to model a linear imaging modality's encodig operation (Fourier transform for MRI, Radon transform for CT, convolution for Microscopy etc.) but also common regularization operators such the identity matrix, the partial derivatives etc.

Here follows a list that briefly describes the linear operators that are used for the reconstruction examples discussed later in this document (Chapter 3, Gadgetron Applications, Chapter 4, Standalone Applications).

2.2.5.1. List of linear operators

The section provides a non-exhaustive list of availble linear operators in Gadgetron toolboxes.

A two-level implementation strategy is used for most of the operators the Gadgetron provide. We first derive a class, say identityOperator, from the linearOperator base class. In this derived class we implement the pure virtual functions of the base class, e.g. mult_M, mult_MH, and mult_MH_M. The overall algorithm and functionality of the operator is implemented at this level. Like its superclass, the identityOperator is however templated on the underlying ARRAY_TYPE and thus cannot contain dedicated implementation code to a specific array implementation. The implementation of mult_M, mult_MH, and mult_MH_M is consequently based on a new set of pure virtual functions of the templated ARRAY_TYPE. We provide another level of inheritance, e.g. cuIdentityOperator, which in this case provides the cuNDArray-specific implementation of the pure virtual function in identityOperator. This hierachy has the desired design goal, that the core algorithm implementation is shared in the base class of the operator. Only the host/device specific sub-components are defined individually. It is thus fairly straightforward to derive both an cuNDArray and an hoNDArray version of an operator.

As an example we provide a simplified declaration of the identityOperator and cuIdentityOperator below. Without specific mentioning for the subsequent operators, many follow a similar inheritance hierachy.

identityOperator

Implements multiplication of a vector with the identity matrix.

// Notice: simplified code without error-checking

template <class REAL, class ARRAY_TYPE> class identityOperator
 : public linearOperator<REAL, ARRAY_TYPE>
{
 public:

  identityOperator() : linearOperator<REAL, ARRAY_TYPE>() {}
  virtual ~identityOperator() {}
  
  // operator_xpy computes "x+y" and stores the result in y
  virtual bool operator_xpy( ARRAY_TYPE *x, ARRAY_TYPE *y ) = 0;

  virtual int mult_M( ARRAY_TYPE *in, ARRAY_TYPE *out, 
                      bool accumulate = false )
  {
    if( accumulate )
      operator_xpy( in, out );
    else 
      *out = *in;
  }

  ... // Similar code for mul_MH and mult_MH_M
};

The Cuda specific implementation:

// Notice: 
// Simplified code without error checking and multi-device support

template <class REAL, class T> 
class cuIdentityOperator 
: public identityOperator< REAL, cuNDArray<T> >
{
 public:

  cuIdentityOperator() : 
    identityOperator< REAL, cuNDArray<T> >() {}
  
  virtual ~cuIdentityOperator() {}
  
  virtual bool operator_xpy( cuNDArray<T> *x, cuNDArray<T> *y )
  { 
    return cuNDA_axpy( T(1), x, y );
  }

 ...
};

Notice that the template arguments to the cuIdentitytOperator differ from its base class. REAL specifies the desired precision (float or double) and T specifies the desired type - which could be identical to REAL or e.g. a complext<REAL>. Also notice how the cuIdentitytOperator class definition directly specifies the ARRAY_TYPE of its superclass (in this case to be of type cuNDArray<T>).

partialDerivativeOperator

Provides the partial derivative of an image in a given spatial dimension.

laplaceOperator

Computes the Laplacian of an image.

imageOperator

Performs multiplication with a diagonal matrix of the element-wise reciprocal of a given image.

convolutionOperator

Performs convolution of an image with a given kernel.

nfftOperator

Implements the non-Cartesian Fast Fourier Transform

senseOperator

Implements the encoding operator for the parallel MRI imaging technique Sense. Comes in two flavours for 1) Cartesian and 2) non-Cartesian reconstruction.

encodingOperatorContainer

As we require exactly one encoding operator (but allow multiple regularization operators) to be added to our solvers (see Section 2.2.6, “Linear Solvers” below), this operator acts as a container when multiple encoding operators are desired. For example: The cost function right below (Section 2.2.6.1, “Conjugate Gradient Method for Linear Least Squares”) has two terms in its general form. Most often the vector p is 0 and consequently the operator R is considered a regularization operator while the operator E the single encoding operator. However, if p is non-zero, both E and R must be added to an encodingOperatorContainer that takes in both m and p during multiplication. A single encodingOperatorContainer is then added to the solver.

2.2.6. Linear Solvers

The Gadgetron's solvers toolbox contains both a generic conjugate gradient solver to solve linear least squares reconstruction problems (see Section 2.2.6, “Linear Solvers” ) and a two flavors of a Split Bregman solver for non-linear problems using l1-norms for regularization (see Section 2.2.7, “Non-linear Solvers”). More solvers can be expected in upcoming releases.

2.2.6.1. Conjugate Gradient Method for Linear Least Squares

The conjugate gradient solver is used to reconstruct an image posed as a minimizer to an l2-based optimization problem:


The unknown image to be reconstructed is denoted here by u and the measured data by m. E is a linear operator modelling the encoding of the imaging modality (e.g. a Fourier transform for MRI, a Radon transform for CT etc.). R is a regularization operator often required to ensure uniqueness of the solution. Lambda is a scalar weight (with a default value of one) associated to each matrix operator and used to balance the various terms in the cost function. Finally p denotes some (possibly blank) prior image in the regularization term. Any number of terms can be added.

The closed form solution to the optimization problem is given by the linear system of equations:


Put extremely short; you set up and run a solver by 1) adding the corresponding linear operators to the solver, and 2) invoking the solve function in the solver providing m (and p if non-zero) as input arguments.

An abbreviated version of the interface to the conjugate gradient solver is shown here

// Defined in solver.h

template <class ARRAY_TYPE_IN, class ARRAY_TYPE_OUT> 
class solver
{
public:

  // Constructor/destructor
  //

  solver() { output_mode_ = OUTPUT_SILENT; }
  virtual ~solver() {}
  
  // Output modes
  //

  enum solverOutputModes { OUTPUT_SILENT = 0, 
                           OUTPUT_WARNINGS = 1, 
                           OUTPUT_VERBOSE = 2, 
                           OUTPUT_MAX = 3 };
  
  // Set/get output mode
  //

  virtual int get_output_mode() { return output_mode_; }

  virtual void set_output_mode( int output_mode ) {
      output_mode_ = output_mode;
  }
  
  // Set/get starting solution/estimate for solver
  //

  virtual void set_x0( boost::shared_ptr<ARRAY_TYPE_OUT> x0 )
    { x0_ = x0; }

  virtual boost::shared_ptr<ARRAY_TYPE_OUT> get_x0()
    { return x0_; }

  // Default error output
  //

  virtual void solver_error( std::string msg ) { ... }

  // Default warning output
  //

  virtual void solver_warning( std::string msg ) { ... }

  // Invoke solver
  //

  virtual boost::shared_ptr<ARRAY_TYPE_OUT> solve
    ( ARRAY_TYPE_IN* ) = 0;
 
protected:
  int output_mode_;
  boost::shared_ptr<ARRAY_TYPE_OUT> x0_;
};
The abstract cgSolver class:

// Defined in cgSolver.h

template <class REAL, 
          class ELEMENT_TYPE, 
          class ARRAY_TYPE> 
class cgSolver : public linearSolver
  <REAL, ELEMENT_TYPE, ARRAY_TYPE>
{
public:

  // Class defining the termination criterium
  //

  friend class cgTerminationCallback
    <REAL, ELEMENT_TYPE, ARRAY_TYPE>;

  // Constructor / destructor
  //

  cgSolver() : linearSolver<REAL, ELEMENT_TYPE, ARRAY_TYPE>() 
   {...}
 
  virtual ~cgSolver() {}

  // Set preconditioner
  //

  virtual void set_preconditioner( 
    boost::shared_ptr< cgPreconditioner<ARRAY_TYPE> > precond ) {
      precond_ = precond;
  }
  
  // Set termination callback
  //

  virtual void set_termination_callback(
    boost::shared_ptr< cgTerminationCallback
      <REAL, ELEMENT_TYPE, ARRAY_TYPE> > cb ){
      cb_ = cb;
  }

  // Set/get maximally allowed number of iterations
  //

  virtual void set_max_iterations( unsigned int iterations ) { 
    iterations_ = iterations; }

  virtual unsigned int get_max_iterations() { return iterations_; }  

  // Set/get tolerance threshold for termination criterium
  //

  virtual void set_tc_tolerance( REAL tolerance ) 
    { tc_tolerance_ = tolerance; }

  virtual REAL get_tc_tolerance() { return tc_tolerance_; }
  
  // Pre/post solver callbacks
  //

  virtual bool pre_solve( ARRAY_TYPE** ) { return true; }
  virtual bool post_solve( boost::shared_ptr<ARRAY_TYPE>& ) 
    { return true; }

  // Pure virtual functions defining core solver functionality
  // Implemented on the host/device respectively in a derived class
  //

  virtual ELEMENT_TYPE solver_dot( ARRAY_TYPE*, ARRAY_TYPE* ) = 0;
  virtual bool solver_clear( ARRAY_TYPE* ) = 0;
  virtual bool solver_scal( ELEMENT_TYPE, ARRAY_TYPE* ) = 0;
  virtual bool solver_dump( ARRAY_TYPE* ) { return true; }
  virtual bool solver_axpy
    ( ELEMENT_TYPE, ARRAY_TYPE*, ARRAY_TYPE* ) = 0;

  //
  // Main solver interfaces
  //

  virtual boost::shared_ptr<ARRAY_TYPE> solve( ARRAY_TYPE *_d ) 
    { ... }

  virtual boost::shared_ptr<ARRAY_TYPE> solve_from_rhs
    ( ARRAY_TYPE *_rhs ) { ... }

  ...
};
The Cuda specific implementation:

// Defined in cuCGSolver.h

template <class REAL, class T> class cuCgSolver 
  : public cgSolver< REAL, T, cuNDArray<T> >
{
public:

  cuCgSolver() : cgSolver< REAL, T, cuNDArray<T> >() { ... }
  virtual ~cuCgSolver() {}

  cuCGSolver() : cgSolver< REAL, T, cuNDArray<T> >() {}
  virtual ~cuCGSolver() {}

  virtual bool pre_solve(cuNDArray<T>**)
   { ... }

  virtual bool post_solve(cuNDArray<T>**)
   { ... }

  virtual void solver_error( std::string err )
   { ... }

  virtual T solver_dot( cuNDArray<T> *x, 
   cuNDArray<T> *y ){ ... }

  virtual bool solver_clear( cuNDArray<T> *x )
   { ... }

  virtual bool solver_scal( T a, 
   cuNDArray<T> *x ){ ... }

  virtual bool solver_axpy( T a, cuNDArray<T> *x, 
   cuNDArray<T> *y ){ ... }

  ...
};
The overall inheritance hierachy is modelled and implemented similarly to the linearOperator class hierachy described above (see Section 2.2.5, “Linear (Matrix) Operators”). To use the solver the user creates an instance of the solver for either the host or device (e.g. the cuCGSolver above for a GPU-based solver). The solver is configured using the functions in the cgSolver base class. The core solve function itself is found in the root of the hierachy; the solver.

Note that any number of terms (linear operators) can be added to the solver (or cost function).

The following code listing provides a short example of how to define a conjugate gradient solver for GPU-based image deblurring given an image and an estimate of the point spread function that degraded the image. It uses the convolutionOperator to model the blurring and a partialDerivativeOperator in each spatial dimension for regularization. The full code can be found in $(GADGETRON_SOURCE)/apps/standalone/gpu/deblurring/2d/deblur_2d_cg.cpp.

{
  << Code that parses the command line 
     and loads the image and kernel from disk >>

  // Define the desired precision
  typedef float _real; 
  typedef complext<_real>::Type _complext;

  // Upload host data to device
  cuNDArray<_complext> data(host_data.get());
  cuNDArray<_complext> kernel(host_kernel.get());
    
  // Setup regularization operators

  boost::shared_ptr
    < cuPartialDerivativeOperator<_real,_complext,2> > 
      Rx( new cuPartialDerivativeOperator<_real,_complext,2>(0) ); 

  boost::shared_ptr
    < cuPartialDerivativeOperator<_real,_complext,2> > 
      Ry( new cuPartialDerivativeOperator<_real,_complext,2>(1) ); 

  Rx->set_weight( lambda );
  Ry->set_weight( lambda );
     
  //
  // Setup conjugate gradients solver
  //

  // Define encoding matrix
  boost::shared_ptr< cuConvolutionOperator<_real,2> > 
    E( new cuConvolutionOperator<_real,2>() );

  E->set_kernel( &kernel );
  E->set_domain_dimensions(data.get_dimensions().get());
    
  // Setup conjugate gradient solver
  cuCGSolver<_real, _complext> cg;

  // encoding matrix
  cg.set_encoding_operator( E );

  // regularization matrix                   
  if( kappa>0.0 ) cg.add_reuglarization_operator( Rx );
  
  // regularization matrix
  if( kappa>0.0 ) cg.add_regularization_operator( Ry ); 

  cg.set_max_iterations( num_iterations );
  cg.set_tc_tolerance( 1e-8 );
  cg.set_output_mode( cuCGSolver<_real, _complext>::OUTPUT_VERBOSE );
                
  //
  // Conjugate gradient solver
  //
  
  boost::shared_ptr< cuNDArray<_complext> > 
    cgresult = cg.solve(&data);

  // All done, write out the result
  
  boost::shared_ptr< hoNDArray<_complext> > 
    host_result = cgresult->to_host();
  
  write_nd_array<_complext>(host_result.get(), 
    (char*)parms.get_parameter('r')->get_string_value());
}
For an overview of the various standalone applications the Gadgetron provides - and instruction on how to run them - we refer to Chapter 4, Standalone Applications.

2.2.7. Non-linear Solvers

2.2.7.1. Split Bregman Solver for L1-regularized Problems

The Gadgetron includes two Split Bregman solvers to solve respectively


where |.|TV denotes the Total Variation norm. The solver to the upper (unconstraint) optimization problem is defined in sbSolver.h while the solver to the latter constraint problem declared in sbcSolver.h. The Split Bregman solver was chosen as it integrates nicely with the linear conjugate solver desribed above (Section 2.2.6, “Linear Solvers”). In fact, most of the work in the two Split Bregman solvers is performed by a linear inner solver (e.g. a conjugate gradient solver), but the input (right hand side) to the inner solver varies from iteration to iteration.

The interface to the unconstraint Split Bregman solver is given here. We have seen the overall inheritance hierachy several times already, so it should suffice to provide only very abbreviated headers here:

// Defined in sbSolver.h


template< class REAL, 
          class ELEMENT_TYPE, 
          class ARRAY_TYPE_REAL, 
          class ARRAY_TYPE_ELEMENT, 
          class INNER_SOLVER,
          class OPERATOR_CONTAINER > class sbSolver 

 : public linearSolver<REAL, ELEMENT_TYPE, ARRAY_TYPE_ELEMENT>
{
public:

  // Constructor
  //

  sbSolver() : linearSolver<REAL, ELEMENT_TYPE, ARRAY_TYPE_ELEMENT>() 
   { ... }
  
  // Destructor
  //

  virtual ~sbSolver() {}
   

  // Add regularization group operator 
  // (isotropic, multiple operators per group allowed)
  //

  virtual bool add_regularization_group_operator( 
    boost::shared_ptr< linearOperator<REAL, ARRAY_TYPE_ELEMENT> > op ) 
  { ... }

  // Add isotroic regularization group (multiple groups allowed)
  //

  virtual bool add_group() { ... }

  // Get regularization group operator
  //
 
  < omitted for brevity>
  
  // Set/get prior image (PICCS style). 
  // I.e. for every regularization operator (group) 
  // R that is added we minimize:
  // alpha|R(x-prior)|_{l1} + (1-alpha)|R(x)|_{l1}
  //

  virtual bool set_prior_image( 
    boost::shared_ptr<ARRAY_TYPE_ELEMENT> prior, REAL alpha )
  { ... }
 
  // Get the prior image and corresponding weighing factor
  //
  
  virtual boost::shared_ptr<ARRAY_TYPE_ELEMENT> get_prior_image() 
    { return prior_; }

  virtual REAL get_prior_alpha() { return alpha_; }


  // Set termination criterium tolerance
  //

  virtual void set_tc_tolerance( REAL tolerance ) 
  { ... }

  // Set/get maximum number of outer Split-Bregman iterations
  //

  virtual void set_max_outer_iterations( 
    unsigned int iterations ) { outer_iterations_ = iterations; }
 
  virtual unsigned int get_max_outer_iterations() { 
    return outer_iterations_; }

  // Set/get maximum number of inner Split-Bregman iterations
  //

  virtual void set_max_inner_iterations( 
    unsigned int iterations ) { inner_iterations_ = iterations; }

  virtual unsigned int get_max_inner_iterations() 
   { return inner_iterations_; }

  // Get the inner solver
  //

  virtual boost::shared_ptr<INNER_SOLVER> get_inner_solver() 
   { return inner_solver_; }
  

  // Core solver functionality to be implemented
  // in a derived class (host/device specific implementations)
  //

  virtual bool solver_clear_real( ARRAY_TYPE_REAL* ) = 0;

  virtual bool solver_clear_element( ARRAY_TYPE_ELEMENT* ) = 0;

  virtual bool solver_sqrt( ARRAY_TYPE_REAL* ) = 0;

  virtual bool solver_scal( ELEMENT_TYPE, 
                            ARRAY_TYPE_ELEMENT* ) = 0;

  virtual bool solver_axpy_real( REAL, ARRAY_TYPE_REAL*, 
                                 ARRAY_TYPE_REAL* ) = 0;

  virtual bool solver_axpy_element( ELEMENT_TYPE, 
                                    ARRAY_TYPE_ELEMENT*, 
                                    ARRAY_TYPE_ELEMENT* ) = 0;

  virtual REAL solver_asum( ARRAY_TYPE_ELEMENT* ) = 0;

  virtual boost::shared_ptr<ARRAY_TYPE_REAL> solver_abs
    ( ARRAY_TYPE_ELEMENT* ) = 0;

  virtual boost::shared_ptr<ARRAY_TYPE_REAL> solver_norm
    ( ARRAY_TYPE_ELEMENT* ) = 0;

  virtual bool solver_shrink1( REAL, ARRAY_TYPE_ELEMENT*, 
                               ARRAY_TYPE_ELEMENT* ) = 0;

  virtual bool solver_shrinkd( REAL, ARRAY_TYPE_REAL*, 
                               ARRAY_TYPE_ELEMENT*, 
                               ARRAY_TYPE_ELEMENT* ) = 0;

  //
  // Main solver interface
  //

  virtual boost::shared_ptr<ARRAY_TYPE_ELEMENT> solve
   ( ARRAY_TYPE_ELEMENT *f ) { ... }

 ...
};
// Defined in cuSbCgSolver.h

template <class REAL, class T> class cuSbCgSolver 
  : public sbSolver< REAL, T, cuNDArray<REAL>, 
                     cuNDArray<T>, cuCgSolver<REAL,T>, 
                     cuEncodingOperatorContainer<REAL,T> >
{
public:
  
  cuSbCgSolver() : sbSolver< REAL, T, cuNDArray<REAL>, 
                             cuNDArray<T>, cuCgSolver<REAL,T>, 
                             cuEncodingOperatorContainer<REAL,T> >() 
  { ... }
  
  virtual ~cuSbCgSolver() {}

  // Implementation of pure virtual functions
  ...
};
To run the algorithm on the GPU the user would create an instance of a cuSbCgSolver providing the two template arguments; the desired precision and data type. Prior to running the solve function with the measured data m, the user should provide 1) the encoding operator, 2) the regularization operators, and 3) the desired domain and codomain dimensions as these cannot in general be deduced from the measured data.

We outline the code required to set up the solver for TV-based image denoising. The full code can be found in $(GADGETRON_SOURCE)/apps/standalone/gpu/denoising/2d/denoise_TV.cpp.

{
  << Command line parsing and data loading >>
  
  //
  // Setup regularization operators
  // 

  boost::shared_ptr< cuPartialDerivativeOperator<_real,_real,2> > 
    Rx( new cuPartialDerivativeOperator<_real,_real,2>(0) ); 

  boost::shared_ptr< cuPartialDerivativeOperator<_real,_real,2> > 
    Ry( new cuPartialDerivativeOperator<_real,_real,2>(1) ); 

  Rx->set_weight( lambda );
  Rx->set_domain_dimensions(data.get_dimensions().get());
  Rx->set_codomain_dimensions(data.get_dimensions().get());

  Ry->set_weight( lambda );
  Ry->set_domain_dimensions(data.get_dimensions().get());
  Ry->set_codomain_dimensions(data.get_dimensions().get());

  // Define encoding operator (identity)
  boost::shared_ptr< cuIdentityOperator<_real,_real> > 
    E( new cuIdentityOperator<_real,_real>() );

  E->set_weight( mu );
  E->set_domain_dimensions(data.get_dimensions().get());
  E->set_codomain_dimensions(data.get_dimensions().get());

  // Setup split-Bregman solver
  //

  cuSbCgSolver<_real,_real> sb;

  sb.set_encoding_operator( E );

  sb.add_regularization_group_operator( Rx );
  sb.add_regularization_group_operator( Ry);
  sb.add_group();

  sb.set_max_outer_iterations(num_outer_iterations);
  sb.set_max_inner_iterations(num_inner_iterations);

  sb.set_output_mode( cuCgSolver<_real,_real>::OUTPUT_VERBOSE );
  
  // Setup inner conjugate gradient solver
  //

  sb.get_inner_solver()->set_max_iterations
   ( num_cg_iterations );
  sb.get_inner_solver()->set_tc_tolerance( 1e-4 );
  sb.get_inner_solver()->set_output_mode
   ( cuCgSolver<_real,_real>::OUTPUT_WARNINGS );  

  //
  // Run split-Bregman solver
  //

  boost::shared_ptr< cuNDArray<_real> > 
   sbresult = sb.solve(&data);

  << do something with the result >>
}
The constrained Split Bregman solver inherits from the unconstaint Split Bregman solver is thus defined with an identical interface.

2.3. Gadgetron Gadgets

Gadgets wrap the functionality of the toolboxes and provide generic building blocks for configuring the streaming reconstruction in the Gadgetron.

2.3.1. MRI Gadgets

One of the original motivations for creating the Gadgetron was to make a high throughput MRI reconstruction engine that could be interfaced to different MRI vendor systems. Consequently, a lot of the functionality present in the initial release toolboxes and Gadgets is focused on MRI reconstruction. In this section we review the basic data structures used to describe MRI data and list some of the MRI Gadgets that are available. These Gadgets are used in several of the example applications in Chapter 3, Gadgetron Applications.

2.3.1.1. MRI Data Structures

MRI data is processed in two different phases. In the first phase individual data (k-space) acquisitions are processed while in the second phase these acquisitions have been combined into images (which may still be in k-space). Correspondingly, there are two different types of Gadgets that dominate the MRI Gadgets; those who operate on individual acquisitions and those who operate on images. Naturally, there are also transitional Gadgets that operate on acquisitions but output images.

The data header structures used by these MRI Gadgets are defined by the ISMRM Raw Data format (http://ismrmrd.github.io).

Most MRI Gadgets inherit from Gadget2 as described in Section 2.1.1, “Gadgets”, i.e. they operate on two argument types, the main two base classes used are:

Gadget2< ISMRMRD::AcquisitionHeader, hoNDArray< std::complex<float> > >
Gadget2< ISMRMRD::ImageHeader, hoNDArray< std::complex<float> > >
As seen, they take a data array (which is typically of complex float type) and a header describing either the acquisition or the image. These headers are defined in ismrmrd.h (from the ISMRM Raw Data format). The definition of ISMRMRD::AcquisitionHeader looks like (abbreviated):

struct EncodingCounters {
 uint16_t kspace_encode_step_1; 
 uint16_t kspace_encode_step_2; 
 uint16_t average;              
 uint16_t slice;                
 uint16_t contrast;             
 uint16_t phase;                
 uint16_t repetition;           
 uint16_t set;                  
 uint16_t segment;              
 uint16_t user[8];              
};

struct AcquisitionHeader
{
 uint16_t           version;                        
 uint64_t           flags;                          
 uint32_t           measurement_uid;                
 uint32_t           scan_counter;                   
 uint32_t           acquisition_time_stamp;         
 uint32_t           physiology_time_stamp[3];       
 uint16_t           number_of_samples;              
 uint16_t           available_channels;             
 uint16_t           active_channels;                
 uint64_t           channel_mask[16];               
 uint16_t           discard_pre;                    
 uint16_t           discard_post;                   
 uint16_t           center_sample;                  
 uint16_t           encoding_space_ref;             
 uint16_t           trajectory_dimensions;          
 float              sample_time_us;                 
 float              position[3];                    
 float              quaternion[4];                  
 float              patient_table_position[3];      
 EncodingCounters   idx;                            
 int32_t            user_int[8];                    
 float              user_float[8];                 
};
It is a simple struct, which mainly serves the purpose of keeping track of a) the encoding properties of a given acquisition (phase ending number, etc.) and b) the spatial position and orientation that the data was acquired from. Different MRI systems have different conventions for how to label data, but in most cases one would be able to convert to this format.

The ISMRMRD::ImageHeader data structure is also just a struct for keeping track of image labels, position, and orientation:

struct ImageHeader
{
uint16_t            version;                        
 uint64_t            flags;                         
 uint32_t            measurement_uid;               
 uint16_t            matrix_size[3];                
 float               field_of_view[3];              
 uint16_t            channels;                      
 float               position[3];                   
 float               quaternion[4];                 
 float               patient_table_position[3];     
 uint16_t            average;                       
 uint16_t            slice;                         
 uint16_t            contrast;                      
 uint16_t            phase;                         
 uint16_t            repetition;                    
 uint16_t            set;                           
 uint32_t            acquisition_time_stamp;        
 uint32_t            physiology_time_stamp[3];      
 uint16_t            image_data_type;               
 uint16_t            image_type;                    
 uint16_t            image_index;  
 uint16_t            image_series_index;
 int32_t             user_int[8];       
 float               user_float[8];     
};
2.3.1.2. List of available MRI Gadgets

This section contains a non-exhaustive list of available MRI Gadgets with a few brief comments on their function. The purpose is to make it easier to read the XML configuration files provided with the Gadgetron and to give some ideas of what modules can be reused in new reconstruction programs.

AccumulatorGadget (gadgetroncore):

Simple Gadget for accumulating k-space profiles in an array and passing it on to next Gadget. Used for simple Cartesian FT MRI reconstructions.

AutoScaleGadget (gadgetroncore):

Does simple histogram analysis of floating point images passing through and scales them. This is typically used upstream of conversion from floating point to unsigned short images.

CoilReductionGadget (gadgetroncore):

Used to reduce the number of coils in a dataset. Typically used to tune the performance of a given reconstruction by eliminating data. This Gadget is commonly used in conjunction with the PCACoilGadget which generates virtual coils based on principal component analysis. The coil reduction can be specified with either a mask or the number of target coils as illustrated below

<gadget>
 <name>CoilReduction</name>
 <dll>gadgetroncore</dll>
 <class>CoilReductionGadget</class>
 <!-- Keep a max of 16 coils -->
 <property><name>coils_out</name><value>16</value></property>
</gadget>

<gadget>
 <name>CoilReduction</name>
 <dll>gadgetroncore</dll>
 <class>CoilReductionGadget</class>
 <!-- Keep only coil 2,3,4,5 and discard the rest-->
 <property>
  <name>coil_mask</name>
  <value>0 1 1 1 0 0 0 0</value>
 </property>
</gadget>
CropAndCombineGadget (gadgetroncore):

This Gadget is used to do a simple RMS coil combination in the image domain and remove 2x oversampling in the first dimension of the image as is commonly used in MRI. This Gadget is intended to be used after FFT of the data.

ExtractGadget (gadgetroncore):

This Gadget is used to extract a given component (magnitude, real, imaginary, phase) from complex images, i.e. it converts complex images to real images containing specific components. The Gadget can be used to extract multiple components using a mask. The bit fields used to define the components are defined as:

#define GADGET_EXTRACT_MAGNITUDE              (1 << 0) //1
#define GADGET_EXTRACT_REAL                   (1 << 1) //2
#define GADGET_EXTRACT_IMAG                   (1 << 2) //4
#define GADGET_EXTRACT_PHASE                  (1 << 3) //8
To specify the components, you just specify the mask, for example, the following specification would extract magnitude (1) and phase (8):

<gadget>
 <name>Extract</name>
 <dll>gadgetroncore</dll>
 <class>ExtractGadget</class>
 <property><name>extract_mask</name><value>9</value></property>
</gadget>
Default behavior is to extract magnitude.

FFTGadget (gadgetroncore):

This Gadget Fourier transforms along the first 3 dimensions of the dataset (frequency, phase, partition encoding directions) and passes on the data to the next Gadget.

FloatToUShortGadget (gadgetroncore):

Converts floating point images to unsigned short images. This Gadget would often be used in conjunction with a scaling step (e.g. AutoScaleGadget) upstream to ensure that the values will not get clipped or overflow during the conversion to unsigned short. This Gadget does not make any attempt to scale the data, it is assumed to be scaled upon entry.

GPUCGGoldenRadial, GPUCGFixedRadial (gadgetroncgsense):

These Gadgets perform conjugate gradient based non-Cartesian SENSE reconstruction (Section 3.3, “Non-Cartesian 2D Parallel MRI (SENSE)”). The reconstruction behavior can be controlled with number of properties:

<gadget>
 <name>GPUCGRadial0</name>
 <dll>gadgetroncgsense</dll>
 <classname>GPUCGGoldenRadialGadget</classname>

 <property>
  <name>deviceno</name>
  <value>0</value>
 </property>
 
 <property>
  <name>sliceno</name>
  <value>0</value>
 </property>
 
 <property>
  <name>profiles_per_frame</name>
  <value>32</value>
 </property>
 
 <property>
  <name>shared_profiles</name>
  <value>0</value>
 </property>

 <property>
  <name>number_of_iterations</name>
  <value>10</value>
 </property>

 <property>
  <name>cg_limit</name>
  <value>1e-6</value>
 </property>

 <property>
  <name>oversampling</name>
  <value>1.5</value>
 </property>

 <property>
  <name>kernel_width</name>
  <value>5.5</value>
 </property>

 <property>
  <name>kappa</name>
  <value>0.1</value>
 </property>

 <property>
  <name>pass_on_undesired_data</name>
  <value>true</value>
 </property>

</gadget>
GrappaGadget, GrappaUnmixingGadget (gadgetrongrappa):

These Gadgets are used together to perform 2D Cartesian parallel imaging on the GPU. The GrappaGadget is responsible for calculating GRAPPA coefficients and the GrappeUnmixingGadget Fourier transforms the raw data and applies the coefficients. The GrappaGadget has the ability to use target channel compression, i.e. it can reconstruct using fewer target channels than input channels to improve performance. See Section 3.2, “Cartesian 2D Parallel MRI (GRAPPA)” for details. The target channel compression is specificied like this:

<gadget>
 <name>Grappa</name>
 <dll>gadgetrongrappa</dll>
 <class>GrappaGadget</class>
 <property><name>target_coils</name><value>8</value></property>
</gadget>
ImageFinishGadgetSHORT, ImageFinishFLOAT, ImageFinishCPLX (gadgetroncore):

These 3 Gadgets are all template instances of the same ImageFinishGadget. The only different between them is that they operate on different types of image data types as indicated by their names. Their purpose is to return the reconstructed images to the output queue of the Gadgetron so that they can be returned to the client.

MRINoiseAdjustGadget (gadgetronmricore):

The Gadgetron has two noise pre-whitening Gadgets with similar names MRINoiseAdjustGadget and NoiseAdjustGadget. They both perform the same operation, which is a) to collect noise adjust data when present, calculate the noise decorrelation matrix, and perform noise decorrelation (when the noise adjustment data is available). The difference between the two Gadgets is that MRINoiseAdjustGadget uses BLAS and LAPACK routines to perform the operation, which makes it much faster than the NoiseAdjustGadget. The latter Gadget is provided to enable reconstruction on systems where those libraries are not available.

NoiseAdjustGadget (gadgetroncore):

See description of MRINoiseAdjustGadget.

PCACoilGadget (gadgetronmricore):

This Gadget is used to create virtual channels based on principal component analysis of a portion of the data. Specifically, data is accumulated for the first frame (for each location, i.e. slice) and a principal component analysis is done of this data. Once the PCA coefficients are available, all subsequent data will be transformed into the virtual channel domain and passed on down the Gadget chain. This Gadget is often combined with the CoilReductionGadget.

RemoveROOversamplingGadget (gadgetroncore):

Removes the 2x oversampling often used in the readout direction for (Cartesian) MRI.

2.3.2. Python Gadgets

The Gadgetron provides a mechanism to do prototype development in Python. Again, we use MRI as the example application.

The Python layer is accessed through a set of Python Gadgets that can encapsulate a Python module. This is seen in Figure 2.3, “Overview of Python Prototyping”, which illustrates a part of a Gadget chain with two Python Gadgets and one C/C++ Gadget. A Gadget chain can have any number of Python Gadgets and Python Gadgets can be mixed with C++ Gadgets.

Figure 2.3. Overview of Python Prototyping

Overview of Python Prototyping

The Python modules that are encapsulated in the Python Gadgets are expected to have certain characteristics. Specifically, the Gadgets must have at least 3 functions and these functions will be called by the Gadgetron framework at certain specific times:

Gadget reference function. A specific function will be called when the Python Gadget is created. This function is expected to receive a GadgetReference which is a class (wrapped in a Python module), which holds a reference to the Gadget, which owns the Python module. The purpose of passing this reference is to allow the Python module to return data to the Gadget when reconstruction outputs are ready. See below for details.

Configuration function. This function is used to receive the configuration (usually in XML format), when it is passed to the Gadget, i.e. it is the Python equivalent of process_config in the Gadget (see Section 2.1.1, “Gadgets”).

Reconstruction function. This function is called when the Gadget receives data, i.e. it is the Python equivalent of the process function in the Gadget (see Section 2.1.1, “Gadgets”).

The user can chose the names of these functions freely in the Python module, but the function names must be specified when the Gadget is inserted in the XML configuration:

<gadget>
 <name>AccReconPython</name>
 <dll>gadgetronpython</dll>
 <class>AcquisitionPythonGadget</class>

 <property>
  <name>python_path</name>
  <value>/home/myuser/scripts/python</value>
 </property>

 <property>
  <name>python_module</name>
  <value>accumulate_and_recon</value>
 </property>

 <property>
  <name>gadget_reference_function</name>
  <value>set_gadget_reference</value>
 </property>

 <property>
  <name>input_function</name>
  <value>recon_function</value>
 </property>

 <property>
  <name>config_function</name>
  <value>config_function</value>
 </property>
</gadget>

Notice how the 3 function names are specified through the gadget_reference_function, input_function, and config_function parameter names. Also notice that it is possible to specify a python_path to let the Python interpreter know where to search for script. By default, the gadgetron/lib is added to the search path. Multiple pathnames can be added by separating the paths with ";".

The Python script referenced in the XML configuration above could look like this:

import numpy as np
import GadgetronPythonMRI as g
import kspaceandimage as ki
import libxml2

myLocalGadgetReference = g.GadgetReference()
myBuffer = 0
myParameters = 0
myCounter = 1;
mySeries = 1;

def set_gadget_reference(gadref):
    global myLocalGadgetReference
    myLocalGadgetReference = gadref

def config_function(conf):
    global myBuffer
    global myParameters

    myParameters = dict()

    doc = libxml2.parseDoc(str(conf))
    context = doc.xpathNewContext()
    context.xpathRegisterNs("ismrm", "http://www.ismrm.org/ISMRMRD")
    myParameters["matrix_x"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:x")[0]).content)
    myParameters["matrix_y"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:y")[0]).content)
    myParameters["matrix_z"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:z")[0]).content)
    myParameters["channels"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:acquisitionSystemInformation/ismrm:receiverChannels")[0]).content)
    myParameters["slices"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodingLimits/ismrm:slice/ismrm:maximum")[0]).content)+1
    myParameters["center_line"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodingLimits/ismrm:kspace_encoding_step_1/ismrm:center")[0]).content)

    myBuffer = (np.zeros((myParameters["channels"],myParameters["slices"],myParameters["matrix_z"],myParameters["matrix_y"],(myParameters["matrix_x"]>>1)))).astype('complex64')

def recon_function(acq, data):
    global myLocalGadgetReference
    global myBuffer
    global myParameters
    global myCounter
    global mySeries

    line_offset = (myParameters["matrix_y"]>>1)-myParameters["center_line"];
    myBuffer[:,acq.idx.slice,acq.idx.kspace_encode_step_2,acq.idx.kspace_encode_step_1+line_offset,:] = data
    
    if (acq.flags & (1<<7)): #Is this the last scan in slice
        image = ki.ktoi(myBuffer,(2,3,4))
        image = image * np.product(image.shape)*100 #Scaling for the scanner
        #Create a new image header and transfer value
        img_head = g.ImageHeader()
        img_head.channels = acq.active_channels
        img_head.slice = acq.idx.slice
        g.img_set_matrix_size(img_head, 0, myBuffer.shape[4])
        g.img_set_matrix_size(img_head, 1, myBuffer.shape[3])
        g.img_set_matrix_size(img_head, 2, myBuffer.shape[2])
        g.img_set_position(img_head, 0,g.acq_get_position(acq,0))
        g.img_set_position(img_head, 1,g.acq_get_position(acq,1))
        g.img_set_position(img_head, 2,g.acq_get_position(acq,2))
        g.img_set_quaternion(img_head, 0,g.acq_get_quaternion(acq, 0))
        g.img_set_quaternion(img_head, 1,g.acq_get_quaternion(acq, 1))
        g.img_set_quaternion(img_head, 2,g.acq_get_quaternion(acq, 2))
        g.img_set_quaternion(img_head, 3,g.acq_get_quaternion(acq, 3))
        g.img_set_patient_table_position(img_head, 0, g.acq_get_patient_table_position(acq,0))
        g.img_set_patient_table_position(img_head, 1, g.acq_get_patient_table_position(acq,1))
        g.img_set_patient_table_position(img_head, 2, g.acq_get_patient_table_position(acq,2))
        img_head.acquisition_time_stamp = acq.acquisition_time_stamp
        img_head.image_index = myCounter;
        img_head.image_series_index = mySeries;

        myCounter = myCounter + 1
        if (myCounter > 5):
            mySeries = mySeries + 1
            myCounter = 1

        #Return image to Gadgetron
        return myLocalGadgetReference.return_image(img_head,image.astype('complex64'))

        #print "Returning to Gadgetron"
        return 0 #Everything OK

There is a lot going on in this script. Let us walk through the different parts and add some explanation. First look at the imports:

import numpy as np
import GadgetronPythonMRI as g
import GadgetronXML
import kspaceandimage as ki
All the Python Gadget modules must include numpy. The arrays (NDArray) are passed to the Python module as numpy arrays. The second module GadgetronPythonMRI is a Python wrapped version of some of the data structures used in the MRI part of the Gadgetron (see Section 2.3.1, “MRI Gadgets”). Specifically, the IMRMRD::AcquisitionHeader and ISMRMRD::ImageHeader headers are wrapped as Python types (using Boost Python). The GadgetronPythonMRI also contains a wrapped version of the GadgetReference class:

class GadgetReference
{

 public:
  GadgetReference();
  ~GadgetReference();
  
  int set_gadget(Gadget* g)
  {
    gadget_ = g;
    return 0;
  }

  template<class T> int return_data(T header, 
          boost::python::numeric::array arr);

  int return_acquisition(ISMRMRD::AcquisitionHeader acq, 
          boost::python::numeric::array arr);

  int return_image(ISMRMRD::ImageHeader img, 
          boost::python::numeric::array arr);

 protected:
  Gadget* gadget_;

};
Using the return functions in this class interface, it is possible for the Python module to return data to the Gadget. GadgetronXML is a Python module provided with the Gadgetron, which contains some XML helper functions that can (it is not a requirement) be used to parse the XML parameters that the module will receive from process_config. kspaceandimage is also a python module provided with the Gadgetron, it contains some simple wrapper functions for performing Fourier transforms (to and from k-space) of MRI data. The following section contains some initialization of global variables in the Python module;

myRef = g.GadgetReference()
myBuffer = 0
myParameters = 0
myCounter = 1;
mySeries = 1;
As described above, each Python module must contain at least 3 functions corresponding to the 3 entry points from the Gadgetron framework. The first one of these functions captures the GadgetReference:

def set_gadget_reference(gadref):
    global myLocalGadgetReference
    myLocalGadgetReference = gadref
Using this reference, the Python module will be able to return images (or acquisitions) to the Gadget. The next function (config_function processes the configuration data and finally, the recon_function simply takes the data as it comes it and stores it in a buffer. Based on the flags field in the header, it is determined when the last acquisition in each slice has arrived. As this happens the buffer is Fourier transformed, an image header is populated, and the result is returned (via the GadgetReference) to the Gadgetron where it will be processed by the next Gadget in the chain.

The Gadgetron distribution comes with a simple Python-based 2D FT MRI reconstruction. The Gadget chain configuration for this reconstruction can be found in gadgets/python/python.xml.

2.3.3. Making a new Gadget Library

The easiest way to get started making a new Gadget library is to follow an example. In this example we create a new Gadget library containing a single Gadget; ThresholdGadget. Its purpose is to set all values below a certain fraction of the max value to zero.

New Gadget libraries can either be created in the Gadgetron source tree, which allows easy access to all the other files in the Gadgetron, or they can be made as external libraries that link against an installed Gadgetron system. In this example we do the latter since this creates a new library that does not "taint" the Gadgetron source tree. It is trivial to move the library inside the Gadgetron source tree at some later point in time if desired. We assume that the Gadgetron is installed on the machine that you are working on. The command line entries, etc. correspond to a Linux console. If you are using Windows you have to adjust a bit.

Start by creating a new folder for the library:

user@mycomputer:~/temp$ mkdir gadgetron_examplelib
user@mycomputer:~/temp$ cd gadgetron_examplelib
We start by creating the class ThresholdGadget. Create the following 3 files: ThresholdGadget.h, ThresholdGadget.cpp, examplelib_export.h (the last file is just to help us make sure that things work on Windows) with the following content:

//ThresholdGadget.h

#ifndef THRESHOLDGADGET_H
#define THRESHOLDGADGET_H

#include "examplelib_export.h"
#include "Gadget.h"
#include "GadgetMRIHeaders.h"
#include "hoNDArray.h"
#include <complex>

class EXPORTGADGETSEXAMPLE ThresholdGadget : 
public Gadget2<ISMRMRD::ImageHeader, hoNDArray< std::complex<float> > >
{
 public:
  GADGET_DECLARE(ThresholdGadget)

 protected:
  virtual int process( GadgetContainerMessage< ISMRMRD::ImageHeader>* m1,
       GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2);

  virtual int process_config(ACE_Message_Block* mb);

  float threshold_level_;

};

#endif //THRESHOLDGADGET_H
//ThresholdGadget.cpp

#include "ThresholdGadget.h"

int ThresholdGadget::process_config(ACE_Message_Block* mb) 
{
  threshold_level_ = get_double_value("level");
  if (threshold_level_ == 0.0) {
    threshold_level_ = 1.0;
  }

  return GADGET_OK;
}

int ThresholdGadget::process( 
   GadgetContainerMessage< ISMRMRD::ImageHeader>* m1,
   GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
{

  std::complex<float>* d = 
    m2->getObjectPtr()->get_data_ptr();

  unsigned long int elements =  
    m2->getObjectPtr()->get_number_of_elements();

  //First find max
  float max = 0.0;
  for (unsigned long int i = 0; i < elements; i++) {
    if (abs(d[i]) > max) {
      max = abs(d[i]);
    }
  }

  //Now threshold
  for (unsigned long int i = 0; i < elements; i++) {
    if (abs(d[i]) < threshold_level_*max) {
      d[i] = std::complex<float>(0.0,0.0);
    }
  }

  //Now pass on image
  if (this->next()->putq(m1) < 0) {
     return GADGET_FAIL;
  }

  return GADGET_OK;
}

GADGET_FACTORY_DECLARE(ThresholdGadget)
//examplelib_export.h

#ifndef EXAMPLE_EXPORT_H_
#define EXAMPLE_EXPORT_H_


#if defined (WIN32)
#if defined (gadgetronexamplelib_EXPORTS)
#define EXPORTGADGETSEXAMPLE __declspec(dllexport)
#else
#define EXPORTGADGETSEXAMPLE __declspec(dllimport)
#endif
#else
#define EXPORTGADGETSEXAMPLE
#endif

#endif /* EXAMPLE_EXPORT_H_ */
Now that we have the files for the Gadget we need to set up the build environment. In the folder gadgetron_examplelib create a file called CMakeLists.txt with the following content:

cmake_minimum_required(VERSION 2.6)

project(examplelib)

if (WIN32)
ADD_DEFINITIONS(-DWIN32 -D_WIN32 -D_WINDOWS)
ADD_DEFINITIONS(-DUNICODE -D_UNICODE)
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3")
endif (WIN32)

###############################################################
#Bootstrap search for libraries 
# (We need to find cmake modules in Gadgetron)
###############################################################
find_path(GADGETRON_CMAKE_MODULES FindGadgetron.cmake HINTS
$ENV{GADGETRON_HOME}/cmake
/usr/local/gadgetron)

if (NOT GADGETRON_CMAKE_MODULES)
  MESSAGE(FATAL_ERROR "GADGETRON_CMAKE_MODULES cannot be found. 
   Try to set GADGETRON_HOME environment variable.")
endif(NOT GADGETRON_CMAKE_MODULES)

set(CMAKE_MODULE_PATH ${GADGETRON_CMAKE_MODULES})
###############################################################

find_package(Gadgetron REQUIRED)
find_package(Boost REQUIRED)
find_package(ACE REQUIRED)

set(CMAKE_INSTALL_PREFIX ${GADGETRON_HOME})

INCLUDE_DIRECTORIES(${ACE_INCLUDE_DIR} 
     ${Boost_INCLUDE_DIR}
     ${GADGETRON_INCLUDE_DIR})

LINK_DIRECTORIES(${GADGETRON_LIB_DIR})

ADD_LIBRARY(gadgetronexamplelib SHARED ThresholdGadget.cpp)

TARGET_LINK_LIBRARIES(gadgetronexamplelib 
                      hondarray 
                      optimized ${ACE_LIBRARIES} 
                      debug ${ACE_DEBUG_LIBRARY})

INSTALL (FILES ThresholdGadget.h
         examplelib_export.h
         DESTINATION include)

INSTALL(TARGETS gadgetronexamplelib DESTINATION lib)

INSTALL(FILES threshold.xml DESTINATION config)
The last thing we need is the XML configuration file to use when running our new ThresholdGadget. In the same folder create the threshold.xml file:

<?xml version="1.0" ?>
<gadgetronStreamConfiguration 
  xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
  xmlns="http://gadgetron.sf.net/gadgetron"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        
    <reader>
      <slot>1008</slot>
      <dll>gadgetroncore</dll>
      <classname>GadgetIsmrmrdAcquisitionMessageReader</classname>
    </reader>
  
    <writer>
      <slot>1004</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterCPLX</classname>
    </writer>
    <writer>
      <slot>1005</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterFLOAT</classname>
    </writer>
    <writer>
      <slot>1006</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterUSHORT</classname>
    </writer>
  
    <gadget>
      <name>Acc</name>
      <dll>gadgetroncore</dll>
      <classname>AccumulatorGadget</classname>
    </gadget>
    <gadget>
      <name>FFT</name>
      <dll>gadgetroncore</dll>
      <classname>FFTGadget</classname>
    </gadget>

    <gadget>
      <name>CropCombine</name>
      <dll>gadgetroncore</dll>
      <classname>CropAndCombineGadget</classname>
    </gadget>

    <!-- This is where we insert our new Gadget -->
    <gadget>
      <name>Threshold</name>
      <dll>gadgetronexamplelib</dll>
      <classname>ThresholdGadget</classname>
      <property><name>level</name><value>0.25</value></property>
    </gadget>

    <gadget>
      <name>Extract</name>
      <dll>gadgetroncore</dll>
      <classname>ExtractGadget</classname>
    </gadget>  
    <gadget>
      <name>ImageFinishFLOAT</name>
      <dll>gadgetroncore</dll>
      <classname>ImageFinishGadgetFLOAT</classname>
    </gadget>

</gadgetronStreamConfiguration>
Check that you have 5 files in your folder:

user@mycomputer:gadgetron_examplelib$ ls
CMakeLists.txt
ThresholdGadget.cpp
ThresholdGadget.h
examplelib_export.h
threshold.xml
Next, let us create a build directory and compile:

user@mycomputer:gadgetron_examplelib$ mkdir build; cd build
In the build folder

user@mycomputer:build$ cmake ../
Assuming the cmake process was successful:

user@mycomputer:build$ make 
Scanning dependencies of target gadgetronexamplelib

[100%] Building CXX object \
    CMakeFiles/gadgetronexamplelib.dir/ThresholdGadget.cpp.o

Linking CXX shared library libgadgetronexamplelib.dylib
[100%] Built target gadgetronexamplelib

user@mycomputer:build$ make install
[100%] Built target gadgetronexamplelib
Install the project...
-- Install configuration: ""
-- Up-to-date: /usr/local/gadgetron/include/ThresholdGadget.h
-- Up-to-date: /usr/local/gadgetron/include/examplelib_export.h
-- Installing: /usr/local/gadgetron/lib/libgadgetronexamplelib.so
-- Up-to-date: /usr/local/gadgetron/config/threshold.xml
You may have to use sudo for the make install command depending on your setup.

You should now be able to run a reconstruction using your new reconstruction chain. Follow the instructions in Section 1.5, “Hello Gadgetron: Your First Image Reconstruction” if you have not yet tried to run a simple reconstruction. After having started up the Gadgetron, run the mriclient:

user@mycomputer:~/temp/test_data$ mriclient \
    -d gadgetron_testdata.h5 \ 
    -c threshold.xml

Gadgetron MRI Data Sender
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      gadgetron_testdata.h5
  -- hdf5 group in   :      simple_gre
  -- conf            :      theshold.xml
  -- loop            :      1
  -- hdf5 file out   :      ./out.h5
  -- hdf5 group out  :      2012-05-11 12:52:14
(31540|140170355443520) Connection from 127.0.0.1:9002
31540, 81, GadgetronConnector, Close Message received
(31540|140170283570944) Handling close...
(31540|140170283570944) svc done...
(31540|140170283570944) Handling close...
If you run it again with the level parameter set to 0.00000001 (remember to re-install the threshold.xml file in gadgetron/config by running make install):

    <gadget>
      <name>Threshold</name>
      <dll>gadgetronexamplelib</dll>
      <class>ThresholdGadget</class>
      <property><name>level</name><value>0.00000001</value></property>
    </gadget>
You should get two different results that look something like Figure 2.4, “Result from ThresholdGadget experiment”.

Figure 2.4. Result from ThresholdGadget experiment

Result from ThresholdGadget experiment

If you create interesting Gadget libraries please consider publishing them online to the benefit of the reconstruction community. An easy way to do this is by sending them to the Gadgetron team for us to publish right away on the web and possibly include in a future release of the Gadgetron.

2.4. Gadgetron Clients

2.4.1. Available Clients

The purpose of this section is to maintain a list over the available clients that are included in the Gadgetron distribution. The current available clients are:

mriclient:

This is the standard client for sending MRI data to the Gadgetron using the ISMRM Raw Data format. In order to get usage information for the client, simply run the client with no arguments.

2.4.2. Making a new Client

The Gadgetron distribution comes with a GadgetronConnector class, which can be used to create clients. An example main.cpp file for a client could look like:

#include "GadgetMessageInterface.h"
#include "GadgetronConnector.h"

int main(int argc, char** argv)
{

  std::string host_name("localhost");
  std::string port("9002");
  std::string config_file("threshold.xml");
  std::string xml_config;

  //Generate some XML configuration in xml_fconfig

  GadgetronConnector con;

  //Register Readers and Writers
  con.register_writer(....);
  con.register_reader(....);
  con.register_reader(....);

  //Open a connection with the gadgetron
  if (con.open(hostname, port_no) != 0) {
    //Deal with errors
  }

  //Tell Gadgetron which XML configuration to run.
  if (con.send_gadgetron_configuration_file(config_file) != 0) {
    //Deal with errors
  }

  if (con.send_gadgetron_parameters(xml_config) != 0) {
     //Deal with errors
  }


  //Send data
  while ( .... ) { //some condition
    GadgetContainerMessage<GadgetMessageIdentifier>* m1 =
      new GadgetContainerMessage<GadgetMessageIdentifier>();
      
      //Create data and add to m1

      if (con.putq(m1) == -1) {
         //Deal with errors
      }
  }

  //Put a close package on the queue

  GadgetContainerMessage<GadgetMessageIdentifier>* m1 =
    new GadgetContainerMessage<GadgetMessageIdentifier>();

  m1->getObjectPtr()->id = GADGET_MESSAGE_CLOSE;

  if (con.putq(m1) == -1) {
   //Deal with errors
  }

  con.wait(); //Wait for recon to finish

  return 0;
}
To compile this client, create a cmake file:

cmake_minimum_required(VERSION 2.6)

project(exampleclient)

if (WIN32)
ADD_DEFINITIONS(-DWIN32 -D_WIN32 -D_WINDOWS)
ADD_DEFINITIONS(-DUNICODE -D_UNICODE)
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")
SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W3")
endif (WIN32)

###############################################################
#Bootstrap search for libraries 
# (We need to find cmake modules in Gadgetron)
###############################################################
find_path(GADGETRON_CMAKE_MODULES FindGadgetron.cmake HINTS
$ENV{GADGETRON_HOME}/cmake
/usr/local/gadgetron)

if (NOT GADGETRON_CMAKE_MODULES)
  MESSAGE(FATAL_ERROR "GADGETRON_CMAKE_MODULES cannot be found. 
   Try to set GADGETRON_HOME environment variable.")
endif(NOT GADGETRON_CMAKE_MODULES)

set(CMAKE_MODULE_PATH ${GADGETRON_CMAKE_MODULES})
###############################################################

find_package(Gadgetron REQUIRED)
find_package(Boost REQUIRED)
find_package(ACE REQUIRED)

set(CMAKE_INSTALL_PREFIX ${GADGETRON_HOME})

INCLUDE_DIRECTORIES(${ACE_INCLUDE_DIR} 
     ${Boost_INCLUDE_DIR}
     ${GADGETRON_INCLUDE_DIR})

LINK_DIRECTORIES(${GADGETRON_LIB_DIR})

add_executable(mygadgetronclient main.cpp)

target_link_libraries(mygadgetronclient 
      optimized gadgettools debug gadgettools${CMAKE_DEBUG_SUFFIX}
      tinyxml 
      optimized ${ACE_LIBRARIES} debug ${ACE_DEBUG_LIBRARY})

install(TARGETS mygadgetronclient DESTINATION bin)
Run cmake and follow the normal make and make install instructions (see Section 2.3.3, “Making a new Gadget Library”).

Chapter 3. Gadgetron Applications

Table of Contents

3.1. Basic 2D FFT MRI
3.2. Cartesian 2D Parallel MRI (GRAPPA)
3.3. Non-Cartesian 2D Parallel MRI (SENSE)
3.1. Basic 2D FFT MRI

A basic example application in the Gadgetron is a simple 2D FT MRI reconstruction. It receives 2D MRI data, collects it into k-space arrays, performs FFT of the data, combines channels (if there are multiple), and returns the images to the client. This example is included in the Gadgetron for testing and demonstration purposes only. It was not intended to be fast or otherwise optimal in any sense.

The Gadgets for this reconstruction are in the core folder and the configuration file to use to run this reconstruction is default.xml. The section Section 1.5, “Hello Gadgetron: Your First Image Reconstruction” describes how to run a simple reconstruction using this Gadget chain and how to download data to test it.

In this section we will take a closer look at the Gadgets in this chain and how they are implemented. The Gadgetron XML configuration file (default.xml) looks like this:

<?xml version="1.0" ?>  
<gadgetronStreamConfiguration 
  xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
  xmlns="http://gadgetron.sf.net/gadgetron"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        
    <reader>
      <slot>1008</slot>
      <dll>gadgetroncore</dll>
      <classname>GadgetIsmrmrdAcquisitionMessageReader</classname>
    </reader>
  
    <writer>
      <slot>1004</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterCPLX</classname>
    </writer>
    <writer>
      <slot>1005</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterFLOAT</classname>
    </writer>
    <writer>
      <slot>1006</slot>
      <dll>gadgetroncore</dll>
      <classname>MRIImageWriterUSHORT</classname>
    </writer>
  
    <gadget>
      <name>Acc</name>
      <dll>gadgetroncore</dll>
      <classname>AccumulatorGadget</classname>
    </gadget>
    <gadget>
      <name>FFT</name>
      <dll>gadgetroncore</dll>
      <classname>FFTGadget</classname>
    </gadget>

    <gadget>
      <name>CropCombine</name>
      <dll>gadgetroncore</dll>
      <classname>CropAndCombineGadget</classname>
    </gadget>

    <gadget>
      <name>Extract</name>
      <dll>gadgetroncore</dll>
      <classname>ExtractGadget</classname>
    </gadget>
  
    <gadget>
      <name>ImageFinishFLOAT</name>
      <dll>gadgetroncore</dll>
      <classname>ImageFinishGadgetFLOAT</classname>
    </gadget>

</gadgetronStreamConfiguration>
The resulting Gadget chain is illustrated in Figure 3.1, “Simple 2D FT Reconstruction Chain”. As described in Section 2.1.3, “Stream Configuration” the Gadgetron configuration contains 3 sections: Readers, Writers, and the Stream. In this particular case, there is only one Reader, which received MRI Acquisitions. This data format is described in Section 2.3.1, “MRI Gadgets”. There are 3 Writers registered with this configuration. They are all used to write MRI images, but responsible for the different data types (complex float, float, or unsigned short). In principle this means that this reconstruction is capable of returning 3 different types of images, but as is seen from the stream configuration, the only output from this reconstruction will be float format images. However, many reconstructions will have all 3 Writers registered to make it easy to switch formats, i.e. it would be trivial to turn this reconstruction into one that outputs unsigned short images (have a look at the file default_short.xml) for an example of how this is done.

Figure 3.1. Simple 2D FT Reconstruction Chain

Simple 2D FT Reconstruction Chain

As is seen in the Gadgets section of the configuration, this reconstruction uses 5 Gadgets. The first Gadget is responsible for accumulating MRI acquisitions. To accomplish this, it uses an accumulation buffer. When a k-space line arrives at the Gadget, it will be inserted into the k-space buffer and when the last acquisition in a slice/repetition has arrived, it will copy the entire buffer and pass it on to the next Gadget.

Let's have a look at the definition of the AccumulatorGadget class:

class EXPORTGADGETSCORE AccumulatorGadget : 
public Gadget2< ISMRMRD::AcquisitionHeader, 
                hoNDArray< std::complex<float> > >
{
  
 public:
  GADGET_DECLARE(AccumulatorGadget);

  AccumulatorGadget();
  ~AccumulatorGadget();

 protected:
  virtual int process_config(ACE_Message_Block* mb);
  virtual int process(
    GadgetContainerMessage< ISMRMRD::AcquisitionHeader >* m1,
    GadgetContainerMessage< hoNDArray< std::complex<float> > > * m2);

  hoNDArray< std::complex<float> >* buffer_;
  std::vector<unsigned int> dimensions_;

  int image_counter_;
  int image_series_;

};
There are a few member variables to help us keep track of the buffer and the data dimensions and the core functionality is implemented in two functions: process_config, is used to set up the buffer, and process, which is responsible for the accumulation of data. Let us examine the process_config function (abbreviated):

int AccumulatorGadget::process_config(ACE_Message_Block* mb)
{
 boost::shared_ptr<ISMRMRD::ismrmrdHeader> cfg = 
    parseIsmrmrdXMLHeader(std::string(mb->rd_ptr()));

 ISMRMRD::ismrmrdHeader::encoding_sequence e_seq = cfg->encoding();

 ISMRMRD::encodingSpaceType e_space = (*e_seq.begin()).encodedSpace();
 ISMRMRD::encodingSpaceType r_space = (*e_seq.begin()).reconSpace();
 ISMRMRD::encodingLimitsType e_limits = (*e_seq.begin()).encodingLimits();

 GADGET_DEBUG2("Matrix size: %d, %d, %d\n", 
                e_space.matrixSize().x(), 
                e_space.matrixSize().y(), 
                e_space.matrixSize().z());

 dimensions_.push_back(e_space.matrixSize().x());
 dimensions_.push_back(e_space.matrixSize().y());
 dimensions_.push_back(e_space.matrixSize().z());

 slices_ = e_limits.slice().present() ? 
             e_limits.slice().get().maximum()+1 : 1;

  return GADGET_OK;
}
The main purpose of this function is to pull parameters out of the XML portion of the ISMRM Raw Data header in order to set up the buffer. As mentioned in Section 2.1.1.1, “Gadget XML Configuration”, the convention is to pass parameters into the Gadgets in XML format. To enable convenient parsing of these parameters, the ISMRMRD library includes a C++ class representation of the header. See http://ismrmrd.github.io for more details.

Now we are ready to receive and buffer data, which is done by the process function:

int AccumulatorGadget::
process(GadgetContainerMessage<ISMRMRD::AcquisitionHeader>* m1,
 GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
{

  if (!buffer_) {
   dimensions_.push_back(m1->getObjectPtr()->active_channels);
   dimensions_.push_back(slices_);

   if (!(buffer_ = new hoNDArray< std::complex<float> >())) {
    GADGET_DEBUG1("Failed create buffer\n");
    return GADGET_FAIL;
   }

   if (!buffer_->create(&dimensions_)) {
    GADGET_DEBUG1("Failed allocate buffer array\n");
    return GADGET_FAIL;
   }

   image_series_ = this->get_int_value("image_series");

  }


  std::complex<float>* b =
    buffer_->get_data_ptr();

  std::complex<float>* d =
    m2->getObjectPtr()->get_data_ptr();

  int samples =  m1->getObjectPtr()->number_of_samples;
  int line = m1->getObjectPtr()->idx.kspace_encode_step_1;
  int partition = m1->getObjectPtr()->idx.kspace_encode_step_2;
  int slice = m1->getObjectPtr()->idx.slice;

  if (samples > static_cast<int>(dimensions_[0])) {
   GADGET_DEBUG1("Wrong number of samples received\n");
   return GADGET_FAIL;
  }

  size_t offset= 0;
  //Copy the data for all the channels
  for (int c = 0; c < m1->getObjectPtr()->active_channels; c++) {
    offset = 
      slice*dimensions_[0]*dimensions_[1]*dimensions_[2]*dimensions_[3] +
      c*dimensions_[0]*dimensions_[1]*dimensions_[2] +
      partition*dimensions_[0]*dimensions_[1] +
      line*dimensions_[0] + (dimensions_[0]>>1)-m1->getObjectPtr()->center_sample;
    
    memcpy(b+offset,
     d+c*samples,
     sizeof(std::complex<float>)*samples);
  }
  
  bool is_last_scan_in_slice = 
     ISMRMRD::FlagBit(ISMRMRD::ACQ_LAST_IN_SLICE).isSet(m1->getObjectPtr()->flags);
  
  if (is_last_scan_in_slice) {
    GadgetContainerMessage<ISMRMRD::ImageHeader>* cm1 = 
      new GadgetContainerMessage<ISMRMRD::ImageHeader>();
    
    cm1->getObjectPtr()->flags = 0;

    GadgetContainerMessage< hoNDArray< std::complex<float> > >* cm2 = 
      new GadgetContainerMessage<hoNDArray< std::complex<float> > >();
    
    cm1->cont(cm2);
    
    std::vector<unsigned int> img_dims(4);
    img_dims[0] = dimensions_[0];
    img_dims[1] = dimensions_[1];
    img_dims[2] = dimensions_[2];
    img_dims[3] = dimensions_[3];
    
    if (!cm2->getObjectPtr()->create(&img_dims)) {
      GADGET_DEBUG1("Unable to allocate new image array\n");
      cm1->release();
      return -1;
    }
    
    size_t data_length = dimensions_[0]*dimensions_[1]*
      dimensions_[2]*dimensions_[3];
    
    offset = slice*data_length;
    
    memcpy(cm2->getObjectPtr()->get_data_ptr(),b+offset,
    sizeof(std::complex<float>)*data_length);
    
    cm1->getObjectPtr()->matrix_size[0]     = img_dims[0];
    cm1->getObjectPtr()->matrix_size[1]     = img_dims[1];
    cm1->getObjectPtr()->matrix_size[2]     = img_dims[2];
    cm1->getObjectPtr()->channels           = img_dims[3];
    cm1->getObjectPtr()->slice   = m1->getObjectPtr()->idx.slice;

    memcpy(cm1->getObjectPtr()->position,
      m1->getObjectPtr()->position,
    sizeof(float)*3);

    memcpy(cm1->getObjectPtr()->quaternion,
      m1->getObjectPtr()->quaternion,
    sizeof(float)*4);
 
    memcpy(cm1->getObjectPtr()->patient_table_position,
      m1->getObjectPtr()->patient_table_position, sizeof(float)*3);

    cm1->getObjectPtr()->image_data_type = ISMRMRD::DATA_COMPLEX_FLOAT;
    cm1->getObjectPtr()->image_index = ++image_counter_;
    cm1->getObjectPtr()->image_series_index = image_series_;

    if (this->next()->putq(cm1) < 0) {
     return GADGET_FAIL;
    }
  } 

  m1->release();
  return GADGET_OK;
}
This function has two basic tasks: insert data into the buffer and when enough data is present, copy the buffer and pass it on to next gadget. Additionally, the data buffer is created in this function if it is not already allocated. In this example we choose to allocate the buffer after the first data elements arrive. This allows us to respond to changes in data sizes introduced by upstream Gadgets, e.g. readout downsampling, coil reduction, etc.

In this case the copying of data is done with a very simple memcpy command. There is a basic check for the image dimensions, but a more robust application may have more checks of the incoming data.

Once the data is in the buffer, we check to see if we should put out an image. This is done with the flags field on the acquisition. Specifically we check if a specific bit (ISMRMRD::ACQ_LAST_IN_SLICE) is set.

If it is determined that this is the last acquisition for this slice, we create a copy of the buffer and pass it on to the next Gadget. Instead of a ISMRMRD::AcquisitionHeader we now need an ISMRMRD::ImageHeader to pass along with the data. This header structure is created and populated with fields (orientation, etc.) from the acquisition header before it is passed on to the Gadget in the stream.

Next Gadget is the FFTGadget. Since the k-space buffering has been taken care of, the Fourier transform is a relatively simple task. The process function uses the FFTW wrapper class (Section 2.2.4, “Fourier Transforms”) to perform the FFT along the first 3 dimensions of the array:

int FFTGadget::process( 
GadgetContainerMessage< ISMRMRD::ImageHeader>* m1,
GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
{
  FFT<float>::instance()->ifft(m2->getObjectPtr(),0);
  FFT<float>::instance()->ifft(m2->getObjectPtr(),1);
  FFT<float>::instance()->ifft(m2->getObjectPtr(),2);

  if (this->next()->putq(m1) < 0) {
     return GADGET_FAIL;
  }

  return GADGET_OK;
}
Now that the images have been Fourier transformed, we need to remove the oversampling that is done in the readout dimensions and we need to combine the receiver channels. In this case, we are making some assumptions, i.e. we assume two-fold oversampling in the readout and we are doing a simple RMS coil combination to obtain combined magnitude images. We will not repeat the source code here, it can be found in gadgets/core/CropAndCombineGadget.cpp.

Last two remaining steps after the coil combination is to extract the magnitude of the data and return the floating point images to the Gadgetron so that they can be returned to the client. This is accomplished in the ExtractGadget and the ImageFinishGadgetFLOAT. Both of these Gadgets are described in Section 2.3.1, “MRI Gadgets”.

3.2. Cartesian 2D Parallel MRI (GRAPPA)

The Gadgetron contains a high-throughput real-time 2D Cartesian parallel imaging reconstruction (GRAPPA) implemented on the GPU. It is beyond the scope of this manual to review all the algorithmic details of this application, but we will give an overview here as an example of a more complicated reconstruction chain.

The Gadget chain is defined in the grappa.xml and the resulting chain is illustrated in Figure 3.2, “GRAPPA Reconstruction Chain”.

To test this configuration, please download the GRAPPA test datasets from http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/testdata/ismrmrd, where you will find the dataset grappa_rate2 It is a Cartesian parallel imaging datasets with rate 2 TSENSE type acquisition. Data were acquired with a 32 channel coil.

In order to run the GRAPPA reconstruction you have to have a CUDA enable GPU on your system and your Gadgetron distribution should be compiled with CUDA and CULA enabled. Please see Section 1.4, “Compiling and Installing Gadgetron” for details for your specific platform.

To run the reconstruction, start up your Gadgetron (in its own terminal window) and use the mriclient to send the data from another terminal:

user@host:~/temp$ wget http://sourceforge.net/projects/gadgetron/files/testdata/ismrmrd/grappa_rate2.h5

user@host:~/temp$ mriclient \
    -d grappa_rate2.h5 \
    -c grappa.xml
Gadgetron MRI Data Sender
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      gadgetron_testdata.h5
  -- hdf5 group in   :      gre_tgrappa_rate4
  -- conf            :      grappa.xml
  -- loop            :      1
  -- hdf5 file out   :      ./out.h5
  -- hdf5 group out  :      2012-05-11 15:43:03
(32580|140398140757824) Connection from 127.0.0.1:9002
32580, 81, GadgetronConnector, Close Message received
(32580|140398068885248) Handling close...
(32580|140398068885248) svc done...
(32580|140398068885248) Handling close...
You should get example images that look similar to the ones in Figure 3.3, “GRAPPA Reconstruction Results”.

Figure 3.2. GRAPPA Reconstruction Chain

GRAPPA Reconstruction Chain

Figure 3.3. GRAPPA Reconstruction Results

GRAPPA Reconstruction Results

Let's take a closer look at some of the components of this reconstruction application.

The first Gadget is the NoiseAdjustGadget. As described in Section 2.3.1, “MRI Gadgets”, the purpose of this Gadget is to decorrelate the noise in the receiver channels. This improves the parallel imaging performance, especially in cases where there is a large amount of noise in just a few receiver elements. There are two versions of this Gadget, one that uses the BLAS/LAPACK routines for performance improvements and one that implements the same functionality without these optimizations. When you call the included grappa.xml configuration, you will use the optimized version. If you do not have BLAS and LAPACK on your system, you can modify the XML configuration to use the one from the gadgets/core library.

Second step is removing the oversampling. This step could also be performed after the reconstruction (as it is done in Section 3.1, “Basic 2D FFT MRI”), but here we opt to remove this excess data to improve downstream performance.

The purpose of the next two Gadgets (PCAGadget and CoilReductionGadget) is to a) transform the receiver coils into PCA virtual coils ordered by their information content and b) remove some of the coils to improve downstream performance. The first step is achieved by buffering the first frame of data and then performing a principal component analysis (PCA) on the first frame of data. Based on the determined PCA transformation all data is then subsequently transformed into virtual coils. In the coil reduction gadget we can now simple eliminate the channels that are above a certain number. See Section 2.3.1, “MRI Gadgets” for details on how to control the channel compression.

The next two Gadgets are responsible for the actual GRAPPA reconstruction. The GrappaGadget calculates the GRAPPA coefficients and GrappaUnmixingGadget performs the Fourier transform of the raw data and applies the GRAPPA coefficients to the aliased imaged to obtain unaliased images.

In general it is assumed that the data is acquired in such a way that a set of neighboring frames can be averaged to yield a fully sampled k-space; the data is acquired with a time-interleaved sampling pattern. When enough calibration data is available to calculate GRAPPA coefficients, i.e. when a fully sampled region of k-space is available, the calibration data is sent to a grappa coefficient calculation object (GrappaWeightsCalculator).

The GrappaWeightsCalculator is an active object, which picks up weight calculation jobs from an input queue and passes them on to the GPU where it uses toolbox functions to calculate GRAPPA unmixing coefficients. These coefficients are Fourier transformed to image space where they are combined for all coils and stored in a GrappaWeights object.

When the GrappaGadget passes on the raw data to the GrappaUnmixingGadget it passes a reference to the GrappaWeights object which is to be used when performing the unmixing operation. Let's have closer look at the GrappaUnmixingGadget.h file:

struct GrappaUnmixingJob
{
 boost::shared_ptr< GrappaWeights<float> > weights_;
};

class GrappaUnmixingGadget: 
public Gadget3<GrappaUnmixingJob, 
               ISMRMRD::ImageHeader, 
               hoNDArray<std::complex<float> > > 
{
public:
 GADGET_DECLARE(GrappaUnmixingGadget);

 GrappaUnmixingGadget();
 virtual ~GrappaUnmixingGadget();
protected:
 virtual int process(GadgetContainerMessage<GrappaUnmixingJob>* m1,
   GadgetContainerMessage<ISMRMRD::ImageHeader>* m2, 
   GadgetContainerMessage<hoNDArray<std::complex<float> > >* m3);

};
We can see that the GrappaUnmixingGadget is an example of a Gadget, which takes 3 arguments and the additional argument in this case holds a reference to the unmixing coefficients.

The GrappaWeightsCalculator will update the coefficients as often as it is instructed to do so and the GrappaGadget is in charge of determining when an update should be done. Specifically, it monitors the incoming data and when the slice orientation changes, a job will be submitted to update the coefficients. If the slice is not changing, it is in principle OK to continue with the current coefficients, but if data is available and the GrappaWeightsCalculator is idle (the queue is empty) a job will be submitted.

With this design, the data passes through the GrappaGadget very quickly and the GrappaUnmixingGadget can reconstruction the images very quickly, i.e. it is simply a Fourier transform and an element wise multiplication and sum over the coils. It is in other words designed for very high throughput.

If the slice orientation changes, new coefficients will be calculated, but this calculation will not be done by the time the data reaches the GrappaUnmixingGadget and consequently, the images will be reconstructed with the "old" coefficients until the coefficients are ready. This design ensures low latency, but when the slice changes, aliasing may occur for a few frames until coefficients are updated.

After the unmixing, the images are scaled and magnitude is extracted before returning images to the client. The AutoScaleGadget has been added in this case to ensure that images are in a reasonable range before converting to unsigned short as the output in this case. Automatic image scaling can be problematic, especially when doing quantitative imaging, but it was added in this case to make the reconstruction more robust to data from different sources. A better solution is to only use data where noise calibration data is available and reconstruct SNR scaled images. Based on typical SNR values for MRI images, it is fairly trivial to keep the images in the appropriate range and perform a proper conversion to unsigned short.

A final comment about the GRAPPA reconstruction is that it allows a second step of channel compression. More specifically, it is possible to reconstruct to a limited number of target channels to further improve performance. Between the upstream and downstream channel compression steps, it is possible to tune the performance of the reconstruction to enable real-time reconstruction on the available hardware.

3.3. Non-Cartesian 2D Parallel MRI (SENSE)

The Gadgetron includes a real-time implementation of a GPU-based real-time non-Cartesian Sense reconstruction published in [SANGILD09]. One of the keys to obtaining real-time performance is an efficient GPU implementation of the non-Cartesian Fast Fourier Transform [SANGILD08]. The application reuses several of the gadgets we have seen in use already for the Cartesian Grappa implementation above (Section 3.2, “Cartesian 2D Parallel MRI (GRAPPA)”). An overview of the non-Cartesian Sense gadget chain is given in figure Figure 3.4, “Gadgetron Chain for Non-Cartesian Sense”.

Figure 3.4. Gadgetron Chain for Non-Cartesian Sense

Gadgetron chain for non-Cartesian Sense


The CGSenseGadget implements the non-Cartesian Sense reconstruction. It contains a conjugate gradient solver (Section 2.2.6, “Linear Solvers”) set up with a nonCartesianSense image encoding matrix and an imageOperator for regularization. Internally it maintains a cyclic buffer of a few seconds of imaging data. It uses this buffer to maintain a fully sampled (i.e. unaliased but blurred) k-space image from which coil sensititivities and regularization images are dynamically estimated. The combination of parallel imaging and image regularization operators allows for alias-suppressed image reconstruction using significant undersampling hereby achieving real-time data acquisition rates per frame. The conjugate gradient solver is able to reconstruct faster than the acquisition time e.g. a 192x192 image from 32 coils using 10 solver iterations on newer graphics hardware.

To test this configuration use the 32 channel radial MRI test dataset (golden_angle.h5), which you can download from http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/testdata/ismrmrd/. We assume that you have added $(GADGETRON_HOME)/bin to your PATH environment variable. You need a CUDA enable GPU on your system and your Gadgetron distribution should be compiled with CUDA and CULA enabled. Please see Section 1.4, “Compiling and Installing Gadgetron” for details for your specific platform.

To run the reconstruction; start up gadgetron (in its own terminal window) and use the mriclient to send the data from another terminal. First start gadgetron:

user@host$ gadgetron 
Configuring services
If asked, allow the gadgetron application to allow incoming network connection. Next start the mriclient:

user@host:~/temp$ wget http://sourceforge.net/projects/gadgetron/files/testdata/ismrmrd/golden_angle.h5

user@host:~/temp$ mriclient \
       -d golden_angle.h5 \
       -c radial_single.xml

Gadgetron MRI Data Sender
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      gadgetron_testdata.h5
  -- hdf5 group in   :      gre_golden_angle
  -- conf            :      radial_single.xml
  -- loop            :      1
  -- hdf5 file out   :      ./out.h5
  -- hdf5 group out  :      2012-05-11 15:47:22
(32608|139797448419136) Connection from 127.0.0.1:9002
32608, 81, GadgetronConnector, Close Message received
(32608|139797376546560) Handling close...
(32608|139797376546560) svc done...
(32608|139797376546560) Handling close...

Your current folder now holds the reconstructed images in the out.h5 HDF5 file. They will look something like the one depicted in Figure 3.5, “Non-Cartesian Sense Reconstruction Results”.

Figure 3.5. Non-Cartesian Sense Reconstruction Results

Non-Cartesian Sense Reconstruction Results


Chapter 4. Standalone Applications

Table of Contents

4.1. Image Denoising
4.2. Image Deblurring
4.3. Non-Cartesian FFT
4.4. Non-Cartesian parallel MRI (SENSE)
This chapter demonstrates through a few examples how to use the Gadgetron toolboxes (Section 2.2, “Gadgetron Toolboxes”) to build standalone applications outside the streaming framework. You need a CUDA enabled GPU on your system and your Gadgetron distribution should be compiled with CUDA (and CULA) enabled. Then the examples are automatically build with the Gadgetron and binaries should consequently be available in $GADGETRON_HOME/bin.

4.1. Image Denoising

This example uses the unconstraint Split Bregman solver for total variation based 2D image denoising. The encoding matrix is defined as an identityOperator and a partialDerivativeOperator is used for each of the two spatial directions to implement the total variation regularization term. The two partial derivatives are added as a "group" of regularization operators to implement isotropic denoising. Alternatively, by changing a few lines of code they can be added as individual regularization operators instead to implement anisotropic denoising.

The full source code for the example can be found at $(GADGETRON_SOURCE)/apps/standalone/gpu/denoising/2d/denoise_TV.cpp.

You can download some noisy Shepp-Logan phantom test datasets from http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/testdata/phantom/shepp.tar.gz

In a terminal, go to the folder in which you unpacked the data. We assume that you have added $(GADGETRON_HOME)/bin to your PATH environment variable.

Try

user@host$ denoise_TV -d shepp_logan_256_256_med_noise.real -O 250 -m 1
Running denoising with the following parameters: 
---------------------------------------------------- 
  Noisy image file name (.real)  : shepp_logan_256_256_med_noise.real 
  Result file name               : denoised_image_TV.real 
  Number of cg iterations        : 20 
  Number of sb inner iterations  : 1 
  Number of sb outer iterations  : 250 
  Regularization weight (mu)     : 1 
---------------------------------------------------- 
...
user@host$
which runs 250 iterations of the solver with a regularization weight of 1. The output is saved in the current folder in the file denoised_image_TV.real.

The noisy and denoised phantom is depicted below.

Figure 4.1. A noisy version of the Shepp-Logan phantom

A noisy version of the Shepp-Logan phantom

Figure 4.2. Result after total variation denoising

Result after total variation denoising

Running denoise_TV with no arguments prints out a brief usage description. We leave it as an exercise to run the algorithm with various settings. The data file you downloaded contains two further dataset (with lower and higher noise levels respectively) to try out as well.

4.2. Image Deblurring

This example uses 1) the linear least squares solver, and 2) the constraint Split Bregman solver for image deblurring. The encoding matrix is defined as a convolutionOperator. A partialDerivativeOperator is added for each of the two spatial directions as regularization terms.

We reuse the Shepp-Logan data from the image denoising experiment above (Section 4.1, “Image Denoising”).

First we generate a blurry Shepp-Logan phantom by convolution with a Gaussian kernel. This is easily achieved using the method mult_M in the convolutionOperator. Source code is provided at $(GADGETRON_SOURCE)/apps/standalone/gpu/deblurring/2d/blur_2d.cpp

In a terminal, go to the folder in which you unpacked the Shepp-Logan phantom.

Try

user@host$ blur_2d -d shepp_logan_256_256_no_noise.real
which generates two complex images; blurred_image.cplx and kernel_image.cplx. For convenience a corresponding magnitudes image is also saved as blurred_image.real.

Next run the conjugate gradient solver. The source code for the example can be found in $(GADGETRON_SOURCE)/apps/standalone/gpu/deblurring/2d/deblur_2d_cg.cpp.

user@host$ deblur_2d_cg -K 1e-4
 Running deblurring with the following parameters: 
---------------------------------------------------- 
  Blurred image file name (.cplx)  : blurred_image.cplx 
  Kernel image file name (.cplx)   : kernel_image.cplx 
  Result file name                 : cg_deblurred_image.cplx 
  Number of iterations             : 25 
  Regularization weight            : 1e-4 
---------------------------------------------------- 
Iterating...
...
user@host$
The result is saved in the current folder in the file cg_deblurred_image.cplx. A magnitudes image is also saved as cg_deblurred_image.real.

Next run the constraint Split Bregman solver. The source code for the example can be found in $(GADGETRON_SOURCE)/apps/standalone/gpu/deblurring/2d/deblur_2d_sb.cpp.

user@host$ deblur_2d_sb -O 100 -L 0.5 -M 0.5
 Running deblurring with the following parameters: 
---------------------------------------------------- 
  Blurred image file name (.cplx)  : blurred_image.cplx 
  Kernel image file name (.cplx)   : kernel_image.cplx 
  Result file name                 : sb_deblurred_image.cplx 
  Number of cg iterations          : 20 
  Number of sb inner iterations    : 1 
  Number of sb outer iterations    : 100 
  Mu                               : 0.5 
  Lambda                           : 0.5 
---------------------------------------------------- 
...
user@host$
The result is saved as sb_deblurred_image.cplx. A magnitudes image is also saved as sb_deblurred_image.real.

The blurred and deblurred phantoms are depicted below.

Figure 4.3. Blurry Shepp-Logan phantom

Blurry Shepp-Logan phantom

Figure 4.4. Deblurred phantom from the Conjugate Gradient solver

Deblurred phantom from the Conjugate Gradient solver

Figure 4.5. Deblurred phantom from the constrained Split Bregman solver

Deblurred phantom from the constrained Split Bregman solver

In the present examples no noise was added to the blurred images before the deconvolution. Consequently for the conjugate gradient solver, a very low weight of the regularization term was "sufficient". We leave it as an exercise to run the algorithms with various settings. In particular, try to add noise to the blurred image before the deconvolution to observe the very ill-posed nature of the problem.

Notice. If the dimensions of the provided convolution kernel is exactly double that of the provided image, the convolution operator zero-pads the image before the convolution and removes the padding again after. As the convolution operator utilizes FFTs in its implementation, this oversampling is a way of avoiding cyclic boundary conditions during the convolution.

4.3. Non-Cartesian FFT

This example shows how to use the forwards and adjoint non-Cartesian Fast Fourier Transform (NFFT and NFFTH respectively) on a 2D image. The source code can be found at $(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/nfft/2d/main_nfft.cpp and $(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/nfft/2d/main_nffth.cpp.

We reuse the Shepp-Logan data downloaded for the previous experiments (Section 4.1, “Image Denoising”).

In the following we run the NFFT followed by the NFFTH. The image matrix size is 2562. We use an oversampled matrix size of 3842, 128 profiles in k-space (undersampling) with 384 samples each. The NFFT Kaiser-Bessel convolution kernel width is set to 5.52 (see [SANGILD08]).

user@host$ nfft -d shepp_logan_256_256_no_noise.real \ 
   -o 384 -p 128 -s 384 -k 5.5

 Running reconstruction with the following parameters: 
---------------------------------------------------- 
  Input image file name (.real)  : shepp_logan_256_256_no_noise.real 
  Result file name               : samples.cplx 
  Oversampled matrix size        : 384 
  Number of profiles             : 128 
  Samples per profiles           : 384 
  Kernel width                   : 5.5 
---------------------------------------------------- 
Loading image from disk
Uploading, normalizing and converting to complex
Initializing plan
Computing golden ratio radial trajectories
NFFT preprocessing
Computing density compensation weights
Computing nfft (inverse gridding)
Output result to disk
user@host$
user@host$ nffth -d samples.cplx -m 256 -o 384 -k 5.5
 Running reconstruction with the following parameters: 
---------------------------------------------------- 
  Input samples file name (.cplx)  : samples.cplx 
  Output image file name (.cplx)   : result.cplx 
  Matrix size                      : 256 
  Oversampled matrix size          : 384 
  Kernel width                     : 5.5 
---------------------------------------------------- 
Loading samples from disk
Uploading samples to device
Initializing plan
Computing golden ratio radial trajectories
NFFT preprocessing
Computing density compensation weights
Computing nffth (gridding)
Output result to disk
user@host$
The result is saved in file result.cplx. A magnitudes image is saved as result.real. As an exercise, experiment with the settings to reduce (or increase) the aliasing.

The $(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/nfft/2d/ folder also contains examples of using the nfftOperator in a Conjugate Gradient solver and a Split Bregman solver respectively.

4.4. Non-Cartesian parallel MRI (SENSE)

This section demonstrates how to run a standalone non-Cartesian parallel MRI reconstruction similar to the one that was previously shown using the streaming framework infrastructure in sectionSection 3.3, “Non-Cartesian 2D Parallel MRI (SENSE)”. More details can be found in [SANGILD09].

In addition to a regularized linear least squares solution to the reconstruction problem, we furthermore use the Split Bregman solver to obtain the solution with minimum total variation subject to the constraint of the encoding operator (compressed sensing).

Download a free-breathing cardiac MRI sample dataset from http://sourceforge.net/projects/gadgetron/files/testdata/mri/fb_data.zip

Source code is found in the files main_cg.cpp and main_sbc.cpp in directory

$(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio.

Both command lines below produce a 2D image sequence, each image with a matrix size of 1922 (-m). 32 projections are used for each frame (-p) for a frame rate of roughly 32 profiles/frame * 2.5 ms/profile = 80 ms ms/frame or 12.5 frames/s. The reconstruction results are written out in both complex and magnitude data as result.cplx and result.real respectively.

If sufficient device memory is available on your GPU (i.e. you are in possession of a high-end card) all frames in the sequence can be reconstructed concurrently (as a 3D volume). On systems that do not hold enough device memory to reconstruct all frames in parallel, they can instead be reconstructed in several batches. The -f option to the command lines below indicate the number of frames that are reconstructed per batch. A negative value indicates "all". If the command below fails to complete due to lack of device memory, try running with argument -f 8 (or an even smaller number) instead.

The following output was obtained on a Geforce GTX 480 GPU.

user@host$ radial_sense_cg -d fb_data.cplx -m 192 -o 256 -p 32 -K 0.01

  Running reconstruction with the following parameters: 
---------------------------------------------------- 
  Sample data file name                             : fb_data.cplx 
  Result file name                                  : result.cplx 
  Matrix size                                       : 192 
  Oversampled matrix size                           : 256 
  Profiles per frame                                : 32 
  Frames per reconstruction (negative meaning all)  : -1 
  Number of iterations                              : 10 
  Kernel width                                      : 5.5 
  Kappa                                             : 0.01 
---------------------------------------------------- 

Loading data: 18.339 ms

#samples/profile: 256
#profiles/frame: 32
#profiles: 2560
#coils: 4
#frames/reconstruction: 80
#profiles/reconstruction: 2560
#samples/reconstruction: 655360

Filling rhs buffer: 283.675 ms
Estimating csm: 3.435 ms
Computing regularization: 0.319 ms
Computing preconditioning weights: 0.081 ms
Iterating...
Iteration 0. rq/rq_0 = 0.453177
Iteration 1. rq/rq_0 = 0.132643
Iteration 2. rq/rq_0 = 0.0413432
Iteration 3. rq/rq_0 = 0.0144378
Iteration 4. rq/rq_0 = 0.00681063
Iteration 5. rq/rq_0 = 0.00450857
Iteration 6. rq/rq_0 = 0.00342872
Iteration 7. rq/rq_0 = 0.00240418
Iteration 8. rq/rq_0 = 0.00146108
Iteration 9. rq/rq_0 = 0.000903398
GPU Conjugate Gradient solve: 2115.7 ms
Full SENSE reconstruction.: 2188.68 ms
Writing out result: 50.111 ms

user@host$
user@host$ radial_sense_sbc -d fb_data.cplx -m 192 -o 256 -p 32 

  Running reconstruction with the following parameters: 
---------------------------------------------------- 
  Sample data file name                             : fb_data.cplx 
  Result file name                                  : result.cplx 
  Matrix size                                       : 192 
  Oversampled matrix size                           : 256 
  Profiles per frame                                : 32 
  Frames per reconstruction (negative meaning all)  : -1 
  Number of cg iterations                           : 20 
  Number of sb inner iterations                     : 1 
  Number of sb outer iterations                     : 20 
  Kernel width                                      : 5.5 
  Mu                                                : 1.0 
  Lambda                                            : 2.0 
---------------------------------------------------- 

Loading data: 16.082 ms

#samples/profile: 256
#profiles/frame: 32
#profiles: 2560
#coils: 4
#frames/reconstruction 80
#profiles/reconstruction 2560
#samples/reconstruction 655360

CSM and regularization estimation: 288.983 ms

...

GPU constrained Split Bregman solve: 57257.4 ms
Full SENSE reconstruction with TV regularization.: 57330.3 ms
Writing out result: 50.421 ms

user@host$
As all 80 frames are reconstructed in parallel it is straightforward to add temporal regularization to the reconstructions. We leave this as a suggested exercise for the reader.

For the interested reader, an implementation of kt-Sense can be found in directory

$(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio_kt.

Additionally, the source code for the user interface demonstrated in the movie accompanying [SANGILD09] can be found (if you configured cmake to inlude Qt support) in direcotry

$(GADGETRON_SOURCE)/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio_gui.

Chapter 5. Frequently Asked Questions (FAQ)

Can I make a branching Gadget chain?

The short answer is no. We plan on supporting this in a future release, but it is not quite ready yet.

How can I help?

We are always looking for people who are interested in helping with the continuing development of the Gadgetron. There are many things you can do:

Use it.

When you develop new Gadgets or Toolboxes, please consider submitting them to us so that we can include them in the archive.

Help us implement some of the future features in Appendix C, Future Features. It is probably a good idea to get in touch with us before you start coding, just in case somebody is already working on it.

Appendix A. Simple Array File Format

When working with the Gadgetron it is often necessary to write files with reconstructed images to disk, either as part of debugging or as the final reconstruction result. We have adopted a very simple multidimensional array file format for this purpose. The main advantage of this file format is its simplicity but there are a number of disadvantages and caveats as well as described in this section.

The simple array files are made up of a) a header followed by b) the data itself. This layout of data and header is illustrated in Figure A.1, “Simple Array File Format”. The header has a single 32-bit integer to indicate the number of dimensions of the dataset followed by one integer for each dimension to indicate the length of that dimension. The data follows immediately after the header. The data is stored such that the first dimension is the fastest moving dimension, second dimension is second fastest, etc. The header contains no information about the size of each individual data element and consequently the user needs to know what type of data is contained in the array. In general, the Gadgetron uses 3 different types of data and the convention is to use the file extension to indicate the data type in the file:

16-bit unsigned short. File extension: *.short

32-bit float. File extension: *.real

32-bit complex float. Two 32-bit floating point values per data element. File extension: *.cplx

Figure A.1. Simple Array File Format

Simple Array
The simple array file format has a header followed by the data. The header consists of one 32-bit integer defining the number of dimensions (N-dimensions) followed by N-dimensions 32-bit unsigned integers each defining the length of each dimensions. In the example, the dataset has 4 dimensions and the size of those dimensions is 128x128x1x1, i.e. 16384 elements.


The Gadgetron framework provides function for reading these files in C++. The functions are located in toolboxes/ndarray/hoNDArray_fileio.h in the Gadgetron source code distribution.

It is also trivial to read the files into Matlab. Below is a function which detects the data type based on the file extension and reads the file into Matlab.


function data = read_gadgetron_array(filename)
%  data = read_gadgetron_array(filename)
%  
%  Reads simplified array format output from the Gadgetron
%
%  The datatype is determined by the file extension.
%     - *.short : 16-bit unsigned integer
%     - *.real  : 32-bit float
%     - *.cplx  : 32-bit complex (two 32-bit values per data element)
%
%
if (~exist(filename,'file')),
    error('File not found.');
end

[path,name,ext] = fileparts(filename);

ext = lower(ext);

if (~strcmp(ext,'.short') && ~strcmp(ext,'.real') && ~strcmp(ext,'.cplx')),
   error('Unknown file extension'); 
end

f = fopen(filename);
ndims = fread(f,1,'int32'); 
dims = fread(f,ndims,'int32'); 

switch ext
    case '.short'
        data = fread(f,prod(dims),'uint16'); 
    case '.real'
        data = fread(f,prod(dims),'float32'); 
    case '.cplx'
        data = fread(f,2*prod(dims),'float32'); 
        data = complex(data(1:2:end),data(2:2:end));
    otherwise     
end

fclose(f);

data = reshape(data,dims');

end

  
Appendix B. HDF5 Files

The Gadgetron framework is used to process many different types of data and it is cumbersome to add specific read and write routines for all these different kinds of data. Consequently we have chosen to use the generic HDF5 file format. A detailed description of this format can be found at http://www.hdfgroup.org/HDF5/.

The HDF5 file format is much like a file system. Data can be organized hierarchically into groups (like folders in a filesystem) and each file can contain multiple groups and datasets. Each dataset can be an array of any type, e.g. an array of images. There is a generic tool hdfview which can be used to view the files. It is available on all the platforms supported by the Gadgetron framework. HDF5 files can also be read easily in newer versions of Matlab.

As an example of a HDF5 file with MRI raw data can be found at http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/testdata/. Download the file gadgetron_testdata.h5. When opened with hdfview, it should look like Figure B.1, “Examining Data with HDFView”. As seen, the file contains 4 groups of data. Each group consists of some data and an XML configuration for the Gadgetron.

Figure B.1. Examining Data with HDFView

Examining Data with HDFView

HDF5 Files can also be used to store images. Several of the Gadgetron clients included with the framework save images in HDF5 files. An example of viewing the output of a reconstruction can be seen in Figure B.2, “Viewing Images in HDF5 Files”.

Figure B.2. Viewing Images in HDF5 Files

Viewing Images in HDF5 Files

Images saved by Gadgetron clients are saved as arrays in the HDF5 files. Due to the array storage conventions in the Gadgetron environment, the first dimension is the slowest varying dimension in the arrays and the last dimension is the fastest varying dimension. That means that an array with 10 images with dimensions 128x128 would be stored in a variable in the HDF5 file with dimensions 10x1x128x128 as seen in Figure B.2, “Viewing Images in HDF5 Files”. To display the images, right click on the data and choose settings as illustrated in Figure B.3, “Setting for viewing HDF5 output images.”.

Figure B.3. Setting for viewing HDF5 output images.

Setting for viewing HDF5 output images.

The HDF5 files can also be read with Matlab. The images in the file above could be read with:

>> images = h5read('out.h5','/2012-05-11 10:57:48/data_0');
>> size(images)

ans =

   128   128     1    10

>> imagesc(images(:,:,1,1));colormap(gray) 
Appendix C. Future Features

The Gadgetron is evolving continuously and there are many things still that we would like to include but have not yet had the time to do. This appendix serves as a to-do list of features to be implement as we go along.

Branching Gadget chains. There is currently no ability to branch and collect in the Gadgetron.

Persistent memory storage across Gadget chains.

Matlab Gadgets. It would be great to have a way to encapsulate Matlab code in a Gadget similar to the way that the Python Gadgets work.

Bibliography

[HANSEN12] M. S. Hansen and T. S. Sørensen. Gadgetron: An Open Source Framework for Medical Image Reconstruction. Magnetic Resonance in Medicine. Submitted. 2012.

[HANSEN08] M. S. Hansen, D. Atkinson, and T. S. Sørensen. Cartesian SENSE and k-t SENSE reconstruction using commodity graphics hardware. Magnetic Resonance in Medicine. 59. 3. 463-468. 2008.

[SANGILD08] T. S. Sørensen, T. Schaeffter, K. O. Noe, and M. S. Hansen. Accelerating the nonequispaced fast fourier transform on commodity graphics hardware. IEEE Trans Med Imaging. 27. 4. 538-47. 2008.

[SANGILD09] T. S. Sørensen, D. Atkinson, T. Schaeffter, and M. S. Hansen. Real-time reconstruction of sensitivity encoded radial magnetic resonance imaging using a graphics processing unit. IEEE Trans Med Imaging. 28. 12. 1974-85. 2009.