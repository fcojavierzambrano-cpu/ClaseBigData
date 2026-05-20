
// abrimos librerias de claisificacion - SparkSession
scala> import org.apache.spark.ml.classification.LogisticRegression
     | import org.apache.spark.sql.SparkSession
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.sql.SparkSession

// abrimimos libreria de level de error 
scala> import org.apache.log4j._
     | Logger.getLogger("org").setLevel(Level.ERROR)
import org.apache.log4j._

// creamos sesion 
scala> val spark = SparkSession.builder().getOrCreate()
val spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.classic.SparkSession@76be352d

// abrimos archivo de csv
scala> val data  = spark.read.option("header","true").option("inferSchema", "true").format("csv").load("/Users/matinahernandez/Desktop/P
royectos/BigData/Spark_LogisticRegression/advertising.csv")
val data: org.apache.spark.sql.DataFrame = [Daily Time Spent on Site: double, Age: int ... 8 more fields]

// imprimis Schema
scala> data.printSchema()
root
 |-- Daily Time Spent on Site: double (nullable = true)
 |-- Age: integer (nullable = true)
 |-- Area Income: double (nullable = true)
 |-- Daily Internet Usage: double (nullable = true)
 |-- Ad Topic Line: string (nullable = true)
 |-- City: string (nullable = true)
 |-- Male: integer (nullable = true)
 |-- Country: string (nullable = true)
 |-- Timestamp: timestamp (nullable = true)
 |-- Clicked on Ad: integer (nullable = true)

// mostramos encabezado
scala> data.head(1)
val res2: Array[org.apache.spark.sql.Row] = Array([68.95,35,61833.9,256.09,Cloned 5thgeneration orchestration,Wrightburgh,0,Tunisia,2016-03-27 00:53:11.0,0])

scala> val colnames = data.columns
     | val firstrow = data.head(1)(0)
     | println("\n")
     | println("Example data row")
     | for(ind <- Range(1, colnames.length)){
     |     println(colnames(ind))
     |     println(firstrow(ind))
     |     println("\n")
     | }


Example data row
Age
35


Area Income
61833.9


Daily Internet Usage
256.09


Ad Topic Line
Cloned 5thgeneration orchestration


City
Wrightburgh


Male
0


Country
Tunisia


Timestamp
2016-03-27 00:53:11.0


Clicked on Ad
0

// asignado nombres de columnas en un arreglo
val colnames: Array[String] = Array(Daily Time Spent on Site, Age, Area Income, Daily Internet Usage, Ad Topic Line, City, Male, Country, Timestamp, Clicked on Ad)
val firstrow: org.apache.spark.sql.Row = [68.95,35,61833.9,256.09,Cloned 5thgeneration orchestration,Wrightburgh,0,Tunisia,2016-03-27 00:53:11.0,0]

// asignando hora en formato timesstamp
scala> val timedata = data.withColumn("Hour",hour(data("Timestamp")))
     | 
val timedata: org.apache.spark.sql.DataFrame = [Daily Time Spent on Site: double, Age: int ... 9 more fields]

scala> val logregdata = timedata.select(data("Clicked on Ad").as("label"), $"Daily Time Spent on Site", $"Age", $"Area Income", $"Daily 
Internet Usage", $"Hour", $"Male")
     | 
val logregdata: org.apache.spark.sql.DataFrame = [label: int, Daily Time Spent on Site: double ... 5 more fields]

// importando libreria de vector assembler
scala> import org.apache.spark.ml.feature.VectorAssembler
     | import org.apache.spark.ml.linalg.Vectors
     | 
import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.linalg.Vectors

// asignado un valores de un assembler
scala> val assembler = (new VectorAssembler()
     |                   .setInputCols(Array("Daily Time Spent on Site", "Age","Area Income","Daily Internet Usage","Hour","Male"))
     |                   .setOutputCol("features"))
val assembler: org.apache.spark.ml.feature.VectorAssembler = VectorAssembler: uid=vecAssembler_3ffc7e9a0924, handleInvalid=error, numInputCols=6

// creado arreglo con porcentajes de training y test

scala> val Array(training, test) = logregdata.randomSplit(Array(0.7, 0.3), seed = 12345)
val training: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: int, Daily Time Spent on Site: double ... 5 more fields]
val test: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: int, Daily Time Spent on Site: double ... 5 more fields]

// asignandio libreria de pipeline
scala> import org.apache.spark.ml.Pipeline
     | 
     | val lr = new LogisticRegression()
import org.apache.spark.ml.Pipeline
val lr: org.apache.spark.ml.classification.LogisticRegression = logreg_c3ed88479dbb

// asinando nuevo pipeline desde un arreglo
scala> val pipeline = new Pipeline().setStages(Array(assembler, lr))
val pipeline: org.apache.spark.ml.Pipeline = pipeline_e0d557a0819f

// asignando un modelo training
scala> val model = pipeline.fit(training)
26/05/19 19:09:29 WARN InstanceBuilder: Failed to load implementation from:dev.ludovic.netlib.blas.JNIBLAS
val model: org.apache.spark.ml.PipelineModel = pipeline_e0d557a0819f

// asignando resultado del modelo test
scala> val results = model.transform(test)
val results: org.apache.spark.sql.DataFrame = [label: int, Daily Time Spent on Site: double ... 9 more fields]

scala> import org.apache.spark.mllib.evaluation.MulticlassMetrics
import org.apache.spark.mllib.evaluation.MulticlassMetrics

scala> val predictionAndLabels = results.select($"prediction",$"label").as[(Double, Double)].rdd
val predictionAndLabels: org.apache.spark.rdd.RDD[(Double, Double)] = MapPartitionsRDD[69] at rdd at <console>:1

// asignando metricos desde valores de prediccion y etiquetas
scala> val metrics = new MulticlassMetrics(predictionAndLabels)
val metrics: org.apache.spark.mllib.evaluation.MulticlassMetrics = org.apache.spark.mllib.evaluation.MulticlassMetrics@7f099be1

// imprimiendo texto
scala> println("Confusion matrix:")
Confusion matrix:

// imprimiendo matrix de confusion
scala> println(metrics.confusionMatrix)
136.0  1.0    
4.0    146.0  

// mostrando matriz de efectividad
scala> metrics.accuracy
val res6: Double = 0.9825783972125436