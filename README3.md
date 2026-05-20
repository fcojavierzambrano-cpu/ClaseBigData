// practica 3
// importando librerias de MultilayerPerceptronClassifier

scala> import org.apache.spark.ml.classification.MultilayerPerceptronClassifier
     | import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator
import org.apache.spark.ml.classification.MultilayerPerceptronClassifier
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator

// carga datos del archivo sample_data
scala> val data = spark.read.format("libsvm").load("/Users/matinahernandez/Desktop/Proyectos/BigData/sa
mple_data.txt")
26/05/20 13:46:08 WARN LibSVMFileFormat: 'numFeatures' option not specified, determining the number of features by going though the input. If you know the number in advance, please specify it via 'numFeatures' option to avoid the extra scan.
val data: org.apache.spark.sql.DataFrame = [label: double, features: vector]

// asigna valores a un arreglo desde data y asignando los porcentajes 60 y 40 con un seed de 12345

scala> val Array(training, test) = data.randomSplit(Array(0.6, 0.4), seed = 12345)
val training: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: double, features: vector]
val test: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: double, features: vector]

// asigna layers desde un arreglo
scala> val layers = Array[Int](4, 5, 4, 3)
val layers: Array[Int] = Array(4, 5, 4, 3)

// asigna nuevo valor a trainer desde un MultilayerPerceptronClassifier

scala> val trainer = new MultilayerPerceptronClassifier().setLayers(layers).setBlockSize(128).setSeed(1
234L).setMaxIter(100)
val trainer: org.apache.spark.ml.classification.MultilayerPerceptronClassifier = mlpc_8f30d5a72e51

// asigna modelo desde trainer
scala> val model = trainer.fit(training)
val model: org.apache.spark.ml.classification.MultilayerPerceptronClassificationModel = MultilayerPerceptronClassificationModel: uid=mlpc_8f30d5a72e51, numLayers=4, numClasses=3, numFeatures=4

scala> val result = model.transform(test)
val result: org.apache.spark.sql.DataFrame = [label: double, features: vector ... 3 more fields]

scala> val predictionAndLabels = result.select("prediction", "label")
val predictionAndLabels: org.apache.spark.sql.DataFrame = [prediction: double, label: double]

// asigna valor a evaluator con una nueva MulticlassClassificationEvaluator

scala> val evaluator = new MulticlassClassificationEvaluator().setMetricName("accuracy")
val evaluator: org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator = MulticlassClassificationEvaluator: uid=mcEval_849e234a17b7, metricName=accuracy, metricLabel=0.0, beta=1.0, eps=1.0E-15

// imprime resultado de la efectividad del evaluator
scala> println(s"Test set accuracy = ${evaluator.evaluate(predictionAndLabels)}")
