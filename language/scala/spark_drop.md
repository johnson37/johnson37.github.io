# Spark_drop

Spark DataFrame provides a drop() method to drop a column/field from a DataFrame/Dataset. drop() method also used to remove multiple columns at a time from a Spark DataFrame/Dataset. 

## Drop
```
1) drop(colName : scala.Predef.String) : org.apache.spark.sql.DataFrame
2) drop(colNames : scala.Predef.String*) : org.apache.spark.sql.DataFrame
3) drop(col : org.apache.spark.sql.Column) : org.apache.spark.sql.DataFrame
```

## Step 0 create one dataset
```
  val structureData = Seq(
    Row("James","","Smith","36636","NewYork",3100),
    Row("Michael","Rose","","40288","California",4300),
    Row("Robert","","Williams","42114","Florida",1400),
    Row("Maria","Anne","Jones","39192","Florida",5500),
    Row("Jen","Mary","Brown","34561","NewYork",3000)
  )

  val structureSchema = new StructType()
    .add("firstname",StringType)
    .add("middlename",StringType)
    .add("lastname",StringType)
    .add("id",StringType)
    .add("location",StringType)
    .add("salary",IntegerType)

  val df = spark.createDataFrame(
    spark.sparkContext.parallelize(structureData),structureSchema)
  df.printSchema()
```

## Step 1 Drop one column from DataFrame
```scala
  val df2 = df.drop("firstname") //First signature
  df2.printSchema()

  df.drop(df("firstname")).printSchema()

  //import org.apache.spark.sql.functions.col is required
  df.drop(col("firstname")).printSchema() //Third signature
```

## Step 2 Drop multiple columns from DataFrame
```scala
  //Refering more than one column
  df.drop("firstname","middlename","lastname")
    .printSchema()

  // using array/sequence of columns
  val cols = Seq("firstname","middlename","lastname")
  df.drop(cols:_*)
    .printSchema()
```