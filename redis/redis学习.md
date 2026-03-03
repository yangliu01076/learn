# Pipeline 和 Transaction 的区别
Pipeline 是 Redis 提供的非原子性批量操作命令，可以一次性执行多个命令，
但是每个命令的执行是独立的，不保证原子性。一个命令失败不会影响其他命令的执行。  
Transaction 是 Redis 提供的原子性事务，
可以一次性执行多个命令, 并且保证这些命令是原子性的。  

# 网络延迟
Redis 的网络延迟大概是 mysql 的十分之一。