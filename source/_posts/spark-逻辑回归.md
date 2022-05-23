---
title: spark-逻辑回归
date: 2021-06-19 21:26:50
tags:
- spark
- 机器学习
cover: https://pica.zhimg.com/80/v2-47e51546ebf1aa028f641bfe578e07b0_1440w.jpg?source=1940ef5c
---

代码实例
```
import org.apache.log4j.{ Level, Logger }
import org.apache.spark.{ SparkConf, SparkContext }
import org.apache.spark.mllib.regression.LinearRegressionWithSGD
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.Vectors
import org.apache.spark.mllib.regression.LinearRegressionModel

object LinearRegression {
  def main(args: Array[String]): Unit = {
    //  构建对象
    val conf = new SparkConf().setAppName("LinearRegressionWithSGD")
    val sc = new SparkContext(conf)
    Logger.getRootLogger.setLevel(Level.WARN)
    // 读取样本数据（1）
      val data_path1 = "/Users/lelingshan/Downloads/pandown/MLlib机器学习/数据/lpsa.data"
      val data = sc.textFile(data_path1)
      val examples = data.map{line =>
        val parts = line.split(',')
        LabeledPoint(parts(0).toDouble, Vectors.dense(parts(1).split(' ').map(_.toDouble)))
      }.cache()
      val numExample = examples.count()


    // 读取样本数据（1）
//    val data_path2 = "/Users/lelingshan/Downloads/pandown/MLlib机器学习/数据/lpsa.data"
//    val examples = MLUtils.loadLibSVMFile(sc,data_path2).cache()
//    val numExample = examples.count()
    // 新建线性回归模型，并设置参数
    val numIterations = 100
    val stepSize = 1
    val miniBatchFraction = 1.0
    val model = LinearRegressionWithSGD.train(examples,numIterations,stepSize,miniBatchFraction)
    // 对样本进行预测
    val prediction = model.predict(examples.map(_.features))
    val predictionAndLabel = prediction.zip(examples.map(_.label))
    val print_predict = predictionAndLabel.take(50)
    println("prediction"+"\t"+"label")
    for(i <- 0 to print_predict.length-1){
      println(print_predict(i)._1+"\t"+print_predict(i)._2)
    }
    // 进行测试误差
    val loss = predictionAndLabel.map{
        case(v,p) => math.pow((v-p),2)}.mean()
    println(s"Test RMSE = $loss.")


    // 模型保存
//    val ModelPath = "/User/Downloads/pandown/MLlib机器学习/"
//    model.save(sc,ModelPath)
//    val sameModel = LinearRegressionModel.load(sc,ModelPath)
  }
}-
```
