Introduction
============

MSBWT is a package for combining strings from sequencing into a data structure known as the multi-string BWT (MSBWT).  
This structure allows for querying a k-mer in O(k) time regardless of how many strings are present in the MSBWT.  This
particular package was created originally for merging MSBWTs after creation.  In short, this allows for multiple data
sets to be combined into a single structure which allows for queries over both data sets simultaneously.

Included in this package are implementations of MSBWT creation algorithms as described by Bauer et al. in "Lightweight
BWT construction for very large string collections" for different file types.  Furthermore, the algorithm for merging
these MSBWTs from Holt and McMillan in "Merging of Multi-String BWTs with Applications" is also implemented.

Currently, some rudimentary query utilities are also in place on the command line interface.  However, we recommend to 
anyone wishing to perform some dynamic/interactive queries to use the API provided by the source code.

For detailed usage, please type after installation:

	msbwt -h

Wiki Pages
==========

Wiki pages for common use cases of the MSBWT can be found on our github page: https://github.com/holtjma/msbwt/wiki

References
==========

Holt, James, and Leonard McMillan. "Merging of multi-string BWTs with applications." Bioinformatics (2014): btu584.

Holt, James, and Leonard McMillan. "Constructing burrows-wheeler transforms of large string collections via merging." 
Proceedings of the 5th ACM Conference on Bioinformatics, Computational Biology, and Health Informatics. ACM, 2014.

System Requirements
===================

Msbwt and its modules have been tested under Python 2.7.  The package is distributed with both Cython and the 
corresponding C files.  We recommend using installing Cython prior to installing this package.

Two Python modules are required to run the code.

[pysam] - Tested with pysam 0.7.4

As a wrapper of Samtools, the pysam module facilitates the manipulation of SAM/BAM files in Python. Its latest 
package can be downloaded from:

	http://code.google.com/p/pysam/


[argparse] - Tested with argparse 1.2.1

The argparse module is used to parse the command line arguments of the module. It has been maintained in Python 
Standard Library since Python 2.7.  Its latest package can be downloaded from:

	http://code.google.com/p/argparse/

[numpy] - Tested with numpy 1.10.1

The numpy module provide various fast numerical functions for Python.  Its latest package can be downloaded from:

	http://www.numpy.org

Installation
============

It is recommended to use easy-install (http://packages.python.org/distribute/easy_install.html) for the 
installation.

	easy_install msbwt

Alternatively, users can download the tarball of source from

	https://github.com/holtjma/msbwt

and then type:

	easy_install msbwt-<version>.tar.gz

By default, the package will be installed under the directory of Python dist-packages, and the executable of 
msbwt can be found under '/usr/local/bin/'.

If you don't have permission to install it in the system-owned directory, you can install it in locally following 
the next steps:

(1) Create a local package directory for python:

	mkdir -p <local_dir>

(2) Add the absolute path of <local_dir> to the environment variable PYTHONPATH:

	export PYTHONPATH=$PYTHONPATH:<local_dir>

(3) Use easy_install to install the package in that directory:

	easy_install -d <local_dir> msbwt-<version>.tar.gz

For example, if you want to install the package under the home directory in
a Linux system, you can type:

	mkdir -p /home/$USER/.local/lib/python/dist-packages/
	export PYTHONPATH=$PYTHONPATH:/home/$USER/.local/lib/python/dist-packages/
	easy_install -d /home/$USER/.local/lib/python/dist-packages/ msbwt-<version>.tar.gz

After installation, msbwt will be located in '/home/$USER/.local/lib/python/dist-packages/'.


Detailed Description
===========

The msbwt package is primarily focused on creation, merging, and querying of MSBWTs.  To perform a query, several
data structures are useful and stored with the MSBWT.  To handle this, each MSBWT is grouped into a directory which
contains standardized filenames.  Here is the description of what each file contains:

 * seqs.npy - Contains the raw input strings in a sorted, numerical format. It can be safely deleted after msbwt.npy has been fully instantiated.
 * offsets.npy - Contains meta-data regarding seq.npy.  For strings of uniform length, this file is very small. If strings have variable size, it will be O(S) where S is the number of strings.  This file can be safely deleted after msbwt.npy has been fully instantiated.
 * about.npy - Contains information regarding the origin of the sorted strings to be stored in the MSBWT.  For example, if multiple FASTQ files are used for input, this file includes both the source file and which read it was in that file.  For paired end reads, this can be useful for finding mates. If deleted, this file is NOT recoverable from msbwt.npy.
 * msbwt.npy - Contains the full, uncompressed MSBWT.  Each symbol in the MSBWT takes one byte of space.  If deleted, the MSBWT is NOT recoverable except from comp_msbwt.npy.
 * fmIndex.npy - Contains a sampled FM-index for msbwt.npy.  This is derived from msbwt.npy and must be created for any queries to work.  If deleted, the file will be automatically re-created if a query is performed on the data set.
 * comp\_msbwt.npy - Contains the compressed MSBWT.  Each byte contains 3 bits for the symbol and 5 bits for a count. Each run is compressed into a symbol/length pairs and lengths too big for 5 bits expanded to multiple bytes such that one byte contains lengths < 32, two bytes < 32^2, three bytes < 32^3, and so on for as many bytes are necessary to store the run length.  If deleted, this file is NOT recoverable except from msbwt.npy.
 * comp\_fmIndex.npy	- Contains a sampled FM-index for comp\_msbwt.npy.  This is derived from comp\_msbwt.npy and must be created for any queries to work.  If deleted, this file will be automatically re-created if a query is performed on the data set.
 * comp\_refIndex.npy - Contains offset information for the sample FM-index.  This is derived from com\p_msbwt.npy and must be created for any queries to work.  If deleted, this file will be automatically re-created if a query if performed on the data set.

The msbwt package is broken down into various sub-functions which include MSBWT creation, merging, and querying.  Below is a
list of each along with the expected output.

 * cffq - Create From FASTQ.  This function takes a list of input FASTQ files and will create the MSBWT from the strings contained in them to be stored in the end result of msbwt.npy.  This is functionally the same as doing 'pp' followed by 'cfpp'.  Note that strings of uniform length can be processed faster than strings of non-uniform length.
 * pp - Pre-Process.  This function takes a list of input FASTQ files and will create the seqs.npy, offsets.npy, and about.npy files for those FASTQ files.  Note that strings of uniform length can be processed faster than strings of non-uniform length.
 * cfpp - Create From Pre-Process.  This function takes a pre-processed directory and creates the MSBWT from the strings contained in seqs.npy to create msbwt.npy.
 * merge - This function takes a list of input MSBWT directories and merges them into a single output MSBWT.  The resulting BWT will always be stored in msbwt.npy of the output directory.
 * query - This function is for performing a single query of a specific k-mer within the data.  Options exist for dumping the associated strings containing that k-mer as well.
 * massquery - This function takes a list of queries in a line separated file and performs those searches storing counts for each query in a CSV file.
 * compress - This function takes an uncompressed MSBWT and converts it to the compressed version.  The auxiliary FM-index will need to be reconstructed for the new data format.
 * decompress - This function takes a compressed MSBWT and converts it back into its original uncompressed format. The auxiliary FM-index will need to be reconstructed for this format.
 * convert - This function converted a raw text input BWT to our run-length encoded version.

The latest release also includes installation of the Cython library used to load the MSBWTs and perform merges.  After 
installation, the libraries are available through regular Python imports like so:

	import MUSCython.MultiStringBWTCython as MultiStringBWT
	msbwt = MultiStringBWT.loadBWT('/path/to/directory')
	msbwt.countOccurrencesOfSeq('CAT')

Other commands are available as well. For more information, refer to the code available at https://github.com/holtjma/msbwt.

References
==========
Holt, James, and Leonard McMillan. "Merging of multi-string BWTs with applications." *Bioinformatics* (2014): btu584.

Holt, James, and Leonard McMillan. "Constructing Burrows-Wheeler transforms of large string collections via merging." *Proceedings of the 5th ACM Conference on Bioinformatics, Computational Biology, and Health Informatics.* ACM, 2014.