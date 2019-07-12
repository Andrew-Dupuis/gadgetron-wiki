Gadgetron offers flexible ability to distribute reconstruction jobs amongst multiple computers. A group of computers will act as computing nodes, working together to complete a task.   

To employ this capability, Gadgetron must be configured to find other worker nodes - a process referred to as 'Node Discovery'. In essence, Gadgetron will need some way to obtain a list of other computers it can send work to. 

Once Gadgetron has a list of potential workers, it can honour requests for distribution in incoming reconstructions. These appear in incoming config files as `Distributed` (and `PureDistributed`) streams. We will cover these in detail. 

## Node Discovery
Node Discovery is the process by which Gadgetron discovers available workers. This process is very flexible by design, as Gadgetron needs to distribute work in a wide range of production environments. 

To discover worker nodes, a Gadgetron instance will examine it's environment variables, and execute the command specified in the `GADGETRON_REMOTE_WORKER_COMMAND` variable. 

This command can be anything. It can read a prepared list of workers, or query a list of running instances from a cloud computing service. There's no restrictions, as long as the output is a list of available workers.

The discovery command is expected to produce a list of nodes (as a JSON list written to standard out). Gadgetron will run the command, and consume the list of nodes as it deems appropriate. Gadgetron will run this command repeatedly, whenever the list of available workers is to be refreshed. You should think of Gadgetron executing the worker command as Gadgetron asking 'What workers are available to me right now?'

If node discovery fails for any reason, Gadgetron will complete the reconstruction without distributing work. Should this happen, Gadgetron will produce a warning. 

#### Example: Node Discovery using a Prepared File
For this example, I have a number of computers in the office, and I'd like to run some distributed reconstructions. I know the IP addresses of these machines, and I'll start Gadgetron on each of them. Gadgetron will then be running on the worker machines, ready to take on work. I'll run Gadgetron on the default port: 9002. 

Node Discovery will not be very complex - I'll simply prepare a list of the machines in my office, and have the Gadgetron node discovery read the file.

This list is a simple JSON list, describing my office machines: 
```json
["192.168.1.84:9002", "192.168.1.86:9002", "192.168.1.87:9002"]
```
I'll save this list of workers as ```~/.gadgetron/workers.json```.

To use this list as the available workers, I will read the contents of the file when asked to provide worker nodes. Starting Gadgetron with the node discovery environment variable could look like this:
```bash
$ env GADGETRON_REMOTE_WORKER_COMMAND="cat ~/.gadgetron/workers.json" gadgetron
```
Every time Gadgetron tries to discover nodes, the file will be read and passed to Gadgetron. Gadgetron will parse the list, and hand work out to the office machines.

To test the setup, I'll run one of the distributed integration tests (these usually have 'distributed' in the name): 
```bash
$ cd path/to/gadgetron/test/integration
$ ./run_tests.py -ep 9002 cases/cmr_cine_binning_2slices_distributed.cfg
```

#### Example: Node Discovery using a Python Relay 
For this example, I'll be using a [Python Relay](https://github.com/dchansen/gadgetron_cloudbus) to do dynamic node discovery. The relay is a tiny Python script, keeping track of the running Gadgetron instances. Each worker will periodically contact the relay, telling it where it can be reached. The main instance of Gadgetron will query the relay to obtain a list of workers. 

This setup mirrors the CloudBus/Relay functionality in older versions of Gadgetron. 

The relay will run on port 5000 by default. We will need the address of the relay when we start the workers, as the workers will need to contact the relay to tell it where they can be found. The relay is a very simple server, maintaining a list of known workers. Gadgetron will query the relay to obtain a list of currently active workers. The relay need not run on a specific machine, it just needs to be accessible from the worker pool.

To start the relay, simply start the relay:
```bash 
$ python3 cloudbus_relay.py
```

Once the relay is running on a known address, starting workers is a simple as running the worker script: 
```bash
$ python3 cloudbus_worker.py --relay 192.168.1.82:5000
```
The worker script will start an instance of Gadgetron, and restart if needed. The worker script will also let the relay know where to reach the Gadgetron worker instance. Post's should appear in the relay log, as the workers report in. 

Having started the workers on a number of machines, the relay maintains a list of workers. We can simply query the cloudbus/workers endpoint to access it: 
```bash
$ curl 192.168.1.82:5000/cloudbus/workers
```
To have Gadgetron do node discovery using the relay, simply set the appropriate environment variable when starting Gadgetron:
```bash
$ env GADGETRON_REMOTE_WORKER_COMMAND="curl 192.168.1.82:5000/cloudbus/workers" gadgetron
```
Gadgetron now queries the relay every time it needs to discover new nodes. 

To test the setup, we can run one of the distributed integration tests: 
```bash
$ cd path/to/gadgetron/test/integration
$ ./run_tests.py -ep 9002 cases/cmr_cine_binning_2slices_distributed.cfg
```

## Distributed

Requesting that Gadgetron distributes part of a reconstruction is a matter of introducing a `Distributed` (or `PureDistributed`) stream in the reconstruction configuration file. I'll refer to [distributed_default.xml](https://github.com/gadgetron/gadgetron/blob/Gadgetron4.0/core/config/distributed_default.xml) when discussing distributed config files in particular, though you should have no trouble generalizing to your own use case. 

I'll try to summarize the config file here, as reading XML can be tedious. I'll then proceed to cover the distributed components in detail.
```
config
    version
    readers/writers
    stream
        distributed
            readers/writers
            distributor
            stream
                gadget [These will run on remote workers]
                gadget ...
        gadget [These will run locally]
        gadget ...
```
The above describes a pretty normal (version 2) config file, with the obvious inclusion of a distributed part. You can read more about config files [here](https://github.com/gadgetron/gadgetron/wiki/Gadgetron-Streaming-Architecture#stream-configuration). 

The `distributed` node can be placed in config files just like a `gadget` node. Here, `distributed` is the first node in the top-level stream, but this is incidental. `distributed` nodes can (like all the new nodes) be used like `gadget` nodes almost anywhere. It is common to see noise adjustment ahead of `distributed` nodes, for instance. 

As should be evident in the reference config file, a `distributed` node defines a number things. These are the things the Gadgetron distributed system needs to know to send data to and from remote workers. A `distributed` node must define:

##### Readers/Writers
These are plain readers and writers, as used by Gadgetron when talking to any client. These are used to (de)serialize the messages passed to the remote worker, so they can be transmitted over a TCP socket. They handle the conversion from stuff to bytes, and bytes to stuff. Gadgetron ships with a number of readers/writers for the ISMRM Raw Data format, along with a handful of other data types. From the worker's point of view, an incoming connection from the distributing Gadgetron instance looks like any other client connection - the same protocol is used for all communication. In many ways, Gadgetron communicates with remote workers by being a regular client, sending first a configuration, then a data header, then some data. 

##### Distributor
The Distributor is a specialized piece of code, tasked with deciding which worker handles what data. The `distributor` config entry needs to know where to find the code, same as `gadget` - the config file needs a reference to a class to load, and a library to load it from. The `distributor` itself is the piece of code that looks at a piece of input, and decides which worker it should be sent to. Here's the relevant bit from the `AcquisitionDistributor` referenced in the config file: 
```c++
void AcquisitionDistributor::process(
        TypedInputChannel<Acquisition> &input,
        ChannelCreator &creator
) {
    std::map<uint16_t, OutputChannel> channels{};
    for (Acquisition acq : input) {
        auto index = selector(std::get<Header>(acq));
        if (!channels.count(index)) channels.emplace(index, creator.create());
        channels.at(index).push(std::move(acq));
    }
}
```
The `Distributor` interface, like `Gadget`, revolves around the process function. However, the `Distributor` takes only an input channel and a `ChannelCreator`. The channel creator is an object that knows how to open connections to remote workers - these behave like normal output channels when opened. 

This particular `Distributor` determines an index from each acquisition by looking at the acquisition header. The exact strategy (the `selector`) depends on the configuration of the `AcquisitionDistributor`; in this example it is configured to look at `idx.repetition` of each acquisition header. Based on the index, an appropriate remote worker is selected (a connection is opened if necessary), and the acquisition is pushed out to the worker. 

##### Stream
The `stream` is a Gadgetron reconstruction passed to the remote workers. It is the answer to the question "_What work should be done?_". As a connection is opened to a remote worker, the distributed stream is packaged as a regular Gadgetron config file, and passed to the remote worker. The remote worker then proceeds to load the appropriate Gadgets, process the data, and send it back, same as for any other client. 

#### Output
Data received from remote workers is pushed to the `distributed` node's output channel as it arrives. This means data passing through fast workers may overtake data from slow workers, for instance. If you rely on the ordering of the output, you may wish to add a sorting Gadget after the `distributed` node (as is done in the reference config file). Alternatively, the `puredistributed` node guarantees consistent ordering of it's output, along with some other excellent features like error handling and load balancing. 

## Pure Distributed




