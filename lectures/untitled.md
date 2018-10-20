# Lecture 1 - MapReduce

![Figure 1: MapReduce execution overview](../.gitbook/assets/image%20%284%29.png)

## Review

**What is distributed systems?**  
`Multiple cooperating computers`

**Why distributed?**  
`* to organize physically separate entities  
* to organize physically separate entities  
* to achieve security via isolation  
* to tolerate faults via replication  
* to scale up throughput via parallel infrastructures`

**What will likely limit the performance?**  
`Bandwidth because all data go through network  
-> Minimizing movement of data over network`

**How does detailed design reduce effect of slow network?**`* Map input is read from GFS replica on local disk, not network  
* Intermediate data goes over network just once  
* Map worker write intermediate data to disk, not GFS. Why?   
* Intermediate data partitioned into files holding many keys, benefitting data transfer`

**Why not stream the records to the reducer\(via TCP\) as they are being produced by the mappers?**  
`* Not parallel any more, performance affected  
* Hard to handle a single node failure, may resend all data again`

 **How to get good load balance**   
`Some tasks may take longer that others, waiting for a single point to finish is bad  
 -> scaling  
Scaling many more tasks than workers, so no task is significant huge than others,  
avoding that it dominates completion time.  
Faster server could thus hands on new task and do more work efficiently` 

**What if a worker crashes during a map/reduce job? and worker crash recovery?**  
`* All the current-running map/reduce tasks will be rearranged  
* Map tasks which have been accomplished will re-run by other workers, because intermediate data may be unreachable in crashed worker local disk  
* Finished reduce tasks are OK because data stores in GFS, having replca`

**Why not re-start the whole job from beginning?**  
`Map and Reduce are designed to be pure functional and relateive independent, which means  
* No communication between Map and Reduce  
* Master do NOT keep state accorssing calls  
* Map and Reduce are just expecting to recieve input and output, that's all  
 -> This is the inflexible aspect of MapReduce, but also its simplicity`

**What if the Master gives two workers the same Map\(\) task?**  
`Master will tell Reduce workers about only one of them`

**What if the Master gives two workers the same Reduce\(\) task?**  
`It is fine to run twice. They will both write to the same output file on GFS. and` **`atomic`**`GFS will rename one and the other will be visible.`

**What if a single worker is very slow?**  
`Using back-up mechanism. Last few jobs which have not been finished will be rescheduled by master to other workers`

~~**What if a worker computes incorrect output due to broken h/w or s/w?**~~

**What if master crashed?**  
`Client should handle it, recovering master from check-point or give up job`

**What applications does NOT MapReduce work well?**  
`* Small data which is not suitable to partition and with high overhead  
* Small update to data, both map and reduce can NOT select input since unpredictable read  
* Multiple shuffles, not efficient`

**How might a real-world web company use MapReduce?**  
`CatBook", a new company running a social network for cats; needs to:`  
`1) build a search index, so people can find other peoples' cats  
2) analyze popularity of different cats, to decide advertising value  
3) detect dogs and remove their profiles  
Can use MapReduce for all these purposes!  
- run large batch jobs over all profiles every night  
1) build inverted index: map(profile text) -> (word, cat_id)  
                         reduce(word, list(cat_id) -> list(word, list(cat_id))  
2) count profile visits: map(web logs) -> (cat_id, "1")  
                         reduce(cat_id, list("1")) -> list(cat_id, count)  
3) filter profiles: map(profile image) -> img analysis -> (cat_id, "dog!")  
                    reduce(cat_id, list("dog!")) -> list(cat_id)`

**Conclusion  
`- Not the most efficient and flexible  
+ Nice Scalability  
+ Easy to program and handle, hidding failures and data movement`**

## Reference

* [x] [Lecture note - Introduction](https://pdos.csail.mit.edu/6.824/notes/l01.txt)
* [x] [Paper - MapReduce\(2004\)](http://www.cs.bu.edu/~jappavoo/jappavoo.github.com/451/papers/mapreduce.pdf)
* [x] [Blog - MapReduce Intro\(Chinese\)](http://airekans.github.io/cloud-computing/2014/01/25/mapreduce-intro)



