---
description: 'http://book.mixu.net/distsys/index.html'
---

# Distributed Systems for Fun and Profit

## Basics

### Terms and concepts

**Why are we developing distributed systems?** 

Problems scale and cost-prohibitive, single node can no longer satisfy.

**Scalability:** 

Moving from small to large, things or problems should have ability to accommodate the growth and not get incrementally worse.

**Performance**:

* Short response time/low latency \(time between something happened and exerted a impact\)
* High throughput
* Low utilization of computing resources

**Availability**:

Whether a system is in a functioning condition. Mostly we are talking about **fault tolerance.**

**Fault tolerance**:

The ability of a **system** to behave in a well-defined manner once faults occur.

### **Design techniques**

**Partition** and **Replicate** -- Divide and Conquer:

![Illustration 1: independently partition and replicate](../.gitbook/assets/image.png)

 **Partition:** technique to reduce the impact of dataset growth.

* Improve performance by limiting the amount of data needed to deal with.
* Improve availability by allowing individual fail independently.

**Replication:** primary way in which we can fight latency.

* Improve performance by making additional computing resources available to new copy of the data.
* Improving availability by creating copies, increasing tolerance of single node failure.
* Maintaining  consistency through consistency model.
  * Strong consistency: --&gt; programming as if having not underlying  replica of data.
  * Weak consistency --&gt; lower latency and higher availability, but existing difference.

## Up and down the level of abstraction 

