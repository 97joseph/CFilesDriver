# CFilesDriver
 A simple driver implementation
 
Let's put a file system into a file!

Using the descriptions in the sections that follow, implement FileFS in C, using any standard libraries. The file system is not to reside in a partition on a disk, but rather in a single fixed-size file. The interface to your file system will be a program with command-line options for writing, reading, removing, and listing files contained in the file system.

Formatting

The fixed-size file that is acting as a stand-in for a disk partition will need to be formatted to provide some basic information about your file system each time it is opened by your program. The formatting will result in a superblock, free list, and inodes being written to your fixed-sized file. The superblock defines the file system, specifying the number of blocks available in the file system, the size and number of inodes, and potentially other useful information. The most straightforward free list might be implemented as a bit map of sufficient size to provide a bit for each block in the file system, specifying whether the corresponding block is free or used. Inodes should initially be unused. The blocks that are used for file data storage will appear after the superblock, free list, and inodes. No action is required for the blocks themselves, since the only way they might be read or written is when the file system structures like the free list or inodes refer to them. Make the size of your file system 10MB. Do not turn your file system storage file in with your solution, I'll create my own to test with.

You can think of the file your file system will reside in as having roughly this structure:


<- 0 byte                                                   10MB  ->

+------------------------------------------------------------------+

|              |  Free   |        |                                |

|  Superblock  |  block  | Inodes | File system data ...           |

|              |  list   |        |                                |

+------------------------------------------------------------------+


Internal File Types

Your file system should support two file types, directories and
regular files. Directories are basically implemented as regular files (they are accessed via their own inode)
that are labeled as directories and whose data blocks contain only directory
entries (filename, type, size, etc.).  Don't worry about compacting directories as entries are added
and removed, instead think about marking directory entries that belong
to removed files as invalid, and potentially reusing them when a new
file is written to that directory. No format should be assumed for
regular files, you are only expected to be able to read and write
these in their entirety. Keep in mind that files (and the files we
call directories) will commonly span more than one block, and that the
blocks need not necessarily be contiguous. 

Inodes

Your inodes are to contain 100 direct block references and one single
indirect block (meaning that your file system should support the
writing files that are bigger than 51200 bytes). You are not reproducing a user environment, so you also don't have to store the user, group and permissions information. I also don't care about time stamps. Just focus on what will allow you to read and write a file in its entirety. Each block that the inode points to should be 512 bytes (your file system will apparently only handle smallish files). I will not be testing with more than 100 files, so have at least 100 inodes in your file system. Each inode should fit into a single 512 byte block.

Functionality

Your program should allow files to be added to your file system, read and removed from it, and for the contents of the file system to be listed. It might be easiest to describe the intended functionality by providing examples of each. For example, the following command line invocation should list the contents of the example file system starting with the root directory:

./filefs -l -f example

The output might be a file system tree, showing the names of each directory and file contained within your file system (think about recursing through it starting with the root, adding a number of leading tabs to each line of output equivalent to the depth of recursion).

The following invocation would add the specified file to your file system (as well as each directory in the path):

./filefs -a /a/b/c -f example

In this example, /a/b/c is the absolute path to an existing regular file c in /a/b/. The intended result is that your file system will likewise contain directory a, subdirectory b, and file c, with c’s contents being identical to the specified file’s. So, you’d find some free inodes and blocks and write directory entries to both /a and /a/b, then get another free inode for c, and copy c into your file system, repurposing free blocks and updating the block mapping in its inode as you perform the copy.

The following invocation would remove the specified file from your file system (as well as any directory that becomes empty after you remove the specified file):

./filefs -r /a/b/c -f example

So, if the only file in your file system is /a/b/c and you remove it as above, then listing the contents of the file system should result in nothing being printed, since both a and a/b would have been removed after c was removed.

The following invocation would read the contents of the specified file to stdout, then redirect that output to file foobar.

./filefs -e /a/b/c -f example > foobar

Reading a file’s contents in this manner should not result in the file being removed from your file system. There should be no difference between the original file and the data returned by your file system.

The following invocation will print each inode and contents of each file in the path. This ought to be very useful for debugging your code. In case of directories, print the name of each file and the associated inode’s number, in case of files, just print that it is a regular file.

./filefs -d /a/b/c -f example

We might see something like the following as a result:

directory ’/’:

’a’ 2

’d’ 5

’e’ 6

directory ’a’:

’b’ 3

’f’ 7

directory ’b’:

’c’ 4

Hard requirements (solutions not meeting all of these requirements are automatically earn 0 points)

Your solution must be implemented according to the specifications described in this document.

File and directory names are stored only in directory files, they do not appear in inodes.

Writing a new file requires an entry to be written in the appropriate directory along with a number identifying its inode. As free blocks are selected to store the file's data, block identifiers are stored in the file's inode. 

Extracted file data will be written to stdout. 

Your file system should be self-contained; i.e. metadata and data comprising the file system should be inside of a single file, formatted according to your interpretation of the specifications. Files are added and read directly from the storage file.

Your file system should correctly store any kind of input file, text or binary.

Your file system should be able to support paths or filenames up to 255 characters total.

A file stored in your file system should, when extracted, match the original file (call diff on the original and extracted file to make sure there are no differences).

Include an up-to-date Makefile. I will not reproduce your build from nothing.

Include a README file that contains any information that would be useful for grading: testing, configuration, compiling, etc. (This would be a good place to describe 
what works, what doesn’t, and how the stuff the works does what it does).

Your solution should compile and run on os.cs.siue.edu. I will not reproduce your environment.
