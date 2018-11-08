# RPC and Threads

![](../.gitbook/assets/image%20%2813%29.png)

![](../.gitbook/assets/image%20%2816%29.png)



![](../.gitbook/assets/image%20%2814%29.png)

* Stub constructs RPC request argument and records it for future response
* RPC library: 
  * read each request/response
  * create a new thread for this request/response
  * call the named method\(dispatch\)
  * marshallls  reply according to RPC rules
  * writes reply on TCP connection
* Dispatch: mapping each RPC request to corresponding handler which matches with the argument of RPC request
* Handler: function that read RPC argument, execute specific jobs, modify reply.

![](../.gitbook/assets/image%20%284%29.png)

## Review

**What is Thread?**  
`a thread is the smallest execution context which is a independent set of registers values of  processor(CPU), including contexts of PC(Program counter), IR(Instruction Register), SP(Stack Pointer/Register)  
* Threads allow one program to logically execute many things at once(snippet shot of execution context)  
* Thread share memory`  
****&gt; ****[what is a "thread"\(really\)? - stackOverFlow](https://stackoverflow.com/questions/5201852/what-is-a-thread-really)

![A process\(execution context\) with two threads of execution, running on one processor](../.gitbook/assets/image%20%287%29.png)

**Why threads?**  
`Threads express concurrency  
* I/O concurrency: while waiting for a response from another server, process next request  
* Multicore: Threads run in parallel on several cores` 

**Threading challenges?**  
`Racing(bug) -- One thread read data that another thread is changing  
    > use Mutexes  
    > or avoid sharing  
Coordination between threads  
    > use go channels or WaitGroup`

**When to use sharing and locks, versus channels?**  
`* Use lock regrading to state sharing and change  
* Use channel regrading to communication between separate thread  
* Use channel regrading to wait for events`

**What does failures look like to the client RPC library?**  
`e.g. lost packet, broken network, slow server, crashed server  
Client never see aby response after some time  
* Server may never receive the request  
* Server may has executed, but fail to reply before a crash  
* Server may has exectued and replyed, but network broke before`

**RPC problem: what to do about these failures?**  
`Two models handling failure:  
> At least once - repeat sending request until succcess  
    * Problem: latent wirte request has undeterministic influence for the responses of other requests  
    * Only works for read-only operations or has a model of dealing with duplicate.  
  
> At most once(Better) - server check duplicate for each requestd`

**How does at-most-once server handle duplicate requests?**  
`Introduce unique [xid] = clientID + RPC sequence number  
if seen[xid]   
    return old[xid]  
else   
    r = handler()  
    old[xid] = r  
    seen[xid] = true  
* Problem: when or how to discard old RPC info?`

**When is discard safe?**  
`Idea: server need to find a way to recognize client has received the last response  
* Client tell server directly: include see all replies <= X with RPC each request. Then server discard old[:x]  
* Only allow client one outstanding RPC at a time: the arrival of seq+1 allows server to discard all <= seq`

How to handle duplicate requests while original is still executing?  
`Using pending flag per executing RPC, pending for sequential duplicate requests`

**What if an at-most-once server crashes and re-starts?**  
`Write  old info to disk or use replicate machine to store state`

## Reference

* Lecture note
  * [RPC and Threads](https://pdos.csail.mit.edu/6.824/notes/l-rpc.txt)
* Concurrency example
  * [Crawler](https://pdos.csail.mit.edu/6.824/notes/crawler.go)
* RPC example
  * [K/V](https://pdos.csail.mit.edu/6.824/notes/kv.go)
* Golang Package
  * [RPC](https://golang.org/pkg/net/rpc/)



