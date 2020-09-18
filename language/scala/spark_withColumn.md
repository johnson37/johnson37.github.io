# Spark_WithColumn

Spark withColumn() function is used to rename, change the value, convert the datatype of an existing DataFrame column and also can be used to create a new column

## Step 0
```
val data = Seq(Row(Row("James ","","Smith"),"36636","M","3000"),
      Row(Row("Michael ","Rose",""),"40288","M","4000"),
      Row(Row("Robert ","","Williams"),"42114","M","4000"),
      Row(Row("Maria ","Anne","Jones"),"39192","F","4000"),
      Row(Row("Jen","Mary","Brown"),"","F","-1")
)

val schema = new StructType()
      .add("name",new StructType()
      .add("firstname",StringType)
      .add("middlename",StringType)
      .add("lastname",StringType))
      .add("dob",StringType)
      .add("gender",StringType)
      .add("salary",StringType)

val df = spark.createDataFrame(spark.sparkContext.parallelize(data),schema)
```

## Step 1 To change column DataType
```scala
df.withColumn("salary",col("salary").cast("Integer"))
```

## Step 2 Change the value of an existing column

```scala
df.withColumn("salary",col("salary")*100)
```

## Step 3 Derive new column from an existing column
```scala
df.withColumn("CopiedColumn",col("salary")* -1)
```
## Step 4  Add a new column
```scala
df.withColumn("Country", lit("USA"))

//chaining to operate on multiple columns
df.withColumn("Country", lit("USA"))
   .withColumn("anotherColumn",lit("anotherValue"))
```
## Step 5 Rename DataFrame column name
```scala
df.withColumnRenamed("gender","sex")
```

## Step 6 Drop a column from Spark DataFrame
```scala
df.drop("CopiedColumn")
```