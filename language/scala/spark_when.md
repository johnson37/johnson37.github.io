# Spark When

## Step 0: Create one dataset
```scala
import org.apache.spark.sql.functions.{when, _}
val spark: SparkSession = SparkSession.builder()
      .master("local[1]")
      .appName("SparkByExamples.com")
      .getOrCreate()

import spark.sqlContext.implicits._
val data = List(("James","","Smith","36636","M",60000),
        ("Michael","Rose","","40288","M",70000),
        ("Robert","","Williams","42114","",400000),
        ("Maria","Anne","Jones","39192","F",500000),
        ("Jen","Mary","Brown","","F",0))

val cols = Seq("first_name","middle_name","last_name","dob","gender","salary")
val df = spark.createDataFrame(data).toDF(cols:_*)
```

## Step 1 Using “when otherwise” on Spark DataFrame.
```scala
val df2 = df.withColumn("new_gender", when(col("gender") === "M","Male")
      .when(col("gender") === "F","Female")
      .otherwise("Unknown"))
```

```scala
val df4 = df.select(col("*"), when(col("gender") === "M","Male")
      .when(col("gender") === "F","Female")
      .otherwise("Unknown").alias("new_gender"))
```

## Step 2 Using && and || operator
```scala
val dataDF = Seq(
      (66, "a", "4"), (67, "a", "0"), (70, "b", "4"), (71, "d", "4"
      )).toDF("id", "code", "amt")
dataDF.withColumn("new_column",
       when(col("code") === "a" || col("code") === "d", "A")
      .when(col("code") === "b" && col("amt") === "4", "B")
      .otherwise("A1"))
      .show()
```