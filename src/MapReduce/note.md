
中间结果用iterator存储，传给reduce function。避免中间结果太大，导致内存装不下。  
map worker的分布式执行通过【分割输入数据】实现。reduce worker的分布式执行通过分割中间结果实现。  
map worker传回给master的是处理完的数据的地址（而不是数据本身）。同样，reduce worker拿到的也是地址。  
reduce worker需要对拿到的中间结果进行排序、合并。如果数据太大导致内存装不下，则需要外部排序。reduce worker把<key, set of intermediate values>传给reduce function  
一个reduce worker生成一个结果文件，通常不需要合并这些文件。一个map worker可能生产多个文件，一个文件交给一个reduce worker处理。reduce worker处理完后，重命名该文件。  
fault tolerance：
1. 心跳检测：master ping workers periodically。
2. map worker的结果数据存在local disk，而reduce worker的数据存在全局文件系统上。所以完成的map task在遇到错误时需要重试；而完成的reduce task不需要。  
3. 如果一个任务换了个worker执行，所有与之相关的reduce worker需要被通知  

GFS会把一份文件的复制三份，保存在不同电脑上。mapReduce的master会尝试把map task安排在拥有输入文件副本的机器上；如果不行，就会尝试安排在离拥有数据的机器很近的机器上（比如在同一个交换网内）  
map task的数据分成m份，然后生产出R份数据供reduce worker执行。master需要记录O(M*R)份状态数据。通常我们要控制M，使map task的输入数据在16MB-64MB之间。R经常由用户决定，推荐是worker数目*一个小系数。  
straggler问题：可能有些机器执行地格外慢。解决：当整个mapreduce任务快完成时，master为剩下的任务开启一个backup执行，backup和原先的执行有一个成功了，就标记该任务成功了。  
