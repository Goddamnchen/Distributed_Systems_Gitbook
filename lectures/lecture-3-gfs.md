# Lecture 3 - GFS

![Figure 1: GFS Architecture](../.gitbook/assets/image%20%2811%29.png)

![Figure 2: GFS Control Flow](../.gitbook/assets/image%20%282%29.png)

1. 
## Review

**What does Master store?**  
`- file namespace (*)  
- access control info e.g. version number of chunk (*)  
- mapping file --> chunks  
- chunk locations  
* these meta data will persist by logging mutations on disk`

**What does Master control?**  
`- chunk lease management  
- garbage collection of orphaned chunks  
- chunk migration between chunk servers  
- heart beat --> send instruction and collect states of chunk servers`

**Won't the master be a performance bottleneck?**  
`No, It wont.  
* Master minimizes its involvement in data transfer(read and write) by only supplying the chunks information of an aiming file.  
* Client will cache that information to also reduce interaction with master.`

**How does GFS determine the location of the nearest replica?**  
`IP address`

**What does 3x replications give us?**   
`* Availability of data  
* Load balance for reading of hot spot files`

**Why are chunks so big?**  
`* Reduce request numbers to master for chunk location  
* Reduce network overhead by keeping a more persistent and extended period of TCP connection  
* Reduce the size of metadata stored on master`

**How does client read a file?**  
`* Send file name and chunk index to master  
* Master replies with set of servers have that chunk  
* Response includes version # of chunk  
* Clients cache that information and ask nearest chunk server  
* Check the version  
* If version # is wrong, re-contact master`  






 

## Reference

* [x] [Lecture Note - GFS](https://pdos.csail.mit.edu/6.824/notes/l-gfs-short.txt)
* [x] [Paper - GFS\(2003\)](https://pdos.csail.mit.edu/6.824/papers/gfs.pdf)
* [x] [FAQ - GFS](https://pdos.csail.mit.edu/6.824/papers/gfs-faq.txt)
* [x] [Case Study ](https://queue.acm.org/detail.cfm?id=1594206)
  * \_\_[_GFS: Evolution on Fast-forward_](https://queue.acm.org/detail.cfm?id=1594206)\_\_
  * [GFS 2.0](http://highscalability.com/blog/2010/9/11/googles-colossus-makes-search-real-time-by-dumping-mapreduce.html)
* [x] [Blog - GFS Note\(Chinese\)](https://www.jianshu.com/p/e9a477ee27c1)
  * [Introduction to Google File System\(Chinese\)](http://blog.bittiger.io/post174/)
  *  [Video](https://www.youtube.com/watch?v=WLad7CCexo8)



