
To get started with the [[Foreign Language Interface]], you need a working [[Gadgetron Installation]]. 

## Installation

To use Matlab with Gadgetron, we must first install the Matlab Gadgetron interface. It's a foreign language interface for Gadgetron - in essence just a ISMRMRD server, with a few extra features. 

The interface is available as a toolbox for Matlab - you can find it by searching for 'gadgetron' in the Add-On manager. 

Once installed, we're good to go! 

## Verification

To make sure everything is in working order, we'll run a Gadgetron integration test. The Matlab foreign language interface comes with a couple of examples that each double as a Gadgetron integration test.

Running a Matlab test looks something like this: 
```bash
$ cd path/to/gadgetron/test/integration 
$ ./run_tests.py cases/external_matlab_acquisition_example.cfg
```

The test should run and pass, with output that looks like this: 
```
Test 1 of 1: cases/external_matlab_acquisition_example.cfg

Running Gadgetron test cases/external_matlab_acquisition_example.cfg with:
  -- ISMRMRD_HOME    : None
  -- GADGETRON_HOME  : None
  -- TEST CASE       : cases/external_matlab_acquisition_example.cfg
Starting Gadgetron instance on port 9003
Converting Siemens data: data/simple_gre/meas_MiniGadgetron_GRE.dat -> test/in.h5
Passing data to Gadgetron: test/in.h5 -> test/out.h5
Gadgetron processing time: 6.90 s
TEST                       [OK] (Norm: 1.5e-07 [0.0001] Scale: 0.0e+00 [0.0001])
Test status: Passed

1 tests passed. 0 tests failed. 0 tests skipped.
Total processing time: 6.90 seconds.
```

## Running Code

Having successfully run an integration test, the next step is to run some code of our own. Grab a Matlab live script, and let's get started. 

First of all, we'll need a function to handle the connection from Gadgetron. We'll start with something simple; a do-nothing function.  

We'll also direct the Gadgetron interface to listen for incoming connections from Gadgetron on port 18000, and call our do-nothing function when a connection is established. This is done using the `gadgetron.external.listen` call. 

```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    
    % TODO: Useful work.
end
```
Run the script. You should see Matlab being busy - it's listening for connections. 

Once Matlab is listening, it's time to run a Gadgetron reconstruction. We'll run a reconstruction with a passive `external` node, i.e. a reconstruction that will connect to a foreign language. You'll most likely tailor a reconstruction chain to your own needs, but we'll start with [this](https://github.com/gadgetron/gadgetron/blob/master/gadgets/examples/config/external_connect_example.xml) simple example. It's a chain that literally does nothing besides opening a connection to Matlab, forwarding all the data. 

To run the reconstruction chain, you'll need to run Gadgetron, and the Gadgetron ISMRMRD client: 

Start Gadgetron:
```bash
$ gadgetron
```

Run the ISMRMRD client: 
```bash 
$ cd path/to/gadgetron/test/integration
$ gadgetron_ismrmrd_client -p 9002 -f test/in.h5 -o test/out.h5 -c external_connect_example.xml
```

Here I'm using the data from the test case we ran earlier as input. 

Upon running the Gadgetron ISMRMRD client, you should see a connection in the Gadgetron log. Mine looks like this: 
```
01-21 12:07:08.017 INFO [Server.cpp:40] Accepted connection from: ::ffff:127.0.0.1
01-21 12:07:08.017 INFO [ConfigConnection.cpp:112] Connection state: [CONFIG]
01-21 12:07:08.017 DEBUG [ConfigConnection.cpp:54] Reading config file: "/home/princess/.gadgetron/share/gadgetron/config/external_connect_example.xml"
01-21 12:07:08.017 INFO [HeaderConnection.cpp:82] Connection state: [HEADER]
01-21 12:07:08.017 INFO [StreamConnection.cpp:75] Connection state: [STREAM]
01-21 12:07:08.017 DEBUG [Stream.cpp:64] Loading External Connect block on port 18000 
01-21 12:07:08.018 INFO [External.cpp:53] Connecting to external module on address: localhost:18000
01-21 12:07:08.191 INFO [Core.cpp:76] Connection state: [FINISHED]
```

More significantly, the listen call in the live script has returned, and has produced the following output: 
```
Starting external MATLAB module 'handle_connection' in state: [PASSIVE]
Waiting for connection from client on port: 18000
handle_connection was called.
```

Note that there is no output yet - maybe not surprising, as the do-nothing handler does little useful work. Let's correct that. 

## Consuming Input and Producing Output

In order to do useful work, we'll need to process our input, and maybe produce some output. Let's discuss input first. 

The `Connection` object - the first argument to the `handle_connection` function when invoked by listen - features a `next` method. Calling next will read an item off the stream (perhaps waiting for an item to become available), and return it to the caller. Data from Gadgetron is accessed by repeatedly calling `next`. 

Modifying our live script, we can have a look:
```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    
    % TODO: Useful work.
    
    connection.next() % Get the first item sent from Gadgetron; an Acquisition.
    connection.next() % Get the second item sent from Gadgetron; another Acquisition.
end
```

Re-run the script, and re-run the ISMRMRD client, and you should see a few acquisitions in the live script output. (I'll not reproduce it here - it takes up quite a few lines)

Calling next will produce items one at a time, until the `Connection` runs dry. When there are no more items to get, calling `next` will produce an exception. 

For this example, we'll be calling next until we have received all the acquisitions making up a slice. We'll assemble them in a single buffer, do an inverse fft, and return the image to Gadgetron (who will then return it to the client). 

Note that these are well-defined independent steps. The inverse fft operates on a simple array, and need not concern itself with how this array is assembled. Likewise, returning an image to Gadgetron need not concern itself with how an image is produced. 

To leverage this independence into clean and composable units, I'll be implementing each in it's own function, using closures for state management. One could also leverage objects to this end, but I find Matlab OO programming cumbersome, and prefer a more functional style. 

### Assembling a Slice

At this point, we only really have the `Connection` to work with. Calling `next` will produce Acquisitions - each a line in k-space. We'll assemble these into a single buffer, and pass it on to the reconstruction. 

Let's start out by writing a function that accumulates the acquisitions in a slice. We'll add it to the live script:

```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    

    % Call produce_slice - have a look at the output.
    produce_slice(@connection.next)    
end

function slice = produce_slice(input)
    
    acquisitions = {}; % Accumulate acquisitions in a cell array. 
   
    while true
        acquisition = input(); % Call input function to produce the next acquisition. 
        acquisitions{end + 1} = acquisition; % Save the acquisition in the cell array. 
        
        % Consider the acquisition flags; is the acquisition last in slice?
        if acquisition.is_flag_set(acquisition.ACQ_LAST_IN_SLICE), break; end
    end
    
    slice = acquisitions;
end
```
I trust you to follow the code above. `produce_slice` is a fairly simple function. We initialize a cell array to hold the acquisitions making up a slice. Then we loop, calling our `input` function to produce new acquisitions. They are written to the cell array, and examined. If we reach an acquisition flagged as last in slice, we end the loop and return the acquisitions. 

Running the code above (rerun the script; rerun the ISMRMRD client) a cell array containing 129 acquisitions is produced. We could call the function again, and get more slices. 

To finish the job, we must now assemble the acquisition data in a single k-space buffer. We'll write a short function that does this (it's essentially just a call to `cat`):

```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    

    next_acquisition = @connection.next;
    next_slice = @() produce_slice(next_acquisition);
        
    kspace = next_slice()
end

function slice = produce_slice(input)
    
    acquisitions = {}; % Accumulate acquisitions in a cell array. 
   
    while true
        acquisition = input(); % Call input function to produce the next acquisition. 
        acquisitions{end + 1} = acquisition; % Save the acquisition in the cell array. 
        
        % Consider the acquisition flags; is the acquisition last in slice?
        if acquisition.is_flag_set(acquisition.ACQ_LAST_IN_SLICE), break; end
    end
    
    function slice = assemble_slice(acquisitions)
        data = cellfun(@(acq) acq.data', acquisitions, 'UniformOutput', false);
        slice = cat(3, data{:}); % Stack the acquisition data. 
    end

    slice = assemble_slice(acquisitions);
end

```
Running the above should output a nice k-space buffer. 

Note that I've begun setting up a sequence of `next_` functions when handling the connection. As we are now able to produce slices, I've introduced a `next_slice` function to be used as input when reconstructing the image.

The independent steps of the reconstruction process will live in different functions, knowing nothing of each other. They'll be composed in the top level handle function, and be anonymously called to produce output successively closer to the desired product.  

### Reconstructing the Image

To reconstruct the image, we'll simply perform an inverse fft. We'll add a function to do this, of course. We'll also combine the channels - a sum of squares will do. 

```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    

    next_acquisition = @connection.next;
    next_slice = @() produce_slice(next_acquisition);
    next_image = @() reconstruct_image(next_slice);
   
    image = next_image()
    imshow(image, []);    
end

function slice = produce_slice(input)
    
    acquisitions = {}; % Accumulate acquisitions in a cell array. 
   
    while true
        acquisition = input(); % Call input function to produce the next acquisition. 
        acquisitions{end + 1} = acquisition; % Save the acquisition in the cell array. 
        
        % Consider the acquisition flags; is the acquisition last in slice?
        if acquisition.is_flag_set(acquisition.ACQ_LAST_IN_SLICE), break; end
    end
    
    function slice = assemble_slice(acquisitions)
        data = cellfun(@(acq) acq.data', acquisitions, 'UniformOutput', false);
        slice = cat(3, data{:}); % Stack the acquisition data. 
    end

    slice = assemble_slice(acquisitions);
end

function image = reconstruct_image(input)
    image = input(); % Call input function to produce the next slice. 
    image = gadgetron.lib.fft.cifftn(image, [2, 3]); % Perform centered inverse fft.    
    image = squeeze(sqrt(sum(abs(image) .^ 2, 1))); % Combine the channels; sum of squares.
end
```

Running the above should yield an image. If you're still using the data from the integration test, you should see an image of a phantom - circular with square impressions. 

### Returning the Image to Gadgetron

Having reconstructed an image, we'll need to return it to Gadgetron (which will then immediately return it to the client). 

To return an image to Gadgetron, we'll need to call the `connection.send` method. This method examines it's input, and selects and appropriate writer based on it's types and/or attributes. In order to send an image back to Gadgetron, we'll initialize an image object (of type `gadgetron.types.Image`), and send it. 

There are multiple ways to initialize an image. We'll use the static initializer `gadgetron.types.Image.from_data`, as it's very convenient. It takes data (the array we already have is suitable), and some reference data. The reference data is just an Acquisition header, used to initialize the image meta-data. 

We'll add a function to initialize the `Image` object, and carry an acquisition header along with the data so we can initialize the image. 

```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    

    next_acquisition = @connection.next;
    next_slice = @() produce_slice(next_acquisition);
    next_image = @() reconstruct_image(next_slice);
    next_image = @() initialize_image_object(next_image);
   
    image = next_image()
    imshow(squeeze(image.data), []);    
end

function slice = produce_slice(input)
    
    acquisitions = {}; % Accumulate acquisitions in a cell array. 
   
    while true
        acquisition = input(); % Call input function to produce the next acquisition. 
        acquisitions{end + 1} = acquisition; % Save the acquisition in the cell array. 
        
        % Consider the acquisition flags; is the acquisition last in slice?
        if acquisition.is_flag_set(acquisition.ACQ_LAST_IN_SLICE), break; end
    end
    
    function slice = assemble_slice(acquisitions)
        data = cellfun(@(acq) acq.data', acquisitions, 'UniformOutput', false);
        slice.data = cat(3, data{:}); % Stack the acquisition data. 
        slice.reference = acquisitions{end}.header; % Save an acquisition header as reference data.
    end

    slice = assemble_slice(acquisitions);
end

function image = reconstruct_image(input)
    image = input(); % Call input function to produce the next slice. 
    image.data = gadgetron.lib.fft.cifftn(image.data, [2, 3]); % Perform centered inverse fft.    
    image.data = sqrt(sum(abs(image.data) .^ 2, 1)); % Combine the channels; sum of squares.
end

function image = initialize_image_object(input)
    image = input(); % Call input function to produce the next reconstructed image.
    image = gadgetron.types.Image.from_data(image.data, image.reference);
end
```

Notice how we are layering useful steps to assemble a reconstruction process. Running the code above, we now produce fully fledged `Image` objects, ready to be shipped back to Gadgetron. We just need to call `connection.send`, until we run out of input. 

Because 'consume until exception' is a common pattern when using the Gadgetron Matlab interface, the `consume` function is provided to make this painless. `consume` will call a function repeatedly, until an exception is thrown. `consume` will then quietly return. 

We'll use it to send the images back to the client: 
```matlab
% Listen for incoming connections on port 18000. Call 'handle_connection'
% when a connection is established. 
gadgetron.external.listen(18000, @handle_connection);

function handle_connection(connection)
    disp("handle_connection was called.")    

    connection.next(); % Discard the first acquisition - it's noise data.
    
    next_acquisition = @connection.next;
    next_slice = @() produce_slice(next_acquisition);
    next_image = @() reconstruct_image(next_slice);
    next_image = @() initialize_image_object(next_image);
    send_image = @() connection.send(next_image());

    tic; gadgetron.consume(send_image); toc    
end

function slice = produce_slice(input)
    
    acquisitions = {}; % Accumulate acquisitions in a cell array. 
   
    while true
        acquisition = input(); % Call input function to produce the next acquisition. 
        acquisitions{end + 1} = acquisition; % Save the acquisition in the cell array. 
        
        % Consider the acquisition flags; is the acquisition last in slice?
        if acquisition.is_flag_set(acquisition.ACQ_LAST_IN_SLICE), break; end
    end
    
    function slice = assemble_slice(acquisitions)
        data = cellfun(@(acq) acq.data', acquisitions, 'UniformOutput', false);
        slice.data = cat(3, data{:}); % Stack the acquisition data. 
        slice.reference = acquisitions{end}.header; % Save an acquisition header as reference data.
    end

    slice = assemble_slice(acquisitions);
end

function image = reconstruct_image(input)
    image = input(); % Call input function to produce the next slice. 
    image.data = gadgetron.lib.fft.cifftn(image.data, [2, 3]); % Perform centered inverse fft.    
    image.data = sqrt(sum(abs(image.data) .^ 2, 1)); % Combine the channels; sum of squares.
end

function image = initialize_image_object(input)
    image = input(); % Call input function to produce the next reconstructed image.
    image = gadgetron.types.Image.from_data(image.data, image.reference);
end
```
There we are. The data contains 10 slices; each will be reconstructed in Matlab and returned to the client. 

Having followed along, you should find the Matlab examples shipping with the Gadgetron interface familiar. You can find them [here](https://github.com/gadgetron/gadgetron-matlab/tree/master/%2Bgadgetron/%2Bexamples), in case you'd like to take a look. 

