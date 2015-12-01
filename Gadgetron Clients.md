### Available Clients

The purpose of this section is to maintain a list over the available clients that are included in the Gadgetron distribution. The current available clients are:

-   `gadgetron_ismrmrd_client`:

    This is the standard client for sending MRI data to the Gadgetron using the ISMRM Raw Data format. In order to get usage information for the client, simply run the client with no arguments.

### Making a new Client

The Gadgetron distribution comes with a GadgetronConnector class, which can be used to create clients. An example `main.cpp` file for a client could look like:

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

    find_package(Ismrmrd REQUIRED)
    find_package(Gadgetron REQUIRED)
    find_package(Boost REQUIRED)
    find_package(ACE REQUIRED)

    set(CMAKE_INSTALL_PREFIX ${GADGETRON_HOME})

    INCLUDE_DIRECTORIES(${ACE_INCLUDE_DIR} 
         ${Boost_INCLUDE_DIR}
         ${GADGETRON_INCLUDE_DIR}
         ${ISMRMRD_SCHEMA_DIR}
         ${ISMRMRD_XSD_INCLUDE_DIR}
         ${ISMRMRD_INCLUDE_DIR})

    LINK_DIRECTORIES(${GADGETRON_LIB_DIR})

    add_executable(mygadgetronclient main.cpp)

    IF(WIN32)
            SET(HDF5_LIB_DIR ${HDF5_INCLUDE_DIR}/../lib)
            target_link_libraries(mygadgetronclient optimized ${HDF5_LIB_DIR}/hdf5dll.lib)
            target_link_libraries(mygadgetronclient optimized ${HDF5_LIB_DIR}/hdf5_cppdll.lib)
            target_link_libraries(mygadgetronclient optimized ${HDF5_LIB_DIR}/hdf5_hldll.lib)
    
            target_link_libraries(mygadgetronclient debug ${HDF5_LIB_DIR}/hdf5ddll.lib)
            target_link_libraries(mygadgetronclient debug ${HDF5_LIB_DIR}/hdf5_cppddll.lib)
            target_link_libraries(mygadgetronclient debug ${HDF5_LIB_DIR}/hdf5_hlddll.lib)
    
            target_link_libraries(mygadgetronclient gadgetron_mricore gadgettools optimized ${ACE_LIBRARIES} debug ${ACE_DEBUG_LIBRARY} ${ISMRMRD_LIBRARIES} ${Boost_LIBRARIES})
    ELSE (WIN32)
            target_link_libraries(mygadgetronclient gadgettools  ${HDF5_LIBRARIES} optimized ${ACE_LIBRARIES} debug ${ACE_DEBUG_LIBRARY} ${ISMRMRD_LIBRARIES} ${Boost_LIBRARIES})
    ENDIF(WIN32)

    install(TARGETS mygadgetronclient DESTINATION bin)

Run cmake and follow the normal `make` and `make install` instructions (see [Linux Installation] [Mac Installation] [Windows Installation]).