# Lecture 4 - Primary-Backup Replication

## Basic Design

![Figure 1: Basic FT Configuration](../.gitbook/assets/image%20%282%29.png)

**Primary VM and Backup VM:** initialize with same state, receive same inputs in same order, same execution and generate same outputs.

**Logging channel:** Medium that transfer all inputs received by primary to keep sync between primary and backup. 

**Shared Disk:** The content in shared disk is naturally correct and available if failover happens. Only primary VM could write on it. The atomic test-and-set it supports, which is before a failover, could avoid split brain. 

{% hint style="info" %}
Shared disk would be a single point failure. The whole service will down if it fails.  
Solution: Using GFS to ensure fault-tolerance for storage.
{% endhint %}

### **Deterministic Replay** 

Deterministic replay achieves synchronize between primary VM and backup VM through logging channel. It records inputs and all possible non-deterministic associated with the VM execution in a stream of log entires.

* For non-deterministic operations: sufficient information is logged to reproduce the operation, same state, and same output.
* For non-deterministic events\(e.g. timer, I/O completion interrupts\): the exact instruction where event occurred is recorded. During replay, the event is delivered at the same point in instructions stream.

### FT Protocol

Additional restrictions with logging entries on the logging channel to achieve fault-tolerance. The FT Protocol ensures that no data lost when primary fails by means of _**Output Requirement**_ which is enforced by _**Output Rule**_.

> * _**Output Requirement:**_ if the backup VM ever takes over after a failure of the primary, the backup VM will continue executing in a way that is entirely consistent with all outputs that the primary VM has sent to the external world.
> * _**Output Rule:**_ the primary VM may not send an output to the external world, until the backup VM has received and acknowledged the log entry associated with the operation producing the output.

Because output operation that primary receive may rely on some non-deterministic events\(e.g. timer interrupt\), which may give rise to a non-deterministic output by backup, the key point is that backup should  receive all log entries including log entry for output-producing operation to achieve consistency of output. 

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



