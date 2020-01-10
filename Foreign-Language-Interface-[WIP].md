While Gadgetron is written in C++, users will frequently prefer to work in another programming language - Matlab and Python are common choices. 

The Foreign Language Interface provides a uniform way to have foreign (non-C++) code as part of a Gadgetron reconstruction chain. This document will describe how the Foreign Language Interface works, and provide a step-by-step guide to get you started. 

## Overview

The Foreign Language Interface uses an ISMRMRD stream to communicate with a foreign language (or runtime). The very same kind of stream used by the client to talk to the Gadgetron server, in fact. In this way, data can flow between Gadgetron and any compatible ISMRMRD server. 

Thus, the Foreign Language Interface consists of two major components: A Gadgetron `External` node, playing the 
part of an ISMRMRD client; and an ISMRMRD server written in your language of choice. 

There are no restrictions on the foreign server (besides ISMRMRD compatibility). It can run in a process of your choosing, on a different machine, in a debugger, container, or wherever else you find it convenient. The Foreign Language Interface is very flexible by design. 

Similarly, the use of an ISMRMRD stream means that all data transmitted must be collapsed into a sequence of bytes during transmission (a process handled by a Reader/Writer pair). This requires communication to feature well defined units of data, that are meaningful on their own. It also let's the foreign language present input to (and expect output from) users in a manner that makes sense within the foreign language, with no concern for the data's representation within Gadgetron. 

Both the Matlab- and Python Foreign Language Interface expose the data from Gadgetron through a `Connection` object. This will let you iterate through input, and return output to Gadgetron. The two languages are different, so the exact functionality varies between the two implementation, but the abstract concept modelled is the same. The following examples will feature Matlab and Python code side by side.  

[comment]: # (I've put the illustration in a personal repository. Dunno if that's a good solution, but at least you can get at them if you need to make changes. -k)

![Foreign Language Interface Overview](https://raw.githubusercontent.com/kristofferknudsen/gadgetron-illustrations/master/external.svg)

In simple terms, any interface to Gadgetron in a foreign language is a simple ISMRMRD server, exposing a sensible interface to the user, as well as a few plumbing details (to facilitate getting started by Gadgetron, etc.). 

## Getting Started

To get started with the Foreign Language Interface, you need a working [[Gadgetron Installation]], and a foreign language of your choice (At the time of writing, Python or Matlab). 

### Installation

The first thing we must do is install the Gadgetron interface for our chosen language. 

For Python, the Gadgetron interface is available as a pip package:
```bash
$ pip3 install gadgetron
```

For Matlab, the Gadgetron interface is available as a Matlab toolbox. Simply search for 'gadgetron', and it should show right up. 

And that's it, we're good to go! 

### Verification

To make sure everything is in working order, we'll run a Gadgetron integration test using our foreign language of choice. Each foreign interface comes with a couple of examples, that each double as a Gadgetron integration test.

Running a Python test looks something like this: (I trust you to make appropriate adjustments for Matlab yourself) 
```bash
$ cd path/to/gadgetron/test/integration 
$ ./run_tests cases/external_python_acquisition_example.cfg
```
### Running Code

Having run the test, the next step is to run some code of our own. For that, we'll need some code to run. We'll start with something simple: a do-nothing function.

Here's a suitable Matlab function:
```matlab
function handle_connection(connection)
    disp("Hello, world!")
end
```

The same function in Python:
```python
def handle_connection(connection):
    print("Hello, world!")
```


## Modes

Gadgetron supports two different modes of interaction with foreign languages (sometimes referred to as external modules). One is active, where Gadgetron will take charge, start your language (interpreter, runtime, etc.), call your function with a connection, and end the process when done. The other is passive, where Gadgetron simply connects to a server you started - you just need to tell it where to find it. 

### Passive - Listen/Connect

### Active - Execute

## The `Connection` Object
### Receiving Items
### Sending Items
### Accessing ISMRMRD Headers
### Filters

## Readers and Writers

Communicating with the foreign language through an ISMRMRD stream imposes certain restrictions. Most obviously, everything sent must be flattened into a sequence of bytes. 
