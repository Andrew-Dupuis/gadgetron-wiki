This section demonstrates how to run a standalone non-Cartesian parallel MRI reconstruction similar to the one that was previously shown using the streaming framework infrastructure in section?. More details can be found in [Non-Cartesian 2D Parallel MRI (SENSE)].

In addition to a regularized linear least squares solution to the reconstruction problem, we furthermore use the Split Bregman solver to obtain the solution with minimum total variation subject to the constraint of the encoding operator (compressed sensing).

Download a free-breathing cardiac MRI sample dataset from <http://sourceforge.net/projects/gadgetron/files/testdata/standalone/fb_data.zip>

Source code is found in the files `main_cg.cpp` and `main_sbc.cpp` in directory

$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio`.

Both command lines below produce a 2D image sequence, each image with a matrix size of 192<sup>2</sup> (`-m`). 32 projections are used for each frame (`-p`) for a frame rate of roughly 32 profiles/frame \* 2.5 ms/profile = 80 ms ms/frame or 12.5 frames/s. The reconstruction results are written out in both complex and magnitude data as `result.cplx` and `result.real` respectively.

If sufficient device memory is available on your GPU (i.e. you are in possession of a high-end card) all frames in the sequence can be reconstructed concurrently (as a 3D volume). On systems that do not hold enough device memory to reconstruct all frames in parallel, they can instead be reconstructed in several batches. The `-f` option to the command lines below indicate the number of frames that are reconstructed per batch. A negative value indicates "all". If the command below fails to complete due to lack of device memory, try running with argument `-f 8` (or an even smaller number) instead.

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

For the interested reader, an implementation of *kt*-Sense can be found in directory

$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio_kt`.

Additionally, the source code for the user interface demonstrated in the [movie](movie) accompanying ? can be found (if you configured `cmake` to inlude Qt support) in direcotry

$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/sense/noncartesian/radial/2d_golden_ratio_gui`.
