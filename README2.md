// Practica 2
// importando librerias de logisticregresion y sparksession

scala> import org.apache.spark.ml.classification.LogisticRegression
     | import org.apache.spark.sql.SparkSession
import org.apache.spark.ml.classification.LogisticRegression
import org.apache.spark.sql.SparkSession

scala> import org.apache.log4j._
     | Logger.getLogger("org").setLevel(Level.ERROR)
import org.apache.log4j._

scala> val spark = SparkSession.builder().getOrCreate()
val spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.classic.SparkSession@641c872d

// asignando datos desde archivo adverstising.csv

scala> val data  = spark.read.option("header","true").option("inferSchema", "true").format("csv").load("/Users/matinahernandez/Deskto
p/Proyectos/BigData/Spark_LogisticRegression/advertising.csv")
val data: org.apache.spark.sql.DataFrame = [Daily Time Spent on Site: double, Age: int ... 8 more fields]

// desplegando schema del dataset data
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


scala> data.head(1)
val res2: Array[org.apache.spark.sql.Row] = Array([68.95,35,61833.9,256.09,Cloned 5thgeneration orchestration,Wrightburgh,0,Tunisia,2016-03-27 00:53:11.0,0])

// asigna valores a variable de las columnas del dataset

scala> val colnames = data.columns
val colnames: Array[String] = Array(Daily Time Spent on Site, Age, Area Income, Daily Internet Usage, Ad Topic Line, City, Male, Country, Timestamp, Clicked on Ad)

// asigna valores del encabezado del primer registro del data set
scala> val firstrow = data.head(1)(0)
val firstrow: org.apache.spark.sql.Row = [68.95,35,61833.9,256.09,Cloned 5thgeneration orchestration,Wrightburgh,0,Tunisia,2016-03-27 00:53:11.0,0]

scala> println("\n")

// desplega texto en pantalla
scala> println("Example data row")
Example data row

// <Despliega el contenido del primer registro por campo y valor>
// ejecuta un ciclo donde se le otorga un rango para su ejecucion

scala> for(ind <- Range(1, colnames.length)){
     |     println(colnames(ind))
     |     println(firstrow(ind))
     |     println("\n")
     | }
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


scala> val timedata = data.withColumn("Hour",hour(data("Timestamp")))
val timedata: org.apache.spark.sql.DataFrame = [Daily Time Spent on Site: double, Age: int ... 9 more fields]

scala> val logregdata = timedata.select(data("Clicked on Ad").as("label"), $"Daily Time Spent on Site", $"Age", $"Area Income", $"Dai
ly Internet Usage", $"Hour", $"Male")
     | 
val logregdata: org.apache.spark.sql.DataFrame = [label: int, Daily Time Spent on Site: double ... 5 more fields]

scala> import org.apache.spark.ml.feature.VectorAssembler
     | import org.apache.spark.ml.linalg.Vectors
import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.linalg.Vectors

scala> val assembler = (new VectorAssembler()
     |                   .setInputCols(Array("Daily Time Spent on Site", "Age","Area Income","Daily Internet Usage","Hour","Male"))
     |                   .setOutputCol("features"))
val assembler: org.apache.spark.ml.feature.VectorAssembler = VectorAssembler: uid=vecAssembler_483ea4da3e75, handleInvalid=error, numInputCols=6

// importa librerias para el vectorassembler

scala> import org.apache.spark.ml.feature.VectorAssembler
     | import org.apache.spark.ml.linalg.Vectors
import org.apache.spark.ml.feature.VectorAssembler
import org.apache.spark.ml.linalg.Vectors

// asigna variable assembler con un nuevo vector creando un nuevo encabezado
scala> val assembler = (new VectorAssembler()
     |                   .setInputCols(Array("Daily Time Spent on Site", "Age","Area Income","Daily Internet Usage","Hour","Male"))
     |                   .setOutputCol("features"))
val assembler: org.apache.spark.ml.feature.VectorAssembler = VectorAssembler: uid=vecAssembler_483ea4da3e75, handleInvalid=error, numInputCols=6

// asigna valores a trainnig y test utilizando el randomsplit y asignando los porsentajes del 70% y 30% y utilizando una seed de 12345 

scala> val Array(training, test) = logregdata.randomSplit(Array(0.7, 0.3), seed = 12345)
val training: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: int, Daily Time Spent on Site: double ... 5 more fields]
val test: org.apache.spark.sql.Dataset[org.apache.spark.sql.Row] = [label: int, Daily Time Spent on Site: double ... 5 more fields]

// importando libreria pipeline

scala> import org.apache.spark.ml.Pipeline
import org.apache.spark.ml.Pipeline

// asignando nuevo valor de logisticregression

scala> val lr = new LogisticRegression()
val lr: org.apache.spark.ml.classification.LogisticRegression = logreg_8f6a58688951

// asignando nuevo pipeline 
scala> val pipeline = new Pipeline().setStages(Array(assembler, lr))
val pipeline: org.apache.spark.ml.Pipeline = pipeline_c3533a4ca647

// asignando nuevo modelo usando el pipeline de training
scala> val model = pipeline.fit(training)
26/05/20 12:34:51 WARN InstanceBuilder: Failed to load implementation from:dev.ludovic.netlib.blas.JNIBLAS
val model: org.apache.spark.ml.PipelineModel = pipeline_c3533a4ca647

// asignando resultado del modelo transformado de test

scala> val results = model.transform(test)
val results: org.apache.spark.sql.DataFrame = [label: int, Daily Time Spent on Site: double ... 9 more fields]

// importando libreria de multiclassmetrics

scala> import org.apache.spark.mllib.evaluation.MulticlassMetrics
import org.apache.spark.mllib.evaluation.MulticlassMetrics

// asignando prediccion desde el resultado del dataset
scala> val predictionAndLabels = results.select($"prediction",$"label").as[(Double, Double)].rdd
val predictionAndLabels: org.apache.spark.rdd.RDD[(Double, Double)] = MapPartitionsRDD[69] at rdd at <console>:1

// asignando nuevos metricos desde la variable predictionandlabels 
scala> val metrics = new MulticlassMetrics(predictionAndLabels)
val metrics: org.apache.spark.mllib.evaluation.MulticlassMetrics = org.apache.spark.mllib.evaluation.MulticlassMetrics@4f9e8566

// imprimiendo texto de confusion matrix
scala> println("Confusion matrix:")
Confusion matrix:

// imprimiendo metricos desde confusionmatrix
scala> println(metrics.confusionMatrix)
136.0  1.0    
4.0    146.0  

// asignando metricos de efectividad
scala> metrics.accuracy
val res8: Double = 0.9825783972125436