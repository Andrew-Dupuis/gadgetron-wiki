The Gadgetron contains a high-throughput real-time 2D Cartesian parallel imaging reconstruction (GRAPPA) implemented on the GPU. It is beyond the scope of this manual to review all the algorithmic details of this application, but we will give an overview here as an example of a more complicated reconstruction chain.

The Gadget chain is defined in the `grappa.xml` and the resulting chain is illustrated in the figure below.

<img src="https://s3.amazonaws.com/gadgetron.github.io/figs/grappa.png" style="width: 400px;" />

To test this configuration, we will generate some accelerated 32 channel data using a simulation application from the ISMRMRD library and then send it through the Gadgetron. Make sure you have the Gadgetron running on your machine:

```
$ ismrmrd_generate_cartesian_shepp_logan -a 2 -c 32 -r 4
Generating Cartesian Shepp Logan Phantom!!!
Acceleration: 2
$ gadgetron_ismrmrd_client -f testdata.h5 -c grappa_cpu.xml
Gadgetron ISMRMRD client
  -- host            :      localhost
  -- port            :      9002
  -- hdf5 file  in   :      testdata.h5
  -- hdf5 group in   :      /dataset
  -- conf            :      grappa_cpu.xml
  -- loop            :      1
  -- hdf5 file out   :      out.h5
  -- hdf5 group out  :      2015-12-01 09:35:32
```                
    
You should get example images that look similar to the ones in the figure below.


<img src="https://s3.amazonaws.com/gadgetron.github.io/figs/grappaout.png" style="width: 600px;" />

Let's take a closer look at some of the components of this reconstruction application.

The first Gadget is the NoiseAdjustGadget. As described in [Gadgetron Gadgets], the purpose of this Gadget is to decorrelate the noise in the receiver channels. This improves the parallel imaging performance, especially in cases where there is a large amount of noise in just a few receiver elements. There are two versions of this Gadget, one that uses the BLAS/LAPACK routines for performance improvements and one that implements the same functionality without these optimizations. When you call the included `grappa.xml` configuration, you will use the optimized version. If you do not have BLAS and LAPACK on your system, you can modify the XML configuration to use the one from the `gadgets/core` library.

Second step is removing the oversampling. This step could also be performed after the reconstruction (as it is done in [Basic 2D FFT MRI]), but here we opt to remove this excess data to improve downstream performance.

The purpose of the next two Gadgets (PCAGadget and CoilReductionGadget) is to a) transform the receiver coils into PCA virtual coils ordered by their information content and b) remove some of the coils to improve downstream performance. The first step is achieved by buffering the first frame of data and then performing a principal component analysis (PCA) on the first frame of data. Based on the determined PCA transformation all data is then subsequently transformed into virtual coils. In the coil reduction gadget we can now simple eliminate the channels that are above a certain number. See [Gadgetron Gadgets] for details on how to control the channel compression.

The next two Gadgets are responsible for the actual GRAPPA reconstruction. The GrappaGadget calculates the GRAPPA coefficients and GrappaUnmixingGadget performs the Fourier transform of the raw data and applies the GRAPPA coefficients to the aliased imaged to obtain unaliased images.

In general it is assumed that the data is acquired in such a way that a set of neighboring frames can be averaged to yield a fully sampled k-space; the data is acquired with a time-interleaved sampling pattern. When enough calibration data is available to calculate GRAPPA coefficients, i.e. when a fully sampled region of k-space is available, the calibration data is sent to a grappa coefficient calculation object (GrappaWeightsCalculator).

The GrappaWeightsCalculator is an active object, which picks up weight calculation jobs from an input queue and passes them on to the GPU where it uses toolbox functions to calculate GRAPPA unmixing coefficients. These coefficients are Fourier transformed to image space where they are combined for all coils and stored in a GrappaWeights object.

When the GrappaGadget passes on the raw data to the GrappaUnmixingGadget it passes a reference to the GrappaWeights object which is to be used when performing the unmixing operation. Let's have closer look at the `GrappaUnmixingGadget.h` file:

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

After the unmixing, the images are scaled and magnitude is extracted before returning images to the client. The AutoScaleGadget has been added in this case to ensure that images are in a reasonable range
before converting to unsigned short as the output in this case. Automatic image scaling can be problematic, especially when doing quantitative imaging, but it was added in this case to make the
reconstruction more robust to data from different sources. A better solution is to only use data where noise calibration data is available and reconstruct SNR scaled images. Based on typical SNR values for MRI images, it is fairly trivial to keep the images in the appropriate range and perform a proper conversion to unsigned short.

A final comment about the GRAPPA reconstruction is that it allows a second step of channel compression. More specifically, it is possible to reconstruct to a limited number of target channels to further improve performance. Between the upstream and downstream channel compression steps, it is possible to tune the performance of the reconstruction to enable real-time reconstruction on the available hardware.
