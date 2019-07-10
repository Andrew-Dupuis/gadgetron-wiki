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

## Pure Distributed




