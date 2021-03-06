New in Gadgetron 4.0 is the ability to create branching streams. This allows data to be separated into different parallel paths, each processed individually. In earlier versions of Gadgetron similar functionality has been achieved by Gadgets accessing their stream context though a `StreamController`, but 4.0 brings a kosher mechanisms for fine-grained control of data flow. 

To use parallel streams in a Gadgetron reconstruction you need simply add a `parallel` node to the reconstruction configuration. In doing so, you must specify what is to happen in each of the parallel streams, by adding multiple `stream` nodes to the `parallel` node, each containing any number of gadgets (or other nodes). You must also specify the logic that determines which stream to send incoming data to, as well as the logic merging output from the parallel streams into a single output. 

The branch and merge logic is handled by a `Branch` and `Merge` object respectively. These are loaded from a shared object library like normal Gadgets. In fact, `Branch` and `Merge` objects are just like Gadgets in almost every way. They just take multiple input- or output-channels (as opposed to a single input and output) as arguments to their process function. 

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
In full, the `parallel` node from the XML looks like this: 
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
It is also important to note that you are not limited to two streams in the `parallel` node. As long as the streams have unique keys (note the keys on each stream in the example; images and weights), they will be accessible from the branch and merge objects.

### Branch
A `Branch` object knows how to distribute data between multiple output channels, one for each parallel stream. It can also do processing, same as a Gadget, prior to selecting an appropriate output channel. 

The `Branch` object interface is the same as for Gadgets, except for the signature of the process function. Where a `ChannelGadget` would only have a single `OutputChannel`, the `Branch` object must accept a `std::map` containing any number of `OutputChannel`s. The keys in the output map will be strings, each being the 'key' of the parallel stream as specified in the configuration file. 

#### Example: Fanout Branch that copies input to all outputs.

The `AcquisitionFanout` branch used in the real-time grappa code is in fact just an instantiation of a `Fanout` branch. The `Fanout` branch object will take take it's input, and copy it to each output channel. It'll need the input type to have a well defined copy operator, but not much else. This is implemented in the core Gadgetron library - let's take a look at the `process` code: 
```c++
template<class... ARGS>
void Fanout<ARGS...>::process(InputChannel<ARGS...> &input, std::map<std::string, OutputChannel> output) {
    for (auto thing : input) {
        for (auto &pair : output) {
            auto copy_of_thing = thing;
            pair.second.push(copy_of_thing);
        }
    }
}
```
Of note is the `std::map` containing the output channels. While the output map is most often accessed using the keys, the fanout branch doesn't care. It simply iterates over all the key-output pairs in the output map, and copies it's input into each output channel. 

Fanouts are a common way to start parallel streams, so Fanout types for common data types already exist in the Gadgetron libraries. You can find Fanouts for the ISMRMRD types (`Acquisition`, `Waveform`, `Image`) in the `gadgetron_core_parallel` library. 

#### Example: Acquisition/Waveform Branch sending each down different streams.

While this example might not be particularly useful, I hope to illustrate how to write a Branch object that does something useful. I've decided to handle both Acquisitions and Waveforms, as this allows me to also illustrate how to deal with input of multiple types. You will find the code amongst the Gadget examples; look for the files in `gadgets/examples`. 

I'm listing all the source code for this example. I hope seeing everything will make the endeavour of writing a Branch node seem accessible. I'll attempt to highlight the choices I've made in this example, along with their effect on the code. 

At this point, I've made a single choice: I'm writing a Branch class that works on `Acquisition` and `Waveform` messages. As I'm not interested in handling generic messages, but know the types I'm interested in, I'll be implementing a `TypedBranch` (as I would implement a `TypedChannelGadget` if I were writing a Gadget). Handling multiple input types is done through an application of `Core::variant`. 

As I hope to point out, this determines entirely the contents of the header file, as well as the structure of the implementation. But let's take it one step at a time, and have a look at the header file:
```c++
#pragma once

#include "parallel/Branch.h"

namespace Gadgetron::Examples {

    using AcquisitionOrWaveform = Core::variant<Core::Acquisition, Core::Waveform>;

    class AcquisitionWaveformBranch : public Core::Parallel::TypedBranch<AcquisitionOrWaveform> {
    public:
        AcquisitionWaveformBranch(const Core::Context &, const Core::GadgetProperties &);
        void process(
                Core::InputChannel<AcquisitionOrWaveform> &,
                std::map<std::string, Core::OutputChannel>
        ) override;
    };
}
```
Of note is the `using` alias, giving a slightly more readable name to a `variant` of `Acquisition` and `Waveform`. 
Apart from that, the header file is simply declaring a TypedBranch of the appropriate type. No choices are made, the header simply follows from the form of a `TypedBranch`. 

The implementation is similarly constrained. In it's entirety, the implementation for the `AcquisitionWaveformBranch` is this:

```c++
#include "AcquisitionWaveformBranch.h"

using namespace Gadgetron::Core;
using namespace Gadgetron::Core::Parallel;

namespace {
    std::string select_channel(const Acquisition &) { return "acquisitions"; }
    std::string select_channel(const Waveform &) { return "waveforms"; }
}

namespace Gadgetron::Examples {

    AcquisitionWaveformBranch::AcquisitionWaveformBranch(
            const Context &,
            const GadgetProperties &properties
    ) : TypedBranch<AcquisitionOrWaveform>(properties) {}

    void AcquisitionWaveformBranch::process(
            InputChannel<AcquisitionOrWaveform> &input,
            std::map<std::string, OutputChannel> output
    ) {
        for (auto acq_or_wav : input) {
            auto &channel = output.at(visit([](auto &aw) { return select_channel(aw); }, acq_or_wav));
            channel.push(std::move(acq_or_wav));
        }
    }

    GADGETRON_BRANCH_EXPORT(AcquisitionWaveformBranch)
}
```
The constructor is fairly innocuous, so I'll focus on the process function. This, as always, is where the work takes place. The Branch process function takes an input channel and a map of output channels. One output channel for each of the parallel streams. The keys in this map are the keys as provided in the config file; this piece of code assumes the stream for `Acquisition` messages will be named "acquisitions"; likewise for waveforms.

The process function itself is simple. It simply examines it's input messages one by one, and chooses an output channel from the map based on the type of the input. The `visit` function is very commonly used with `variant` types to specify what is to be done, depending on the actual type present.  

Note that the class must be exported appropriately, much the same way gadgets are exported. This ensures that appropriate symbols are present in the compiled shared object files, which are in turn used by Gadgetron when the Branch object is to be loaded and used.

### Merge

A Merge object knows how to assemble a single output from multiple parallel streams. It can also do processing, same as a Gadget, before producing output. 

The Merge object interface is the same as for Gadgets, except for the signature of the process function. Where a Gadget would only have a single InputChannel, the Merge object features a std::map containing any number of InputChannels. The keys in the output map will be strings, each being the 'key' of the parallel stream as specified in the configuration file.

It is noteworthy that there is no 'TypedMerge'. As each parallel stream can produce widely different outputs, and such types can't be reliably encoded in the map of input channels, writing a `Merge` often entails a manual step of escalating to appropriately typed `InputChannel`s before processing begins.

#### Example: Merging image output from different streams. 

To illustrate how to Merge streams, I'm including an example merging images from two parallel streams. 

I'll skip the header file, as it's not very interesting. It's contents is entirely dictated by the form of a Merge node. You can find the appropriate header in `gadgets/examples/ImageLayerer.h`. 

The implementation of the `ImageLayerer` looks like this:  
```c++
namespace Gadgetron::Examples {

    ImageLayerer::ImageLayerer(const Context &, const GadgetProperties &properties) : Merge(properties) {}

    void ImageLayerer::process(std::map<std::string, GenericInputChannel> input, OutputChannel output) {

        auto unchanged = InputChannel<AnyImage>(input.at("unchanged"), output);
        auto inverted = InputChannel<AnyImage>(input.at("inverted"), output);

        for (auto image : unchanged) {
            auto merged = Core::visit(
                    [](const auto &a, const auto &b) -> AnyImage { return merge(a, b); },
                    image,
                    inverted.pop()
            );

            GINFO_STREAM("Images combined; pushing out result.");

            output.push(std::move(merged));
        }
    }

    GADGETRON_MERGE_EXPORT(ImageLayerer)
}
```
I've left out the details of the `merge` function. It is simple enough, but not particularly relevant to the Merge mechanics.    

Note that we escalate from `GenericInputChannel` to `InputChannel`. This is done early, and is simply a matter of 'expecting images' form the two streams. As always, any message not containing an image will be silently passed to the output channel unchanged.

Also note that `visit` again plays a role in making sure the right processing takes place. `AnyImage` is a `variant` of a number of different `Image` types (`Image<float>`, `Image<double>`, etc.), and the application of `visit` makes sure that the correct version of `merge` is called. 

The loop structure, due to the presence of multiple inputs, is a little different than a normal Gadget loop. In essence, we consume one input channel as if we were a Gadget, and it was the only input. The other input is consumed during the loop. So each iteration will of the loop will consume from first one channel, then the other. This will work fine in this example (`gadgets/examples/config/parallel_bypass_example.xml`) as we are sure an equal number of images passes through the parallel streams. 

In order to support more advanced consumption schemes, `GenericInputChannel` also have a `try_pop` function. This is an excellent primitive for building consumption schemes that are not fixed ratio. Consider having a look at the real-time grappa merge as an excellent example of uneven consumption. `gadgets/grappa/Unmixing.cpp` contains the relevant code.