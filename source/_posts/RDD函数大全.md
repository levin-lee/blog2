---
title: RDD函数大全
date: 2021-11-27 14:20:45
tags: spark
cover: https://pic2.zhimg.com/80/v2-029fb7a5d9f9dbff547ca7ba91f35cf9_1440w.jpg
---
## map 
>是对 RDD 中的每个元素都执行一个指定的函数来产生一个新的 RDD。 任何 原 RDD 中的元素在新 RDD 中都有且只有一个元素与之对应。 举例： 

```
scala>val a=sc.parallelize(1to9,3) 
scala>val b=a.map(x=>x*2) 
scala>a.collect res10:Array[Int]=Array(1,2,3,4,5,6,7,8,9) 
scala>b.collect res11:Array[Int]=Array(2,4,6,8,10,12,14,16,18) 
//上述例子中把原 RDD 中每个元素都乘以 2 来产生一个新的 RDD。
```

## filter
>filter 是对 RDD 中的每个元素都执行一个指定的函数来过滤产生一个新的 RDD。 任何原 RDD 中的元素在新 RDD 中都有且只有一个元素与之对应。

```
val rdd=sc.parallelize(List(1,2,3,4,5,6)) 
val filterRdd=rdd.filter(_>5) 
filterRdd.collect()//返回所有大于 5 的数据的一个 Array， Array(6,8,10,12)
```

## flatMap
>与 map 类似，区别是原 RDD 中的元素经 map 处理后只能生成一个元素，而原 RDD 中的元素经 flatmap 处理后可生成多个元素来构建新 RDD。举例：对原 RDD 中的每个元素 x 产生 y 个元素（从 1 到 y，y 为元素 x 的值） 

```
scala>vala=sc.parallelize(1to4,2) scala>valb=a.flatMap(x=>1tox)
scala>b.collect res12:Array[Int]=Array(1,1,2,1,2,3,1,2,3,4)
```

## mapPartitions
>mapPartitions 是 map 的一个变种。map 的输入函数是应用于 RDD 中每个元素， 而 mapPartitions 的输入函数是应用于每个分区，也就是把每个分区中的内容作 为整体来处理的。 它的函数定义为： def mapPartitions[U: ClassTag](f: Iterator[T] => Iterator[U], preservesPartitioning: Boolean=false):RDD[U] f 即为输入函数，它处理每个分区里面的内容。每个分区中的内容将以 Iterator[T] 传递给输入函数 f， f 的输出结果是 Iterator[U]。最终的 RDD 由所有分区经过输入 函数处理后的结果合并起来的。 举例：

``` 
scala>val a=sc.parallelize(1to9,3) 
scala>def myfuncT:Iterator[(T,T)]={ 
var res=List(T,T) 
var pre=iter.next 
while(iter.hasNext){ 
	valcur=iter.next 
	res.::=(pre,cur) 
	pre=cur } res.iterator } 
scala>a.mapPartitions(myfunc).collect 
res0:Array[(Int,Int)]=Array((2,3),(1,2),(5,6),(4,5),(8,9),(7,8))
```

 上述例子中的函数myfunc是把分区中一个元素和它的下一个元素组成一个Tuple。 因为分区中最后一个元素没有下一个元素了，所以(3,4)和(6,7)不在结果中。 mapPartitions 还有些变种，比如 mapPartitionsWithContext，它能把处理过程中的 一些状态信息传递给用户指定的输入函数。还有 mapPartitionsWithIndex，它能 把分区的 index传递给用户指定的输入函数。
## mapPartitionsWithIndex
def mapPartitionsWithIndex[U](f: (Int, Iterator[T]) => Iterator[U], preservesPartitioning:Boolean=false)(implicitarg0:ClassTag[U]):RDD[U] 函数作用同 mapPartitions，不过提供了两个参数，第一个参数为分区的索引。

```
var rdd1=sc.makeRDD(1to5,2) //rdd1 有两个分区 
var rdd2=rdd1.mapPartitionsWithIndex{ 
(x,iter)=>{ varresult=ListString 
var i=0 
while(iter.hasNext){ i+=iter.next() } result.::(x+"|"+i).iterator
}
} 
```

//rdd2 将 rdd1 中每个分区的数字累加，并在每个分区的累加结果前面加了分区 索引 scala>rdd2.collect res13:Array[String]=Array(0|3,1|12)

## mapWith
>mapWith 是 map 的另外一个变种，map 只需要一个输入函数，而 mapWith 有两 个输入函数。它的定义如下： def mapWith[A: ClassTag, U: ](constructA: Int => A, preservesPartitioning: Boolean = false)(f:(T,A)=>U):RDD[U] 第一个函数 constructA 是把 RDD 的 partitionindex（index 从 0 开始）作为输入， 输出为新类型 A； 第二个函数 f 是把二元组(T,A)作为输入（其中 T 为原 RDD 中的元素，A 为第一个 函数的输出），输出类型为 U。 举例：把 partitionindex 乘以 10 加 2,作为新的 RDD 的元素。 

```
val x=sc.parallelize(List(1,2,3,4,5,6,7,8,9,10),3) 
x.mapWith(a=>a*10)((b,a)=>(b,a+2)).collect 
// 结果： 
(1,2) (2,2) (3,2) (4,12)
(5,12) (6,12) (7,22) (8,22) (9,22) (10,22)
```

## flatMapWith
>flatMapWith 与 mapWith 很类似，都是接收两个函数，一个函数把 partitionIndex 作为输入，输出是一个新类型 A；另外一个函数是以二元组（T,A）作为输入，输 出为一个序列，这些序列里面的元素组成了新的 RDD。它的定义如下： 
def flatMapWith[A:ClassTag, U: ClassTag](constructA:Int =>A, preservesPartitioning: Boolean=false)(f:(T,A)=>Seq[U]):RDD[U] 举例： 

```
scala>vala=sc.parallelize(List(1,2,3,4,5,6,7,8,9),3) 
scala>a.flatMapWith(x=>x,true)((x,y)=>List(y,x)).collect 
res58:Array[Int]=Array(0,1,0,2,0,3,1,4,1,5,1,6,2,7,2, 8,2,9)
```
## coalesce
>def coalesce(numPartitions: Int, shuffle: Boolean = false)(implicit ord: Ordering[T] = null):RDD[T] 该函数用于将 RDD 进行重分区，使用 HashPartitioner。 第一个参数为重分区的数目，第二个为是否进行 shuffle，默认为 false; 以下面的例子来看： 

```
scala>var data=sc.parallelize(1to12,3) 
scala>data.collect 
scala>data.partitions.size 
scala>var rdd1=data.coalesce(1) 
scala>rdd1.partitions.size 
scala>varrdd1=data.coalesce(4) 
scala>rdd1.partitions.size res2:Int=1 
//如果重分区的数目大于原来的分区数，那么必须指定 shuffle 参 数为 true，//否则，分区数不便 scala>varrdd1=data.coalesce(4,true) scala>rdd1.partitions.size
res3:Int=4
```

## repartition

>defrepartition(numPartitions:Int)(implicitord:Ordering[T]=null):RDD[T] 该函数其实就是 coalesce 函数第二个参数为 true 的实现 
```
scala>vardata=sc.parallelize(1to12,3) 
scala>data.collect scala>data.partitions.size 
scala>var rdd1=data.repartition(1) 
scala>rdd1.partitions.size 
scala>varrdd1=data.repartition(4) 
scala>rdd1.partitions.size res3:Int=4
```

## randomSplit
>def randomSplit(weights: Array[Double], seed: Long = Utils.random.nextLong): Array[RDD[T]] 该函数根据 weights 权重，将一个 RDD 切分成多个 RDD。 该权重参数为一个 Double 数组 第二个参数为 random 的种子，基本可忽略。 

```scala>varrdd=sc.makeRDD(1to12,12) rdd:org.apache.spark.rdd.RDD[Int]=ParallelCollectionRDD[16]atmakeRDDat:21
scala>rdd.collect res6:Array[Int]=Array(1,2,3,4,5,6,7,8,9,10)
scala>varsplitRDD=rdd.randomSplit(Array(0.5,0.1,0.2,0.2)) splitRDD: Array[org.apache.spark.rdd.RDD[Int]] = Array(MapPartitionsRDD[17] at randomSplitat:23, MapPartitionsRDD[18]atrandomSplitat:23, MapPartitionsRDD[19]atrandomSplitat:23, MapPartitionsRDD[20]atrandomSplitat:23)
//这里注意：randomSplit 的结果是一个 RDD 数组
scala>splitRDD.size res8:Int=4 //由于 randomSplit 的第一个参数 weights 中传入的值有 4 个，因此，就会切分成 4 个 RDD, //把原来的 rdd 按照权重 0.5,0.1,0.2,0.2，随机划分到这 4 个 RDD 中，权重高的 RDD，划分到//的几率就大一些。 //注意，权重的总和加起来为 1，否则会不正常 scala>splitRDD(0).collect res10:Array[Int]=Array(1,4)
scala>splitRDD(1).collect res11:Array[Int]=Array(3)
scala>splitRDD(2).collect res12:Array[Int]=Array(5,9)
scala>splitRDD(3).collect res13:Array[Int]=Array(2,6,7,8,10)
```

## glom
>defglom():RDD[Array[T]] 该函数是将 RDD 中每一个分区中类型为 T 的元素转换成 Array[T]，这样每一个分 区就只有一个数组元素。

```
scala>var rdd=sc.makeRDD(1to10,3) 
rdd:org.apache.spark.rdd.RDD[Int]=ParallelCollectionRDD[38]atmakeRDDat:21 
scala>rdd.partitions.size res33:Int=3 
//该 RDD 有 3 个分区 
scala>rdd.glom().collect 
res35:Array[Array[Int]]=Array(Array(1,2,3),Array(4,5,6),Array(7,8,9,10)) //glom 将每个分区中的元素放到一个数组中，这样，结果就变成了 3 个数组
```

## union 并集
```
val rdd1=sc.parallelize(List(5,6,4,3)) 
val rdd2=sc.parallelize(List(1,2,3,4)) //求并集 
val rdd3=rdd1.union(rdd2) 
rdd3.collect
```

## distinct
>去重 

```
val rdd1=sc.parallelize(List(5,6,4,3)) 
val rdd2=sc.parallelize(List(1,2,3,4)) //求并集 valrdd3=rdd1.union(rdd2) //去重输出 rdd3.distinct.collect
```
## intersection 交集
```
val rdd1=sc.parallelize(List(5,6,4,3)) 
valrdd2=sc.parallelize(List(1,2,3,4)) //求交集 valrdd4=rdd1.intersection(rdd2) rdd4.collect
```

## subtract
>def subtract(other:RDD[T]):RDD[T] defsubtract(other:RDD[T],numPartitions:Int):RDD[T] def subtract(other: RDD[T], partitioner: Partitioner)(implicit ord: Ordering[T] = null): RDD[T] 该函数返回在 RDD 中出现，并且不在 otherRDD 中出现的元素，不去重。

```
val rdd1=sc.parallelize(List(5,6,6,4,3)) 
val rdd2=sc.parallelize(List(1,2,3,4)) //求差集 valrdd4=rdd1.subtract(rdd2) rdd4.collect
```

## subtractByKey
>defsubtractByKeyW(implicitarg0:ClassTag[W]):RDD[(K,V)] def subtractByKey[W](other: RDD[(K, W)], numPartitions: Int)(implicit arg0: ClassTag[W]):RDD[(K,V)] def subtractByKey[W](other: RDD[(K, W)], p: Partitioner)(implicit arg0: ClassTag[W]): RDD[(K,V)]
subtractByKey 和基本转换操作中的 subtract 类似，只不过这里是针对 K 的，返回 在主 RDD 中出现，并且不在 otherRDD 中出现的元素。 参数 numPartitions 用于指定结果的分区数 参数 partitioner 用于指定分区函数

```
var rdd1=sc.makeRDD(Array(("A","1"),("B","2"),("C","3")),2) 
var rdd2=sc.makeRDD(Array(("A","a"),("C","c"),("D","d")),2) 
scala>rdd1.subtractByKey(rdd2).collect res13:Array[(String,String)]=Array((B,2))
```

## groupbyKey
```
val rdd1=sc.parallelize(List(("tom",1),("jerry",3),("kitty",2))) 
val rdd2=sc.parallelize(List(("jerry",2),("tom",1),("shuke",2))) //求并集 
val rdd4=rdd1.union(rdd2) //按 key 进行分组 
val rdd5=rdd4.groupByKey 
rdd5.collect
```

## reduceByKey
>顾名思义，reduceByKey 就是对元素为 KV 对的 RDD 中 Key 相同的元素的 Value 进行 reduce，因此，Key 相同的多个元素的值被 reduce 为一个值，然后与原 RDD 中的 Key 组成一个新的 KV 对。 举例: 

```
valrdd1=sc.parallelize(List(("tom",1),("jerry",3),("kitty",2))) 
valrdd2=sc.parallelize(List(("jerry",2),("tom",1),("shuke",2))) //求并集 
val rdd4=rdd1unionrdd2 //按 key 进行分组 
val rdd6=rdd4.reduceByKey(+) rdd6.collect()
```

## sortByKey
>将 List(("tom",1),("jerry",3),("kitty",2), ("shuke",1))和 List(("jerry",2),("tom",3), ("shuke",2),("kitty",5))做 wordcount，并按名称排序 
```
val rdd1=sc.parallelize(List(("tom",1),("jerry",3),("kitty",2),("shuke",1))) 
val rdd2=sc.parallelize(List(("jerry",2),("tom",3),("shuke",2),("kitty",5))) 
val rdd3=rdd1.union(rdd2) //按 key 进行聚合 
val rdd4=rdd3.reduceByKey(+) //false 降序 
val rdd5=rdd4.sortByKey(false) 
rdd5.collect
```

## sortBy
>将 List(("tom",1),("jerry",3),("kitty",2), ("shuke",1))和 List(("jerry",2),("tom",3), ("shuke",2),("kitty",5))做 wordcount，并按数值排序 

```
val rdd1=sc.parallelize(List(("tom",1),("jerry",3),("kitty",2),("shuke",1))) 
valrdd2=sc.parallelize(List(("jerry",2),("tom",3),("shuke",2),("kitty",5))) 
val rdd3=rdd1.union(rdd2) 
//按 key 进行聚合 
valrdd4=rdd3.reduceByKey(+)
//false 降序 
valrdd5=rdd4.sortBy(_._2,false) rdd5.collect

## zip
>def zipU(implicitarg0:ClassTag[U]):RDD[(T,U)]
zip 函数用于将两个 RDD 组合成 Key/Value 形式的 RDD,这里默认两个 RDD 的 partition 数量以及元素数量都相同，否则会抛出异常。

```
scala>varrdd1=sc.makeRDD(1to5,2) rdd1:org.apache.spark.rdd.RDD[Int]=ParallelCollectionRDD[1]atmakeRDDat:21
scala>varrdd2=sc.makeRDD(Seq("A","B","C","D","E"),2) rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[2] at makeRDD at:21
scala>rdd1.zip(rdd2).collect res0:Array[(Int,String)]=Array((1,A),(2,B),(3,C),(4,D),(5,E))
scala>rdd2.zip(rdd1).collect res1:Array[(String,Int)]=Array((A,1),(B,2),(C,3),(D,4),(E,5))
scala>varrdd3=sc.makeRDD(Seq("A","B","C","D","E"),3) rdd3: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[5] at makeRDD at:21 scala>rdd1.zip(rdd3).collect java.lang.IllegalArgumentException: Can't zip RDDs with unequal numbers of partitions //如果两个 RDD 分区数不同，则抛出异常
```

## zipPartitions
>zipPartitions 函数将多个 RDD 按照 partition 组合成为新的 RDD，该函数需要组合
的 RDD 具有相同的分区数，但对于每个分区内的元素数量没有要求。 该函数有好几种实现，可分为三类：
参数是一个 RDD

```
def zipPartitions[B, V](rdd2: RDD[B])(f: (Iterator[T], Iterator[B]) => Iterator[V])(implicit arg0: ClassTag[B],arg1:ClassTag[V]):RDD[V]
def zipPartitions[B, V](rdd2: RDD[B], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B]) => Iterator[V])(implicitarg0:ClassTag[B],arg1:ClassTag[V]):RDD[V]
```
这两个区别就是参数 preservesPartitioning，是否保留父 RDD 的 partitioner 分区信息
映射方法 f 参数为两个 RDD 的迭代器。

```
scala>varrdd1=sc.makeRDD(1to5,2) rdd1:org.apache.spark.rdd.RDD[Int]=ParallelCollectionRDD[22]atmakeRDDat:21
scala>varrdd2=sc.makeRDD(Seq("A","B","C","D","E"),2) rdd2:org.apache.spark.rdd.RDD[String]=ParallelCollectionRDD[23]atmakeRDDat:21
//rdd1 两个分区中元素分布： scala>rdd1.mapPartitionsWithIndex{ | (x,iter)=>{ | varresult=ListString | while(iter.hasNext){ | result::=("part_"+x+"|"+iter.next()) | } | result.iterator | | } | }.collect res17:Array[String]=Array(part_0|2,part_0|1,part_1|5,part_1|4,part_1|3)
//rdd2 两个分区中元素分布 scala>rdd2.mapPartitionsWithIndex{ | (x,iter)=>{
| varresult=ListString | while(iter.hasNext){ | result::=("part_"+x+"|"+iter.next()) | } | result.iterator | | } | }.collect res18:Array[String]=Array(part_0|B,part_0|A,part_1|E,part_1|D,part_1|C)
//rdd1 和 rdd2 做 zipPartition scala>rdd1.zipPartitions(rdd2){ | (rdd1Iter,rdd2Iter)=>{ | varresult=ListString | while(rdd1Iter.hasNext&&rdd2Iter.hasNext){ | result::=(rdd1Iter.next()+""+rdd2Iter.next()) | } | result.iterator | } | }.collect res19:Array[String]=Array(2_B,1_A,5_E,4_D,3_C)
参数是两个 RDD
def zipPartitions[B, C, V](rdd2: RDD[B], rdd3: RDD[C])(f: (Iterator[T], Iterator[B], Iterator[C]) => Iterator[V])(implicitarg0:ClassTag[B],arg1:ClassTag[C],arg2:ClassTag[V]):RDD[V]
def zipPartitions[B, C, V](rdd2: RDD[B], rdd3: RDD[C], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B], Iterator[C]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2:ClassTag[V]):RDD[V]
```

用法同上面，只不过该函数参数为两个 RDD，映射方法 f 输入参数为两个 RDD 的迭代器。

```
scala>varrdd1=sc.makeRDD(1to5,2) rdd1:org.apache.spark.rdd.RDD[Int]=ParallelCollectionRDD[27]atmakeRDDat:21
scala>varrdd2=sc.makeRDD(Seq("A","B","C","D","E"),2) rdd2:org.apache.spark.rdd.RDD[String]=ParallelCollectionRDD[28]atmakeRDDat:21
scala>varrdd3=sc.makeRDD(Seq("a","b","c","d","e"),2) rdd3:org.apache.spark.rdd.RDD[String]=ParallelCollectionRDD[29]atmakeRDDat:21
//rdd3 中个分区元素分布 scala>rdd3.mapPartitionsWithIndex{ | (x,iter)=>{ | varresult=ListString | while(iter.hasNext){ | result::=("part"+x+"|"+iter.next()) | } | result.iterator | | } | }.collect res21:Array[String]=Array(part_0|b,part_0|a,part_1|e,part_1|d,part_1|c)
//三个 RDD 做 zipPartitions scala>varrdd4=rdd1.zipPartitions(rdd2,rdd3){ | (rdd1Iter,rdd2Iter,rdd3Iter)=>{ | varresult=ListString | while(rdd1Iter.hasNext&&rdd2Iter.hasNext&&rdd3Iter.hasNext){ | result::=(rdd1Iter.next()+""+rdd2Iter.next()+""+rdd3Iter.next()) | } | result.iterator | } | } rdd4:org.apache.spark.rdd.RDD[String]=ZippedPartitionsRDD3[33]atzipPartitionsat:27
scala>rdd4.collect res23:Array[String]=Array(2_B_b,1_A_a,5_E_e,4_D_d,3_C_c)
参数是三个 RDD
def zipPartitions[B, C, D, V](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D])(f: (Iterator[T], Iterator[B],
Iterator[C], Iterator[D]) => Iterator[V])(implicit arg0: ClassTag[B], arg1: ClassTag[C], arg2: ClassTag[D],arg3:ClassTag[V]):RDD[V]
def zipPartitions[B, C, D, V](rdd2: RDD[B], rdd3: RDD[C], rdd4: RDD[D], preservesPartitioning: Boolean)(f: (Iterator[T], Iterator[B], Iterator[C], Iterator[D]) => Iterator[V])(implicit arg0: ClassTag[B],arg1:ClassTag[C],arg2:ClassTag[D],arg3:ClassTag[V]):RDD[V]
```
用法同上面，只不过这里又多了个一个 RDD 而已。

## zipWithIndex
>defzipWithIndex():RDD[(T,Long)] 该函数将 RDD 中的元素和这个元素在 RDD 中的 ID（索引号）组合成键/值对。 

```
scala>var rdd2=sc.makeRDD(Seq("A","B","R","D","F"),2) 
rdd2: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[34] at makeRDD at:21 
scala>rdd2.zipWithIndex().collect res27:Array[(String,Long)]=Array((A,0),(B,1),(R,2),(D,3),(F,4))
```

##  zipWithUniqueId
>def zipWithUniqueId():RDD[(T,Long)] 该函数将 RDD 中元素和一个唯一 ID 组合成键/值对，该唯一 ID 生成算法如下： 每个分区中第一个元素的唯一 ID 值为：该分区索引号， 每个分区中第 N 个元素的唯一 ID 值为： (前一个元素的唯一 ID 值)+(该 RDD 总的 分区数) 看下面的例子： scala>varrdd1=sc.makeRDD(Seq("A","B","C","D","E","F"),2) rdd1: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[44] at makeRDD at:21 //rdd1 有两个分区， scala>rdd1.zipWithUniqueId().collect res32:Array[(String,Long)]=Array((A,0),(B,2),(C,4),(D,1),(E,3),(F,5))
//总分区数为 2 //第一个分区第一个元素 ID 为 0，第二个分区第一个元素 ID 为 1 //第一个分区第二个元素 ID 为 0+2=2，第一个分区第三个元素 ID 为 2+2=4 //第二个分区第二个元素 ID 为 1+2=3，第二个分区第三个元素 ID 为 3+2=5
键值转换

## partitionBy
>def partitionBy(partitioner:Partitioner):RDD[(K,V)] 该函数根据 partitioner 函数生成新的 ShuffleRDD，将原 RDD 重新分区。 scala>varrdd1=sc.makeRDD(Array((1,"A"),(2,"B"),(3,"C"),(4,"D")),2) rdd1:org.apache.spark.rdd.RDD[(Int,String)]=ParallelCollectionRDD[23]atmakeRDDat:21 scala>rdd1.partitions.size res20:Int=2

```
//查看 rdd1 中每个分区的元素 
scala>rdd1.mapPartitionsWithIndex{ | (partIdx,iter)=>{ | varpart_map=scala.collection.mutable.MapString,List[(Int,String)] | while(iter.hasNext){ | varpart_name="part_"+partIdx; | varelem=iter.next() | if(part_map.contains(part_name)){ | varelems=part_map(part_name) | elems::=elem | part_map(part_name)=elems | }else{ | part_map(part_name)=List[(Int,String)]{elem} | } | } | part_map.iterator | | } | }.collect res22:Array[(String,List[(Int,String)])]=Array((part_0,List((2,B),(1,A))),(part_1,List((4,D),(3,C))))
//(2,B),(1,A)在 part_0 中，(4,D),(3,C)在 part_1 中
//使用 partitionBy 重分区 
scala>varrdd2=rdd1.partitionBy(neworg.apache.spark.HashPartitioner(2)) rdd2:org.apache.spark.rdd.RDD[(Int,String)]=ShuffledRDD[25]atpartitionByat:23
scala>rdd2.partitions.size res23:Int=2
//查看 rdd2 中每个分区的元素 
scala>rdd2.mapPartitionsWithIndex{ | (partIdx,iter)=>{ | varpart_map=scala.collection.mutable.MapString,List[(Int,String)] | while(iter.hasNext){ | varpart_name="part_"+partIdx; | varelem=iter.next() | if(part_map.contains(part_name)){ | varelems=part_map(part_name) | elems::=elem | part_map(part_name)=elems | }else{ | part_map(part_name)=List[(Int,String)]{elem} | } | } | part_map.iterator | } | }.collect res24:Array[(String,List[(Int,String)])]=Array((part_0,List((4,D),(2,B))),(part_1,List((3,C),(1,A)))) //(4,D),(2,B)在 part_0 中，(3,C),(1,A)在 part_1 中
```

##  mapValues
>mapValues 顾名思义就是输入函数应用于 RDD 中 Kev-Value 的 Value，原 RDD 中的 Key 保持 不变，与新的 Value 一起组成新的 RDD 中的元素。因此，该函数只适用于元素为 KV 对的 RDD。 举例： 

```
scala>vala=sc.parallelize(List("dog","tiger","lion","cat","panther","eagle"),2) 
scala>valb=a.map(x=>(x.length,x)) 
scala>b.mapValues("x"+_+"x").collect
res5: Array[(Int, String)] = Array((3,xdogx), (5,xtigerx), (4,xlionx),(3,xcatx), (7,xpantherx), (5,xeaglex))
```

##  flatMapValues
>flatMapValues 类似于 mapValues，不同的在于 flatMapValues 应用于元素为 KV 对的 RDD 中 Value。每个一元素的 Value 被输入函数映射为一系列的值，然后这些值再与原 RDD 中的 Key 组成一系列新的 KV 对。 举例 vala=sc.parallelize(List((1,2),(3,4),(5,6))) valb=a.flatMapValues(x=>1.to(x)) b.collect.foreach(println)

## combineByKey
>def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) =>C):RDD[(K,C)] def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) =>C,numPartitions:Int):RDD[(K,C)] def combineByKey[C](createCombiner: (V) => C, mergeValue: (C, V) => C, mergeCombiners: (C, C) => C, partitioner: Partitioner, mapSideCombine: Boolean = true, serializer: Serializer = null): RDD[(K,C)] 该函数用于将 RDD[K,V]转换成 RDD[K,C],这里的 V 类型和 C 类型可以相同也可以不同。 其中的参数： createCombiner：组合器函数，用于将 V 类型转换成 C 类型，输入参数为 RDD[K,V]中的 V,输 出为 C,分区内相同的 key 做一次 mergeValue：合并值函数，将一个C类型和一个V类型值合并成一个C类型， 输入参数为(C,V)， 输出为 C，分区内相同的 key 循环做 mergeCombiners：分区合并组合器函数，用于将两个 C 类型值合并成一个 C 类型，输入参数 为(C,C)，输出为 C，分区之间循环做 numPartitions：结果 RDD 分区数，默认保持原有的分区数 partitioner：分区函数,默认为 HashPartitioner mapSideCombine：是否需要在 Map 端进行 combine 操作，类似于 MapReduce 中的 combine， 默认为 true
看下面例子：

```
scala>var rdd1=sc.makeRDD(Array(("A",1),("A",2),("B",1),("B",2),("C",1))) 
rdd1:org.apache.spark.rdd.RDD[(String,Int)]=ParallelCollectionRDD[64]atmakeRDDat:21 
scala>rdd1.combineByKey( | (v:Int)=>v+"", | (c:String,v:Int)=>c+"@"+v, | (c1:String,c2:String)=>c1+"1),(B,1_” +c2//合并 C 类型和 C 类型，中间加$，返回 C(String) 其他参数为默认值。 最终，将 RDD[String,Int]转换为 RDD[String,String]。
```

再看例子：
```
rdd1.combineByKey( (v:Int)=>List(v), (c:List[Int],v:Int)=>v::c, (c1:List[Int],c2:List[Int])=>c1:::c2 ).collect 
res65:Array[(String,List[Int])]=Array((A,List(2,1)),(B,List(2,1)),(C,List(1))) 最终将 RDD[String,Int]转换为 RDD[String,List[Int]]。
```

## foldByKey
```
deffoldByKey(zeroValue:V)(func:(V,V)=>V):RDD[(K,V)]
deffoldByKey(zeroValue:V,numPartitions:Int)(func:(V,V)=>V):RDD[(K,V)]
deffoldByKey(zeroValue:V,partitioner:Partitioner)(func:(V,V)=>V):RDD[(K,V)]
```

该函数用于 RDD[K,V]根据 K 将 V 做折叠、合并处理，其中的参数 zeroValue 表示先根据映射 函数将 zeroValue 应用于 V,进行初始化 V,再将映射函数应用于初始化后的 V.
例子：
```
scala>varrdd1=sc.makeRDD(Array(("A",0),("A",2),("B",1),("B",2),("C",1))) scala>rdd1.foldByKey(0)(+).collect res75:Array[(String,Int)]=Array((A,2),(B,3),(C,1)) //将 rdd1 中每个 key 对应的 V 进行累加，注意 zeroValue=0,需要先初始化 V,映射函数为+操 //作，比如("A",0),("A",2)，先将 zeroValue 应用于每个 V,得到：("A",0+0),("A",2+0)，即： //("A",0),("A",2)，再将映射函数应用于初始化后的 V，最后得到(A,0+2),即(A,2)
```
再看：
```
scala>rdd1.foldByKey(2)(+).collect res76:Array[(String,Int)]=Array((A,6),(B,7),(C,3)) //先将 zeroValue=2 应用于每个 V,得到：("A",0+2), ("A",2+2)，即：("A",2), ("A",4)，再将映射 函 //数应用于初始化后的 V，最后得到：(A,2+4)，即：(A,6)
```
再看乘法操作：
```
scala>rdd1.foldByKey(0)(*).collect res77:Array[(String,Int)]=Array((A,0),(B,0),(C,0)) //先将 zeroValue=0 应用于每个 V,注意，这次映射函数为乘法，得到：("A",00),("A",20)， //即：("A",0),("A",0)，再将映射函//数应用于初始化后的 V，最后得到：(A,00)，即：(A,0) //其他 K 也一样，最终都得到了 V=0
scala>rdd1.foldByKey(1)(__).collect res78:Array[(String,Int)]=Array((A,0),(B,2),(C,1)) //映射函数为乘法时，需要将 zeroValue 设为 1，才能得到我们想要的结果。
```
在使用 foldByKey 算子时候，要特别注意映射函数及 zeroValue 的取值。

## reduceByKeyLocally
>defreduceByKeyLocally(func:(V,V)=>V):Map[K,V]
该函数将 RDD[K,V]中每个 K 对应的 V 值根据映射函数来运算，运算结果映射到一个 Map[K,V] 中，而不是 RDD[K,V]。

```
scala>var rdd1=sc.makeRDD(Array(("A",0),("A",2),("B",1),("B",2),("C",1))) 
rdd1:org.apache.spark.rdd.RDD[(String,Int)]=ParallelCollectionRDD[91]atmakeRDDat:21
scala>rdd1.reduceByKeyLocally((x,y)=>x+y) 
res90:scala.collection.Map[String,Int]=Map(B->3,A->2,C->1)
```

## sample
>sample(withReplacement, fraction, seed):以指定的随机种子随机抽样出数量为fraction的数据，withReplacement表示是抽出的数据是否放回，true为有放回的抽样，false为无放回的抽样，seed用于指定随机数生成器种子。例子从RDD中随机且有放回的抽出50%的数据，随机种子值为3（即可能以1 2 3的其中一个起始值） 

```
Val red = sc.makeRDD(1 to 10)
Val sample = rdd.sample(true,0.4,.2)
``

