# Spark Filter

Spark filter() or where() function is used to filter the rows from DataFrame or Dataset based on the given condition or SQL expression.

## Filter Synax
```
1) filter(condition: Column): Dataset[T]
2) filter(conditionExpr: String): Dataset[T] //using SQL expression
3) filter(func: T => Boolean): Dataset[T]
4) filter(func: FilterFunction[T]): Dataset[T]
```

## Step 0 create a DataFrame.
```scala
val arrayStructureData = Seq(
    Row(Row("James","","Smith"),List("Java","Scala","C++"),"OH","M"),
    Row(Row("Anna","Rose",""),List("Spark","Java","C++"),"NY","F"),
    Row(Row("Julia","","Williams"),List("CSharp","VB"),"OH","F"),
    Row(Row("Maria","Anne","Jones"),List("CSharp","VB"),"NY","M"),
    Row(Row("Jen","Mary","Brown"),List("CSharp","VB"),"NY","M"),
    Row(Row("Mike","Mary","Williams"),List("Python","VB"),"OH","M")
)

val arrayStructureSchema = new StructType()
    .add("name",new StructType()
    .add("firstname",StringType)
    .add("middlename",StringType)
    .add("lastname",StringType))
    .add("languages", ArrayType(StringType))
    .add("state", StringType)
    .add("gender", StringType)

val df = spark.createDataFrame(
    spark.sparkContext.parallelize(arrayStructureData),arrayStructureSchema)
df.printSchema()
df.show()
```

## Step 1 DataFrame filter() with Column condition
```scala
df.filter(df("state") === "OH")
    .show(false)
```

## Step 2 Filtering with multiple conditions
```scala
//multiple condition
df.filter(df("state") === "OH" && df("gender") === "M")
    .show(false)
```

## Step 3: Filtering on an Array column
```scala
df.filter(array_contains(df("languages"),"Java"))
    .show(false)
```
## Step 4: Filtering on Nested Struct columns
```scala
df.filter(df("name.lastname") === "Williams")
    .show(false)
```