The Gadgetron includes a real-time implementation of a GPU-based real-time non-Cartesian Sense reconstruction published in [IEEE Trans Med Imaging. 2009 Dec;28(12):1974-85](http://www.ncbi.nlm.nih.gov/pubmed/19628452). One of the keys to obtaining real-time performance is an efficient GPU implementation of the non-Cartesian Fast Fourier Transform [IEEE Trans Med Imaging. 2008 Apr;27(4):538-47](http://www.ncbi.nlm.nih.gov/pubmed/18390350). The application reuses several of the gadgets we have seen in use already for the Cartesian Grappa implementation above ([Cartesian 2D Parallel MRI (GRAPPA)]). An overview of the non-Cartesian Sense gadget chain is given in the figure below. 

<img src="https://s3.amazonaws.com/gadgetron.github.io/figs/cgsense.png" style="width: 400px;" />

This figure originates from the [Gadgetron paper](http://www.ncbi.nlm.nih.gov/pubmed/22791598). As described in [Gadgetron Gadgets] this gadget has since been broken up to support both linear SENSE and kt-SENSE and non-linear compressed sensing SENSE. The CGSenseGadget (now the [gpuRadialSensePrepGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_radial_sense_prep_gadget.html) and [gpuCgSenseGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_cg_sense_gadget.html)) implements the linear non-Cartesian Sense reconstruction. It contains a conjugate gradient solver ([Linear Solvers](../Gadgetron%20Toolboxes/#sectionlinearsolvers)) set up with a nonCartesianSense image encoding matrix and an imageOperator for regularization. Internally it maintains a cyclic buffer of a few seconds of imaging data. It uses this buffer to maintain a fully sampled (i.e. unaliased but blurred) k-space image from which coil sensititivities and regularization images are dynamically estimated. The combination of parallel imaging and image regularization operators allows for alias-suppressed image reconstruction using significant undersampling hereby achieving real-time data acquisition rates per frame. The conjugate gradient solver is able to reconstruct faster than the acquisition time e.g. a 192x192 image from 32 coils using 10 solver iterations on newer graphics hardware.

To test this configuration use the 32 channel radial MRI test dataset (`golden_angle.h5`), which you can download from [https://gadgetrondata.blob.core.windows.net/gadgetrongithubio/files/testdata/ismrmrd/](https://gadgetrondata.blob.core.windows.net/gadgetrongithubio/files/testdata/ismrmrd/). We assume that you have added \$(GADGETRON\_HOME)/bin to your PATH environment variable. You need a CUDA enable GPU on your system and your Gadgetron distribution should be compiled with CUDA and CULA enabled. Please see ? for details for your specific platform.

To run the reconstruction; start up gadgetron (in its own terminal window) and use the gadgetron_ismrmrd_client to send the data from another terminal. First start gadgetron:

    user@host$ gadgetron
    Configuring services

If asked, allow the gadgetron application to allow incoming network
connection. Next start the gadgetron_ismrmrd_client:

    user@host:~/temp$ wget http://sourceforge.net/projects/gadgetron/files/testdata/ismrmrd/golden_angle.h5

    user@host:~/temp$ gadgetron_ismrmrd_client \
           -f golden_angle.h5 \
           -c golden_radial_mode2_realtime.xml

    Gadgetron MRI Data Sender
      -- host            :      localhost
      -- port            :      9002
      -- hdf5 file  in   :      golden_angle.h5
      -- hdf5 group in   :      gre_golden_angle
      -- conf            :      golden_radial_mode2_realtime.xml
      -- loop            :      1
      -- hdf5 file out   :      ./out.h5
      -- hdf5 group out  :      2012-05-11 15:47:22
    (32608|139797448419136) Connection from 127.0.0.1:9002
    32608, 81, GadgetronConnector, Close Message received
    (32608|139797376546560) Handling close...
    (32608|139797376546560) svc done...
    (32608|139797376546560) Handling close...

Your current folder now holds the reconstructed images in the `out.h5`
HDF5 file. They will look something like the one depicted in the figure below.

<img src="https://s3.amazonaws.com/gadgetron.github.io/figs/examplecgsenseresult.png" style="width: 400px;" />
