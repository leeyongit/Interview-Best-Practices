# MySQL 主从复制

### MySQL 主从复制原理
1. 主库将所有的写操作记录在binlog日志中，并生成logdump线程，将binlog日志传给从库的I/O线程
2. 从库生成两个线程，一个是I/O线程，另一个SQL线程
3. I/O线程去请求主库的binlog日志，并将binlog日志中的文件写入relay log（中继日志）中
4. SQL线程会读取relay log中内容，并解析成具体的操作，来实现主从操作的一致，大道最终数据一致的目的