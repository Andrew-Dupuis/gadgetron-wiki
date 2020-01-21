While Gadgetron is written in C++, users will frequently prefer to work in another programming language - Matlab and Python are common choices. 

The Foreign Language Interface provides a uniform way to have foreign (non-C++) code as part of a Gadgetron reconstruction chain. This document will describe how the Foreign Language Interface works.

## Overview

The Foreign Language Interface uses an ISMRMRD stream to communicate with a foreign language (or runtime). The very same kind of stream used by the client to talk to the Gadgetron server, in fact. In this way, data can flow between Gadgetron and any compatible ISMRMRD server, written in any language. 

Thus, the Foreign Language Interface consists of two major components: A Gadgetron `External` node, playing the 
part of an ISMRMRD client; and an ISMRMRD server written in your language of choice. 

There are no restrictions on the foreign server (besides ISMRMRD compatibility). It can run in a process of your choosing, on a different machine, in a debugger, container, or wherever else you find it convenient. The Foreign Language Interface is very flexible by design. 

Similarly, the use of an ISMRMRD stream means that all data transmitted must be collapsed into a sequence of bytes during transmission (a process handled by a Reader/Writer pair). This requires communication to feature well defined units of data, that are meaningful on their own. It also let's the foreign language present input to (and expect output from) users in a manner that makes sense within the foreign language, with no concern for the data's representation within Gadgetron. 

[comment]: # (I've put the illustration in a personal repository. Dunno if that's a good solution, but at least you can get at them if you need to make changes. -k)

![Foreign Language Interface Overview](https://raw.githubusercontent.com/kristofferknudsen/gadgetron-illustrations/master/external.svg)

In simple terms, any interface to Gadgetron in a foreign language is a simple ISMRMRD server, exposing a sensible interface to the user, as well as a few plumbing details (to facilitate getting started by Gadgetron, etc.). 

Both the Matlab- and Python Foreign Language Interface expose the data from Gadgetron through a `Connection` object. This will let you iterate through input, and return output to Gadgetron. The two languages are different, so the exact functionality varies between the two implementations, but the abstract concept modelled is the same. 

## Modes

Gadgetron supports two different modes of interaction with foreign languages. One is active, where Gadgetron will take charge, start your language (interpreter, runtime, etc.), call your function with a connection, and end the process when done. The other is passive, where Gadgetron simply connects to a server you started - you just need to tell it where to find it. 

Mode is determined from the config file you send Gadgetron. Specifically, Gadgetron examines the `external` node in your chain to determine what to do. Gadgetron expects to find either a `connect` directive (causing it to enter passive mode), or an `execute` directive, describing what code you would like to run. We'll take a look at each in detail.

### Passive - Listen/Connect

To enter passive mode, you must simply direct Gadgetron to connect to your foreign language on a port of your choosing. It's your responsibility to make sure your foreign language is listening, and waiting to accept connections. This allows you to easily run your code with a debugger, a profiler, or other useful tools, making passive mode particularly useful during development. 

To enter Passive mode, simply specify a `connect` directive in the appropriate `external` node. Here's the relevant snippet from a config file: ([full config](https://github.com/gadgetron/gadgetron/blob/master/gadgets/examples/config/external_connect_example.xml))

```xml
        <external>
            <!-- Connect to a running process on port 18000. -->
            <connect port="18000"/>

            <!-- The configuration is sent to the external process. It's left pretty empty here. -->
            <configuration/>
        </external>
```

Besides a port, the `connect` directive also accepts an address to connect to. This defaults to 'localhost', but can be set however you like. It's useful for connecting to foreign languages outside a docker container, for instance. 

### Active - Execute

To enter active mode, you must simply tell Gadgetron what foreign code to actively pursue. This is done by adding an `execute` directive to the appropriate `external` node. Active mode let's Gadgetron 'get on with it' on it's own. It'll start everything up, make sure your code is run, and close everything down nicely afterwards. 

Where passive mode made it easy to debug/profile/develop your foreign language code, active mode is about getting it to run without a manual listen-step from you. It's intended to be useful once your foreign language code has been written, and is doing something useful you'd like to use in a chain. 

Consider this example: ([full config](https://github.com/gadgetron/gadgetron/blob/master/gadgets/examples/config/external_python_acquisition_example.xml))

```xml
        <external>
            <execute name="gadgetron.examples" target="recon_acquisitions" type="python"/>
            <configuration/>
        </external>
```

The `execute` directive takes a number of arguments, the semantics of which will vary slightly between foreign language. In all cases, however, they answer the question 'What should I run?' as posed by Gadgetron. 

Starting from the rear, the `type` specifies which foreign language is to be invoked. The example specifies Python. The `target` is the name of the function that is to be invoked. This function will be called with a `Connection` object as soon as possible. The `name` is the name of the package (or module, etc.) in which Gadgetron is to find the target.

The example above will thus invoke the `gadgetron.exmaples.recon_acquisitions` function, that ships with the Python foreign language interface. 

Some languages (notably Matlab) does not need the package separately specified. Just putting a fully qualified name in `target` will suffice. 

## Configuration

The `external` node allows you to specify some configuration for your foreign code, analogous to the parameters some Gadgets take. The `configuration` found in the `external` node is forwarded to the foreign language interface, and is available through the `Connection` object. As is the ISMRMRD XML header describing the data.

They can usually be accessed as `connection.config` and `connection.header` respectively. 

## Readers and Writers

Communicating with the foreign language through an ISMRMRD stream imposes certain restrictions. Most obviously, everything sent must be flattened into a sequence of bytes. 



