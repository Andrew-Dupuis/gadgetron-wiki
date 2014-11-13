The easiest way to get started making a new Gadget library is to follow an example. In this example we create a new Gadget library containing a single Gadget; ThresholdGadget. Its purpose is to set all values below a certain fraction of the max value to zero.

New Gadget libraries can either be created in the Gadgetron source tree, which allows easy access to all the other files in the Gadgetron, or they can be made as external libraries that link against an installed Gadgetron system. In this example we do the latter since this creates a new library that does not "taint" the Gadgetron source tree. It is trivial to move the library inside the Gadgetron source tree at some later point in time if desired. We assume that the Gadgetron is installed on the machine that you are working on. The command line entries, etc. correspond to a Linux console. If you are using Windows
you have to adjust a bit.

Start by creating a new folder for the library:

    user@mycomputer:~/temp$ mkdir examplelib
    user@mycomputer:~/temp$ cd examplelib
 
We start by creating the class ThresholdGadget. Create the following 3 files: `ThresholdGadget.h`, `ThresholdGadget.cpp`, `examplelib_export.h` (the last file is just to help us make sure that things work on Windows) with the following content.

Header file:

    //ThresholdGadget.h
    
    #ifndef THRESHOLDGADGET_H
    #define THRESHOLDGADGET_H
    
    #include "examplelib_export.h"
    #include "Gadget.h"
    #include "GadgetMRIHeaders.h"
    #include "hoNDArray.h"
    #include <complex>
    #include <ismrmrd.h>
    
    namespace Gadgetron
    {
    
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
    
    }
    #endif //THRESHOLDGADGET_H

Implementation file:

    //ThresholdGadget.cpp
    
    #include "ThresholdGadget.h"
    
    using namespace Gadgetron;
    
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
    

Exports file

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

Now that we have the files for the Gadget we need to set up the build environment. In the folder `gadgetron_examplelib` create a file called `CMakeLists.txt` with the following content:

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
    
    find_package(Ismrmrd REQUIRED)
    find_package(Gadgetron REQUIRED)
    find_package(Boost REQUIRED)
    find_package(ACE REQUIRED)
    
    set(CMAKE_INSTALL_PREFIX ${GADGETRON_HOME})
    
    INCLUDE_DIRECTORIES(${ACE_INCLUDE_DIR} 
         ${Boost_INCLUDE_DIR}
         ${GADGETRON_INCLUDE_DIR}
         ${ISMRMRD_INCLUDE_DIR}
         ${ISMRMRD_SCHEMA_DIR}
         ${ISMRMRD_XSD_INCLUDE_DIR}
         )
    
    LINK_DIRECTORIES(${GADGETRON_LIB_DIR})
    
    ADD_LIBRARY(gadgetronexamplelib SHARED ThresholdGadget.cpp)
    
    TARGET_LINK_LIBRARIES(gadgetronexamplelib 
                          cpucore 
                          optimized ${ACE_LIBRARIES} 
                          debug ${ACE_DEBUG_LIBRARY})
    
    INSTALL (FILES ThresholdGadget.h
             examplelib_export.h
             DESTINATION include)
    
    INSTALL(TARGETS gadgetronexamplelib DESTINATION lib)
    
    INSTALL(FILES threshold.xml DESTINATION config)

The last thing we need is the XML configuration file to use when running our new ThresholdGadget. In the same folder create the `threshold.xml` file:

    <?xml version="1.0" encoding="UTF-8"?>
    <gadgetronStreamConfiguration xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
            xmlns="http://gadgetron.sf.net/gadgetron"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            
        <reader>
          <slot>1008</slot>
          <dll>gadgetron_mricore</dll>
          <classname>GadgetIsmrmrdAcquisitionMessageReader</classname>
        </reader>
      
        <writer>
          <slot>1004</slot>
          <dll>gadgetron_mricore</dll>
          <classname>MRIImageWriterCPLX</classname>
        </writer>
        <writer>
          <slot>1005</slot>
          <dll>gadgetron_mricore</dll>
          <classname>MRIImageWriterFLOAT</classname>
        </writer>
        <writer>
          <slot>1006</slot>
          <dll>gadgetron_mricore</dll>
          <classname>MRIImageWriterUSHORT</classname>
        </writer>
      
        <gadget>
          <name>Acc</name>
          <dll>gadgetron_mricore</dll>
          <classname>AccumulatorGadget</classname>
        </gadget>
        <gadget>
          <name>FFT</name>
          <dll>gadgetron_mricore</dll>
          <classname>FFTGadget</classname>
        </gadget>
        <gadget>
          <name>CropCombine</name>
          <dll>gadgetron_mricore</dll>
          <classname>CropAndCombineGadget</classname>
        </gadget>
    
        <!-- This is where we insert our new Gadget -->
        <gadget>
          <name>Threshold</name>
          <dll>gadgetronexamplelib</dll>
          <classname>ThresholdGadget</classname>
          <property><name>level</name><value>0.50</value></property>
        </gadget>
    
        <gadget>
          <name>Extract</name>
          <dll>gadgetron_mricore</dll>
          <classname>ExtractGadget</classname>
        </gadget>  
        <gadget>
          <name>ImageFinishFLOAT</name>
          <dll>gadgetron_mricore</dll>
          <classname>ImageFinishGadgetFLOAT</classname>
        </gadget>
    
    </gadgetronStreamConfiguration>

Check that you have 5 files in your folder:

     
    CMakeLists.txt
    ThresholdGadget.cpp
    ThresholdGadget.h
    examplelib_export.h
    threshold.xml

Next, let us create a `build` directory and compile:

     user@mycomputer:gadgetron_examplelib$ mkdir build; cd build


In the `build` folder

     user@mycomputer:build$ cmake ../

Assuming the cmake process was successful:

    user@mycomputer:~/temp/examplelib/build$ sudo make install
    [100%] Built target gadgetronexamplelib
    Install the project...
    -- Install configuration: ""
    -- Up-to-date: /usr/local/gadgetron/include/ThresholdGadget.h
    -- Up-to-date: /usr/local/gadgetron/include/examplelib_export.h
    -- Installing: /usr/local/gadgetron/lib/libgadgetronexamplelib.so
    -- Removed runtime path from "/usr/local/gadgetron/lib/libgadgetronexamplelib.so"
    -- Up-to-date: /usr/local/gadgetron/config/threshold.xml

  
You may have to use sudo for the `make install` command depending on your setup.

You should now be able to run a reconstruction using your new reconstruction chain. Follow the instructions in ? if you have not yet tried to run a simple reconstruction. After having started up the Gadgetron, run the mriclient:

    user@mycomputer:~/temp:~/temp$ ismrmrd_generate_cartesian_shepp_logan -r 10
    Generating Cartesian Shepp Logan Phantom
    Accelleration: 1

    user@mycomputer:~/temp$ mriclient -d testdata.h5 -c threshold.xml 
    Gadgetron MRI Data Sender
      -- host            :      localhost
      -- port            :      9002
      -- hdf5 file  in   :      testdata.h5
      -- hdf5 group in   :      /dataset
      -- conf            :      threshold.xml
      -- loop            :      1
      -- hdf5 file out   :      ./out.h5
      -- hdf5 group out  :      2014-01-23 20:10:19
    (21140|140214396131136) Connection from 127.0.0.1:9002
    21140, 87, GadgetronConnector, Close Message received
    (21140|140214312244992) Handling close...
    (21140|140214312244992) GadgetronConnector svc done...
    (21140|140214312244992) Handling close...

If you run it again with the `level` parameter set to 0.00000001 (remember to re-install the `threshold.xml` file in `gadgetron/config` by running `make
        install`):

        <gadget>
          <name>Threshold</name>
          <dll>gadgetronexamplelib</dll>
          <class>ThresholdGadget</class>
          <property><name>level</name><value>0.00000001</value></property>
        </gadget>

You should get two different results that look something like the figure below.

<img src="http://gadgetron.sf.net/figs/examplelibout.png" style="width: 600px;"/>

If you create interesting Gadget libraries please consider publishing them online to the benefit of the reconstruction community. An easy way to do this is by sending them to the Gadgetron team for us to publish right away on the web and possibly include in a future release of the Gadgetron.