A basic example application in the Gadgetron is a simple 2D FT MRI reconstruction. It receives 2D MRI data, collects it into k-space arrays, performs FFT of the data, combines channels (if there are multiple), and returns the images to the client. This example is included in the Gadgetron for testing and demonstration purposes only. It was not intended to be fast or otherwise optimal in any sense.

The Gadgets for this reconstruction are in the `core` folder and the configuration file to use to run this reconstruction is `default.xml`. [Gadgetron Hello World] describes how to run a simple reconstruction using this Gadget chain and how to download data to test it.

In this section we will take a closer look at the Gadgets in this chain and how they are implemented. The Gadgetron XML configuration file (`default.xml`) looks like this:

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

The resulting Gadget chain is illustrated in the figure below. As described in [Gadgetron Streaming Architecture] the Gadgetron configuration contains 3 sections: Readers, Writers, and the Stream. In this particular case, there is only one Reader, which received MRI Acquisitions. This data format is described in [Gadgetron Gadgets]. There are 3 Writers registered with this configuration. They are all used to write MRI images, but responsible for the different data types (complex float, float, or unsigned short). In principle this means that this reconstruction is capable of returning 3 different types of images, but as is seen from the stream configuration, the only output from this reconstruction will be float format images. However, many reconstructions will have all 3 Writers registered to make it easy to switch formats, i.e. it would be trivial to turn this reconstruction into one that outputs unsigned short images (have a look at the file `default_short.xml`) for an example of how this is done.

<img src="http://gadgetron.sf.net/figs/simple2dft.png" style="width: 400px;" />

As is seen in the Gadgets section of the configuration, this reconstruction uses 5 Gadgets. The first Gadget is responsible for accumulating MRI acquisitions. To accomplish this, it uses an accumulation buffer. When a k-space line arrives at the Gadget, it will be inserted into the k-space buffer and when the last acquisition in a slice/repetition has arrived, it will copy the entire buffer and pass it on to the next Gadget.

Let's have a look at the definition of the AccumulatorGadget class:

    class EXPORTGADGETSCORE AccumulatorGadget : 
    public Gadget2< ISMRMRD::AcquisitionHeader, 
                    hoNDArray< std::complex<float> > >
    {
      
     public:
      GADGET_DECLARE(AccumulatorGadget);

      AccumulatorGadget();
      ~AccumulatorGadget();

     protected:
      virtual int process_config(ACE_Message_Block* mb);
      virtual int process(
        GadgetContainerMessage< ISMRMRD::AcquisitionHeader >* m1,
        GadgetContainerMessage< hoNDArray< std::complex<float> > > * m2);

      hoNDArray< std::complex<float> >* buffer_;
      std::vector<unsigned int> dimensions_;

      int image_counter_;
      int image_series_;

    };

There are a few member variables to help us keep track of the buffer and the data dimensions and the core functionality is implemented in two functions: `process_config`, is used to set up the buffer, and
`process`, which is responsible for the accumulation of data. Let us examine the `process_config` function (abbreviated):

    int AccumulatorGadget::process_config(ACE_Message_Block* mb)
    {
     boost::shared_ptr<ISMRMRD::ismrmrdHeader> cfg = 
        parseIsmrmrdXMLHeader(std::string(mb->rd_ptr()));

     ISMRMRD::ismrmrdHeader::encoding_sequence e_seq = cfg->encoding();

     ISMRMRD::encodingSpaceType e_space = (*e_seq.begin()).encodedSpace();
     ISMRMRD::encodingSpaceType r_space = (*e_seq.begin()).reconSpace();
     ISMRMRD::encodingLimitsType e_limits = (*e_seq.begin()).encodingLimits();

     GADGET_DEBUG2("Matrix size: %d, %d, %d\n", 
                    e_space.matrixSize().x(), 
                    e_space.matrixSize().y(), 
                    e_space.matrixSize().z());

     dimensions_.push_back(e_space.matrixSize().x());
     dimensions_.push_back(e_space.matrixSize().y());
     dimensions_.push_back(e_space.matrixSize().z());

     slices_ = e_limits.slice().present() ? 
                 e_limits.slice().get().maximum()+1 : 1;

      return GADGET_OK;
    }

The main purpose of this function is to pull parameters out of the XML portion of the ISMRM Raw Data header in order to set up the buffer. As mentioned in [Gadget XML Configuration](../Gadgetron%20Streaming%20Architecture/#sectiongadgetxmlconfiguration), the convention is to pass parameters into the Gadgets in XML format. To enable convenient parsing of these parameters, the
ISMRMRD library includes a C++ class representation of the header. See <http://ismrmrd.github.io> for more details.

Now we are ready to receive and buffer data, which is done by the `process` function:

    int AccumulatorGadget::
    process(GadgetContainerMessage<ISMRMRD::AcquisitionHeader>* m1,
     GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
    {

      if (!buffer_) {
       dimensions_.push_back(m1->getObjectPtr()->active_channels);
       dimensions_.push_back(slices_);

       if (!(buffer_ = new hoNDArray< std::complex<float> >())) {
        GADGET_DEBUG1("Failed create buffer\n");
        return GADGET_FAIL;
       }

       if (!buffer_->create(&dimensions_)) {
        GADGET_DEBUG1("Failed allocate buffer array\n");
        return GADGET_FAIL;
       }

       image_series_ = this->get_int_value("image_series");

      }


      std::complex<float>* b =
        buffer_->get_data_ptr();

      std::complex<float>* d =
        m2->getObjectPtr()->get_data_ptr();

      int samples =  m1->getObjectPtr()->number_of_samples;
      int line = m1->getObjectPtr()->idx.kspace_encode_step_1;
      int partition = m1->getObjectPtr()->idx.kspace_encode_step_2;
      int slice = m1->getObjectPtr()->idx.slice;

      if (samples > static_cast<int>(dimensions_[0])) {
       GADGET_DEBUG1("Wrong number of samples received\n");
       return GADGET_FAIL;
      }

      size_t offset= 0;
      //Copy the data for all the channels
      for (int c = 0; c < m1->getObjectPtr()->active_channels; c++) {
        offset = 
          slice*dimensions_[0]*dimensions_[1]*dimensions_[2]*dimensions_[3] +
          c*dimensions_[0]*dimensions_[1]*dimensions_[2] +
          partition*dimensions_[0]*dimensions_[1] +
          line*dimensions_[0] + (dimensions_[0]>>1)-m1->getObjectPtr()->center_sample;
        
        memcpy(b+offset,
         d+c*samples,
         sizeof(std::complex<float>)*samples);
      }
      
      bool is_last_scan_in_slice = 
         ISMRMRD::FlagBit(ISMRMRD::ACQ_LAST_IN_SLICE).isSet(m1->getObjectPtr()->flags);
      
      if (is_last_scan_in_slice) {
        GadgetContainerMessage<ISMRMRD::ImageHeader>* cm1 = 
          new GadgetContainerMessage<ISMRMRD::ImageHeader>();
        
        cm1->getObjectPtr()->flags = 0;

        GadgetContainerMessage< hoNDArray< std::complex<float> > >* cm2 = 
          new GadgetContainerMessage<hoNDArray< std::complex<float> > >();
        
        cm1->cont(cm2);
        
        std::vector<unsigned int> img_dims(4);
        img_dims[0] = dimensions_[0];
        img_dims[1] = dimensions_[1];
        img_dims[2] = dimensions_[2];
        img_dims[3] = dimensions_[3];
        
        if (!cm2->getObjectPtr()->create(&img_dims)) {
          GADGET_DEBUG1("Unable to allocate new image array\n");
          cm1->release();
          return -1;
        }
        
        size_t data_length = dimensions_[0]*dimensions_[1]*
          dimensions_[2]*dimensions_[3];
        
        offset = slice*data_length;
        
        memcpy(cm2->getObjectPtr()->get_data_ptr(),b+offset,
        sizeof(std::complex<float>)*data_length);
        
        cm1->getObjectPtr()->matrix_size[0]     = img_dims[0];
        cm1->getObjectPtr()->matrix_size[1]     = img_dims[1];
        cm1->getObjectPtr()->matrix_size[2]     = img_dims[2];
        cm1->getObjectPtr()->channels           = img_dims[3];
        cm1->getObjectPtr()->slice   = m1->getObjectPtr()->idx.slice;

        memcpy(cm1->getObjectPtr()->position,
          m1->getObjectPtr()->position,
        sizeof(float)*3);

        memcpy(cm1->getObjectPtr()->read_dir,
          m1->getObjectPtr()->read_dir,
        sizeof(float)*3);

        memcpy(cm1->getObjectPtr()->phase_dir,
          m1->getObjectPtr()->phase_dir,
        sizeof(float)*3);

        memcpy(cm1->getObjectPtr()->slice_dir,
          m1->getObjectPtr()->slice_dir,
        sizeof(float)*3);
     
        memcpy(cm1->getObjectPtr()->patient_table_position,
          m1->getObjectPtr()->patient_table_position, sizeof(float)*3);

        cm1->getObjectPtr()->image_data_type = ISMRMRD::DATA_COMPLEX_FLOAT;
        cm1->getObjectPtr()->image_index = ++image_counter_;
        cm1->getObjectPtr()->image_series_index = image_series_;

        if (this->next()->putq(cm1) < 0) {
         return GADGET_FAIL;
        }
      } 

      m1->release();
      return GADGET_OK;
    }

This function has two basic tasks: insert data into the buffer and when enough data is present, copy the buffer and pass it on to next gadget. Additionally, the data buffer is created in this function if it is not already allocated. In this example we choose to allocate the buffer after the first data elements arrive. This allows us to respond to changes in data sizes introduced by upstream Gadgets, e.g. readout downsampling, coil reduction, etc.

In this case the copying of data is done with a very simple `memcpy` command. There is a basic check for the image dimensions, but a more robust application may have more checks of the incoming data.

Once the data is in the buffer, we check to see if we should put out an image. This is done with the `flags` field on the acquisition. Specifically we check if a specific bit (`ISMRMRD::ACQ_LAST_IN_SLICE`)
is set.

If it is determined that this is the last acquisition for this slice, we create a copy of the buffer and pass it on to the next Gadget. Instead of a ISMRMRD::AcquisitionHeader we now need an ISMRMRD::ImageHeader to pass along with the data. This header structure is created and populated with fields (orientation, etc.) from the acquisition header before it is passed on to the Gadget in the stream.

Next Gadget is the FFTGadget. Since the k-space buffering has been taken care of, the Fourier transform is a relatively simple task. The `process` function uses the FFTW wrapper class (see [Gadgetron Toolboxes]) to perform the FFT along the first 3 dimensions of the array:

    int FFTGadget::process( 
    GadgetContainerMessage< ISMRMRD::ImageHeader>* m1,
    GadgetContainerMessage< hoNDArray< std::complex<float> > >* m2)
    {
      FFT<float>::instance()->ifft(m2->getObjectPtr(),0);
      FFT<float>::instance()->ifft(m2->getObjectPtr(),1);
      FFT<float>::instance()->ifft(m2->getObjectPtr(),2);

      if (this->next()->putq(m1) < 0) {
         return GADGET_FAIL;
      }

      return GADGET_OK;
    }

Now that the images have been Fourier transformed, we need to remove the oversampling that is done in the readout dimensions and we need to combine the receiver channels. In this case, we are making some
assumptions, i.e. we assume two-fold oversampling in the readout and we are doing a simple RMS coil combination to obtain combined magnitude images. We will not repeat the source code here, it can be found in `gadgets/core/CropAndCombineGadget.cpp`.

Last two remaining steps after the coil combination is to extract the magnitude of the data and return the floating point images to the Gadgetron so that they can be returned to the client. This is accomplished in the ExtractGadget and the ImageFinishGadgetFLOAT. Both of these Gadgets are described in [Gadgetron Gadgets].