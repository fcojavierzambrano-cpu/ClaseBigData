// Practica # 6
// Asignando Librerias de naivebayes

scala> import org.apache.spark.mllib.classification.{NaiveBayes, NaiveBayesModel}
     | import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.mllib.classification.{NaiveBayes, NaiveBayesModel}
import org.apache.spark.mllib.util.MLUtils

// Cargando archivo desde la ruta asinada
scala> val data = MLUtils.loadLibSVMFile(sc, "/Users/matinahernandez/Desktop/Proyectos/BigData/sample_d
ata.txt")
     | 
val data: org.apache.spark.rdd.RDD[org.apache.spark.mllib.regression.LabeledPoint] = MapPartitionsRDD[6] at map at MLUtils.scala:88

// Asignando la separacion de porcentajes 60% training 40% test
scala> val Array(training, test) = data.randomSplit(Array(0.6, 0.4))
     | 
val training: org.apache.spark.rdd.RDD[org.apache.spark.mllib.regression.LabeledPoint] = MapPartitionsRDD[7] at randomSplit at <console>:1
val test: org.apache.spark.rdd.RDD[org.apache.spark.mllib.regression.LabeledPoint] = MapPartitionsRDD[8] at randomSplit at <console>:1

// asignado el modelo naivebayes utilizando el typo multinomial
// creando la prediccion en base el modelo creado previamente
// asignando la efectividad calculando la prediccion entre el conteo de test

scala> val model = NaiveBayes.train(training, lambda = 1.0, modelType = "multinomial")
     | val predictionAndLabel = test.map(p => (model.predict(p.features), p.label))
     | val accuracy = 1.0 * predictionAndLabel.filter(x => x._1 == x._2).count() / test.count()
val model: org.apache.spark.mllib.classification.NaiveBayesModel = org.apache.spark.mllib.classification.NaiveBayesModel@46b6ce71
val predictionAndLabel: org.apache.spark.rdd.RDD[(Double, Double)] = MapPartitionsRDD[24] at map at <console>:2
val accuracy: Double = 1.0

// se crea modelo salvando en archivo temponal dentro de una ruta asignada
// se la ruta ya existe y archivo tambien enviar mensaje de error indicando que ya existe

scala> model.save(sc, "target/tmp1/myNaiveBayesModel")

// en este paso se esta cargando el archivo en una variable del modelo

scala> val sameModel = NaiveBayesModel.load(sc, "target/tmp/myNaiveBayesModel")
val sameModel: org.apache.spark.mllib.classification.NaiveBayesModel = org.apache.spark.mllib.classification.NaiveBayesModel@1a04a12
