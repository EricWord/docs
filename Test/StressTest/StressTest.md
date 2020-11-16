# 压力测试相关技术总结
## 1. ab压测工具
### 1.1 使用命令示例
#### 1.1.1 示例1
ab -n 1  -c 10 -p /post.txt -T 'application/json ' -H 'your url' 
#### 1.1.2 示例2
ab -H "data:{\"data\": \"your data\"}"  -n 1000  -c 150 -p "/post.txt" -T "application/json"  "your url"

### 1.2 Get请求示例
ab  -c 100 your_url?field1=xxx

