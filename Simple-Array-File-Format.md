When working with the Gadgetron it is often necessary to write files with reconstructed images to disk, either as part of debugging or as the final reconstruction result. We have adopted a very simple multidimensional array file format for this purpose. The main advantage of this file format is its simplicity but there are a number of disadvantages and caveats as well as described in this section.

The simple array files are made up of a) a header followed by b) the data itself. This layout of data and header is illustrated below. The header has a single 32-bit integer to indicate the number of dimensions of the dataset followed by one integer for each dimension to indicate the length of that dimension. The data follows immediately after the header. The data is stored such that the first dimension is the fastest moving dimension, second dimension is second fastest, etc. The header contains no information about the size of each individual data element and consequently the user needs to know what type of data is contained in the array. In general, the Gadgetron uses 3 different types of data and the convention is to use the file extension to indicate the data type in the file:

-   16-bit unsigned short. File extension: `*.short`

-   32-bit float. File extension: `*.real`

-   32-bit complex float. Two 32-bit floating point values per data element. File extension: `*.cplx`

<img src="https://s3.amazonaws.com/gadgetron.github.io/figs/arrayfileformat.png" style="width: 400px;" />

The Gadgetron framework provides function for reading these files in C++. The functions are located in `toolboxes/ndarray/hoNDArray_fileio.h` in the Gadgetron source code distribution.

It is also trivial to read the files into Matlab. Below is a function which detects the data type based on the file extension and reads the file into Matlab.


    function data = read_gadgetron_array(filename)
    %  data = read_gadgetron_array(filename)
    %  
    %  Reads simplified array format output from the Gadgetron
    %
    %  The datatype is determined by the file extension.
    %     - *.short : 16-bit unsigned integer
    %     - *.real  : 32-bit float
    %     - *.cplx  : 32-bit complex (two 32-bit values per data element)
    %
    %
    if (~exist(filename,'file')),
        error('File not found.');
    end

    [path,name,ext] = fileparts(filename);

    ext = lower(ext);

    if (~strcmp(ext,'.short') && ~strcmp(ext,'.real') && ~strcmp(ext,'.cplx')),
       error('Unknown file extension'); 
    end

    f = fopen(filename);
    ndims = fread(f,1,'int32'); 
    dims = fread(f,ndims,'int32'); 

    switch ext
        case '.short'
            data = fread(f,prod(dims),'uint16'); 
        case '.real'
            data = fread(f,prod(dims),'float32'); 
        case '.cplx'
            data = fread(f,2*prod(dims),'float32'); 
            data = complex(data(1:2:end),data(2:2:end));
        otherwise     
    end

    fclose(f);

    if numel(dims)>1
        data = reshape(data,dims');
    end

    end

      
