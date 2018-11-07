# Lecture 4 - Primary-Backup Replication

## Basic Design

![Figure 1: Basic FT Configuration](../.gitbook/assets/image%20%282%29.png)

**Primary VM and Backup VM:** initialize with same state, receive same inputs in same order, same execution and generate same outputs.

**Logging channel:** Medium that transfer all inputs received by primary to keep sync between primary and backup. 

**Shared Disk:**

### **Deterministic Replay** 

Deterministic replay achieves synchronize between primary VM and backup VM through logging channel. It records inputs and all possible non-deterministic associated with the VM execution in a stream of log entires.

* For non-deterministic operations
* For non-deterministic events\(e.g. timer, I/O completion interrupts\)

### FT Protocol

![Figure 2: FT Protocol](../.gitbook/assets/image%20%285%29.png)

## Review

**Why would physical servers be more difficult to ensure deterministic than VMs?**  
`VM is running on top of a hypervisor, which has full control over the execution of a VM, ensuring the hypervisor to capture all necessary information about non-deterministic operations on primary VM and to replay these operations correctly on the backup VM.(e.g. percise timing of interrupt delivery, random generator)`

**What is a hypervisor?**  
`The hypervisor is a type of system, normally in software, emulates computer resources, hardwares, devices to create a virtualized environment which makes virtual machine believe they are booting up and running with these available physical resources.  
* Type 1 bare-metal hypervisor - directly acting as a host and running on the hardware to manage guest operating system based on itself.  
* Type 2 hosted hypersivor - runing as an application on the top of an existing host operating system which between type2 hypersivor and hardware` 

**What is** _**deterministic reply**_**?**  
`Basic technology that allows us to record the execution of a primary and ensure the backup executes identically. It is added with extra protocals and functionality to bulid a complete fault-tolerant VM system.`

## Reference

* Lecture Note 
  * [Primary-Backup Replication](https://pdos.csail.mit.edu/6.824/notes/l-vm-ft.txt)
* Paper 
  * [Fault-Tolerant Virtual Machines \(2010\)](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)
* FAQ
  * [Fault-Tolerant Virtual Machines](https://pdos.csail.mit.edu/6.824/papers/vm-ft-faq.txt)
* Video
  * [What is a Hypervisor?](https://www.youtube.com/watch?v=s8Buw__C3k0)



