# 写在前面
Spark 是目前最主流的大数据计算框架，其常用算子是最重要的考察点之一，而常用算子中groupByKey和 reduceByKey 两个算子的辨析又是必须考察的。本文就来天一谈这两个算子


groupByKey方法主要作用是将键相同的所有的键值对分组到一个集合序列当中，如（a,1）,(a,3),(b,1),(c,1),(c,3),分组后结果是（(a,1),(a,3)）,(b,1),((c,1),(c,3)),分组后的集合中的元素顺序是不确定的，比如键a的值集合也可能是(（a,3）,（a,1)).
相对而言，groupByKey方法是比较昂贵的操作，意思就是说比较消耗资源。所以如果你的目的是分组后对每一个键所对应的所有值进行求和或者取平均的话，那么请使用PairRDD中的reduceByKey方法或者aggregateByKey方法，这两种方法可以提供更好的性能
groupBykey是把所有的键值对集合都加载到内存中存储计算，所以如果一个键对应的值太多的话，就会导致内存溢出的错误，这是需要重点关注的地方
# reduceByKey与groupByKey进行对比
## 返回值类型不同
返回结果的外在形式相同，但返回值类型不同。reduceByKey返回的是RDD[(K, V)]，而groupByKey返回的是RDD[(K, Iterable[V])]，详细可见后文的案例比较。
## 中间计算结果不同
举例来说这两者的区别。比如含有数据（a,1）,(a,2),(a,3),(b,1),(b,2),(c,1)的rdd分别用上面两个方法做求和，reduceByKey产生的中间结果(a,6),(b,3),(c,1)，而groupByKey产生的中间结果结果为（(a,1)(a,2)(a,3)）,((b,1)(b,2)),(c,1)。可见groupByKey的结果更加消耗资源。
## 作用不同
作用不同，reduceByKey作用是聚合，异或等，groupByKey作用主要是分组，也可以做聚合（分组之后）
map端中间结果对键对应的值得聚合方式不同

举例说区别
下面我们用单词计数的例子来说明reduceByKey与groupByKey两种方式的区别。

单词技术的代码如下。
```
val words = Array("a", "a", "a", "b", "b", "b")  

val wordPairsRDD = sc.parallelize(words).map(word => (word, 1))  

val wordCountsWithReduce = wordPairsRDD.reduceByKey(_ + _)  //reduceByKey

val wordCountsWithGroup = wordPairsRDD.groupByKey().map(t => (t._1, t._2.sum))  //groupByKey
```

上面两种方法的计算结果是相同的，但是计算过程，中间结果却有很大的区别：
reduceByKey在每个分区移动数据之前，会对每一个分区中的key所对应的values进行求和，然后再利用reduce对所有分区中的每个键对应的值进行再次聚合。整个过程如图：
<div  align="center"><img src="https://github.com/cld378632668/work-notes-for-spark/blob/master/illustration/reducebyKey.png" alt="1.1" align="center" /> <br><br/> This is a picture in the center.</div><br><br/>
wordcount的 reduceByKey 的计算过程

groupByKey是把分区中的所有的键值对都进行移动，然后再进行整体求和，这样会导致集群节点之间的开销较大，传输效率较低，也是上文所说的内存溢出错误出现的根本原因
<div  align="center"><img src="https://github.com/cld378632668/work-notes-for-spark/blob/master/illustration/groupbyKey.png" alt="1.1" align="center" /> <br><br/> This is a picture in the center.</div><br><br/>

wordcount的 groupByKey 的计算过程




本文字符数 ：1530
