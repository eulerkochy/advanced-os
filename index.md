## A Trip down the OS Memory Lane 
# A Look Back at some fundamental papers which shaped modern OS development


### Authors: Koustav Chowdhury, Rupayan Sarkar, Jyoti Agrawal, Jyotisman Das, Bedant Agrawal

## As modern Operating Systems and Processors continue to amaze us with their ultra - high parallelism and super convenient user interfaces, and providing us virtually infinite cloud storage with the advent of virtualization technology and cloud computing, let us take a step back and look at what design features, or what user needs, did inspire the modern OS design. We shall be looking at 2 operating systems specifically, namely the **Hydra OS** and the **Pilot OS**, and shall be describing their architecture, their merits and shortcomings, and how they parallel modern OS which led to  today’s standard OS design, and is taught to students of computer science all over the world.


# Hydra OS: An OS for a multiprocessor system 

The Hydra OS was developed at Carnegie Mellon University (CMU) during the 1970s, and it was one of the only Operating systems at that time to have multiprocessing support, and have a process scheduling and memory model similar to today’s Linux. Together with its hardware architecture, the **PDP-11**, it supported multiprocessing, page tables and virtual memory, subsystems dedicated towards special functions, and had a very strong protection policy mediated by the kernel, via an elaborate capability list mechanism. It was able to achieve the policy - mechanism isolation which is one of the core principles of Linux and its kernel modules.

The hardware architecture, called **C.mmp** consists of 16 PDP-11 Processors and an interprocessor bus. There was no master - slave relation between the processors, and the processors themselves were asynchronous, functioning as a distributed system communicating via a bus, and jobs were allocated to each processor as the OS deemed fit. Each PDP-11 could generate only 16-bit addresses, and each 8KB chunk of address was divided into **pages**. Interprocessor communication was supported by interface registers, and similar to today’s **processor affinity masks**, any processor could instruct another processor or itself some instruction via setting the i’th bit in the interface register for that instruction. This would instruct the i’th processor to execute the `HALT`, `START`, `CONTINUE` or `INTERRUPT-L` instruction, and evidently, these registers were 16-bit wide.

Hydra was described as an OS which separates the kernel from the userspace, and the Hydra kernel was designed so as to provide a **Virtual Machine** interface to the userspace programs, providing **virtual resources** such as files, directories, virtual address space and enabling each program to run as if **isolated**. The Hydra kernel was in charge of handling interrupts, and the physical hardware. This philosophy is core to modern day monolithic kernels also. 

According to *Wulf et. al.* in “**Overview of the Hydra Operating System Development**”, the Hydra kernel took additional steps to:

- Enforce protection
- Enforce no policy by the kernel on user programs
- Creation of new virtual resources (creation of new “types”)
- Specification and modification of “types” or resources
- Abstraction of “types”

Thus, we can see here a precursor to modern OS modules and device drivers, which present an identical abstraction over low level machine and device details via function calls.

When protection is to be enforced, it can be enforced in two ways: 
1. By only providing a global knowledge about a virtual resource in the form of some **external specifications**, and other subsystems which need to access this resource can only do so via these specifications,
2. By attaching rights lists or **capability lists** to every user program and process and accordingly, control the behavior and access patterns of these user programs.

The makers of Hydra enforced point 1 via 2 actually, and the capability mechanism can also provide for sharing of resources to provide parallelism, by intelligently granting rights to cooperating users or programs. The subsystem concept prevailed throughout, and modules could only be accessed via some functions. Each module was implemented as 2 files : one which had internal details of the data structures in the module, and another described the interface of the module to be provided to the external world. The second file achieved the abstraction or hiding concept, and both files were to be compiled together. Hydra was implemented in the **BLISS programming language**. 

Elaborating on the policy - mechanism separation, it should serve the purpose of not letting a malicious user take control over system resources. Speaking in a broad sense, the kernel should provide the mechanisms to access the resources, but should not dictate how the resources should be accessed; that is the job of the policy or the user space program. The kernel however must enforce that no userspace program can gain access to the hardware directly, and must contain policies for manipulation of hardware resources. We consider the protection mechanism (*Cohen et. al.*, “**Protection in the Hydra Operating System**”) and later consider the paging and scheduling mechanisms in Hydra (*Levin et. al*., “**Policy / Mechanism separation in Hydra**”).

### Protection mechanism in Hydra: Objects and C-LISTS

The protection problem is an interesting one, and the authors had divided it into:

- Prior vs Future Decisions: Some user or process may be granted or denied some permission prior to some event or in the future.
- Unilateral vs Negotiated Decisions: An unilateral decision is similar to an administrator restricting guest users’ rights, while a negotiated decision is a decision as to what rights the other may possess.

The policy / mechanism separation dictated that Hydra didn’t have any policy granting or denying access rights to a resource, but rather had a mechanism built-in to do so as needed by another user or program. This design is similar to the **Exokernel** design, where library OSs on top of the exokernel could decide the policy of resource allocation, but the protection and granting of resources rights rested with the Exokernel only.

Hydra referred to any process, procedure (function) or semaphore, etc., as an **object**(not in the sense of object oriented programming). Objects were distinguished by **types**, and achieved abstraction as described above. Generic kernel operations could manipulate objects, and objects as a whole could only be manipulated by a program via a **capability list** or `C-LIST`. Each `C-LIST` contains a large number of access rights which determine how the object named by the `C-LIST` can be accessed.

In Hydra, a **procedure** or **function call** had its own `C-LIST` and executed in a different environment than the caller of the procedure. Since the procedure, if from a proper subsystem, should be able to manipulate the object at the data structure and contents level, the procedure had a provision of **rights amplification**, which enabled it to have more rights than its caller, however the caller couldn’t access the contents of the object via its own C-LIST.

Every program had a C-LIST which was a linked list of pairs of an object and a set of access rights for the object in each node. Each **executing program** was also deemed to be an object, and was known as a **Local Namespace (LNS)**. Each object had a *Data-part* and a `C-LIST`
The Kernel Multiprocessing System (KMPS) was that portion of the kernel which implemented the mechanism to support policies for scheduling user processes. Each process had an associated Policy Module (PM), which was responsible for making scheduling decisions related to that process, e.g., one PM may control scheduling of time-sharing processes, and another may control scheduling background batch processes.

### Kernel calls and rights in Hydra

Syscalls manipulating the data part:
1. `Getdata(c-list, data_offset, length_to_read, buffer)`
2. `Putdata`
3.` Adddata`

Syscalls manipulating the C-LIST of the object:
1. `Load(c-list_path, LNS_slot)`
2. `Store(c-list_path, LNS_slot, rights_bitmask)`
3. `Append`
4. `Destroy`
5. `Copy`
6. `Create`

It is imperative that Hydra would have a rights list defining how objects in a C-LIST could access another object, and hence, the rights list was implemented as a bit vector of length 24, a set bit representing the presence of that right. Objects with the same rights could share resources, as defined by the rights list. The generic rights list applicable to types of all kinds is:

1. `GETRTS` - to get data from an object’s data part
2. `PUTRTS` - to put data into an object’s data part
3. `ADDRTS` - to append data onto an object’s data part
4. `LOADRTS` - to load a capability from an object’s C-LIST into the LNS
5. `STORTS` - to store a capability into an object’s C-LIST
6. `APPRTS` - to append a capability onto an object’s C-LIST
7. `KILLRTS` - to delete a capability from an object’s C-LIST
8. `COPYRTS` - to copy an object
9. `OBJRTS` - to destroy an object

Remaining rights were `ENVRTS` (environment rights), `MDFYRTS` (modify rights), `FRZRTS` (freeze rights), `DLRTS` (delete rights), and `TMPLRTS` (template rights).

### Procedures and Process structure in Hydra

In Hydra, a function was known as a procedure, and procedures were also deemed to be objects, with procedure arguments known as templates, and were generic C-LIST accepting arguments. The process structure was similar to the process address space in today’s architectures, starting with intrinsic program data, similar to the bss segment in executing programs, and having numerous executing stack frames, similar to function stack frames being pushed and popped. Thus a process consisted of a stack of LNS frames and initial data. Hydra deemed **“everything is an object”** similar to the UNIX philosophy that **“everything is a file”**. Several types of objects were:

- LNS
- Procedure
- Process
- Page
- Semaphore
- Port
- Device
- Policy
- Data
- Universal
- Type

Templates formed a part of the Type subsystem, and could be used to create objects of new types, or amplify rights of procedure arguments.

This design provided a powerful solution to the protection problem: direct protection, for example, was provided by the C-LIST for a file object, like read and write accesses, while procedure objects could provide for a rights induced secure environment for a LNS to execute (so-called **procedural embedding**).

### Revocation Protocol

Similar to the **Exokernel** design, in Hydra objects participated in a revocation protocol, and addressed issues like immediate, permanent, selective, partial, temporal and right to revocation. To design this, the creators made a new entity, called an **Alias**, which was a pointer with certain access rights from a C-LIST to its object. Aliases had an additional right ALLYRTS, which when unset from the alias severed the resource from its C-LIST and thus, revoked the access rights to the object. 

### Problems solved by the Hydra protection mechanism design

1. **Mutual suspicion**: By allowing each procedure to have its own C-LIST and execute in a different environment than its callee, thereby not letting inheritance of rights unless explicitly passed in the procedure template
2. **Inadvertent modification of objects**: By requiring that any rights that can modify an object should have an additional right MDFYRTS.
3. **Secure environment of execution**: The right ENVRTS bound a LNS or procedure to its own environment only, and ENVRTS could not be inherited or gained by rights amplification. This is similar to the concept of containers in modern cloud computing.
Freezing of C-LISTs and objects: Passing FRZRTS to a C-LIST made the object and C-LIST incapable of being modified, and hence was a safeguard against other users.

We thus gain an insight into the very powerful and elaborate protection mechanism and the process structure and execution in Hydra. With this knowledge, we can explore the different OS specific structures in Hydra, namely paging and scheduling.

### Scheduling in Hydra

The **Kernel Multiprocessing System (KMPS)** was that portion of the kernel which implemented the mechanism to support policies for scheduling user processes. Each process had an associated **Policy Module (PM)**, which was responsible for making scheduling decisions related to that process, e.g., one PM may control scheduling of time-sharing processes, and another may control scheduling background batch processes.

The policy module for a process contained scheduling parameters such as **priority, processor mask, time quantum** and **maximum current pageset size** (the set of pages present in “core” or RAM in modern day). The PM could start and stop the process, and when it started it was controlled by the KMPS, which was responsible for making the pages required by the process available to it. The KMPS also defined operations to get the current **Process Context Block (PCB)** of the current process, and operations to get messages from the PM for a process. Thus, the PM controlled the scheduling decisions, permitting user-level control of scheduling policies, while the KMPS was only in charge of executing it. It was assumed that the PM was only in charge of scheduling decisions and not the internal data of the LNS.

However, this risked the issue that a demonic process could starve other processes by using a major portion of CPU cycles unfairly, hence the KMPS was in-charge of the **fairness policy**. Each PM had a guaranteed CPU time, and when it ended, the KMPS stopped it, and moved it to the end of the current runqueue. All processes of the same priority were scheduled in a **round robin manner**, and processes were ordered by the **priority-indexed runqueues**, similar to the **active array in Linux scheduling**. A question may arise as to how to set the priorities of the processes, who does it, and do priorities change dynamically or are statically set? We have no satisfactory answer from the paper.

### Paging and Paging Policy in Hydra

Paging is strongly architecture dependent, and PDP-11 could only generate 16-bit addresses, and it's a rather small address space. Hydra was incapable of supporting **demand paging**. Each page was of **8KB** size. The hierarchy of paging for a LNS was a RPS (relocation pageset or currently executing page set or hot pages), CPS (current page set or in-memory pageset or warm pages) and LNS (total pageset, including cold pages, which needed to fetched from storage). Kernel operations like `CPSLOAD(cpsslot, page)`and `RPSLOAD(rpsslot, cpsslot)` loaded pages from a hierarchy to its subset, thus referencing current pages.

The paging policy supported by Hydra was:

1. Whenever a PM started, all pages referenced by the top CPS of a LNS were brought into memory or core.
2. Whenever a process stopped, the pages of the top CPS of the stopped process were made eligible to be transferred out of memory.
3. Whenever the top CPS changed, pages in the new CPS were brought in and the old CPS was made eligible to be paged out.

It was expensive for the KMPS to handle each page fault, and hence the PM had parameters to set the CPS size and execute `CPSLOAD` in order to mediate the pages to be brought into memory. The PM could alter CPS limits, and stop processes until the pages required were available in memory, hence page fault servicing was under the PM’s policy. However, as the authors mention, there clearly was no best policy of dividing the decisions among the KMPS and PMs, the PMs were expected to handle paging decisions, but when problems of shared pages or total pages referenced by all the CPSs exceeded the memory size came, one PM had no way of knowing the issue.

This was a severe problem the creators faced, and the dynamic nature of page referencing and faulting could not allow for two centres of authority for paging decisions, and ultimately the design was to be altered so that the KMPS was the sole arbiter and manager of pages. Each page also had the necessary provision of setting off a **dirty bit**, which indicated whether the page had been modified in memory since it was last written onto disk, and hence, should be updated in disk before being paged out.

The replacement policy followed was a priority based indexing of pages, with pages having same priority replaced on a **LRU (Least Recently Used)** basis, and the priorities *P* decided by $P = \sum c(r)$ where the summation is taken over the CPSs referenced by this page and **c(r)** is a cost function which had the value of 2 if r was in some top CPS, 1 if next-to-top and 0 if lower than that. No prefetching, demand paging, or frequency dependent policies were implemented. The need to define a CPS explicitly was also inconvenient, and could have been avoided if demand paging were implemented.

### Policy Module guarantees

One could have questioned whether the PMs themselves were trustworthy or not with regard to scheduling. This was handled by a system administrator, who could define the reliability and trustworthiness of each PM. Each PM had identification data which caused a LNS to identify and be associated with it. Hydra also implemented an error mechanism, where the kernel could throw an error when a PM did an erroneous scheduling, or stopping of a process. No process could be restarted without modifying or updating the process or PM data, or else the kernel operations returned an error. The kernel provided a `RUNTIME(time, pages)` operation, which guaranteed that a process had at least time `time` and pages `pages` allocated to it. If these were erroneous, an error was thrown.

## Thus we come to the end of a satisfactory discussion on the Hydra OS, having described almost all of its architectural features. We find that Hydra had many features of modern OSs, especially Linux. However, the C-LIST and rights mechanism aren’t used today, and today’s hardware support is much more powerful. Nevertheless, Hydra was a milestone in implementing so many features with so little hardware support, and it’s elaborate protection mechanism and multiprocessing support amazes us. With that, we shall be comparing and contrasting **Pilot OS** now.

# Pilot OS: An Operating System for a Personal Computer

Developed at Xerox Business Systems by Redell et.al., during the late 1970s, it was an OS designed for personal computers, and supported a high resolution bitmap display, a keyboard, a pointing device, a moving arm disk storage and a high bandwidth network to a LAN or remote servers. Written in the **Mesa programming language**, it was a single language single user system, and amusingly, the Mesa language depended on Pilot OS for its runtime support. 

Mesa consisted of **program modules** for defining and storing the internal data structures of a module, and **definitions module** for exporting interfaces needed to handle the module. It was similar to the Hydra design and modern day abstraction in object oriented programming. Interfaces exported were files, streams, network communication and virtual memory, and we shall consider each in turn.

### Files

Pilot defined a **File interface** for data storage, and a **Volume interface** for the actual storage device. Files had no directory hierarchy and were a flat namespace. A maximum of 2<sup>64</sup> files could be stored. The `File.Create` operation created a file and files were identified by unique **64-bit universal** identifiers or uids. These uids were unique for every system across all times, and hence, uniquely identified files even on different systems. Each uid was a concatenation of the machine serial number and the real time system clock. 

Each file had 4 attributes or metadata: **size, type, permanence** and **immutability**. Having a permanence attribute supported an innovative failure recovery system: if between `File.Create` and `File.MakePermanent` calls the system crashes, a scavenger or garbage collector removed invalid and temporary files. Thus, before valuable data was to be stored in the file, it was to be made permanent. The immutability attribute simply said whether a file could be further modified or not, and aided in portability and security.
 
Pilot could support multiple volumes or storage devices (physical volumes) and present them as a single large logical volume, or a large physical volume could also be divided into smaller logical volumes.

### Virtual Memory

Pilot had a single linear address space of 2<sup>3</sup> 16-bit words, with runs of addresses divided into the `Space` interface. It was a tree kind of structure, the root `Space` having all memory and any new `Space` defined as a subspace of the current referenced Space.

Each space had three flags: referenced, written and write-protected, and the operations supported by the `Space` interface were:

1. `Space.Create` - create a new space as a subspace of some other space
2. `Space.Map` - associate a file or set of files to a space
3. `Space.Activate` - swap in this space as it is to be referenced soon
4. `Space.Deactivate` - swap out this space, and/or update in disk if it is dirty
5. `Space.Kill` - overwrite this space as it is useless and no swapping is needed

Whenever a program referenced a space, it would look for at least a file which was mapped into the space. If no file was mapped into the referenced space, the fatal error `AddressFault` was raised, and a debugger was to be invoked. The separation of mapping and disk swapping decoupled buffer operations (file read/write) from disk accesses, and enabled reduction in disk traffic. `Space.Kill` also helped in overwriting useless spaces which need not be swapped and written onto disk.

### Streams and I/O devices

Similar to modern Linux, devices had a device driver interface which had generic function calls which could be invoked by a client program. Pilot also defined a `Stream` interface which allowed a communication and data stream from a program to a device, similar to the **UNIX pipe**, and multiple pipes could be chained together. A special module called a **transducer** imported a device driver and exported a stream, while a filter module imported a stream and exported another. Hence, a transducer and a set of filters could be cascaded to form a pipeline, and the authors do reference the UNIX pipe here.

### Network Communication

During the time Pilot was constructed, it was clear that the basic networking system was the same as today’s computer networks, and routers and telnet had been developed. Pilot supported **socket communication** on both same and different machines, similar to UNIX’s design. Network addresses were represented as a triple of a 16-bit network number, a 32-bit processor ID and a 16-bit socket number, and exported as a system wide datatype `System.NetworkAddress`. The **ARPA inter-network protocol** family was used, and it had 3 levels similar to today’s OSI layers, with each level encapsulating and padding layer specific data and checksums to a packet. Both reliable communication via **TCP** and unreliable communication via **datagrams** were supported, and TCP implemented a connection-oriented and error correcting sequenced protocol as it does today. The other details were the same as conventional socket programming, i.e., socket listening and binding, and an early form of **DNS** was also in force at that time; it was called a **clearinghouse**, which was a network backed database mapping network addresses to hostnames. 

### The CoPilot Debugger

Pilot had the facility of saving the entire machine state and doing a **world-swap** to the CoPilot debugger, which had a separate boot file and enabled strong isolation between machine execution and debugging. The machine state could be restored by doing a second world-swap.

## Architecture and Components

The creators kept the kernel a simple one, consisting of the **filer** and **swapper**, together with low-level Mesa support and basic network drivers. The filer handled file mappings, and the swapper handled disk accesses, and on top of these, the **virtual memory manager** and the **file manager** allowed for additional resource and feature management. A page or space cache was also present, containing swap units, and it supported demand paging and a viable page replacement policy. 

The process implementation in Pilot had provisions of `FORK`, `WAIT` and `MONITORS`, similar to UNIX. A file scavenger was also designed to repair damaged filesystems. Regarding the mapping between a volume and a file, a B-Tree keyed on `<file_uid, page_number>` was used, and it returned the device address of the page given a key, i.e., the physical address of the page given the virtual page number and file uid. 

## Hence, we see that Pilot paralleled UNIX in many ways, and was thus influential in modern OS development. The protection policy in Pilot was a defensive one though, the kernel handling only errors and for the major part trusting applications. This may be a shortcoming, albeit if it were coupled with the strong protection policy in Hydra, it would have made a system similar in robustness and applicability as UNIX is.

