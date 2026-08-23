# Outline

- [Outline](#outline)
- [iSCSI](#iscsi)
  - [What it is](#what-it-is)
  - [Key Components: Initiators and Targets](#key-components-initiators-and-targets)
  - [Fiber Channel](#fiber-channel)
  - [Networking Best Practices for iSCSI](#networking-best-practices-for-iscsi)
- [SAN](#san)
  - [SAN, NAS, DAS](#san-nas-das)
  - [Disk Group, Storage Pool, Volume, LUN](#disk-group-storage-pool-volume-lun)
  - [Pool/Controller Management](#poolcontroller-management)




# iSCSI

## What it is

- Internet Small Computer Systems Interface

- It is a networking protocol used to connect data storage servers to computers or hypervisors (like ESXi) over standard ethernet cables

- `SCSI`

    - `SCSI` is the traditional protocol computers use to talk to internal hard drives using commands like "write this data to sector 4" or "read sector 12"

    - `iSCSI` takes those exact same low-level storage commands, wraps them inside standard TCP/IP packets, and sends them over your regular network wires

    - When the data reaches the storage array, the network packets are stripped away, leaving the original `SCSI` storage commands to be executed on the disks

## Key Components: Initiators and Targets

- The iSCSI Initiator (The Client): This is the device that needs storage
  
    - `ESXi` host acts as the initiator

    - It issues commands to read and write data

    - Initiators can be software built into the OS, or dedicated hardware network cards

- The iSCSI Target (The Server/Storage)

    - This is the storage appliance that has the disks (often a SAN—Storage Area Network—or a NAS like a Synology or TrueNAS)

    - The target advertises specific chunks of storage, which are called LUNs (Logical Unit Numbers)

    - To your ESXi host, a LUN looks exactly like a blank, unformatted physical hard drive

## Fiber Channel

- Before iSCSI, if you wanted high-speed, centralized storage for servers, you had to use Fibre Channel (FC)

- Fibre Channel is incredibly fast but requires expensive, dedicated optical switches, specialized Host Bus Adapter (HBA) cards, and separate cabling

## Networking Best Practices for iSCSI

- `iSCSI` traffic should live on its own isolated VLAN and use dedicated physical NICs (`vmnic`)

- Change the packet size limit (MTU) to 9000 bytes across the ESXi VMkernel port, the physical switches, and the storage target

    - Standard network packets are 1,500 bytes

    - Storage data moves in massive blocks

- No Routing (Keep it Layer 2): iSCSI traffic should never pass through a router or firewall if it can be avoided

- The ESXi host (vmk2) and the Storage Target should be on the exact same subnet/VLAN to minimize latency




# SAN

## SAN, NAS, DAS

- SAN

    - How it connects: Connects via specialized high-speed networks (Fibre Channel or iSCSI)

    - How it functions: Presents storage to servers at the block level. The server's operating system treats the storage array like a locally attached hard drive

    - Use Cases: Ideal for heavy-duty applications like virtual machine hypervisors, databases, and large-scale enterprise data centers

- NAS

    - How it connects: Connects directly to your local area network (LAN) using standard Ethernet cables

    - How it functions: Presents storage at the file level (using protocols like NFS or SMB/CIFS). Users and servers interact directly with files and folder structures

    - Use Cases: Perfect for general-purpose file sharing, active collaboration, and archival storage across multiple users in a home or office network

- DAS

    - How it connects: Plugs directly into a single server or workstation via cables like SAS, SATA, or USB

    - How it functions: Does not communicate over a network; it is dedicated exclusively to the host device it is physically attached to

    - Use Cases: Good for local server expansion, isolated workstation storage, or personal backups

## Disk Group, Storage Pool, Volume, LUN

- Disk Group

    - A Disk Group (sometimes called a VDisk or RAID Group) is a collection of physical drives (HDDs or SSDs) bound together to form a protected array 

    - What it does: It applies a specific RAID level (e.g., RAID 10, RAID 5, RAID 6) across those disks to provide data redundancy and performance aggregation

- Storage Pool

    - A Storage Pool is a logical container that aggregates the storage space from one or more Disk Groups into a single large reservoir of capacity 

    - What it does: It abstracts the underlying disk groups so you can manage storage as a single, flexible block
    
    - Modern SANs use pools to enable features like thin provisioning, auto-tiering (moving active data to SSDs and cold data to HDDs), and snapshot overhead management 

    - Analogy: If a Disk Group is a bucket of water, a Storage Pool is a large water tank created by pouring multiple buckets together

- Volume

    - A Volume is a specific portion of storage carved out from the Storage Pool 

    - What it does: This is the administrative boundary where you define capacity (e.g., a "500GB Volume"), set thin/thick provisioning options, and manage array-level snapshots or replication

    - Analogy: Carving a 500GB volume out of your storage pool is like drawing a 500GB partition line inside that large water tank

- LUN

    - A LUN is technically an address/identifier used by the SAN fabric and host servers to direct read/write commands to a specific volume over iSCSI or Fibre Channel 

    - Engineers often use "volume" and "LUN" interchangeably in daily conversation, the Volume is the storage unit on the SAN
    
    - LUN ID (e.g., LUN 0, LUN 1) is how the host hypervisor or OS like (VMware ESXi or Windows Server) identifies and addresses that volume across the SAN fabric

    - Analogy: If the Volume is a specific apartment unit inside a building, the LUN is the street address number on the front door that tells the delivery driver (the server) where to drop off the package (data)

## Pool/Controller Management

- Dual-controller Architecture

    - Splitting physical drives across Pool A (owned by Controller A) and Pool B (owned by Controller B)

    - By splitting disks into 2 symmetrical pools, read/write workloads are distributed evenly across both controllers

    - Example

        ![SAN Dual-Controller Architecture](images/san_dual_controller_architecture.png)

- Never mix drive types, speeds, or capacities within the same Disk Group

- Example: A 6-disk group should consist of 6x identical 1.2TB 10K SAS drives

- Example Enterprise Layout

    ![SAN Controller Enterprise Layout](images/san_controller_enterprise_layout.png)