---
layout: post
title: Crontab purge binary logs
---
生产环境中，我们经常会遇到MySQL数据库日志占满磁盘的情况，比如，MySQL数据迁移或者高IO读写的MySQL实例等。经过分析，可以发现占满磁盘的都是binlog日志，那么我们有什么办法可以定期去清理这些日志呢？
- 通过修改expire_logs_days参数。
- 通过定时脚本去清理。

【注意】当然MySQL有自带binlog清理机制，通过修改expire_logs_days即binlog过期时间进行控制。但是由于该参数的取值范围是1-31天，所以无法精确清理小时级别的日志。

## .sh脚本案例
```
while /bin/true; do
    #把16hour改成需要清理n小时前的binlog的值即可
    time=$(date "+%Y-%m-%d %H:%M:%S" -d "-16hour")
    #把$ip，端口$port，用户名$user，密码$passward改成对应MySQL实例的
    mysql -e "purge binary logs to ${time}" -h$ip -P$port -u$user -p$passward
    #把5改成需要的间隔时间
    sleep 5
done

```
## Crontab设置方法
vim /etc/crontab

![定时任务写法](http://ucloudspt.cn-sh2.ufileos.com/Blogs/定时任务写法.png)
- 每三小时执行一次定时脚本
- 分、时、日、月、周、用户、执行的命令


Author:UStarGao
