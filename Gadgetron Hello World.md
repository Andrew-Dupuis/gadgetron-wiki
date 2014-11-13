In the following example we demonstrate how to send raw data through the Gadgetron client/server streaming framework infrastructure and obtain images in return. The command line examples are from a Linux installation. You may have to adjust for your specific installation. 

Open up two terminal windows. We will use one for the Gadgetron server and one for the Gadgetron client. In the server window you should be able to simply type `gadgetron` and see the following output:

    ubuntu@ip-10-184-82-58:~/temp$ gadgetron
    Configuring services, Running on port 9002

This means that your Gadgtron server is ready to receive data. 

Next (in the client terminal) we generate some data and send it in. The ISMRMRD library has an application that allows you to generate a simple dataset for testing purposes. On the comand line type

    ismrmrd_generate_cartesian_shepp_logan -r 10

which generates a dataset with 8 coils and 10 repetitions. 
Send it to the Gadgetron server using the following command:

    mriclient -d testdata.h5 

You should see something similar to the following in the client window:

    ubuntu@ip-10-184-82-58:~$ mriclient -d testdata.h5 
    Gadgetron MRI Data Sender
      -- host            :      localhost
      -- port            :      9002
      -- hdf5 file  in   :      testdata.h5
      -- hdf5 group in   :      /dataset
      -- conf            :      default.xml
      -- loop            :      1
      -- hdf5 file out   :      ./out.h5
      -- hdf5 group out  :      2014-01-23 16:51:58
    (18267|139953229416256) Connection from 127.0.0.1:9002
    18267, 87, GadgetronConnector, Close Message received
    (18267|139953146058496) Handling close...
    (18267|139953146058496) GadgetronConnector svc done...
    (18267|139953146058496) Handling close...


In the server window, you should see something like this:

    (18265|140484607932224) Connection from 127.0.0.1:43385
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 219] Running configuration: /usr/local/gadgetron/config/default.xml
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 274] Found 1 readers
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 275] Found 3 writers
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 276] Found 5 gadgets
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 287] --Found reader declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 288]   Reader dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 289]   Reader class: GadgetIsmrmrdAcquisitionMessageReader
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 290]   Reader slot: 1008
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 317] --Found writer declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 318]   Reader dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 319]   Reader class: MRIImageWriterCPLX
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 320]   Reader slot: 1004
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 317] --Found writer declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 318]   Reader dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 319]   Reader class: MRIImageWriterFLOAT
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 320]   Reader slot: 1005
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 317] --Found writer declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 318]   Reader dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 319]   Reader class: MRIImageWriterUSHORT
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 320]   Reader slot: 1006
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 336] Processing 5 gadgets in reverse order
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 346] --Found gadget declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 347]   Gadget Name: ImageFinishFLOAT
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 348]   Gadget dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 349]   Gadget class: ImageFinishGadgetFLOAT
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 364]   Gadget parameters: 0
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 346] --Found gadget declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 347]   Gadget Name: Extract
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 348]   Gadget dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 349]   Gadget class: ExtractGadget
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 364]   Gadget parameters: 0
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 346] --Found gadget declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 347]   Gadget Name: CropCombine
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 348]   Gadget dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 349]   Gadget class: CropAndCombineGadget
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 364]   Gadget parameters: 0
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 346] --Found gadget declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 347]   Gadget Name: FFT
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 348]   Gadget dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 349]   Gadget class: FFTGadget
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 364]   Gadget parameters: 0
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 346] --Found gadget declaration
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 347]   Gadget Name: Acc
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 348]   Gadget dll: gadgetron_mricore
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 349]   Gadget class: AccumulatorGadget
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 364]   Gadget parameters: 0
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 380] Gadget Stream configured
    [file /home/ubuntu/mrprogs/gadgetron/gadgets/mri_core/AccumulatorGadget.cpp, line 38] Matrix size: 512, 256, 1
    [file /home/ubuntu/mrprogs/gadgetron/gadgets/mri_core/AccumulatorGadget.cpp, line 46] FOV: 600.000000, 300.000000, 6.000000
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 81] Received close signal from client. Closing stream...
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (Acc) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (Acc) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (Acc) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (Acc) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (Acc)
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (FFT) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (FFT) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (FFT) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (FFT) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (FFT)
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (CropCombine) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (CropCombine) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (CropCombine) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (CropCombine) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (CropCombine)
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (Extract) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (Extract) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (Extract) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (Extract) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (Extract)
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (ImageFinishFLOAT) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (ImageFinishFLOAT) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (ImageFinishFLOAT) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (ImageFinishFLOAT) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (ImageFinishFLOAT)
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/EndGadget.h, line 19] Close called in EndGadget with flags 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/EndGadget.h, line 30] Calling close in base class  with flags 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (EndGadget) Close Called with flags = 1
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 115] Gadget (EndGadget) waiting for thread to finish
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/EndGadget.h, line 19] Close called in EndGadget with flags 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/EndGadget.h, line 30] Calling close in base class  with flags 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 105] Gadget (EndGadget) Close Called with flags = 0
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 117] Gadget (EndGadget) thread finished
    [file /home/ubuntu/mrprogs/gadgetron/apps/gadgetron/Gadget.h, line 48] Shutting down Gadget (EndGadget)
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 83] Stream closed
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 84] Closing writer task
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 86] Writer task closed
    (18265|140484607932224) GadgetStreamController, unable to read message identifier
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 156] handle_close called
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 161] Shutting down stream and closing up shop...
    [file /home/ubuntu/mrprogs/gadgetron/toolboxes/gadgettools/GadgetStreamController.cpp, line 192] Stream is closed

The images are saved in the folder in which you started the mriclient. The client appends the result to an HDF5 file called out.h5 (if no other file name is specified). A group is created with the current time and data and the images are stored in that group. If you run multiple reconstructions one after another, the results will be added to the same file with a new group is created for each run. This makes it easy to compare results from different reconstructions. The images are stored in a single precision format as specified by the default.xml configuration file. You can read and display the data using [hdfview](http://www.hdfgroup.org/products/java/hdf-java-html/hdfview/) or in Matlab with:

    images = h5read('out.h5','/<INSERT CORRECT DATE HERE>/image_0.img');
    imagesc(images(:,:,1,1));colormap(gray);
