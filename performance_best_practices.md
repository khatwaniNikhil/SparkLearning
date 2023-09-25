# Top Spark Cluster Tuning Strategies
## 1. Identify and handle data skewness
#### How to identify
1. Go to Stages tab in Spark UI: the skewed partitions hang within a stage and don’t seem to progress for a while on a few partitions. If we look at the summary metrics, the max column usually has a much larger value than the medium and more records count. Then we know we have encountered a data skew issue.
2. go to stage detail page in Spark UI for visual representation of the DAG. 
3. use WholeStageCodegen ids and go to the Spark Data Frame tab to find the code and hover on the SQL plan graphs to know the detail of what’s running in your code.

#### Fixing data skewness
1. revisit "spark.sql.shuffle.partitions" value
2. **salting**: Add the salt key as part of the key as a new column. The newly added key forces Spark to hash the new key to a different hash value

## 2. Structured way to do cluster infra planning
https://help.hitachivantara.com/Documentation/Pentaho/Data_Integration_and_Analytics/9.3/Setup/Determining_Spark_resource_requirements

## 3. Good Tenant config. in case of multiple tenants/teams sharing Spark Infra
set max limits around executors requested by each tenant in multi tenant env. don't abuse Spark dynamic allocation

## 4. Parition metrics to look for
1. Each partition size should be smaller than 200 MB to gain optimized performance.
2. The number of partitions should be 1x to 4x of the number of cores you have to gain optimized performance (which means create a cluster that matches your data scale is also important).


# References
1. https://nealanalytics.com/blog/databricks-spark-jobs-optimization-techniques-shuffle-partition-technique-part-1/
