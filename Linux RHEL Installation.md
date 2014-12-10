RHEL 6.3 Derivatives
====================

Installation instruction for Red Hat Enterprise Linux and its derivatives (CentOS, Scientific Linux, etc).  These notes have been tested on a clean install of CentOS 6.3 with the "Software Development Workstation" package option.

CUDA, CULA and GPU Driver
-------------------------
**The steps below must be executed as a super user.**

* Install the CUDA (4.2) toolkit, driver and SDK
    - wget http://developer.download.nvidia.com/compute/cuda/4_2/rel/toolkit/cudatoolkit_4.2.9_linux_64_rhel6.0.run
    - sh ./cudatoolkit_4.2.9_linux_64_rhel6.0.run
* Install CULA
    - download the .run file from http://www.culatools.com/downloads/dense/
    - sh ./cula_dense_free_R15-linux.run
* GPU Driver 
  (Optional: The GPU Driver is not needed for building)
    - Download the nvidia driver corresponding to your card.  
      This will give you a file named something like this:  NVIDIA-Linux-x86_64-304.37.run
    - Stop the XServer, install, start the XServer, check the GPU
        * init 3
        * ./NVIDIA-Linux-x86_64-304.37.run
        * init 5
        * lspci | grep NVIDIA

Enabling some none-default repositories
---------------------------------------
There are a few dependencies not in the default repositories, so we add the EPEL and ACE repositories.

**The steps below must be executed as a super user.**

* Add EPEL to the list of know repositories
    - Download the EPEL repo RPM from here: http://download.fedoraproject.org/pub/epel/6/i386/repoview/epel-release.html
    - Install it with yum: `yum install epel-release-6-8.noarch.rpm`
* Add the ACE development repository from http://download.opensuse.org/repositories/devel:/libraries:/ACE:/minor/, e.g.
    - `cd /etc/yum.repos.d/`
    - `wget http://download.opensuse.org/repositories/devel:/libraries:/ACE:/minor/CentOS_CentOS-6/devel:libraries:ACE:minor.repo`

Gadgetron dependencies
----------------------
**The steps below must be executed as a super user.**

* yum install qt-devel fftw-devel freeglut-devel hdf5-devel glew-devel lapack-devel xerces-c-devel xsd
* yum install docbook-utils-pdf docbook5-schemas docbook5-style-xsl
* yum install cmake28
* yum install ace-devel

  Note: Depending on your installation, you may need to install a few additional packages.  Here is a list: boost-devel, doxygen, git, libxml2-devel, libxslt-devel.


Python Options
--------------
* CentOS6.3 ships with python 2.6.6 and some somewhat outdated python packages.  They work just fine, but if you want the latest and greatest, you'll want to install python 2.7.3 and set up a virtualenv and use pip to install the necessary bits.  Alternatively you could stay on python 2.6.6 and just install/upgrade the packages you want using pip, e.g.:
    - yum install python-pip 
    - pip-python install --upgrade numpy scipy matplotlib ipython

ISMRMRD and Gadgetron
---------------------
**The steps may be executed as a regular user.**

* Create install directory
    - mkdir ~/local
* ISMRMRD
    - git clone https://github.com/ismrmrd/ismrmrd.git
    - cd ismrmrd
    - mkdir build; cd build
    - cmake28 -DCMAKE_INSTALL_PREFIX=~/local -DCMAKE_PREFIX_PATH=~/local/ ../
    - make;make install
* Gadgetron
    - git clone https://github.com/gadgetron/gadgetron.git
    - git checkout development
    - cd gadgetron
    - mkdir build; cd build
    - cmake28 -DCMAKE_INSTALL_PREFIX=~/local -DCMAKE_PREFIX_PATH=~/local/ ../
    - make; make install