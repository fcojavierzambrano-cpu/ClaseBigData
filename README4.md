// Practica 4
// Asignando librerias de sparksession
scala> import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.SparkSession

// Construyendo seccion de sql
scala> val spark = SparkSession.builder()
     |   .appName("DecisionTreeCreditExample")
     |   .getOrCreate()
26/05/19 20:21:23 WARN SparkSession: Using an existing Spark session; only runtime SQL configurations will take effect.
val spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.classic.SparkSession@6eb0d0ea

// asignando valores al dataset

scala> val data = Seq(
     |   (1.0,0.0,1.0,1.0),
     |   (1.0,0.0,1.0,1.0),
     |   (1.0,1.0,1.0,1.0),
     |   (1.0,0.0,0.0,0.0),
     |   (1.0,1.0,0.0,0.0),
     |   (0.0,1.0,0.0,0.0),
     |   (0.0,1.0,0.0,0.0),
     |   (0.0,0.0,1.0,1.0),
     |   (0.0,1.0,1.0,0.0),
     |   (0.0,0.0,0.0,0.0)
     | ).toDF("income_high","has_debt","good_history","label")
val data: org.apache.spark.sql.DataFrame = [income_high: double, has_debt: double ... 2 more fields]

// desplegando los datos previamente creados

scala> data.show()
+-----------+--------+------------+-----+
|income_high|has_debt|good_history|label|
+-----------+--------+------------+-----+
|        1.0|     0.0|         1.0|  1.0|
|        1.0|     0.0|         1.0|  1.0|
|        1.0|     1.0|         1.0|  1.0|
|        1.0|     0.0|         0.0|  0.0|
|        1.0|     1.0|         0.0|  0.0|
|        0.0|     1.0|         0.0|  0.0|
|        0.0|     1.0|         0.0|  0.0|
|        0.0|     0.0|         1.0|  1.0|
|        0.0|     1.0|         1.0|  0.0|
|        0.0|     0.0|         0.0|  0.0|
+-----------+--------+------------+-----+

// asignando libreria de vectorassembler

scala> import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.feature.VectorAssembler

// creado un nuevo vector assembler utilizando los datos de las columnas income_high,
// has_debt, goof_history

scala> val assembler = new VectorAssembler()
     |   .setInputCols(Array("income_high","has_debt","good_history"))
     |   .setOutputCol("features")
val assembler: org.apache.spark.ml.feature.VectorAssembler = VectorAssembler: uid=vecAssembler_8795036a2ee0, handleInvalid=error, numInputCols=3

// asignando un dataset utilizando los datos del assembler 
// utilizando las columnas feactures y labels

scala> val dataset = assembler.transform(data)
     | dataset.select("features","label").show()
+-------------+-----+
|     features|label|
+-------------+-----+
|[1.0,0.0,1.0]|  1.0|
|[1.0,0.0,1.0]|  1.0|
|[1.0,1.0,1.0]|  1.0|
|[1.0,0.0,0.0]|  0.0|
|[1.0,1.0,0.0]|  0.0|
|[0.0,1.0,0.0]|  0.0|
|[0.0,1.0,0.0]|  0.0|
|[0.0,0.0,1.0]|  1.0|
|[0.0,1.0,1.0]|  0.0|
|    (3,[],[])|  0.0|
+-------------+-----+

val dataset: org.apache.spark.sql.DataFrame = [income_high: double, has_debt: double ... 3 more fields]

// asignando arreglo utilidando el dataser haciendoi un randomplit donde se le asigna el porsentaje de separacion de los valores

scala> val Array(trainingData, testData) = dataset.randomSplit(Array(0.7, 0.3), seed = 42)
val trainingData: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [income_high: double, has_debt: double ... 3 more fields]
val testData: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [income_high: double, has_debt: double ... 3 more fields]

// asignandio libreria de decisiontreeclassifier
scala> import org.apache.spark.ml.classification.DecisionTreeClassifier
     
     // asignando dataset creando una nueva decisiontreeclassifier
     // asinando etiquetas en las columnas

     | val dt = new DecisionTreeClassifier()
     |   .setLabelCol("label")
     |   .setFeaturesCol("features")
     |   .setMaxDepth(3)

     // importando libreria  decisiontreeclassifier
import org.apache.spark.ml.classification.DecisionTreeClassifier
val dt: org.apache.spark.ml.classification.DecisionTreeClassifier = dtc_e1d5f8a37a0f

// creando modelo utilizando el dataset trainingdata

scala> val model = dt.fit(trainingData)
26/05/19 20:24:25 WARN DecisionTreeMetadata: DecisionTree reducing maxBins from 32 to 7 (= number of training instances)
val model: org.apache.spark.ml.classification.DecisionTreeClassificationModel = DecisionTreeClassificationModel: uid=dtc_e1d5f8a37a0f, depth=2, numNodes=5, numClasses=2, numFeatures=3

// imprimiendo modelo mostrando las condiciones en base al features dando como predicion el resultado

scala> println(model.toDebugString)
DecisionTreeClassificationModel: uid=dtc_e1d5f8a37a0f, depth=2, numNodes=5, numClasses=2, numFeatures=3
  If (feature 1 <= 0.5)
   If (feature 2 <= 0.5)
    Predict: 0.0
   Else (feature 2 > 0.5)
    Predict: 1.0
  Else (feature 1 > 0.5)
   Predict: 0.0

//  asignado la prediction basandose al modelo del testdata

scala> val predictions = model.transform(testData)
val predictions: org.apache.spark.sql.DataFrame = [income_high: double, has_debt: double ... 6 more fields]

// se selecciona la prediccion en base a las columnas mencionadas en la condicion asignada

scala> predictions.select("features","label","prediction","probability").show(false)
+-------------+-----+----------+-----------+
|features     |label|prediction|probability|
+-------------+-----+----------+-----------+
|[1.0,0.0,1.0]|1.0  |1.0       |[0.0,1.0]  |
|[1.0,1.0,1.0]|1.0  |0.0       |[1.0,0.0]  |
|[1.0,0.0,0.0]|0.0  |0.0       |[1.0,0.0]  |
+-------------+-----+----------+-----------+

// importando libreria multiclassclassificationevaluator

scala> import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator

// asignando nuevo multiclassclassificationevaluator a la variable evaluator

scala> val evaluator = new MulticlassClassificationEvaluator()
     |   .setLabelCol("label")
     |   .setPredictionCol("prediction")
     |   .setMetricName("accuracy")
val evaluator: org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator = MulticlassClassificationEvaluator: uid=mcEval_50a2e35f667f, metricName=accuracy, metricLabel=0.0, beta=1.0, eps=1.0E-15

// asinando eficacia desde las prediciones del evaluator
scala> val accuracy = evaluator.evaluate(predictions)
val accuracy: Double = 0.6666666666666666

// imprimiendo etiqueta mas valor de eficacia

scala> println("Accuracy = " + accuracy)
Accuracy = 0.6666666666666666