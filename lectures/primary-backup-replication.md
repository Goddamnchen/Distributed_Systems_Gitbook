# Primary-Backup Replication

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

* **For non-deterministic operations:** sufficient information is logged to reproduce the operation, same state, and same output.
* **For non-deterministic events\(e.g. timer, I/O completion interrupts**\): the exact instruction where event occurred is recorded. During replay, the event is delivered at the same point in instructions stream.

### FT Protocol

Additional restrictions with logging entries on the logging channel to achieve fault-tolerance. The FT Protocol ensures that no data lost when primary fails by means of _**Output Requirement**_ which is enforced by _**Output Rule**_.

> * _**Output Requirement:**_ if the backup VM ever takes over after a failure of the primary, the backup VM will continue executing in a way that is entirely consistent with all outputs that the primary VM has sent to the external world.
> * _**Output Rule:**_ the primary VM may not send an output to the external world, until the backup VM has received and acknowledged the log entry associated with the operation producing the output.

Because output operation that primary receive may rely on some non-deterministic events\(e.g. timer interrupt\), which may give rise to a non-deterministic output on backup, the key point is that backup should  receive all log entries including log entry for output-producing operation to achieve consistency of output. 

![Figure 2: FT Protocol](../.gitbook/assets/image%20%285%29.png)

What happen when the primary crashes just after getting ACK from backup, but before the primary emits the output?

Failover Occurrence:

1. The backup got some log entries from the primary
2. The backup continues executing those log entries WITH OUTPUT SUPPRESSED
3. After the last log entry, the backup starts emitting outputs
4. Therefore it will emits the output that the primary failed to emit

FT protocol can not guarantee that all outputs are produced exactly once when having failover, which means the backup may emit duplicate outputs after taking over. However, it's OK for TCP and writing to shared disk, since receivers could ignore duplicate sequence number and backup will write same data to same block\#.

### Detecting and Responding to Failure

Two situations of failures here: 

<table>
  <thead>
    <tr>
      <th style="text-align:left">Backup VM fails</th>
      <th style="text-align:left">Primary VM fails</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">The primary VM will <em>go live</em> (leave recording mode and stop sending
        entries ti log channel)</td>
      <td style="text-align:left">
        <p>The backup VM keep replaying execution until having consumed the last
          log entry.</p>
        <p>Then stop replaying and turn to normal mode, starting executing as a new
          primary VM</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">Restore new redundancy backup VM on another host</td>
      <td style="text-align:left">
        <p>During transition, the new promoted VM need to:</p>
        <ol>
          <li>advertise its MAC address on network</li>
          <li>reissue some disk I/Os</li>
          <li>Restore new redundancy backup VM on another host</li>
        </ol>
      </td>
    </tr>
  </tbody>
</table>#### Failure Detection:

* UDP heart beating
* Logging traffic monitoring

However, these two resorts are susceptible to a split-brain problem. We need additional resort based on shared storage to resolve split brain problem.

#### Split Brain Detection

{% hint style="info" %}
Whenever either a primary or backup VM wants to go live, it executes an atomic _test-and-set_ operation on the shared storage.
{% endhint %}

If the primary or backup thinks the other server is dead, and thus that it should take over by itself, it first sends a test-and set operation to the disk server. The server executes this roughly this code:

{% code-tabs %}
{% code-tabs-item title="Pseudocode of test-and-set" %}
```text
// The primary (or backup) only takes over ("goes live")
// if test-and-set returns true.
test-and-set() {
  acquire_lock()
  if flag == true:
    release_lock()
    return false
  else:
    flag = true
    release_lock()
    return true
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> The higher-level view is that, if the primary and backup lose network contact with each other, we want only one of them to go live. The danger is that, if both are up and the network has failed, both may go live and develop split brain. If only one of the primary or backup can talk to the disk server, then that server alone will go live. But what if both can talk to the disk server? Then the network disk server acts as a tie-breaker; test-and-set returns true only to the first call.

## Practical Design and Implementation

Aside form basic design, we need more details of design and implementation to achieve a usable, robust, and automatic system of our design. Such as initial-start and re-start of a backup VM, _VMotion_, detail about managing logging channel, Operations on FT, issues for network/disk I/Os.

### Initial-start and Re-start of Backup VM

Aims: To acquire a mechanism which will start a backup VM in the same state as a primary and does NOT significantly disrupt the execution of primary.

Solution: **Modified** **VMotion**, which will clone a exact running copy of VM to a remote host with minimal disrupt\(pause less that a second\) and will set up a logging channel between primary and remote 

### Managing the Logging Channel

![Figure 3: FT logging Buffers and Channel](../.gitbook/assets/image%20%2810%29.png)

The hypervisors of Primary and Backup VM both maintain a log buffer for log entries.   
The primary will produce log entires to its log buffer when executing. And these log entries will immediately flush out to logging channel. The backup log buffer will read in log entires from channel as soon as possible. Each time backup log buffer reads in some log entires, it will reply a ACK to primary, which will allow the delayed output to be release to external world.

{% hint style="warning" %}
The full/empty log buffer will lead to a execution halt of primary/backup. Although the halt is ok for backup VM because of no external communication, we must minimize the possibility that the primary log buffer may fills up, in which case the pause could affect client. 
{% endhint %}

The idea to resolve it is actually keep maintaining a matched speed between primary and backup.

{% hint style="success" %}
The VM FT uses the mechanism of limiting CPU resources to accommodate the speed of execution between primary and backup VM.
{% endhint %}

### Operations on FT VMs

For operations like shutting down or resource management change of primary VM, it is necessary to send special control entries to backup VM, ensuring that backup VM will exert appropriate operation corresponding to primary VM. 

* Most operations are initiated only on primary VM
* VMotion is a special case, which can be done independently by primary and backup

VMotion need to deal with switchover connection appropriately

> i.e. When trying to establish a new connection and switchover successfully, after which the survived one takes over and uses VMotion to re-start backup when primary/backup failed, the VMotion is required all outstanding\(unresolved\) to be completed at the final switchover point.

So VMotion could only deal with the operation of re-start and connection establishment when completing all I/Os. Although it's easy for primary VM to keep waiting for the completeness of outstanding I/Os, more complexity appears for backup VM owing to the reason that it must replay primary, in which case the primary may issuing I/Os constantly with certain workload. Therefore VM FT use an unique method to solve this:  **It requests that primary need temporarily complete all of its I/O at** _**the final switchover point.**_

## Review

**Why would physical servers be more difficult to ensure deterministic than VMs?**  
`VM is running on top of a hypervisor, which has full control over the execution of a VM, ensuring the hypervisor to capture all necessary information about non-deterministic operations on primary VM and to replay these operations correctly on the backup VM.(e.g. percise timing of interrupt delivery, random generator)`

**What is a hypervisor?**  
`The hypervisor is a type of system, normally in software, emulates computer resources, hardwares, devices to create a virtualized environment which makes virtual machine believe they are booting up and running with these available physical resources.  
* Type 1 bare-metal hypervisor - directly acting as a host and running on the hardware to manage guest operating system based on itself.  
* Type 2 hosted hypersivor - runing as an application on the top of an existing host operating system which between type2 hypersivor and hardware` 

**What is** _**deterministic reply**_**?**  
`Basic technology that allows us to record the execution of a primary and ensure the backup executes identically. It is added with extra protocals and functionality to bulid a complete fault-tolerant VM system.`

**Why would VM FT ensures that neither VM is moved to the server where the other VM is?**  
`Since the situation will no longer provide fault tolerance`

## Reference

* Lecture Note 
  * [Primary-Backup Replication](https://pdos.csail.mit.edu/6.824/notes/l-vm-ft.txt)
* Paper 
  * [Fault-Tolerant Virtual Machines \(2010\)](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)
* FAQ
  * [Fault-Tolerant Virtual Machines](https://pdos.csail.mit.edu/6.824/papers/vm-ft-faq.txt)
* Video
  * [What is a Hypervisor?](https://www.youtube.com/watch?v=s8Buw__C3k0)



