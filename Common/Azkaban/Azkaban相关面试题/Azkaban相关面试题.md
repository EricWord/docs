[TOC]

# 1. 如何减少任务调度执行的时间

通过提高并行job数

修改Executor Server配置

```bash
flow.num.job.threads=20#默认值是10
```

# 2. Azkaban的优缺点

1. Azkaban中如果有任务出现失败，只要进程有效执行，那么任务就算执行成功
2. 出现失败的情况时，会丢失所有的工作流