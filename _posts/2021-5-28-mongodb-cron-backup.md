---
layout: post
title: MongoDB Backup Script
---
数据安全是每个企业的重中之重，及时做好数据备份和风险管控是每个企业运维部门的责任与义务。近些年来各种删库跑路的事件，屡见不鲜，对企业造成了严重的经济损失和信誉影响。

针对MongoDB的数据定时（业务低峰期开始）备份，推荐参考以下脚本：

## MongoDB自动备份.sh脚本案例
vim /opt/crontab/mongodb_backup.sh

```
#!/bin/sh
# dump 命令执行路径，根据mongodb安装路径而定
DUMP=/usr/bin/mongodump
# 临时备份路径
OUT_DIR=/data/mongodata/mongod_backup
# 压缩后的备份存放路径
TAR_DIR=/data/mongodata/mongod_backup_list
# 当前系统时间
DATE=`date +%Y-%m-%d`
# 数据库账号
DB_USER=user
# 数据库密码
DB_PASS=password
# 代表删除7天前的备份，即只保留近 7 天的备份
DAYS=7
# 最终保存的数据库备份文件
TAR_BAK="mongod_backup._$DATE.tar.gz"
cd $OUT_DIR
rm -rf $OUT_DIR/*
mkdir -p $OUT_DIR/$DATE
$DUMP -h 127.0.0.1:27017 -u $DB_USER -p $DB_PASS -d dbname -o $OUT_DIR/$DATE
# 压缩格式为 .tar.gz 格式
tar -zcvf $TAR_DIR/$TAR_BAK $OUT_DIR/$DATE
# 删除 15 天前的备份文件
find $TAR_DIR/ -mtime +$DAYS -delete
 
exit
```
## crontab介绍
查看当前用户的crontab，输入 crontab -l；
编辑crontab，输入 crontab -e；
删除crontab，输入 crontab -r

## crontab文件参数解读
# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
  分 时 日 月 周 命令
第1列表示分钟1～59 每分钟用或者 /1表示
第2列表示小时1～23（0表示0点）
第3列表示日期1～31
第4列表示月份1～12
第5列标识号星期0～6（0表示星期天）


Author:UStarGao
