# References
1. https://engineering.salesforce.com/how-to-optimize-your-apache-spark-application-with-partitions-257f2c1bb414/
2. https://chengzhizhao.com/deep-dive-into-handling-apache-spark-data-skew/
3. https://nealanalytics.com/blog/databricks-spark-jobs-optimization-techniques-shuffle-partition-technique-part-1/
4. https://www.pepperdata.com/blog/spark-performance-tuning-tips-expert/
5. https://spark.apache.org/docs/latest/sql-performance-tuning.html

# Spark Cluster Tuning Strategies

## Correct Partitions count 
https://engineering.salesforce.com/how-to-optimize-your-apache-spark-application-with-partitions-257f2c1bb414/
1. capture narrow and wide transformations happening in the current business logic. Notice that often when we have x amount of partitions and we are doing a wide transformation (i.e. groupBy), Spark will first groupBy in each initial partition and only then shuffle the data, partition it by key, and groupBy again in each shuffled partition, to increase efficiency and reduce the amount of rows while shuffling
2. capture the partitions count per stage and identify which stage partitions needs tuning to improve that stage completion time.
3. control the no. of partitions for offending stage using different spark config's like
   1. spark.sql.files.maxPartitionBytes
   2. spark.sql.files.minPartitionNum
   3. spark.sql.adaptive.advisoryPartitionSizeInBytes
   4. spark.sql.adaptive.coalescePartitions.initialPartitionNum
   5. spark.sql.adaptive.coalescePartitions.parallelismFirst
5. Recheck the cluster resources
   1. To start with - Spark’s official recommendation is that you have ~3x the number of partitions than available cores in cluster
   2. If our tasks are rather simple (taking less than a second), we might want to consider decreasing our partitions (to avoid the overhead), even if it means less than 3x
   3. If memory of each executor is overflowing, we might want to create more partitions, even if it’s more than 3x our available cores, so that each will be smaller, or increase the memory of our executors.
6. Data cardinality to double check the no. of partition are correct
   1. Partioning key might needs to be transformed(salting technique etc.) if data is distributed as evenly as possible across your different partitions

## 1. Data skewness debug and fix
https://chengzhizhao.com/deep-dive-into-handling-apache-spark-data-skew/

#### How skewness is introduced
1. spark wide transformations like groupby and join are key based and keys are hashed and mappe to partitions. Thus shuffle(costly operation) of existing data is unavoidable. Post shuffle, hotspots may be created/uneven data distribution. Processing will slow down as other than hotspot partition, rest are idle most of the time.
2. Filtering operation can lead to uneven partitions size post filtering.

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
2. revisit "spark.sql.shuffle.partitions" value to reduce skewness.
https://engineering.salesforce.com/how-to-optimize-your-apache-spark-application-with-partitions-257f2c1bb414/
Spark’s official recommendation is that you have ~3x the number of partitions than available cores in cluster, to maximize parallelism along side with the overhead of spinning more executors. But this is not quite so straight forward. If our tasks are rather simple (taking less than a second), we might want to consider decreasing our partitions (to avoid the overhead), even if it means less than 3x. We should also take under consideration the memory of each executor, and make sure we are not exceeding it. If we do, we might want to create more partitions, even if it’s more than 3x our available cores, so that each will be smaller, or increase the memory of our executors.
3. **salting**: Add the salt key as part of the key as a new column. The newly added key forces Spark to hash the new key to a different hash value

## 3. Proper Cluster infra planning
1. https://github.com/khatwaniNikhil/SparkLearning/blob/main/infra_config.md
2. https://help.hitachivantara.com/Documentation/Pentaho/Data_Integration_and_Analytics/9.3/Setup/Determining_Spark_resource_requirements

## 4. Good Tenant config. in case of multiple tenants/teams sharing Spark Infra
set max limits around executors requested by each tenant in multi tenant env. don't abuse Spark dynamic allocation
