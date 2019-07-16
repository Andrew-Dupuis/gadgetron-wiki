New in Gadgetron 4.0 is the ability to create branching streams. This allows data to be separated into different parallel paths, each processed individually. In earlier versions of Gadgetron similar functionality has been achieved by Gadgets accessing their stream context though a `StreamController`, but 4.0 brings a kosher mechanisms for fine-grained control of data flow. 

To use parallel streams in a Gadgetron reconstruction you need simply add a `parallel` node to the reconstruction configuration. In doing so, you must specify what is to happen in each of the parallel streams, by adding multiple `stream` nodes to the `parallel` node, each containing any number of gadgets. You must also specify the logic that is to determine which parallel stream incoming data is sent down, as well as the logic merging output from the parallel streams into a single output. 

The `branch` and `merge` logic is handled by a `Branch` and `Merge` object respectively. These are loaded from a shared object library like normal Gadgets. In fact, `Branch` and `Merge` objects are just like Gadgets in almost every way. They just take multiple input- or output-channels (as opposed to a single input and output) as arguments to their process function. 

#### Configuration

In order to examine a branching chain in detail, I'll be having a look at a reconstruction from the real-time grappa package. I'll be referring to [grappa_cpu.xml](https://github.com/gadgetron/gadgetron/blob/Gadgetron4.0/gadgets/grappa/config/grappa_cpu.xml) as an example of a branching chain. I trust you can easily generalize to your own use case. 

As reading XML can be a little tedious, I'll attempt to summarize the config here. Of particular interest is of course the parallel part of the reconstruction. In this case, each stream is only a single Gadget long. This is incidental, each stream can contain any number of Gadgets (or even other nodes, such as distributed parts). 
```
config
    version
    readers/writers
    stream
        gadget 
        gadget ...
        parallel
            branch
            stream [key: images]
                gadget [ImageAccumulator]
            stream [key: weights]
                gadget [WeightsCalculator]
            merge
        gadget 
        gadget ...
```
In full, the `distributed` node from the XML looks like this: 
```xml
<parallel>
    <branch>
        <dll>gadgetron_grappa</dll>
        <classname>AcquisitionFanout</classname>
    </branch>
    <stream key="images">
        <gadget>
            <dll>gadgetron_grappa</dll>
            <classname>ImageAccumulator</classname>
        </gadget>
    </stream>
    <stream key="weights">
        <gadget>
            <dll>gadgetron_grappa</dll>
            <classname>cpuWeightsCalculator</classname>
        </gadget>
    </stream>
    <merge>
        <dll>gadgetron_grappa</dll>
        <classname>Unmixing</classname>
    </merge>
</parallel>
``` 
It is also important to note that you are not limited to two streams in the `parallel` node. As long as the streams have unique keys (note the keys on each stream in the example; "images" and "weights"), they will be accessible from the branch and merge objects.  

### Branch

#### Example: Fanout Branch that copies input to all outputs.

#### Example: Encoding Space Branch that selects output based on encoding space.

### Merge

#### Example: ???