# Linux Notes

## rz sz

## mount umount 挂载磁盘

## centos 查看系统配置

## 查看指定进程占用资源（cpu、内存、耗时）

ps aux | grep qpress | awk '{print($3" "$4);}'

ps -e -o 'pid,comm,args,pcpu,rsz,vsz,stime,user,uid' | grep qpress

pmap -d pid

## linux笔记：查看磁盘空间大小和所有用户各自占用空间 df -h

https://blog.csdn.net/abc13526222160/article/details/84962310
