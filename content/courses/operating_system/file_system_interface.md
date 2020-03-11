---
title: File System Interface
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 9
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 8
---

## File Concept

- A logical storage unit created by OS (v.s. physical storage unit in disk (sector, track))
  <img src="https://i.loli.net/2019/03/20/5c91ae7d9b269.png" width="400px"/>
- file attributes (identifier, Name, type, Location, Size, protection, Last-access time, last-updated time)
- file operations (creating, reading, writing, repositioning, deleting, truncating)
- file types: .exe .com .obj .cc .mov (Hint for OS to operate file in a resonable way)
- Process: open-file table
- OS: system-wide table (process shared)

### Open-File Tables & System-wide table

- Per-process table
  - Tracking all files opened by this process
  - Current file pointer for each opened file
  - Access rights and accounting information
- System-wide table
  - Each entry in the per-process table points to this table
  - Process-independent information such as disk location, access dates, file size
  <img src="https://i.loli.net/2019/03/20/5c91b0ff4221e.png" width="400px"/>

### Access Methods

- Sequential access
  - Read/write next (block)
  - Reset: repositoning the file pointer to the beginning
  - Skip/rewind n records
- Direct(relative) access
  - Access an element at an arbitrary positin in a sequence
  - File operation include the block # as parameter
  - Often use random access to refer the access pattern from direct access
  <img src="https://i.loli.net/2019/03/20/5c91cb4992681.png" width="400px"/>
- Index Access Methods
  - Index: contains pointers to blocks of a file
  - To find a record in a file (search the index file $\rightarrow$ find the pointer, use the pointer to directly access the record)
  - with a large file $\rightarrow$ index could become too large
  <img src="https://i.loli.net/2019/03/20/5c91cbfd09552.png" width="400px"/>

## Directory Structure

- Partition (formatted or raw)
  - raw partiton (no file system): UNIX swap space, database
  - Formatted partition with file system is called volume
  - a partition can be a portion of a disk or group of multiple disks (distributed system)
  - some storage devices (e.g.: floopy disk) dose not and cannot have partition
- Directories are used by file system to store the information about the files in the partition
<img src="https://i.loli.net/2019/03/20/5c91cdf01ff41.png" width="400px"/>
- Directory Operations (search, create, delete, list, rename, traverse)

### Single-Level Directory

All files in one directory, filename has to be unique, poor efficiency in locating a file as number of file increases
<img src="https://i.loli.net/2019/03/20/5c91ceedaf6f6.png" width="400px"/>

### Two-Level Dicectory

- a separate dir for each user
- path = user name + file name
- single-level dir problems still exist per user
<img src="https://i.loli.net/2019/03/20/5c91cf80d477b.png" width="400px"/>

### Tree-Structured Directory

- Absolute path: starting from the root
- Relative path: starting from a directory

### Acyclic-Graph Directory

- use links to share files or directories (symbolic link `ln`)
- a file can have multiple absolute paths
- when dose a file actually get deleted?
  - deleting the link but not the file
  - deleting the file but leaves the link $\rightarrow$ dangling pointer
<img src="https://i.loli.net/2019/03/20/5c91d075632c0.png" width="400px"/>

### General-Graph Directory

- May contain cycles
  - Reference count dose not work any more (e.g. self-reference file)
  - How can we deal with cycles? (Garbage collection)
  - First pass traverses the entire graph and marks accessible files or directories
  - second pass collect and free everything that is un-marked
  - poor performance on millions of files
- Use cycle-detection algorithm when a link is created

## File-System Mounting & File sharing

### File-System Mounting

- A file system must be mounted before it can be accessed
- Mount point: the root path that a FS will be mounted to
- Mount timing: boot time, automatically at run-time, manually at run time
  <div class="note info"><p> mount -t type deivce dir (mount -t ext2 /dev/sda0 /mnt) </p></div>

### File sharing On Multiple Users

- Each user: (userID, groupID)
  - ID is associated with every ops/process/thread/ the user issues
- each file has 3 sets of attributes (owner, group, others)
- Owner attributes describe the privileges for the owner of the file
  - same for group/others attributes
  - group/others attributes are set by owner or root

### Access-Control List

- We can create an access-control list (ACL) for each user
  - check requested file access against ACL
  - problem: unlimited # of users
- 3 classes of users $\rightarrow$ 3 ACL (RWX) for each file
  - owner (e.g 7 = RWX = 111)
  - group (e.g. 6 = RWX = 110)
  - others (e.g. 4 = RWX = 100)

### File Protection

- File owner/creator should be able to control (what can be done, by whom, access control list(ACL))
- file should be kept from (
  - physical damage (reliability): RAID
  - improper access (protection): i.e. password