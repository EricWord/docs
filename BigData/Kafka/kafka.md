# kafka相关技术总结
## 1.golang调用kafka常用第三方库
https://github.com/bsm/sarama-cluster  
在主页有使用方法的示例代码，如果想要采用多线程的方式来使用，可以使用下面的代码

## 2.注意事项
topic的命名尽量不要包含下划线  
## 3.自带脚本的常用命令方法
### 3.1 创建一个带有3个副本的topic
sh kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partitions 1 --topic "topic name"  
### 3.2 查看所有的topic
sh kafka-topics.sh --list --zookeeper localhost:2182
### 3.3 查看某个topic的内容
sh kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "topic name" --from-beginning  
### 3.4 向指定topic中写入内容
sh kafka-console-producer.sh --broker-list 127.0.0.1:9092,127.0.0.1:9093,127.0.0.1:9094 --topic "topic name"  
### 3.5 创建消费者组
sh kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic "topic name" --consumer-property group.id = 消费者组名称  
### 3.6 创建生产者
sh kafka-console-producer.sh --broker-list localhost:9092 --topic "topic name"  
### 3.7 创建topic
sh kafka-topics.sh --create --topic topicname --replication-factor 1 --partitions 1 --zookeeper localhost:2181  
+ topic指定topic name  
+ partitions指定分区数，这个参数需要根据broker数和数据量决定，正常情况下，每个broker上两个partition最好  
+ replication-factor指定partition的replicas数，建议设置为2；  

