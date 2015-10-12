# WhiskiWrap
Wrapper around whiski for multiprocessing, python, hdf5

This module provides tools for making whiski (whiskertracking.janelia.org) more efficient and easy to use.

Some of the main issues with whiski are:
1) Very particular about the input file format. I think this is because it relies on libav. I have found that it can only read files encoded with mpeg4 or tiff stacks. mpeg4 is not an ideal codec because it does not support lossless encoding and is quite large.
2) Not parallelized, so tracing takes a very long time.
3) The output file formats (.whiskers, .measurements) do not reliably transfer between operating systems.

The key idea of WhiskiWrap is to split the input video into many smaller chunks, run whiski on each chunk, and stitch the results together. This means that we can give whiski exactly the format that it wants and parallelize its running.

Overall flow:
1) Split the entire video into epochs of about 100K frames (~100MB of data). We can load this much data without running out of memory or taking up too much space with temporary files. 
2) For each epoch:
   a) Split it into chunks of about 1000 frames, each of which will be traced separately.
      (Optional) Crop each frame here
   b) Write each chunk to disk as a tiff stack (note: these files are quite large).
   c) Trace each chunk with whiski. This step is done with ~4 parallel processes.
      This generates a whiskers file for each chunk.
   d) Combine the results of each chunk into an HDF5 file
   e) (Optional) delete the intermediate chunk files here
3) Finally we combine the results from each epoch.


Requirements:
ffmpeg
pip uninstall numpy
conda create -n whiski_wrap python=2.7 pip numpy matplotlib pyqt tables pandas ipython
source activate whiski_wrap
pip install tifffile
git clone WhiskiWrap
echo "/home/chris/dev" > ~/.local/lib/python2.7/site-packages/chris.pth

Installing whiski:
Option 1: Build from source. Requires various dependencies (gl.h) to build so could be troublesome.
Once it is built, we need to make the python files into modules so we can import them.
cd ~/dev
git clone https://github.com/cxrodgers/whisk.git # already made modules
cd whisk
mkdir build
cmake ..
make
And we need to copy a library into an expected location
cp ~/dev/whisk/build/libwhisk.so ~/dev/whisk/python
Hopefully this directory should be above the built-in trace

Option 2: Pre-built binary
The problem with this way is that we don't have access to my changes, which is only useful for Debug_Load_whiskers and __init__ stuff
So, install binary to folder ~/whisk-1.1.0d-Linux
But then we need to make it a module so that we can import the python files
touch ~/whisk-1.1.0d-Linux/share/whisk/__init__.py
touch ~/whisk-1.1.0d-Linux/share/whisk/python/__init__.py
echo "/home/chris/whisk-1.1.0d-Linux/share" >> ~/.local/lib/python2.7/site-packages/chris.pth
Now we can: from whisk.python import traj, trace
