# BFI checksum_scripts

The BFI National Archive have been running checksum speed comparisons, with an aim to reducing bottlenecks caused by an increasing volume of digital media files. One such bottleneck is caused by our use of hashlib in Python2 scripts to generate MD5 checksums for every media file before being written to LTO tape storage. We have decided to run some comparisons between CRC32 and MD5, as we currently only have support for these within our tape library system. (Ruling out the supported SHA options as we have no need for the cryptographic functionality and prefer the speed gain over that functionality).

The scripts in this repository use Python standard library zlib and hashlib to generate CRC32 and MD5 hashes respectively. They both use timeit to measure the speed that it takes to run each checksum pass. As the media files are generally many GBs in size the test repeat number is set to 1 in timeit, so the script is only run once per file, every four hours from crontab. Timeit however, was developed to create averages across multiple speed tests on short snippets of code. If you are checksum testing smaller files in a collection you can change the `number=1` setting to another figure, such as number=100 and run the test to receive an average time across those 100. Timeit's default is 1,000,000 (when no number is specified) so do ensure you set a number for checksum tests that is achievable.

There are two versions of the checksum_speed_test script that allow for single use checksum testing or automated testing of directories, and both will run on Python2.7 or Python3.

There is potential for furher expansion of these tests by altering buffersize and chunk size allocations for the checksum functions. We've not had a chance to experiment with this yet, but welcome feedback and advice about optimising hash generation.

Result from the tests, run on 8 thread 12GB Ubuntu VM with 10Gbps network connection to two different network shares: CRC32 Python 3 implementation is fastest, with MD5 Python 2 implementation slowest.

Methodology         Time per MB in Secs   MB per Second
MD5 Python 2        0.010090932           230.3371534
MD5 Python 3        0.004238077           349.0446795
CRC32 Python 2      0.004238086           362.3504839
CRC32 Python 3      0.003920059           394.4417439


## checksum_speed_test.py

This script allows for a single file to be input and tested against zlib CRC32 and hashlib MD5 modules of Python to see which is quicker. You can drag/drop a file after the python script name to make sure the path is correct.

To run the script:
`python checksum_speed_test.py /path_to_file/file.mkv`

The script performs the following functions:
1. Checks the path supplied is legitimate and present.
2. If both are True it stores sys.argv[1] (the path you supplied) as variable 'filename'.
3. Makes timeit[lambda: ] calls to the following functions supplying the filename:
  - crc(filename): Opens the file in bytes, and passes to zlib.crc32 in buffersizes of 65536, until the whole of the file
    has been checksum evaluated. Prints the CRC32 checksum to the terminal output, formatted 08x.
  - md5(filename): Opens the input file in bytes, splits the file into chunks and iterates through these (size 4096)
    until the hash file is completed. Prints the MD5 checksum, formatted hexdigest.
4. Outputs the time taken for each function, along with the input file name.


## checksum_speed_test_crontab.py

Mostly identical to checksum_speed_tests.py, but with print statements removed so runs silently and appends to a log file at a specified path. To run this script you need to edit the paths variable (line 37/38) and specify a path for your log ouput (line 30). If you're testing in both Python2 and Python3 then you can also amend the version in line 87 and 88.

The script performs the following functions:
1. Iterates through path list, stored in paths variable, and checks each path is legitimate.
2. For each path it iterates through the files within it (there is no check here for file type).
3. The script creates a filepath variable for each file, and runs a size check against it, in MB.
4. Passes the filepath to the following functions using timeit to record the duration taken.
  - crc(file): Opens the media file read only in bytes. Passes to zlib.crc32 in buffersizes of 65536 until the
    total file has been evaluated. Returns the CRC32 checksum, formatted 08x.
  - md5(file): Opens the input file in read only bytes. Splits the file into chunks, iterates through 4096
    bytes at a time. Returns the MD5 checksum, formatted hexdigest.
5. Outputs to log the following data, tab separated:
   Filepath     MD5/CRC32      Size in MB      Time taken in seconds       Python version


## test_checksum_speed_test.py

Test script for the md5 and crc functions of checksum_speed_test scripts. This is a first attempt to work with PyTest asserts. No import of pytest is required to run this script. This test script should be kept in the same directory as the script it's testing so pytest can find it, and should have the same name as the script to be tested, name appended 'test_'.

You will need to install pytest. Thes easiest way is to use pip, or pip3:

`pip install pytest`

There are two ways to run a pytest. Change directory of your terminal console to the folder that holds both the scripts and run:
`py.test -v`
This will run any/all scripts prefixed "test_".

Alternatively, from any directory run:
`pytest -v /path_to_script/test_checksum_speed_tests.py`
The -v requests verbosity, useful when you have multiple test functions within the one script.

Python 3 compliant, not tested on Python2.
