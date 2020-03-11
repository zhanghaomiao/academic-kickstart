---
title: File System Implementation
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 10
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 9
---

## File-System Structure

- I/O transfer between memory and disk are performed in units of blocks
  - one block is one or more sectors
  - one sector is usually 512 bytes
- One OS can support more than 1 FS types (NTFS, FAT32)
- Two design problems in FS
  - interface to user programs
  - interface to physical storage (disk)
- Layered File System
  <img src="https://i.loli.net/2019/03/20/5c91d9f708bcb.png" width="400px"/>

## Data Structure

### On-Disk Structure

- Boot control block (**per partition**): information needed to boot an OS from that partition
  - typical the first block of the partition (empty means no OS)
  - UFS (Unix File Sys): boot block, NTFS: partition boot sector
- Partition Control Block (**per partion**): partion details
  - details: # of blocks, block size, free-block-size, free FCB pointers
  - USF: superblock, NTFS: Master File Table
- File control block (**per file**): details regrading a file
  - details: permission, size, location of data blocks
  - UFS: inode, NTFS: stored in MFT(relational database)
- Directory structure (**pef file system**): organize files
<img src="https://i.loli.net/2019/03/20/5c91e69eb521e.png" width="400px"/>

### In-Memory Structure

- in-memory partition table: information about each **mounted parition**
- in-memory directory structure: information of **recently accessed directories**
- system-wide open-file table: contain a copy of each **opened file's FCB (file control block)**
- per-process open-file table: **pointer (file handler / descriptor)** to the corresponding entry in the above table

### Example

<img src="https://i.loli.net/2019/03/20/5c91e9048c086.png" width="400px"/>

<img src="https://i.loli.net/2019/03/20/5c91e976881de.png" width="400px"/>

- create
  1. OS allocates a new FCB
  2. update directory structure
      1. OS reads in the corresponding directory
      2. Updates the dir structure with the new file name and the FCB
      3. (After file being closed), OS writes back the directory structure back to disk
  3. the file appears in user's dir command

## Virtual File System

- VFS provides an object-oriented way of implementing file systems
- VFS allows the same system call interface to be used for different types of FS
- VFS calls the appropriate FS routines based on the partition info
  <img src="https://i.loli.net/2019/03/20/5c91eadf85fd1.png" width="400px"/>
- Four main object types defined by Linux VFS:
  - inode (an individual file, file control block)
  - file object (an open file)
  - superblock object (an entire file system)
  - dentry object (an individual directory entry)
- VSF defines a set of operations that must be implemented (e.g. for file object)
  - int open(...) (open a file)
  - ssize_t read() (read from a file)
- Directory implementation
  - Linear Lists (list of names with pointers to data blocks, easy to program but poor performance)
  - Hash table -- linear list w/hash data structure, constant time for searching, linked list for collosions on a hash entry, hash table usually has fixed number of entires

## Allocation Methods

- An allocation method refers to how disk blocks are allocated for files
- Allocation strategy: Contiguous allocation, Linked allocation, Indexed allocation

### Contiguous Allocation

<img src="https://i.loli.net/2019/03/20/5c921846bbb60.png" width="400px"/>
- Each file occupies a set of contiguous blocks
  - num of disk seeks is minimized
  - The dir entry for each file = (starting num, size)
- Both sequential & random access can be implemented efficiently
- Problems
  - External fragmentation $\rightarrow$ compaction
  - file cannot grow $\rightarrow$ extend-based FS

#### Extent-Based File System

- Many newer file system use a modified contiguous allocation scheme
- Extent-based file system allocate disk blocks in extents
- An extent is a contiguous blocks of disks
  - A file contains one or more extents
  - An extent: (starting block num, length, pointer to next extent)
- Random access become more costly
- Both internal & external fragmentation are possible

### Linked Allocation

- each file is a linked list of blocks
  - each block contains a pointer to the next block
  - data portion: block size -- pointer size
- file read: following through the list
  <img src="https://i.loli.net/2019/03/20/5c921a6e40243.png" width="400px"/>
- Problems
  - only good for sequential-access files
  - random access requires traversing through the link list
  - each access to link listis a disk I/O (link pointer is stored inside the data block)
  - space required for pointer (4/512 = 0.78%) (solution: cluster of blocks)
  - Reliability (one missing link breaks the whole file)

#### FAT (File Allocation Table) file system

<img src="https://i.loli.net/2019/03/20/5c921c3220923.png" width="400px"/>
- FAT32
  - store all links in a table
  - 32 bits per table entry
  - located in a section of disk at the beginning of each partition
- FAT(table) is often cached in memory
  - Random access is improved
  - Disk head find the location of any block by reading FAT

### Index Allocation Example

- The directory contains the address of the file index block
- each file has its own index block
- index block stores block # for file data
<img src="https://i.loli.net/2019/03/20/5c921c3220923.png" width=400px/>
- Bring all the pointers together into one location: the index block (one for each file)
- Implement direct and random access efficiently
- No external fragmentation
- Easy to create file (no allocation problem)
- Disadvantages
  1. space for index blocks
  2. how large the index block should be:
      - linked scheme
      - multilevel index
      - combined scheme (inode in BSD UNIX)

#### Linked Indexed Scheme

<img src="https://i.loli.net/2019/03/20/5c9220756eea4.png" width="400px"/>

#### Multilevel Scheme (two-level)

<img src="https://i.loli.net/2019/03/20/5c92213d6c2a5.png" width="400px"/>

#### combined scheme: unix inode

- File pointer: 4B (32 bits)
- Let each data/index block be 4KB
<img src="https://i.loli.net/2019/03/20/5c92220210861.png" width="400px"/>

## Free space

- Free-space list: records all free disk blocks
- Scheme
  - Bit vector
  - Linked List (same as linked allocation)
  - Grouping (same as linked index allocation)
  - Counting (same as contiguous allocation)
- file system usually manage free space in the same way as a file

### Bit vector

- Bit vector(bitmap): one bit foreach block
- simplicity, efficient (HW support bit-manipulation instruction)
- bitmap must be cached for gooe performance
  - A 1-TB(4KB block) disk needs 32MB bitmap
<img src="https://i.loli.net/2019/03/20/5c9223f4ac1e5.png" width="250px"/>