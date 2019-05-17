# zset
```
127.0.0.1:6379> zadd myset 1 'one'
(integer) 1
127.0.0.1:6379> zadd myset 1 'uno'
(integer) 1
127.0.0.1:6379> zadd myzset 2 'two' 3 'three'
(integer) 2
127.0.0.1:6379> zrange myzset 0 -1 withscores
1) "two"
2) "2"
3) "three"
4) "3"
127.0.0.1:6379> zadd myzset 1 'one'
(integer) 1
127.0.0.1:6379> zadd myzset 2 'two'
(integer) 0
127.0.0.1:6379> zcard myzset 
(integer) 3

```