# Spark Distinct

Duplicate rows could be remove or drop from Spark DataFrame using distinct() and dropDuplicates() functions, 
distinct() can be used to remove rows that have the same values on all columns 
whereas dropDuplicates() can be used to remove rows that have the same values on multiple selected columns.

## Step 0: Create one dataset

```scala
import spark.implicits._

val simpleData = Seq(("James", "Sales", 3000),
  ("Michael", "Sales", 4600),
  ("Robert", "Sales", 4100),
  ("Maria", "Finance", 3000),
  ("James", "Sales", 3000),
  ("Scott", "Finance", 3300),
  ("Jen", "Finance", 3900),
  ("Jeff", "Marketing", 3000),
  ("Kumar", "Marketing", 2000),
  ("Saif", "Sales", 4100)
)
val df = simpleData.toDF("employee_name", "department", "salary")
df.show()
```

## Step 1: Use distinct() to remove duplicate rows on DataFrame
```scala
//Distinct all columns
val distinctDF = df.distinct()
println("Distinct count: "+distinctDF.count())
distinctDF.show(false)
```

## Step 2: Use dropDuplicate() to remove duplicate rows on DataFrame
Spark doesn’t have a distinct method which takes columns
```scala
//Distinct using dropDuplicates
val dropDisDF = df.dropDuplicates("department","salary")
println("Distinct count of department & salary : "+dropDisDF.count())
dropDisDF.show(false)
```