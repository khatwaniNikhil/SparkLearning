# Spark Cluster Tuning Strategies

## 1. Fixing slownes due to data skewness
#### How skewness is introduced
1. spark wide transformations like groupby and join are key based and keys are hashed and mappe to partitions. Thus shuffle(costly operation) of existing data is unavoidable. Post shuffle, hotspots may be created/uneven data distribution. Processing will slow down as other than hotspot partition, rest are idle most of the time.

#### Monitor skewness
##### Spark UI
1. Go to Stages tab in Spark UI: the skewed partitions hang within a stage and don’t seem to progress.
2. Look at the summary metrics, the max column usually has a much larger value. Then we know we have encountered a data skew issue.

##### Partitions count and stats per partition
1. df.withColumn("partitionId", spark_partition_id())
2. df.groupby([df.partitionId]).count().sort(df.partitionId).show()
3. df.alias("left").join(df.alias("right"),"value", "inner").count()
4. check the plan - amount of data in each partition and duration(min, med, max) - is max very high as compared to med
![](https://github.com/khatwaniNikhil/SparkLearning/blob/main/images/spark_df_no_skew_verify_via_self_join_is_sort_merge_join.png)

#### Identify specific piece of code causing skewness 
1. Go to stage tab and capture Whole Stage Code Generation id's  
2. Crossreference the stage code gen id's with SQL plan graphs to know the detail of what’s running in your code.

#### Fixing data skewness
1. Adaptive Query Execution (AQE) setting - spark tries to optimise
2. revisit "spark.sql.shuffle.partitions" value to reduce skewness
3. **salting**: Add the salt key as part of the key as a new column. The newly added key forces Spark to hash the new key to a different hash value

## 2. Structured way to do cluster infra planning
1. https://github.com/khatwaniNikhil/SparkLearning/blob/main/infra_config.md
2. https://help.hitachivantara.com/Documentation/Pentaho/Data_Integration_and_Analytics/9.3/Setup/Determining_Spark_resource_requirements

## 3. Good Tenant config. in case of multiple tenants/teams sharing Spark Infra
set max limits around executors requested by each tenant in multi tenant env. don't abuse Spark dynamic allocation

## 4. Partition metrics to look for
1. Each partition size should be smaller than 200 MB to gain optimized performance.
2. The number of partitions should be 1x to 4x of the number of cores you have to gain optimized performance (which means create a cluster that matches your data scale is also important).


# References
1. https://nealanalytics.com/blog/databricks-spark-jobs-optimization-techniques-shuffle-partition-technique-part-1/
2. https://chengzhizhao.com/deep-dive-into-handling-apache-spark-data-skew/
3. https://www.pepperdata.com/blog/spark-performance-tuning-tips-expert/
