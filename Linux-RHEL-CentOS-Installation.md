RHEL 6 Derivatives
==================

Installation instruction for Red Hat Enterprise Linux and its derivatives (CentOS, Scientific Linux, etc).  These notes have been tested on CentOS 7 with the "Software Development Workstation" package option.

Enabling some none-default repositories
---------------------------------------
There are a few dependencies not in the default repositories, so we add the EPEL, ACE, and Boost repositories.

**The steps below must be executed as a super user.**

* Add EPEL to the list of know repositories:
  - `wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm`
  - `yum localinstall epel-release-6-8.noarch.rpm`
* Install ACE
  - `mkdir ~/software && cd ~/software`
  - `wget ftp://ftp.pbone.net/mirror/ftp5.gwdg.de/pub/opensuse/repositories/devel:/libraries:/ACE:/micro/CentOS_7/x86_64/ace-6.3.3-55.1.x86_64.rpm`
  - `yum install ./ace-6.3.3-55.1.x86_64.rpm`
* Add the Boost >=1.55 repo:
  - `wget http://repo.enetres.net/enetres.repo -O /etc/yum.repos.d/enetres.repo`
* Add SLC @ CERN repository:
  - `wget http://linuxsoft.cern.ch/cern/scl/slc6-scl.repo -O /etc/yum.repos.d/slc6-scl.repo`

Gadgetron dependencies
----------------------
**The steps below must be executed as a super user.**

* `yum group install "Development Tools"`
* `yum install qt-devel fftw-devel freeglut-devel hdf5-devel glew-devel lapack-devel xerces-c-devel xsd`
* `yum install docbook-utils-pdf docbook5-schemas docbook5-style-xsl`
* `yum install cmake cmake3 cmake-gui cmake3-gui`
* `yum install ace-devel`
* `yum install boost-devel doxygen git libxml2-devel libxslt-devel openblas-devel armadillo-devel gtest`
* `yum install scl-utils devtoolset-3-gcc-c++`

  Optional: Libraries below can be updated from third party repos:
  
  - `fftw-devel 3.2.2`: atrpms
  - `libglew-devel 1.7.0`: linuxtech-release

Intel MKL (Optional)
------------------------------------
```
cd ~/software
wget http://registrationcenter-download.intel.com/akdlm/irc_nas/tec/12414/l_mkl_2018.1.163.tgz
tar -xvf ./l_mkl_2018.1.163.tgz
cd l_mkl_2018.1.163
sudo ./install.sh`
```

Matlab (Optional)
------------------------------------
* Try Matlab at https://www.mathworks.com/programs/trials/trial_request.html
* Require JDK to compile Matlab module
  - `wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u45-b14/jdk-8u45-linux-x64.rpm`
  - `yum localinstall jdk-8u45-linux-x64.rpm`

CUDA and GPU Driver (Optional)
------------------------------------
**The steps below must be executed as a super user.**

* Install Nvidia CUDA (7.0) Toolkit and GPU driver
  - `wget http://developer.download.nvidia.com/compute/cuda/repos/rhel6/x86_64/cuda-repo-rhel6-7.0-28.x86_64.rpm`
  - `yum localinstall cuda-repo-rhel6-7.0-28.x86_64.rpm`
  - `yum install cuda`

DICOM Support (Optional)
------------------------------------
**The steps below must be executed as a super user.**

* Create `/etc/yum.repos.d/PUIAS_6_computational.repo` and add the following lines:

    `[PUIAS_6_computational]`
    `name=PUIAS computational Base $releasever - $basearch`
    `mirrorlist=http://puias.math.ias.edu/data/puias/computational/$releasever/$basearch/mirrorlist`
    `gpgcheck=1`
    `gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-puias`

* Download and install the gpg key.
  - `wget http://springdale.math.ias.edu/data/puias/6/x86_64/os/RPM-GPG-KEY-puias`
  - `rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-puias`
  - `yum install dcmtk360-devel`

Python Options
--------------
```
sudo yum install python python-devel python-pip
sudo pip install --upgrade pip
sudo pip install setuptools
sudo pip install Cython
sudo pip install numpy 
sudo pip install pyxb psutil lxml pydicom
sudo pip install 'h5py==2.5.0'
```
ISMRMRD and Gadgetron
---------------------
**The steps may be executed as a regular user.**

* Environment variables for finding Matlab and DCMTK
    - `export MATLAB_ROOT=/usr/local/MATLAB/R2013a`
    - `export DCMTK_HOME=/usr/local/dcmtk/3.6.0`
* Create install directory
    - `mkdir ~/local`
* Switch GCC to 4.9.1
    - `scl enable devtoolset-3 bash`
* ISMRMRD
    - `git clone https://github.com/ismrmrd/ismrmrd.git`
    - `cd ismrmrd`
    - `mkdir build; cd build`
    - `cmake -D CMAKE_INSTALL_PREFIX=~/local -D CMAKE_PREFIX_PATH=~/local/ ../`
    - `make;make install`
* Gadgetron
    - `git clone https://github.com/gadgetron/gadgetron.git`
    - `cd gadgetron`
    - `mkdir build; cd build`
    - `cmake -D CMAKE_INSTALL_PREFIX=~/local -D CMAKE_PREFIX_PATH=~/local/ ../`
    - `make; make install`

Environment Variables
---------------------
The final step is to add/modify a few environment variables in your `~/.bashrc` file:

    export GADGETRON_HOME=/usr/local/gadgetron
    export ISMRMRD_HOME=/usr/local
    export PATH=$PATH:$GADGETRON_HOME/bin:$ISMRMRD_HOME/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GADGETRON_HOME/lib:$ISMRMRD_HOME/lib

Rename the example configuration file `$GADGETRON_HOME/config/gadgetron.xml.example` to `$GADGETRON_HOME/config/gadgetron.xml`.
