// Practica Evaluatoria
// Cargar en un dataframe de la fuente de datos Iris.csv

scala> import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.SparkSession

scala> val spark = SparkSession.builder().getOrCreate()
val spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.classic.SparkSession@505ba679

scala> import org.apache.log4j._
     | Logger.getLogger("org").setLevel(Level.ERROR)
import org.apache.log4j._

scala> val data  = spark.read.option("header","true").option("inferSchema", "true").format("csv").load("/Users/matinaher
nandez/Desktop/Proyectos/BigData/iris.csv")
val data: org.apache.spark.sql.DataFrame = [sepal_length: double, sepal_width: double ... 3 more fields]


// 2. ¿Cuáles son los nombres de las columnas?
scala> data.show
warning: 1 deprecation (since 2.13.3); for details, enable `:setting -deprecation` or `:replay -deprecation`
+------------+-----------+------------+-----------+-------+
|sepal_length|sepal_width|petal_length|petal_width|species|
+------------+-----------+------------+-----------+-------+
|         5.1|        3.5|         1.4|        0.2| setosa|
|         4.9|        3.0|         1.4|        0.2| setosa|
|         4.7|        3.2|         1.3|        0.2| setosa|
|         4.6|        3.1|         1.5|        0.2| setosa|
|         5.0|        3.6|         1.4|        0.2| setosa|
|         5.4|        3.9|         1.7|        0.4| setosa|
|         4.6|        3.4|         1.4|        0.3| setosa|
|         5.0|        3.4|         1.5|        0.2| setosa|
|         4.4|        2.9|         1.4|        0.2| setosa|
|         4.9|        3.1|         1.5|        0.1| setosa|
|         5.4|        3.7|         1.5|        0.2| setosa|
|         4.8|        3.4|         1.6|        0.2| setosa|
|         4.8|        3.0|         1.4|        0.1| setosa|
|         4.3|        3.0|         1.1|        0.1| setosa|
|         5.8|        4.0|         1.2|        0.2| setosa|
|         5.7|        4.4|         1.5|        0.4| setosa|
|         5.4|        3.9|         1.3|        0.4| setosa|
|         5.1|        3.5|         1.4|        0.3| setosa|
|         5.7|        3.8|         1.7|        0.3| setosa|
|         5.1|        3.8|         1.5|        0.3| setosa|
+------------+-----------+------------+-----------+-------+
only showing top 20 rows

// Pregunta 3
scala> data.printSchema()
root
 |-- sepal_length: double (nullable = true)
 |-- sepal_width: double (nullable = true)
 |-- petal_length: double (nullable = true)
 |-- petal_width: double (nullable = true)
 |-- species: string (nullable = true)

// Pregunta 4
// 4. Imprime las primeras 5 columnas.

scala> data.show(5)
+------------+-----------+------------+-----------+-------+
|sepal_length|sepal_width|petal_length|petal_width|species|
+------------+-----------+------------+-----------+-------+
|         5.1|        3.5|         1.4|        0.2| setosa|
|         4.9|        3.0|         1.4|        0.2| setosa|
|         4.7|        3.2|         1.3|        0.2| setosa|
|         4.6|        3.1|         1.5|        0.2| setosa|
|         5.0|        3.6|         1.4|        0.2| setosa|
+------------+-----------+------------+-----------+-------+
only showing top 5 rows

// Pregunta 5

scala> data.describe().show()
+-------+------------------+-------------------+------------------+------------------+---------+
|summary|      sepal_length|        sepal_width|      petal_length|       petal_width|  species|
+-------+------------------+-------------------+------------------+------------------+---------+
|  count|               150|                150|               150|               150|      150|
|   mean| 5.843333333333335| 3.0540000000000007|3.7586666666666693|1.1986666666666672|     NULL|
| stddev|0.8280661279778637|0.43359431136217375| 1.764420419952262|0.7631607417008414|     NULL|
|    min|               4.3|                2.0|               1.0|               0.1|   setosa|
|    max|               7.9|                4.4|               6.9|               2.5|virginica|
+-------+------------------+-------------------+------------------+------------------+---------+

// Pregunta 6

scala> val Array(training, test) = data.randomSplit(Array(0.7, 0.3), seed = 12000)
val training: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [sepal_length: double, sepal_width: double ... 3 more fields]
val test: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [sepal_length: double, sepal_width: double ... 3 more fields]

scala> val layers = Array[Int](4, 5, 4, 3)
val layers: Array[Int] = Array(4, 5, 4, 3)

import org.apache.spark.ml.classification.MultilayerPerceptronClassifier
     | import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator
import org.apache.spark.ml.classification.MultilayerPerceptronClassifier
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator

scala> val trainer = new MultilayerPerceptronClassifier().setLayers(layers).setBlockSize(128).setSeed(1234L).setMaxIter(
100)
val trainer: org.apache.spark.ml.classification.MultilayerPerceptronClassifier = mlpc_b85b4eb1c53c

// Pregunta 7
import org.apache.spark.mllib.classification.{NaiveBayes, NaiveBayesModel}
import org.apache.spark.mllib.util.MLUtils

scala> val data = MLUtils.loadLibSVMFile(sc, "/Users/matinahernandez/Desktop/Proyectos/BigData/sample_data.txt")
val data: org.apache.spark.rdd.RDD[org.apache.spark.mllib.regression.LabeledPoint] = MapPartitionsRDD[46] at map at MLUtils.scala:88

scala> val model = NaiveBayes.train(training, lambda = 1.0, modelType = "multinomial")
val model: org.apache.spark.mllib.classification.NaiveBayesModel = org.apache.spark.mllib.classification.NaiveBayesModel@4b19b2db

// la arquetectura que utilize fue Naivebayes ya que con el archivo <.csv> no logre crear el modelo se estuvo marcando error el cual no logre desifrar

// Pregunta 8

scala> val predictionAndLabel = test.map(p => (model.predict(p.features), p.label))
     | val accuracy = 1.0 * predictionAndLabel.filter(x => x._1 == x._2).count() / test.count()
val predictionAndLabel: org.apache.spark.rdd.RDD[(Double, Double)] = MapPartitionsRDD[62] at map at <console>:1
val accuracy: Double = 1.0

// como resultado nos mostro un efectividad doble equivalente a 1.0
// las ultimas 2 pregunta no fueron muy bien entendidas ya que por el archivo iris.csv no se logro 
// generar el modelo de forma correcta.