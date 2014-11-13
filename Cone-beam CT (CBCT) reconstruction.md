Besides MRI reconstruction, Gadgetron also supports reconstruction of 3d and 4d cone-beam CT reconstruction. This is currently done through stand alone applications, though it will eventually be implemented as a gadget chain, similarly to what is done for MRI.
The code is currently in an alpha stage, meaning that it can and will change. Some things, such as the data format, are not expected to change however.

Testdata can be downloaded from <https://sourceforge.net/projects/gadgetron/files/testdata/standalone/cbct/>. The folder includes the dataset (projections.hdf5) and a binning file (binning_1ph.hdf5).
Binning files contain information about which projections belong to which time-phase for 4D CT, but can also be used to exclude projections with heavy artifacts. 


To reconstruct the test data, run

    CBCT_reconstruct_FDK_3d -d projections.hdf5 -b binning_1ph.hdf5 -r fdk.real -m 512 512 160 -f 210 210 210

This will do a standard 3D FDK reconstruction of a 210x210x210 mm cube, with a resolution of 512x512x160 pixels.

The CBCT code currently outputs all images in the simple .real format, which is a simple binary blob, with a 16 byte header for 3D data and a 20 byte header for 4D, followed by the raw image bytes as 32-bit floats. This can easily be opened in programs like imageJ.

Gadgetron currently only supports a "hard" filter for the FDK reconstruction, which is why the reconstructed image appears noisy. A less noisy image can be obtained by downsampling the projections, which is done by running

    CBCT_reconstruct_FDK_3d -d projections.hdf5 -b binning_1ph.hdf5 -r fdk.real -m 512 512 160 -f 210 210 210 -D 1

The main focus of the CBCT code in Gadgetron is however for iterative reconstruction. A simple iterative example can be run with 

    CBCT_reconstruct_CG -a projections.hdf5 -f iterative.real -s [512,512,160] -d [210,210,210] -b binning_1ph.hdf5 -D 0 -i 20

