[TOC]

# 1. AWK相关



## 1.1 统计日志文件中出现次数最多的ip

搜索log文本中访问次数最多的ip并给出次数；

cat test.log
POST 200 192.168.1.1
POST 200 192.168.10.1
POST 200 192.168.1.1
POST 200 192.168.20.1

```bash
awk '{a[$3] += 1;} END {for (i in a) printf("%d %s\n", a[i], i);}' ip_count.txt | sort -n | tail -n 1
```

















































