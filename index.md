## A Trip down the OS Memory Lane 
# A Look Back at some fundamental papers which shaped modern OS development


### Authors: Koustav Chowdhury, Rupayan Sarkar, Jyoti Agrawal, Jyotisman Das, Bedant Agrawal

As modern Operating Systems and Processors continue to amaze us with their ultra - high parallelism and super convenient user interfaces, and providing us virtually infinite cloud storage with the advent of virtualization technology and cloud computing, let us take a step back and look at what design features, or what user needs, did inspire the modern OS design. We shall be looking at 3 operating systems specifically, namely the Hydra OS, the Pilot OS and the CAL OS, and shall be describing their architecture, their merits and shortcomings, and how they parallel modern OS which led to  today’s standard OS design, and is taught to students of computer science all over the world.


# Hydra OS: An OS for a multiprocessor system 

The Hydra OS was developed at Carnegie Mellon University (CMU) during the 1970s, and it was one of the only Operating systems at that time to have multiprocessing support, and have a process scheduling and memory model similar to today’s Linux. Together with its hardware architecture, the PDP-11, it supported multiprocessing, page tables and virtual memory, subsystems dedicated towards special functions, and had a very strong protection policy mediated by the kernel, via an elaborate capability list mechanism. It was able to achieve the policy - mechanism isolation which is one of the core principles of Linux and its kernel modules.

The hardware architecture, called C.mmp consists of 16 PDP-11 Processors and an interprocessor bus. There was no master - slave relation between the processors, and the processors themselves were asynchronous, functioning as a distributed system communicating via a bus, and jobs were allocated to each processor as the OS deemed fit. Each PDP-11 could generate only 16-bit addresses, and each 8KB chunk of address was divided into pages. Interprocessor communication was supported by interface registers, and similar to today’s processor affinity masks, any processor could instruct another processor or itself some instruction via setting the i’th bit in the interface register for that instruction. This would instruct the i’th processor to execute the HALT, START, CONTINUE or INTERRUPT-L instruction, and evidently, these registers were 16-bit wide.

Hydra was described as an OS which separates the kernel from the userspace, and the Hydra kernel was designed so as to provide a Virtual Machine interface to the userspace programs, providing virtual resources such as files, directories, virtual address space and enabling each program to run as if isolated. The Hydra kernel was in charge of handling interrupts, and the physical hardware. This philosophy is core to modern day monolithic kernels also. 

According to Wulf et. al. in “Overview of the Hydra Operating System Development”, the Hydra kernel took additional steps to:

- Enforce protection
- Enforce no policy by the kernel on user programs
- Creation of new virtual resources (creation of new “types”)
- Specification and modification of “types” or resources
- Abstraction of “types”

Thus, we can see here a precursor to modern OS modules and device drivers, which present an identical abstraction over low level machine and device details via function calls.

When protection is to be enforced, it can be enforced in two ways: 1) By only providing a global knowledge about a virtual resource in the form of some external specifications, and other subsystems which need to access this resource can only do so via these specifications, and 2) By attaching rights lists or capability lists to every user program and process and accordingly, control the behavior and access patterns of these user programs.

The makers of Hydra enforced point 1 via 2 actually, and the capability mechanism can also provide for sharing of resources to provide parallelism, by intelligently granting rights to cooperating users or programs. The subsystem concept prevailed throughout, and modules could only be accessed via some functions. Each module was implemented as 2 files : one which had internal details of the data structures in the module, and another described the interface of the module to be provided to the external world. The second file achieved the abstraction or hiding concept, and both files were to be compiled together. Hydra was implemented in the BLISS programming language. 

Elaborating on the policy - mechanism separation, it should serve the purpose of not letting a malicious user take control over system resources. Speaking in a broad sense, the kernel should provide the mechanisms to access the resources, but should not dictate how the resources should be accessed; that is the job of the policy or the user space program. The kernel however must enforce that no userspace program can gain access to the hardware directly, and must contain policies for manipulation of hardware resources. We consider the protection mechanism (Cohen et. al., “Protection in the Hydra Operating System”) and later consider the paging and scheduling mechanisms in Hydra (Levin et. al., “Policy / Mechanism separation in Hydra”).

The protection problem is an interesting one, and the authors had divided it into:

Prior vs Future Decisions: Some user or process may be granted or denied some permission prior to some event or in the future.
Unilateral vs Negotiated Decisions: An unilateral decision is similar to an administrator restricting guest users’ rights, while a negotiated decision is a decision as to what rights the other may possess.

The policy / mechanism separation dictated that Hydra didn’t have any policy granting or denying access rights to a resource, but rather had a mechanism built-in to do so as needed by another user or program.

Hydra referred to any process, procedure (function) or semaphore, etc., as an object (not in the sense of object oriented programming). Objects were distinguished by types, and achieved abstraction as described above. Generic kernel operations could manipulate objects, and objects as a whole could only be manipulated by a program via a capability list or C-LIST. Each C-LIST contains a large number of access rights which determine how the object named by the C-LIST can be accessed.

In Hydra, a procedure or function call had its own C-LIST and executed in a different environment than the caller of the procedure. Since the procedure, if from a proper subsystem, should be able to manipulate the object at the data structure and contents level, the procedure had a provision of rights amplification, which enabled it to have more rights than its caller, however the caller couldn’t access the contents of the object via its own C-LIST.

Every program had a C-LIST which was a linked list of pairs of an object and a set of access rights for the object in each node. Each executing program was also deemed to be an object, and was known as a Local Namespace (LNS). Each object had a Data-part and a C-LIST
The Kernel Multiprocessing System (KMPS) was that portion of the kernel which implemented the mechanism to support policies for scheduling user processes. Each process had an associated Policy Module (PM), which was responsible for making scheduling decisions related to that process, e.g., one PM may control scheduling of time-sharing processes, and another may control scheduling background batch processes.


