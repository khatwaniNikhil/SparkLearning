# Spark + Yarn + HDFS
1. https://sparkbyexamples.com/hadoop/apache-hadoop-installation/(hadoop is a prerequisite for yarn)
2. https://sparkbyexamples.com/spark/spark-setup-on-hadoop-yarn/

# Spark Cluster Structure
1. Namenode - namenode.XXX.infra
2. Datanode1 - datanode1.XXX.infra
3. Datanode2 - datanode2.XXX.infra
4. Datanode3 - datanode3.XXX.infra
5. Namenode web Ui is accessible via a.b.c.d:9870
6. Spark History server is accessible on e.f.g.h:18080
7. Spark Driver and Executor config file is located on the namenode server - /spark/conf/spark-defaults.conf
