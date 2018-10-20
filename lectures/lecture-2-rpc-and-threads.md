# Lecture 2 - RPC and Threads

![](../.gitbook/assets/image%20%287%29.png)

![](../.gitbook/assets/image%20%289%29.png)



![](../.gitbook/assets/image%20%288%29.png)

![](../.gitbook/assets/image%20%282%29.png)

## Review

**What is Thread?**  
`a thread is the smallest execution context which is a independent set of registers values of  processor(CPU), including contexts of PC(Program counter), IR(Instruction Register), SP(Stack Pointer/Register)  
* Threads allow one program to logically execute many things at once(snippet shot of execution context)  
* Thread share memory`  
****&gt; ****[what is a "thread"\(really\)? - stackOverFlow](https://stackoverflow.com/questions/5201852/what-is-a-thread-really)

![A process\(execution context\) with two threads of execution, running on one processor](../.gitbook/assets/image%20%284%29.png)

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

**RPC problem: what to do about failures?**

**What does a failure look like to the client RPC library?**

**What if an at-most-once server crashes and re-starts?**

## Reference

* [x] [Lecture note - RPC and Threads](https://pdos.csail.mit.edu/6.824/notes/l-rpc.txt)
* [x] [Concurrency example - Crawler](https://pdos.csail.mit.edu/6.824/notes/crawler.go)
* [x] [RPC example - K/V](https://pdos.csail.mit.edu/6.824/notes/kv.go)
* [x] [Golang Package - RPC](https://golang.org/pkg/net/rpc/)



