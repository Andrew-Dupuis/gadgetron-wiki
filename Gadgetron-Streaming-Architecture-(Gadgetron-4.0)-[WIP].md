The Gadgetron consists of a streaming processing architecture and a set of toolboxes. The toolboxes are used within the streaming components but come as individual shared libraries and can thus also be used in standalone applications. The architecture is outlined in the figure below.

TODO: Illustrate a Gadgetron reconstruction chain.

The Gadgetron receives connections from clients through a TCP/IP connection. A client can be any application from which you can open a TCP/IP socket and send data. Once a connection to a client has been established (see [Communication Sequence](#communicationsequencelink)), Gadgetron will read data from the socket and pass it on down a chain of processing steps. The responsibility of reading and writing packages on the socket is dispatched to a set of Readers and Writers (see [Readers and Writers](#readerswriterslink)). Each step in the processing chain is implemented in a module (referred to as a Gadget - see [Gadgets](#gadgetslink)). A reconstruction process is defined by defining a chain of Gadgets. The assembly of Gadgets is done dynamically at run-time (see [Stream Configuration](#streamconfigurationlink)).

### <a name="gadgetslink"/>Gadgets

A Gadget is the functional unit of the Gadgetron. You can think of the Gadget as a device with an input and output. Data passes through the device and is modified and/or transformed between input and output. By wiring multiple Gadgets together you create a reconstruction program. A schematic outline of a Gadget is seen below.

TODO: Illustrate Gadget - Maybe even the new interface. 

A Gadget is a module processing input, producing output. A Gadget will typically run in it's own thread of execution, and access it's input (and produce output) through channels provided by the Gadgetron framework. Messages in the input channel will be provided by the next upstream Gadget, or a Reader (in case of the first Gadget). The typical behaviour of a Gadget boils down to 'while I have input, take input, process, and produce some output'. 

Receiving input in a Gadget is a matter of reading a message from an input channel. This is commonly done by calling the `pop()` method, or by iterating over the channel (The Channels provided by Gadgetron has a modern iterator interface, and can be used in most standard library functions and loops, etc.). 

The channels provided by the Gadgetron framework come in several flavours. Most commonly used are instances of `TypedInputChannel`. The provide access to a Gadget's input in a typed and convenient manner. Typed channels will also automatically pass on messages that do not match the type profile. 

Producing is a simple as receiving input. Simply `push(...)` your output to the output channel. Push accepts multiple arguments, and 'bundles' these together in a single message for the next Gadget. In this manner, an acquisition header and the acquisition data (for instance) pass from Gadget to Gadget in a single bundle. 

As an example, let's look at a very simple Gadget, which receives acquisitions (in the ISMRM Raw Data format) and bundles them together into larger packages (one bundle for each slice). First the header file [SliceAccumulator.h](https://github.com/gadgetron/gadgetron/blob/Gadgetron4.0/gadgets/grappa/SliceAccumulator.h):
```c++
#pragma once

#include <vector>

#include "Types.h"
#include "Node.h"

namespace Gadgetron::Grappa {

    using Slice = std::vector<Acquisition>;

    class SliceAccumulator : public Core::TypedChannelGadget<Acquisition> {
    public:
        SliceAccumulator(const Core::Context &, const GadgetProperties &);
        void process(Core::TypedInputChannel<Acquisition> &in, Core::OutputChannel &out) override;
    };
}
```
Gadgetron will initialize a Gadget by calling it's constructor. Gadgetron provides two arguments: A `Context` object, and a `GadgetProperties` object. The context contains information about the world around the Gadget. This includes (but is not limited to) the current working directory of Gadgetron, and the ISMRM Data Format XML header describing the data under reconstruction. The `GadgetProperties` is a simple map, containing any properties specified for the Gadget in the reconstruction XML configuration. See [Stream Configuration](#streamconfigurationlink) for more details. 

Note that we define a `Slice` as `std::vector<Acquisition>` - a vector of Acquisitions. And that the Gadget's process method (Called by Gadgetron when the Gadget is loaded) takes a `TypedInputChannel<Acquisition>` as it's first argument.

Implementing the slice accumulator looks like [SliceAccumulator.cpp](https://github.com/gadgetron/gadgetron/blob/Gadgetron4.0/gadgets/grappa/SliceAccumulator.cpp): 
```c++
#include "SliceAccumulator.h"

#include "Channel.h"
#include "Types.h"

namespace Gadgetron::Grappa {
    using namespace Gadgetron;
    using namespace Gadgetron::Core;

    SliceAccumulator::SliceAccumulator(const Context &, const GadgetProperties &props) : TypedChannelGadget(props) {}

    void SliceAccumulator::process(TypedInputChannel<Acquisition> &in, OutputChannel &out) {

        std::vector<Acquisition> acquisitions{};

        for (auto acquisition : in) {
            acquisitions.emplace_back(std::move(acquisition));

            if (acquisitions.back().isFlagSet(ISMRMRD::ISMRMRD_ACQ_LAST_IN_SLICE)) {
                out.push(std::move(acquisitions));
                acquisitions = std::vector<Acquisition>{};
            }
        }
    }
    GADGETRON_GADGET_EXPORT(SliceAccumulator);
}
```
It is worth noting that the input channel is consumed using for loop. This will process each incoming acquisition in turn, waiting for the next if none are available. Once the channel is closed, and the last acquisition processed, the loop will complete. 

The flow of the code itself is simple; we read acquisitions and save them to the current buffer. If the acquisition read ends a slice, we push the entire buffer to the out channel, and start over with a new buffer.

The last thing to notice is the `GADGETRON_GADGET_EXPORT(SliceAccumulator)` macro. This declares functions which will allow Gadgetron to load the SliceAccumulator Gadget on runtime. 

For a tutorial on how to make your own Gadget library see [Making a New Gadget Library](./Making-a-New-Gadget-Library).

#### <a name="sectiongadgetxmlconfiguration"/>Gadget XML Configuration

In addition to defining a Gadget's behavior in response to a data package, it is also possible for the Gadgets to receive configuration information and parameters. 

The user can define the Gadgets behavior in response to configuration information by implementing the `process_config` function in the Gadget header file. The configuration information or parameters is typically transmitted in the beginning of the reconstruction process from the client (see [Communication Sequence](#communicationsequencelink)). The configuration information can in principle be in any format (a given application can use a binary format or a text format defined for the specific purpose), but conventionally the parameters are transmitted in XML format and for the MRI Gadgets, the XML configuration is the XML header from the ISMRM Raw Data file. More details on this format and how to easily parse it with the included C++ XML data binding classes can be found at <http://ismrmrd.github.io>.

An example of a parameter XML file for an MRI data set is shown here:

    <?xml version="1.0"?>

    <ismrmrdHeader xmlns="http://www.ismrm.org/ISMRMRD" 
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
      xmlns:xs="http://www.w3.org/2001/XMLSchema" 
      xsi:schemaLocation="http://www.ismrm.org/ISMRMRD ismrmrd.xsd">

      <subjectInformation>
        <patientName>phantom</patientName>
        <patientWeight_kg>72.5748</patientWeight_kg>
      </subjectInformation>
      <acquisitionSystemInformation>
        <systemVendor>SIEMENS</systemVendor>
        <systemModel>Avanto</systemModel>
        <systemFieldStrength_T>1.494</systemFieldStrength_T>
        <receiverChannels>32</receiverChannels>
        <relativeReceiverNoiseBandwidth>0.79</relativeReceiverNoiseBandwidth>
      </acquisitionSystemInformation>
      <experimentalConditions>
        <H1resonanceFrequency_Hz>63620740</H1resonanceFrequency_Hz>
      </experimentalConditions>
      <encoding>
        <trajectory>cartesian</trajectory>
        <encodedSpace>
          <matrixSize>
            <x>256</x>
            <y>128</y>
            <z>1</z>
          </matrixSize>
          <fieldOfView_mm>
            <x>600</x>
            <y>300</y>
            <z>5</z>
          </fieldOfView_mm>
        </encodedSpace>
        <reconSpace>
          <matrixSize>
            <x>128</x>
            <y>128</y>
            <z>1</z>
          </matrixSize>
          <fieldOfView_mm>
            <x>300</x>
            <y>300</y>
            <z>5</z>
          </fieldOfView_mm>
        </reconSpace>
        <encodingLimits>
          <kspace_encoding_step_1>
            <minimum>0</minimum>
            <maximum>127</maximum>
            <center>64</center>
          </kspace_encoding_step_1>
          <kspace_encoding_step_2>
            <minimum>0</minimum>
            <maximum>0</maximum>
            <center>0</center>
          </kspace_encoding_step_2>
          <slice>
            <minimum>0</minimum>
            <maximum>0</maximum>
            <center>0</center>
          </slice>
          <set>
            <minimum>0</minimum>
            <maximum>0</maximum>
            <center>0</center>
          </set>
        </encodingLimits>
      </encoding>
      <sequenceTiming>
        <TR>5.86</TR>
        <TE>2.96</TE>
      </sequenceTiming>
    </ismrmrdHeader>

The user/developer can use any XML parsing technique to extract parameters from this XML header, but we encourage developers to use the C++ XML Data Binding classes that are included with the ISMRM Raw Data C++ library. For example, to parse encoding limits (example from `AccumulatorGadget.cpp`):

    int AccumulatorGadget::process_config(ACE_Message_Block* mb)
    {
     
     //Calling parsing convenience function found in GadgetIsmrmrdReadWrite.cpp
     boost::shared_ptr<ISMRMRD::ismrmrdHeader> cfg = parseIsmrmrdXMLHeader(std::string(mb->rd_ptr()));

     ISMRMRD::ismrmrdHeader::encoding_sequence e_seq = cfg->encoding(); 
     if (e_seq.size() != 1) {
      GADGET_DEBUG2("Number of encoding spaces: %d\n", e_seq.size());
      GADGET_DEBUG1("Only supports one encoding space supported\n");
      return GADGET_FAIL;
     }

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

### <a name="readerswriterslink" />Readers and Writers

As illustrated in the [Figure](#figuregadgetron) the Gadgetron uses a set of Readers and Writers to deal with the incoming communication on the TCP/IP socket. Readers are responsible for deserialization of packages and Writers are responsible for serialization of packages. All packages that arrive on the socket will start with a message ID. Based on this ID, the Gadgetron delegates the responsibility of reading the package of the socket to a particular instance of a GadgetMessageReader defined by the following abstract class:

    class GadgetMessageReader
    {
     public:
      virtual ACE_Message_Block* read(ACE_SOCK_Stream* stream) = 0;
    };

In order to be able to read a specific type of data, the `read` function must be implemented for that data type. As an example here is the GadgetIsmrmrdAcquisitionMessageReader, which reads an MRI data
acquisition from the socket.

    class GadgetIsmrmrdAcquisitionMessageReader 
    : public GadgetMessageReader
    {
     public:
      GADGETRON_READER_DECLARE(GadgetIsmrmrdAcquisitionMessageReader);
      virtual ACE_Message_Block* read(ACE_SOCK_Stream* socket);
    };

Note the `GADGETRON_READER_DECLARE(GadgetIsmrmrdAcquisitionMessageReader)` declaration. This is equivalent to the declaration needed for the Gadgets (see [Gadgets](#gadgetslink)) in order to make them load properly from shared libraries.

The implementation of this particular reader is as follows (this is an abbreviated version without error checking, etc.):

    ACE_Message_Block* GadgetIsmrmrdAcquisitionMessageReader::read(ACE_SOCK_Stream* sock)
    {
     GadgetContainerMessage<ISMRMRD::AcquisitionHeader>* m1 =
       new GadgetContainerMessage<ISMRMRD::AcquisitionHeader>();

     GadgetContainerMessage<hoNDArray< std::complex<float> > >* m2 =
       new GadgetContainerMessage< hoNDArray< std::complex<float> > >();

     m1->cont(m2);

     ssize_t recv_count = 0;

     if ((recv_count = stream->recv_n(m1->getObjectPtr(), sizeof(ISMRMRD::AcquisitionHeader))) <= 0) {
      m1->release();
      return 0;
     }

     if (m1->getObjectPtr()->trajectory_dimensions) {
      GadgetContainerMessage<hoNDArray< float > >* m3 =
        new GadgetContainerMessage< hoNDArray< float > >();

     m2->cont(m3);

     std::vector<unsigned int> tdims;
     tdims.push_back(m1->getObjectPtr()->trajectory_dimensions);
     tdims.push_back(m1->getObjectPtr()->number_of_samples);

     if (!m3->getObjectPtr()->create(&tdims)) {
      m1->release();
      return 0;
     }

     if ((recv_count =
       stream->recv_n
        (m3->getObjectPtr()->get_data_ptr(),
         sizeof(float)*tdims[0]*tdims[1])) <= 0) {

         m1->release();

       return 0;
     }

     std::vector<unsigned int> adims;
     adims.push_back(m1->getObjectPtr()->number_of_samples);
     adims.push_back(m1->getObjectPtr()->active_channels);

     if (!m2->getObjectPtr()->create(&adims)) {
       m1->release();
       return 0;
     }

     if ((recv_count =
          stream->recv_n
          (m2->getObjectPtr()->get_data_ptr(),
          sizeof(std::complex<float>)*adims[0]*adims[1])) <= 0) {

        m1->release();

        return 0;
     }

    return m1;
    }

    GADGETRON_READER_FACTORY_DECLARE(GadgetIsmrmrdAcquisitionMessageReader)

The Reader allocates two [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message.html) data blocks to contain the incoming data. First an [ISMRMRD::AcquisitionHeader](http://ismrmrd.github.io/api/struct_i_s_m_r_m_r_d_1_1_acquisition_header.html) is read. Based hereon the length of each acquisition (number of samples) and the number of acquisition channels are determined. An [hoNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_array.html) is allocated to store the data read from the socket. Notice that the two [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message.html) are chained together using the `cont` function.

A final important statement to notice is:

    GADGETRON_READER_FACTORY_DECLARE(GadgetIsmrmrdAcquisitionMessageReader)

This macro declares create and destroy functions to load the reader from a shared library on all platforms supported.

Whereas the Readers are responsible for deserialization, the GadgetMessageWriter is responsible for the opposite operation (serialization). In practice, Gadgets that produce an output for the client application can hand that data back to the Gadgetron framework where it is placed on the output queue along with a message ID. This is for instance done in this (abbreviated) code from an [ImageFinishGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget.html):

    template <typename T>
    int ImageFinishGadget<T>
    ::process(GadgetContainerMessage<ISMRMRD::ImageHeader>* m1,
       GadgetContainerMessage< hoNDArray< T > >* m2)
    {
      if (!this->controller_) {
        return -1;
      }

      GadgetContainerMessage<GadgetMessageIdentifier>* mb =
        new GadgetContainerMessage<GadgetMessageIdentifier>();

      switch (sizeof(T)) {
      case 2: //Unsigned short
       mb->getObjectPtr()->id = 
          GADGET_MESSAGE_IMAGE_REAL_USHORT;
       break;
      case 4: //Float
       mb->getObjectPtr()->id = 
          GADGET_MESSAGE_IMAGE_REAL_FLOAT;
       break;
      case 8: //Complex float
       mb->getObjectPtr()->id = 
          GADGET_MESSAGE_IMAGE_CPLX_FLOAT;
       break;
      default:
       GADGET_DEBUG2("Wrong data size detected: %d\n", sizeof(T));
       mb->release();
       m1->release();
       return GADGET_FAIL;
      }

      mb->cont(m1);

      int ret =  this->controller_->output_ready(mb);

      if ( (ret < 0) ) {
       GADGET_DEBUG1("Failed to return massage to controller\n");
       return GADGET_FAIL;
      }

      return GADGET_OK;
    }

Notice that the Gadget has a reference to the Gadgetron framework through the `controller_` member variable, which is set during initialization.

In the framework (more specifically in the [GadgetStreamController](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_stream_controller.html)) there is an active thread responsible for writing messages that are put on to the output queue. This is done by investigating the message ID and then picking the [GadgetMessageWriter](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_message_writer.html) associated with this ID. A Writer must implement the following abstract class:

    class GadgetMessageWriter
    {
     public:
      virtual int write(ACE_SOCK_Stream* stream, 
                        ACE_Message_Block* mb) = 0;
    };

The Writer is handed control of the socket along with the message block. A Writer declaration could look like:

    class MRIImageWriter 
      : public GadgetMessageWriter
    {

    public:
       GADGETRON_WRITER_DECLARE(MRIImageWriter);
       virtual int write(ACE_SOCK_Stream* sock, 
                         ACE_Message_Block* mb);
    };

Notice again the `GADGETRON_WRITER_DECLARE(MRIImageWriter)` which ensures proper run-time linking behavior. The implementation could look like (abbreviated with no error checking, etc.):

    int MRIImageWriter
         ::write(ACE_SOCK_Stream* sock, 
                 ACE_Message_Block* mb)
    {

       GadgetContainerMessage<ISMRMRD::ImageHeader>* imagemb = 
          AsContainerMessage<ISMRMRD::ImageHeader>(mb);
      
       GadgetContainerMessage< hoNDArray< float > >* datamb =
          AsContainerMessage< hoNDArray< float > >(imagemb->cont());
      
       if (!datamb || !imagemb) {
          //Deal with errors
       }
       
       GadgetMessageIdentifier id;
       //Example for real flow image.
       id.id = GADGET_MESSAGE_ISMRMRD_IMAGE_REAL_FLOAT; 
     
       sock->send_n (&id, sizeof(GadgetMessageIdentifier));

       sock->send_n (imagemb->getObjectPtr(), sizeof(ISMRMRD::ImageHeader));

       sock->send_n (datamb->getObjectPtr()->get_data_ptr(), 
          sizeof(float)*datamb->getObjectPtr()->get_number_of_elements());

       return 0;
    }

    GADGETRON_WRITER_FACTORY_DECLARE(MRIImageWriter)

Once again notice the required `GADGETRON_WRITER_FACTORY_DECLARE(MRIImageWriter)` macro. Also notice that the message ID is transmitted to the client. The client is expected to follow the same communication model as the Reader, but it is determined entirely by the Writer implementation how the message is transmitted.

Readers and Writers are loaded dynamically at run-time along with the Gadgets (see [Stream Configuration](#streamconfigurationlink)). The input and output behaviour can be adapted by manipulating which Readers and Writers are associated with which message IDs.

### <a name="streamconfigurationlink" />Stream Configuration

A Gadgetron reconstruction is made up of modules, i.e. Readers, Writers, and Gadgets. New reconstruction programs can be created by simply assembling existing components in a new way. The configuration of the Gadgetron stream is done at run-time and new configuration chains can be created without recompiling any of the underlying Gadgets. More specifically, the configuration is specified in an XML file that the Gadgetron will read before receiving data. The best way to explain the format is by looking at a (simplified) example:

    <?xml version="1.0" encoding="UTF-8"?>
    <gadgetronStreamConfiguration 
      xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
      xmlns="http://gadgetron.sf.net/gadgetron"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
            
        <reader>
          <slot>1008</slot>
          <dll>gadgetroncore</dll>
          <classname>GadgetIsmrmrdAcquisitionMessageReader</classname>
        </reader>
      
        <writer>
          <slot>1004</slot>
          <dll>gadgetroncore</dll>
          <classname>MRIImageWriterCPLX</classname>
        </writer>
        <writer>
          <slot>1005</slot>
          <dll>gadgetroncore</dll>
          <classname>MRIImageWriterFLOAT</classname>
        </writer>
        <writer>
          <slot>1006</slot>
          <dll>gadgetroncore</dll>
          <classname>MRIImageWriterUSHORT</classname>
        </writer>
      
        <gadget>
          <name>Acc</name>
          <dll>gadgetroncore</dll>
          <classname>AccumulatorGadget</classname>
        </gadget>
        <gadget>
          <name>FFT</name>
          <dll>gadgetroncore</dll>
          <classname>FFTGadget</classname>
        </gadget>
        <gadget>
          <name>Extract</name>
          <dll>gadgetroncore</dll>
          <classname>ExtractGadget</classname>
        </gadget>  
        <gadget>
          <name>ImageFinishFLOAT</name>
          <dll>gadgetroncore</dll>
          <classname>ImageFinishGadgetFLOAT</classname>
        </gadget>

    </gadgetronStreamConfiguration>

The stream configuration XML layout is defined in the `GADGETRON_HOME/schema/gadgetron.xsd`. A stream configuration must conform to this schema definition or an error will be generated when the Gadgetron attempts to load the configuration.

The configuration file format contains 3 sections: 1) Readers, 2) Writers, 3) Stream (with Gadgets) corresponding to the 3 different types of components that can be assembled in the Gadgetron.

In the example above, the Readers section contains only one reader, which is the [GadgetIsmrmrdAcquisitionMessageReader](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_ismrmrd_acquisition_message_reader.html) mentioned previously. The message ID associated with this Reader is 1008. Every time a message with ID 1008 arrives on the socket, responsibility for reading the message will be delegated to the [GadgetIsmrmrdAcquisitionMessageReader](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_ismrmrd_acquisition_message_reader.html). When the Gadgetron configuration is loaded, the framework will load the [GadgetIsmrmrdAcquisitionMessageReader](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_ismrmrd_acquisition_message_reader.html) from the DLL (shared library) `gadgetron_mricore`. On the Linux platform this would be a shared library called `libgadgetron_mricore.so` and on the Windows platform it would be called `gadgetron_mricore.dll`.

The Gadgetron framework knows how to load the components from the DLLs assuming that they have been declared properly as described in [Readers and Writers](#readersandwriterslink) and [Gadgets](#gadgetslink).

The example Gadgetron configuration has two Writers, i.e. it is capable of outputting two different types of data. Again the declarations cause the Gadgetron framework to load specific instances of GadgetMessageWriter and associate them with specific ID numbers.

There are certain built-in Readers and Writers in addition to those specified in the configuration file. As an example, there are Readers for receiving configurations to be used by the Gadgetron and for receiving the parameters that will be passed to all Gadgets (see [Communication Sequence](#communicationsequencelink)). If the Gadgetron receives a message with an ID for which there is no associated Reader or encounters a message on the output queue for which there is no associated Writer an error will be generated, the Gadgetron stream shuts down, and the connection to the client will be closed.

In the example above, we have 4 Gadgets in the reconstruction chain. The first Gadget is an [AccumulatorGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_accumulator_gadget.html), which collects individual lines and inserts them in k-space. When the k-space image is complete it is sent to the next Gadget in the chain, the [FFTGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_f_f_t_gadget.html), which is responsible for Fourier transforming the data into image space. The next Gadget, [ExtractGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_extract_gadget.html), extracts the magnitude of the complex image. Finally the last Gadget in the chain, [ImageFinishGadgetFLOAT](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget_f_l_o_a_t.html), sends the reconstructed image back to the Gadgetron framework where it is added to the output queue.

It is also possible to send configuration parameters to Gadgets using the XML file. For example, to set a parameter in a Gadget, one could write:

      <gadget>
       <name>Accumulator</name>
       <dll>gadgetroncore</dll>
       <classname>AccumulatorGadget</classname>
       <property><name>MyTestProperty</name>
       <value>Blah Blah</value></property>
       <property><name>MyTestProperty2</name>
       <value>98776.862187</value></property>
      </gadget>

The two properties will now be accessible inside the Gadget using the parameter access functions defined in `Gadget.h`:

    class Gadget : public ACE_Task<ACE_MT_SYNCH>
    {

    //Other definitions

    int get_bool_value(const char* name);
    int get_int_value(const char* name);
    double get_double_value(const char* name);

    };

Additionally it is also possible to specify how many active threads there should be in a Gadget. This is specified with:

      <gadget>
       <name>Accumulator</name>
       <dll>gadgetroncore</dll>
       <classname>AccumulatorGadget</classname>
       <property><name>threads</name><value>5</value></property>
      </gadget>

Which would make the AccumulatorGadget have 5 threads.

### <a name="communicationsequencelink" />Communication Sequence

Communication between a client and the Gadgetron follows a straightforward communication protocol. When the Gadgetron is started it will be expecting a connection on a specific port (port 9002 is the default). The communication sequence is as follows:

1.  The client makes connection

2.  The Gadgetron accepts the connection and creates a new instance of a [GadgetStreamController](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_stream_controller.html) (see [Gadgetron Figure](#figuregadgetron)). After creating the [GadgetStreamController](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_stream_controller.html) the Gadgetron returns to accept connections on the socket such that multiple clients can be connected simultaneously.

3.  The [GadgetStreamController](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_stream_controller.html) takes control of the socket and expects to read a specific type of message, which either contains the filename of a specific stream configuration (see [Stream Configuration](#streamconfigurationlink)) or alternatively it can receive the actual XML stream specification directly on the socket. These two types of messages are read with Readers that are always registered for the Gadgetron (see [Readers and Writers](#readersandwriterslink)). If the Gadgetron receives the filename of a Gadget stream it expects to be able to find that configuration file in the `gadegtron/config` folder (see [File Organization](#sectionfileorganization)).

4.  The [GadgetStreamController](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_stream_controller.html) is then expecting to receive parameters that will be transmitted to each individual Gadget. In principle the "parameters" is just a raw buffer of characters that will be transmitted as such to each individual Gadget. It is the convention however to send the parameters in an XML format. It is up to each individual Gadget to interpret the parameters. The user can implement any behavior in response to the parameters by implementing the `process_config` function (see [Gadgets](#gadgetslink)). The client can send parameters at any time during a reconstruction and they will always be transmitted to all Gadgets through the `process_config` function.

5.  The client then starts transmitting data packages that the Gadgetron processes. Images are returned to the client.

6.  When the client has no more data it will send a closure package. This package causes all Gadgets (in order) to process all remaining data on their input queue and then shut down.

7.  Once the final Gadget has shut down, the connection with the client is terminated.

To make it easier to create a new client, the Gadgetron comes with a [GadgetronConnector](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadgetron_connector.html) class:

    class GadgetronConnector: 
      public ACE_Svc_Handler<ACE_SOCK_STREAM, ACE_MT_SYNCH> {

    public:

     int open (std::string hostname, std::string port);   
     int putq  (ACE_Message_Block * mb ,  
         ACE_Time_Value *  timeout = 0);

     int register_reader(unsigned int slot, 
         GadgetMessageReader* reader);

     int register_writer(unsigned int slot, 
         GadgetMessageWriter* writer);

     int send_gadgetron_configuration_file(std::string config_xml_name);   
     int send_gadgetron_configuration_script(std::string config_xml_name);
     int send_gadgetron_parameters(std::string xml_string);
    };

This class can be used to create simple clients that open a connection with the Gadgetron using the `open` function and then communicate with the Gadgetron through the Readers and Writers registered with the connector. See the gadgetron_ismrmrd_client  example application (`gadgetron/apps/clients/gadgetron_ismrmrd_client` in the source code archive) for a simple example of how to build a Gadgetron client.

### <a name="sectionfileorganization" />File Organization

This section provides a brief overview of the file organization in the Gadgetron installation. Once you have compiled the Gadgetron and installed it (see [Linux Installation](./Linux-Installation), [Mac Installation](./Mac-OS-X-Installation), [Windows Installation](Windows-Installation)), it will reside in its designated installation folder (`GADGETRON_HOME`). For the purposes of this description, we will assume that the Gadgetron was installed in `/usr/local/gadgetron`.

In `GADGETRON_HOME` you should find the following folders:

-   `bin`: Contains all executables from the Gadgetron framework including the gadgetron executable itself and all clients and standalone applications.

-   `config`: Contains Gadgetron XML configuration files (see [Stream Configuration](#streamconfigurationlink)). This is where the Gadgetron searches for the configurations requested by the clients during initialization of the Gadget chain (see [Communication Sequence](#communicationsequencelink)). This folder must also contain a global `gadgetron.xml` configuration file, which is used to set global configuration parameters such as the port number for the Gadgetron. You can e.g. copy the `gadgetron.xml.example` to `gadgetron.xml` to create this file. The file is not created for you automatically in order to avoid overwriting any custom modifications during subsequent installations.

-   `lib:` Contains all shared libraries (Gadgets and toolboxes). Additionally, this is the default path where Python Gadgets look for Python modules.

-   `include`: Contains all header files for the Gadgets and Toolboxes in order that they can be linked into external applications and Gadget libraries compiled outside the Gadgetron source tree.

-   `schema`: Contains all the XML schema definitions used by the Gadgetron (e.g. `gadgetron.xsd`) and also serves as a container for schema files used by client applications and copied to this folder during installation.

-   `cmake:` Contains a set of helpful CMake scripts that can be used if you wish to build applications or Gadget libraries outside the Gadgetron source tree. Among other things it contains a `FindGadgetron.cmake` script, which can be used to localize and set paths for the Gadgetron using CMake.
