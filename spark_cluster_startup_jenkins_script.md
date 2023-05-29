#!/bin/bash +x

Server_list=i-$server1,i-$server2,i-$server3,i-$server4
serverName=namenode.xyz.infra

if [[ "${Spark_Cluster}" == "Stop" ]]; then
	for i in $(echo $Server_list | sed "s/,/ /g")
		do
			/usr/bin/aws ec2 stop-instances --region ap-south-1 --instance-ids $i
		done
        
elif [[ "${Spark_Cluster}" == "Start" ]]; then
	for i in $(echo $Server_list | sed "s/,/ /g")
		do
			/usr/bin/aws ec2 start-instances --region ap-south-1 --instance-ids $i
		done
fi
