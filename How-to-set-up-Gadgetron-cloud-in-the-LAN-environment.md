Gadgetron offers flexible ability to distribute the reconstruction jobs over the cloud. That is, a group of computers act as  multiple computing nodes. Existing Gadgetron chain can be modified to run over a cloud in the LAN environment with minimal changes. The LAN based cloud solution can be as simple as a few computers in a research lab.

The wiki page will go through necessary steps to convert an example chain to run on cloud.

### Gadgetron chain for real-time cine reconstruction

Let's take an example Gadgetron chain, e.g. [Generic_Cartesian_Grappa_RealTimeCine.xml](https://github.com/gadgetron/gadgetron/blob/master/gadgets/mri_core/config/Generic_Cartesian_Grappa_RealTimeCine.xml)

This chain is an instance of so-called Gadgetron generic reconstruction framework. It starts with AcquisitionAccumulateTriggerGadget which accumulates data by slice. The BucketToBufferGadget will assemble kspace data as a 7D array [RO E1 E2 CHA PHS SET SLC]. For a cardiac cine scan, this means data from every slice will be sent down the chain as one recon job. The following GenericReconCartesianGrappaGadget will reconstruct complex image array in [RO E1 E2 1 PHS SET SLC]. This array is passed down the chain for kspace filtering, partial fourier handling and finally sent back to client.

If a cine scan is prescribed with multiple slices (e.g. 10-15 short axis slices), it is natural to send data from every slice to a computing node, in the cloud environment. 

Gadgetron can achieve this type of data splitting by adding distribution and collecting gadgets. 

### Distribute multiple slices to multiple computing nodes

First, add the distribute gadget before the accumulating gadget:

```
    <!-- Distribute slice over nodes -->
    <gadget>
      <name>Distribute</name>
      <dll>gadgetron_distributed</dll>
      <classname>IsmrmrdAcquisitionDistributeGadget</classname>
      <property><name>parallel_dimension</name><value>slice</value></property>
      <property><name>use_this_node_for_compute</name><value>false</value></property>
    </gadget>
```
Since we are sending each slice to a computing node, the parallel_dimension is slice. This variable should be modified if other dimensions are used for parallelization.

The result for every slice data will be sent back to the gateway node and can be collected by adding the collect gadget:

```
    <!-- Collect images from nodes -->
    <gadget>
        <name>Collect</name>
        <dll>gadgetron_distributed</dll>
        <classname>CollectGadget</classname>
    </gadget>
```

### Gadgetron cloudbus relay

In a LAN environment, a computing node need to be aware of the existence of other nodes in the network. Gadgetron achieves this by getting all nodes to register in a relay application:

```
gadgetron_cloudbus_relay

```
Suppose the IP address of server to run the cloudbus relay is 192.168.2.2, every computing node should get its Gadgetron to look at the relay:

```
gadgetron -p 9002 -r 192.168.2.2
```
The relay will add a computing node if a new node reports in for duty. It will also remove a computing node, if gadgetron stops on that computer.

This reporting process can take a few seconds. After all nodes are reported into the relay, all computing nodes are aware of any other nodes.

In this way, the Gadgetron cloud is symmetric, in the sense that every computing nodes know all other nodes. 

### An example set up to demonstrate this process

After adding the distribution and collecting gadgets to the realtime cine chain, the cloud version of this chain is : 
```
<?xml version="1.0" encoding="utf-8"?>
<gadgetronStreamConfiguration xsi:schemaLocation="http://gadgetron.sf.net/gadgetron gadgetron.xsd"
        xmlns="http://gadgetron.sf.net/gadgetron"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">

    <!--
        Gadgetron generic recon chain for 2D and 3D cartesian sampling

        RealTimeCine imaging
        Triggered by slice
        Recon N is phase and S is set

        Author: Hui Xue
        Magnetic Resonance Technology Program, National Heart, Lung and Blood Institute, National Institutes of Health
        10 Center Drive, Bethesda, MD 20814, USA
        Email: hui.xue@nih.gov
    -->

    <!-- reader -->
    <reader><slot>1008</slot><dll>gadgetron_mricore</dll><classname>GadgetIsmrmrdAcquisitionMessageReader</classname></reader>
    <reader><slot>1022</slot><dll>gadgetron_mricore</dll><classname>MRIImageReader</classname></reader>

    <!-- writer -->
    <writer><slot>1022</slot><dll>gadgetron_mricore</dll><classname>MRIImageWriter</classname></writer>
    <writer><slot>1008</slot><dll>gadgetron_mricore</dll><classname>GadgetIsmrmrdAcquisitionMessageWriter</classname></writer>

    <!-- Noise prewhitening -->
    <gadget><name>NoiseAdjust</name><dll>gadgetron_mricore</dll><classname>NoiseAdjustGadget</classname></gadget>

    <!-- RO asymmetric echo handling -->
    <gadget><name>AsymmetricEcho</name><dll>gadgetron_mricore</dll><classname>AsymmetricEchoAdjustROGadget</classname></gadget>

    <!-- RO oversampling removal -->
    <gadget><name>RemoveROOversampling</name><dll>gadgetron_mricore</dll><classname>RemoveROOversamplingGadget</classname></gadget>

    <!-- Distribute slice over nodes -->
    <gadget>
      <name>Distribute</name>
      <dll>gadgetron_distributed</dll>
      <classname>IsmrmrdAcquisitionDistributeGadget</classname>
      <property><name>parallel_dimension</name><value>slice</value></property>
      <property><name>use_this_node_for_compute</name><value>false</value></property>
    </gadget>

    <!-- Data accumulation and trigger gadget -->
    <gadget>
        <name>AccTrig</name>
        <dll>gadgetron_mricore</dll>
        <classname>AcquisitionAccumulateTriggerGadget</classname>
        <property><name>trigger_dimension</name><value>slice</value></property>
        <property><name>sorting_dimension</name><value></value></property>
    </gadget>

    <gadget>
        <name>BucketToBuffer</name>
        <dll>gadgetron_mricore</dll>
        <classname>BucketToBufferGadget</classname>
        <property><name>N_dimension</name><value>phase</value></property>
        <property><name>S_dimension</name><value>set</value></property>
        <property><name>split_slices</name><value>true</value></property>
        <property><name>ignore_segment</name><value>true</value></property>
    </gadget>

    <!-- Prep ref -->
    <gadget>
        <name>PrepRef</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconCartesianReferencePrepGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>true</value></property>
        <property><name>verbose</name><value>true</value></property>

        <!-- averaging across repetition -->
        <property><name>average_all_ref_N</name><value>true</value></property>
        <!-- every set has its own kernels -->
        <property><name>average_all_ref_S</name><value>false</value></property>
        <!-- whether always to prepare ref if no acceleration is used -->
        <property><name>prepare_ref_always</name><value>true</value></property>
    </gadget>

    <!-- Coil compression -->
    <gadget>
        <name>CoilCompression</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconEigenChannelGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>true</value></property>
        <property><name>verbose</name><value>true</value></property>

        <property><name>average_all_ref_N</name><value>true</value></property>
        <property><name>average_all_ref_S</name><value>true</value></property>

        <!-- Up stream coil compression -->
        <property><name>upstream_coil_compression</name><value>true</value></property>
        <property><name>upstream_coil_compression_thres</name><value>-1</value></property>
        <property><name>upstream_coil_compression_num_modesKept</name><value>0</value></property>
    </gadget>

    <!-- Recon -->
    <gadget>
        <name>Recon</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconCartesianGrappaGadget</classname>

        <!-- image series -->
        <property><name>image_series</name><value>0</value></property>

        <!-- Coil map estimation, Inati or Inati_Iter -->
        <property><name>coil_map_algorithm</name><value>Inati</value></property>

        <!-- Down stream coil compression -->
        <property><name>downstream_coil_compression</name><value>true</value></property>
        <property><name>downstream_coil_compression_thres</name><value>0.002</value></property>
        <property><name>downstream_coil_compression_num_modesKept</name><value>0</value></property>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>true</value></property>
        <property><name>verbose</name><value>true</value></property>

        <!-- whether to send out gfactor -->
        <property><name>send_out_gfactor</name><value>true</value></property>
    </gadget>

    <!-- Partial fourier handling -->
    <gadget>
        <name>PartialFourierHandling</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconPartialFourierHandlingPOCSGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>false</value></property>
        <property><name>verbose</name><value>false</value></property>

        <!-- if incoming images have this meta field, it will not be processed -->
        <property><name>skip_processing_meta_field</name><value>Skip_processing_after_recon</value></property>

        <!-- Parfial fourier POCS parameters -->
        <property><name>partial_fourier_POCS_iters</name><value>6</value></property>
        <property><name>partial_fourier_POCS_thres</name><value>0.01</value></property>
        <property><name>partial_fourier_POCS_transitBand</name><value>24</value></property>
        <property><name>partial_fourier_POCS_transitBand_E2</name><value>16</value></property>
    </gadget>

    <!-- Kspace filtering -->
    <gadget>
        <name>KSpaceFilter</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconKSpaceFilteringGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>false</value></property>
        <property><name>verbose</name><value>false</value></property>

        <!-- if incoming images have this meta field, it will not be processed -->
        <property><name>skip_processing_meta_field</name><value>Skip_processing_after_recon</value></property>

        <!-- parameters for kspace filtering -->
        <property><name>filterRO</name><value>Gaussian</value></property>
        <property><name>filterRO_sigma</name><value>1.0</value></property>
        <property><name>filterRO_width</name><value>0.15</value></property>

        <property><name>filterE1</name><value>Gaussian</value></property>
        <property><name>filterE1_sigma</name><value>1.0</value></property>
        <property><name>filterE1_width</name><value>0.15</value></property>

        <property><name>filterE2</name><value>Gaussian</value></property>
        <property><name>filterE2_sigma</name><value>1.0</value></property>
        <property><name>filterE2_width</name><value>0.15</value></property>
    </gadget>

        <!-- FOV Adjustment -->
    <gadget>
        <name>FOVAdjustment</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconFieldOfViewAdjustmentGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>debug_folder</name><value></value></property>
        <property><name>perform_timing</name><value>false</value></property>
        <property><name>verbose</name><value>false</value></property>
    </gadget>

        <!-- Image Array Scaling -->
    <gadget>
        <name>Scaling</name>
        <dll>gadgetron_mricore</dll>
        <classname>GenericReconImageArrayScalingGadget</classname>

        <!-- parameters for debug and timing -->
        <property><name>perform_timing</name><value>false</value></property>
        <property><name>verbose</name><value>false</value></property>

        <property><name>min_intensity_value</name><value>64</value></property>
        <property><name>max_intensity_value</name><value>4095</value></property>
        <property><name>scalingFactor</name><value>10.0</value></property>
        <property><name>use_constant_scalingFactor</name><value>true</value></property>
        <property><name>auto_scaling_only_once</name><value>true</value></property>
        <property><name>scalingFactor_dedicated</name><value>100.0</value></property>
    </gadget>

    <!-- ImageArray to images -->
    <gadget>
        <name>ImageArraySplit</name>
        <dll>gadgetron_mricore</dll>
        <classname>ImageArraySplitGadget</classname>
    </gadget>

    <!-- Collect images from nodes -->
    <gadget>
        <name>Collect</name>
        <dll>gadgetron_distributed</dll>
        <classname>CollectGadget</classname>
    </gadget>

    <!-- after recon processing -->
    <gadget>
        <name>ComplexToFloatAttrib</name>
        <dll>gadgetron_mricore</dll>
        <classname>ComplexToFloatGadget</classname>
    </gadget>

    <gadget>
        <name>FloatToShortAttrib</name>
        <dll>gadgetron_mricore</dll>
        <classname>FloatToUShortGadget</classname>
    </gadget>

    <gadget>
        <name>ImageFinish</name>
        <dll>gadgetron_mricore</dll>
        <classname>ImageFinishGadget</classname>
    </gadget>

</gadgetronStreamConfiguration>

```
### Run gadgetron cloud with docker

It is much more convenient to run gadgetron docker images at a number of nodes which consists of a local/remote cloud. In this way, user will not need to compile gadgetron on every node; instead, simply pulling the docker images. To find how to run gadgetron with docker, check the wiki page [gadgetron and docker](https://github.com/gadgetron/gadgetron/wiki/Using-Docker).

Let's go through an example. Assume there is a local cluster, consisting of N=5 computers. Their ip addresses are 192.168.2.2-6. To set up the gadgetron docker based cloud:

1) Install docker-ce and optional nvidia-docker at all nodes (check [gadgetron and docker](https://github.com/gadgetron/gadgetron/wiki/Using-Docker))

2) Start docker at every node as :
```
docker run --name=gadgetron -e "GADGETRON_RELAY_HOST=192.168.2.2" --publish=9888:8888 --publish=9002:9002 --publish=9080:9080 --publish=8090:8090 --publish=8002:8002 --publish=18002:18002 --publish=9001:9001 --volume=/tmp/gadgetron_data:/tmp/gadgetron_data  --restart=unless-stopped --detach -t gadgetron/ubuntu_1604_cuda80
```

Here the gadgetron is started inside docker and cloud bus relay is started with host ip 192.168.2.2. This ip will be the cloud gateway and distributes the incoming jobs to other computing nodes.

3) To run the cloud, send data to 192.168.2.2. Suppose there is a multi-slice data in /home/gtuser/cine_example folder:

```
cd /home/gtuser/cine_example
# convert noise to ismrmrd format
siemens_to_ismrmrd -f ./tpat3_RT_Cine_Res160.dat -o noise.h5 -z 1

# convert imaging data
siemens_to_ismrmrd -f ./tpat3_RT_Cine_Res160.dat -o data.h5 -z 2

# send noise to cloud gateway
gadgetron_ismrmrd_client -f noise.h5 -p 9002 -a 192.168.2.2 -c default_measurement_dependencies.xml

# send imaging data to cloud gateway
gadgetron_ismrmrd_client -f data.h5 -p 9002 -a 192.168.2.2 -c Generic_Cartesian_Grappa_RealTimeCine_Cloud.xml -o res.h5
```
Here we send a multi-slice cine dataset to gateway. Every slice will be forwarded to a node for reconstruction. The result images are stored in res.h5.

