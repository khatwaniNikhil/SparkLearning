1. In parallel JDBC read from different production multi tenant clusters and terminated tenants db
2. For each JDBC read
    1. read step
        1. fetchSize = 50K
        2. numPartitions default value of 1 is used as already we have parallelism at tenant level
    2. transform step
        1. parallelize data massaging(transformation/cleansing/filtering) by repartition spark dataframe
    3. write step
        1. batchSizeForJDBCWrite=50k
        2. isolationLevel = NONE for faster writes and business case feasability
        3. 
