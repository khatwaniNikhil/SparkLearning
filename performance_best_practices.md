1. Each partition size should be smaller than 200 MB to gain optimized performance.
2. The number of partitions should be 1x to 4x of the number of cores you have to gain optimized performance (which means create a cluster that matches your data scale is also important).


# References
1. https://nealanalytics.com/blog/databricks-spark-jobs-optimization-techniques-shuffle-partition-technique-part-1/
