Prototype Matlab MRI Gadgets
============================

The current implementation works on Linux and Mac.  It should be straight to get a working Windows implementation.

The code lives in gadgets/matlab.  There is a simple configuration (matlab.xml) which sets up a recon with three gadgets.  These are examples if the three possible types of MRI gadgets:

1.  scale.m: acquisition in, acquisition out
*  accumulate_and_recon.m: acquisition in, image out
*  mask.m: image in, image out

Users should be able to get pretty far using these gadgets as examples.

Installation
------------
Setting the MATLAB_HOME environment variable is set at the time cmake is called, the appropriate version of matlab gets used for buy


N.B Quick additional notes:

Java command server has now been removed.
csh must be installed - is needed by Matlab for engine.

Architecture Details
====================

Matlab Gadget
-------------
The MatlabGadget.h, MatlabGadget.cpp is the glue between the Gadgetron and Matlab.  You can have as many of these gadgets as you like.  Each one has a script associated with it.  To prevent collisions in the matlab interpreter, each gadget starts up its own matlab engine. Once the engine is started, the gadget sets up the matlab paths and starts the Java command server.

Matlab Gadget Scripts
---------------------
Matlab scripts should be derived from the BaseGadget.m class.  Each class has the following methods:
- init
* config
* run_process
* process
* emptyQ
* putQ

ISMRMRD Data Types and XML Header
---------------------------------
The matlab scripts use the ISMRMRD matlab API to handle the AcquisitionHeader and ImageHeader and the XML header.

As things stand now, the headers that go back and forth between the C++ code are passed as byte arrays (much faster than sending matlab structs), so they have to be packed and unpacked on the matlab side.  On the C++ side we just use memcopy.  On the matlab side, the header structs are fragile.  It's very easy to break the type of one of the elements of the struct.  It is not clear how to handle this without really clobbering performance.  Using a more complicated matlab object, with getters and setters and type checking, becomes extremely expensive.

Java Command Server
-------------------
There is a small Java class (MatlabCommandServer) that significantly improves the performance of issuing commands to the matlab engine as compared to using engEvalString.  We need a few engEvalStrings to get things going, but after the command server starts up, all messages got from the C++ gadget to matlab using the send_matlab_command function over a TCP/IP socket.  
