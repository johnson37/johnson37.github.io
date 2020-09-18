# rename nested column

We often need to rename one or multiple columns on Spark DataFrame, Especially when a column is nested it becomes complicated. 

## Step 0: create one dataset

```scala
val data = Seq(Row(Row("James ","","Smith"),"36636","M",3000),
  Row(Row("Michael ","Rose",""),"40288","M",4000),
  Row(Row("Robert ","","Williams"),"42114","M",4000),
  Row(Row("Maria ","Anne","Jones"),"39192","F",4000),
  Row(Row("Jen","Mary","Brown"),"","F",-1)
)

val schema = new StructType()
  .add("name",new StructType()
    .add("firstname",StringType)
    .add("middlename",StringType)
    .add("lastname",StringType))
  .add("dob",StringType)
  .add("gender",StringType)
  .add("salary",IntegerType)
  
  val df = spark.createDataFrame(spark.sparkContext.parallelize(data),schema)
df.printSchema()
```
![schema](./spark-rename-column.jpg)

## Step 1: Using Spark withColumnRenamed – To rename DataFrame column name
```scala
df.withColumnRenamed("dob","DateOfBirth")
    .printSchema()
```

## Step 2: Using withColumnRenamed – To rename multiple columns
```scala
val df2 = df.withColumnRenamed("dob","DateOfBirth")
           .withColumnRenamed("salary","salary_amount")
df2.printSchema()
```

## Step 3: Using Spark StructType – To rename a nested column in Dataframe
```scala
val schema2 = new StructType()
    .add("fname",StringType)
    .add("middlename",StringType)
    .add("lname",StringType)
	
df.select(col("name").cast(schema2),
  col("dob"),
  col("gender"),
  col("salary"))
    .printSchema()
```

## Step 4: Using Select – To rename nested elements.
```scala
df.select(col("name.firstname").as("fname"),
  col("name.middlename").as("mname"),
  col("name.lastname").as("lname"),
  col("dob"),col("gender"),col("salary"))
  .printSchema()
  

```

## Step 5: Using Spark DataFrame withColumn – To rename nested columns
When you have nested columns on Spark DatFrame and if you want to rename it, 
use withColumn on a data frame object to create a new column from an existing
 and we will need to drop the existing column. Below example creates a “fname” column
 from “name.firstname” and drops the “name” column
 
```scala
val df4 = df.withColumn("fname",col("name.firstname"))
      .withColumn("mname",col("name.middlename"))
      .withColumn("lname",col("name.lastname"))
      .drop("name")
df4.printSchema()
```

## Step 6: Using col() function – To Dynamically rename all or multiple columns
```scala

val old_columns = Seq("dob","gender","salary","fname","mname","lname")
    val new_columns = Seq("DateOfBirth","Sex","salary","firstName","middleName","lastName")
    val columnsList = old_columns.zip(new_columns).map(f=>{col(f._1).as(f._2)})
    val df5 = df4.select(columnsList:_*)
    df5.printSchema()
```

## Step 7: Using toDF() – To change all columns in a Spark DataFrame
```scala
val newColumns = Seq("newCol1","newCol2","newCol3")
val df = df.toDF(newColumns:_*)
```