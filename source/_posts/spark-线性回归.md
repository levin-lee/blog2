---
title: spark-线性回归
date: 2021-06-19 21:24:24
tags:
- spark
- 机器学习
cover: https://pica.zhimg.com/80/v2-82068a2b6e53002cc203a6d47cefcee4_1440w.jpg?source=1940ef5c
---

代码实例
```
import org.apache.hadoop.util.XMLUtils
import org.apache.log4j.{Level, Logger}
import org.apache.spark.ml.classification.LogisticRegressionModel
import org.apache.spark.mllib.classification.LogisticRegressionWithLBFGS
import org.apache.spark.mllib.evaluation.MulticlassMetrics
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.mllib.regression.LabeledPoint


object LogisticRegression {
  def main(args: Array[String]): Unit = {
    //1 构建对象
    val conf = new SparkConf().setAppName("LogisticRegression")
    val sc = new SparkContext(conf)
    Logger.getRootLogger.setLevel(Level.WARN)


    //2 读取样本数据，为libsvm格式
    val  data = MLUtils.loadLibSVMFile(sc,"/Users/lelingshan/Downloads/pandown/MLlib机器学习/数据/sample_libsvm_data.txt")


    //3 处理样本数据
    val splits = data.randomSplit(Array(0.6,0.4),seed=11L)
    val training = splits(0).cache()
    val test = splits(1)


    // 4 新建逻辑回归模型，并训练
    val model = new LogisticRegressionWithLBFGS().setNumClasses(10).run(training)


    // 5 对测试样本进行测试
    val predictionAndLabels = test.map{
      case LabeledPoint(label, features) =>
        val prediction = model.predict(features)
        (prediction,label)
    }
    val print_predict = predictionAndLabels.take(20)
    println("prediction" + "\t" + "label")
    for (i <- 0 to print_predict.length - 1){
      println(print_predict(i)._1+"\t"+print_predict(i)._2)
    }


    // 6 计算误差
    val metrics = new MulticlassMetrics(predictionAndLabels)
    val precision = metrics.precision
    println("Precision = " + precision)


//    // 7保存模型
//    val ModelPath = ""
//    model.save(sc,ModelPath)
//    val sameModel = LogisticRegressionModel.load(sc,ModelPath)
  }
}
```
