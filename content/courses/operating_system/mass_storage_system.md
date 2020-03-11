---
title: Mass Storage System
toc: true
type: docs
date: "2019-05-05T00:00:00+01:00"
draft: false

menu:
  operating_system:
   parent: Concepts
   weight: 11
# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 10
---


## Disk Structure

- Disk drives are addressed as large 1-dim array of logical blocks (logical block: smallest unit of transfer(sector))
- Logical blocks are mapped onto disk sequentially
  - Sector 0: 1st sector of 1st track on the outermost cyl
  - go from outermost cylinder to innermost one
- Disk drive attached to a computer by an I/O bus

### Sectors Per Track

- Constant linear velocity (CLV)
  - density of bits per track is uniform
  - more sectors on a track in outer cylinders
  - keeping same data rate
    increase rotation speed in inner cylinders
  - applications: CD-ROM and DVD-ROM
- Constant angular velocity (CAV)
  - keep same rotation speed
  - larger bit density on inner tracks
  - keep same data rate
  - applications: hard disks

## Disk Scheduling

- Disk-access time has 3 major components
  - Seek time: move disk arm to the desired cylinder
  - rotational latency: rotate disk head to the desired sector
  - read time: constant transfer time
- Disk bandwidth:
  number of bytes transferred / (complete of last req - start of first req)
  <img src="https://i.loli.net/2019/03/20/5c92296114e16.png" width="400px"/>
- Minimize seek time
- illustrate with a request queue(0-199) (98, 183, 37, 122, 14, 124, 65, 67)

### Algorithm

- FCFS (First come first served)
- SSTF(Shortest-seek-time-first)
  - SSTF scheduling is a form of SJF scheduling; may cause starvation of some requests
  - total head movement: 236 cylinders
  - common and has a natural appeal, but not optimal
- SCAN scheduling
  - disk head move from one end to the other end
  - A.k.a elvator algorithm
  - total head movement: 236 cylinders
  - perform better for disks with heavy load
  - No staravation problem
- C-SCAN scheduling
  - Disk head move in one direction only
  - A variant of SCAN to provide more uniform wait time
  - More uniform wait time
- C-LOOK scheduling
  - version of C-SCAN
  - Disk head moves only to the last request location
- Performance is also influenced by the file-allocation method
  - Contiguous: less head movement
  - Indexed & linked: greater head movement

## Disk Management

### Disk Formatting

- Low-level formatting (or physical formatting): dividing a disk into sectors that disk controller can read and write
- each sector = header + data area + trailer
  - header & trailer : sector number and ECC (error-correction code)
  - ECC is calculated based on all bytes in data area
  - data area size: 512B, 1KB, 4KB
- OS does the next 2 steps to use the disk
  - partition the disk into one or more groups of cylinders
  - logical formatting (i.e. creation of a file system)

### Boot Block

- Bootstrap program
  - Initialize CPU, registers, device, controllers, memory, and then starts OS
  - First boostrap code stored in ROM
  - complete bootstrap in the boot block of the boot disk (aka system disk)

#### Windows 2000
  
1. Run bootstrap code in ROM
2. Read boot code in MBR (Master Boot Record)
3. Find boot partition from partition table
4. read boot sector/block and continue booting
<img src="https://i.loli.net/2019/03/20/5c922eec7cc23.png" width="400px"/>

### Bad Blocks

- Simple disks like IDE disks
- Sophisticated disks like SCSI disks
- Sector sparing (forwarding): remap bad block to a spare one
  - Could affect disk-scheduling performance
  - A few spare sectors in each cylinder during formatting
- Sector slipping: ships sectors all down one spot

### Swap-Space Management

- swap-space: Virtual memory use disk space (swap-space) as an extension of main memory
- UNIX: allows use of multiple swap spaces
- Location
  - part of a normal file system (e.g. NT) less efficient
  - separate disk partition (raw partition) size is fixed
  - allows access to both types (e.g. LINUX)

#### Swap Space Allocation

- 1st version: copy entire process between contiguous disk regions and memory
- 2nd version: copy pages to swap space
  - Solaris 1: text segments read from file system, thrown away when pageout, only anonymous memory (stack, heap, etc) store in swap space
  - Solaris 2: swap-space allocation only when pageout rather than vitrual memory creation time
- Data structures for Swapping
<img src="https://i.loli.net/2019/03/20/5c9232c43fbc0.png" width="400px"/>

## RAID Structure

- RAID = Redundant Arrays of Inexpensive Disks
  - Provide reliability via redundacy
  - improve performance via parallelism
- RAID is arranged into different levels (Striping, mirror, Error-correcting code (ECC) & Parity bit)

### RAID 0 & RAID 1

- RAID 0: non-redundant striping
  - improve performance via parallelism
  - I/O bandwidth is proportional to the striping count (Both read and write BW increase by N times)
- RAID 1: Mirrored disks
  - Provide reliability via redundancy
    - Read BW increases by N times
    - Write BW remains the same
  <img src="https://i.loli.net/2019/03/22/5c946b246195c.png" width="400px"/>

### RAID 2: Hamming code

- E.g.: [Hamming code](https://en.wikipedia.org/wiki/Hamming_code) (7,4)
  - 4 data bits (on 4 disks) + 3 parity bits (on 3 disks)
  - each parity bit is linear code of 3 data bits
- Recover from any single disk failure
  - can detect up to two disk (i.e. bits) error
  - but can only "correct" one bit error
- better space efficient than RAID1 (75% overhead)
  <img src="https://i.loli.net/2019/03/22/5c946b80ad6ba.png" width="400px"/>

### RAID 3 & 4: Parity Bit

- Disk controller can detect whether a sector has been read correctly
- a single parity bit is enough to correct error from a single disk failure
- RAID 3: bit-level striping; RAID 4: Block-level striping
- Even though space efficiency
- Cost to compute & store parity bit
- RAID4 has higher I/O throughput, because controller does not need to reconstruct block from multiple disks
<img src="https://i.loli.net/2019/03/22/5c946bf615735.png" width="400px"/>

### RAID 5: Distributed Parity

- Spread data & parity across all disks
- Prevent over use of a single disk
<img src="https://i.loli.net/2019/03/22/5c946c1f69fc1.png" width="400px"/>
- Read BW increases by N times, because all four disks can serve a read request
- write BW:
  - Method 1: (1) read out all unmodified (N-2) data bits (2) re-compute parity bit (3) write both modified bit and parity bit to disks (BW = N/(N-2+2) = 1) remains the same
  - Method 2: (1) only read the parity bit and modified bit. (2) re-compute parity bit by the difference. (3) write both modified bit and parity bit (BW = N/(2+2) = N/4 times faster)
    <img src="https://i.loli.net/2019/03/22/5c946d18c09fb.png" width="500px"/>

### RAID 6: P + Q Dual Parity Redundancy

- Like RAID 5, but stores extra redundant information to guard against multiple disk failure
- Use Ecc code (i.e. Error correction code) instead of single parity bit
- Parity bits are also striped across disks
<img src="https://i.loli.net/2019/03/22/5c946dc4da720.png" width="400px"/>
  
### Hybird RAID

- RAID 0+1: Stripe then replicate
- RAID 1+0: Replicate then stripe
<img src="https://i.loli.net/2019/03/22/5c946e29393a7.png" width="400px"/>
- First level control by a controller. Therefore, RAID 10 has better fault tolerance than RAID01 when multiple disk failes
