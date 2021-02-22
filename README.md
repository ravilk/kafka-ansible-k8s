$ kubectl get pods
NAME                                      READY   STATUS    RESTARTS   AGE
kafka-broker1-587cdc764-7gzhg             1/1     Running   0          4m15s
kafka-test-client                         1/1     Running   0          4m14s
zookeeper-deployment-1-7dfbcdb969-wkrgb   1/1     Running   0          4m18s

$ kubectl get svc
NAME            TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
kafka-service   LoadBalancer   10.103.142.48   10.236.1.1    9092:32355/TCP               4m58s
kubernetes      ClusterIP      10.96.0.1       <none>        443/TCP                      24m
zoo1            ClusterIP      10.109.89.20    <none>        2181/TCP,2888/TCP,3888/TCP   4m59s

$ kubectl exec -it kafka-test-client -- /usr/bin/kafka-console-producer --broker-list 10.236.1.1:9092 --topic topic1
>Hello
>World
>

$ kubectl exec -it kafka-test-client -- /usr/bin/kafka-console-consumer --bootstrap-server 10.236.1.1:9092 --topic topic1 --from-beginning
Hello
World

$ kubectl exec kafka-test-client -- /usr/bin/kafka-topics --zookeeper zoo1:2181 --list
__consumer_offsets
topic1

$ kubectl exec kafka-test-client -- /usr/bin/kafka-topics --zookeeper zoo1:2181 --topic topic2 --create --partitions 1 --replication-factor 1
Created topic "topic2".

$ kubectl exec kafka-test-client -- /usr/bin/kafka-producer-perf-test --num-records 90000000 --record-size 100 --topic topic2 --throughput 500000 --producer-props acks=0 bootstrap.servers=10.236.1.1:9092 batch.size=1000
168009 records sent, 33595.1 records/sec (3.20 MB/sec), 2583.3 ms avg latency, 4062.0 max latency.
513184 records sent, 102493.3 records/sec (9.77 MB/sec), 3346.5 ms avg latency, 5354.0 max latency.
588816 records sent, 117763.2 records/sec (11.23 MB/sec), 2281.5 ms avg latency, 2322.0 max latency.
599808 records sent, 119937.6 records/sec (11.44 MB/sec), 2254.4 ms avg latency, 2306.0 max latency.
599368 records sent, 119849.6 records/sec (11.43 MB/sec), 2240.3 ms avg latency, 2298.0 max latency.
589296 records sent, 117859.2 records/sec (11.24 MB/sec), 2273.2 ms avg latency, 2350.0 max latency.
593480 records sent, 118696.0 records/sec (11.32 MB/sec), 2253.1 ms avg latency, 2309.0 max latency.
593752 records sent, 118750.4 records/sec (11.32 MB/sec), 2256.8 ms avg latency, 2311.0 max latency.
264696 records sent, 52939.2 records/sec (5.05 MB/sec), 4324.7 ms avg latency, 5093.0 max latency.

$ kubectl exec kafka-test-client -- /usr/bin/kafka-consumer-perf-test --messages 90000000 --topic topic2 --broker-list 10.236.1.1:9092
start.time, end.time, data.consumed.in.MB, MB.sec, data.consumed.in.nMsg, nMsg.sec, rebalance.time.ms, fetch.time.ms, fetch.MB.sec, fetch.nMsg.sec
2018-12-06 02:00:46:338, 2018-12-06 02:01:13:409, 8583.0688, 317.0577, 90000000, 3324590.8906, 14, 27057, 317.2217, 3326311.1210
