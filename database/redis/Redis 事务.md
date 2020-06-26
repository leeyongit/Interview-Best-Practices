# Redis 事务
[Redis 事务 & 秒杀案例实现](https://www.youtube.com/watch?v=aCXsdLjQLcM)

## Redis 的事务定义
* Redis 事务是一个单独的隔离操作：事务中的所有命令都会被序列化、按顺序执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* Redis 事务的主要作用就是串联多个命令防止别的命令插队。

## Multi、Exec、discard
* 从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。
* 组队的过程中可以通过discard来放弃组队。

**事务的错误处理**
* 组队中某个命令出现了报告错误，执行时整个的所有队列会都会被取消
* 如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会被执行，不会回滚

## 为什么要用事务
一个场景：
    有很多人有你的账户，同时去参加双十一抢购
* 事务冲突的问题
* 三个请求 （余额有10000）
    * 一个请求想给金额减8000
    * 一个请求想给金额减5000
    * 一个请求想给金额减1000

* 悲观锁(Pessimistic Lock), 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁。**传统的关系型数据库里就用到了很多这种锁机制**，比如行锁、表锁，读锁，写锁等，表都是在操作之前先上锁。
* 乐观锁(Optimistic Lock), 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但时在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量。**Redis就是利用这种check-and-set机制实现事务的。

### WATCH key [key ...]
* 在执行multi之前，先执行watch key1 [key2], 可以监视一个（或多个）key, 如果在事务执行之前这个（或这些）key 被其他命令所改动，那么事务将被打断。
```sh
> WATCH balance
ok
> MULITI
ok
> DECRBY balance 10
QUEUED
> INCRBY debt 10
QUEUED
> EXEC
1) (integer) 60
2) (integer) 40
```

### unwatch
* 取消WATCH命令对所有key的监视。
* 如果在执行watch命令之后，EXEC命令或discard命令先执行了的话，那么就不需要再执行unwatch了

## Redis 事务三特性
* 单独的隔离操作
    - 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
* 没有隔离级别的概念
    - 队列中的命令没有提交之前都不会被实际的被执行，因为事务提交前任何指令都不会被实际执行，也就不存在“事务内的查询要看到事务里的更新，在事务外查询不能看到”这个让人万分头痛的问题
* 不保证原子性
    - Redis 同一个事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚

## Redis 事务--秒杀案例
**解决计数器和人员记录的事务操作**
商品库存
| key        | string |
|------------|--------|
|sk:prodid:qt|剩余个数 ｜
秒杀成功者清单
| key          | set           |
|--------------|---------------|
|sk:prod-id:usr|成功者的user_id ｜
|              |成功者的user_id ｜
|              |成功者的user_id ｜
```php
// 查看当前的用户是否秒杀过
$prodid = 1;
$userid = 2;
$memberKey = "sk:".$prodid+":users";
$flag = $redis->sismember($memberKey, $userid) ;
// 已经秒杀过，提示错误信息
if ($flag) {
    exit("该用户已经秒杀过，无法继续参加此活动");
}
$redis->watch($qtKey); // 监视一个(或多个) key
// 没有秒杀过
// 获取商品库存
$qtKey = "sk:".$prodid.":qt";
$qt = $redis->get($qtKey);
// 判断库存大小
// 小于等于0，返回消息false
if ($qt <= 0) {
    exit('没有库存');
}

// 大于0
// 将库存减一
$redis->multi(Redis::MULTI); // 开启事务
$redis->decr($qtKey);
// 增加秒杀成功者清单
$redis->sadd($memberKey, $userid);
$redis->exec();  //执行所有事务块内的命令
```
## Lua 脚本在Redis 中的优势
* 将复杂的或者多步的redis操作，写一个脚本，一次提交给redis执行，减少反复连接redis的次数。提升性能。
* Lua 脚本死类似redis 事务，有一定原子性，不会被其他命令插队，可以完成一些redis事务性的操作。
* 但是要注意redis的lua脚本功能，只有在2.6以上的版本才可以使用

Lua 脚本实现
```lua
local userid=KEYS[1];
local prodid=KEYS[2];
local qtkey="sk:"..prodid..":qt";
local usersKey="sk:"..prodid.":usr";
local usrExists=redis.call("sismember",usersKey,userid);
if tonumber(userExists)==1 then
    return 2
end
local num=reids.call("get", qtkey);
if tonumber(num)<=0 then
    return 0;
else
    redis.call('decr', qtkey);
    redis.call("sadd", usersKey, userid);
end
return 1;
```