# hash基本操作
```
127.0.0.1:6379> hmset runoobkey name 'redis tutorial' description 'redis basic commands for caching' likes 20 visitors 23000
OK
127.0.0.1:6379> hgetall runnoobkey
(empty list or set)
127.0.0.1:6379> hgetall runoobkey
1) "name"
2) "redis tutorial"
3) "description"
4) "redis basic commands for caching"
5) "likes"
6) "20"
7) "visitors"
8) "23000"
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 1
127.0.0.1:6379> hdel myhash field1
(integer) 1
127.0.0.1:6379> hdel myhash field2
(integer) 0
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 1
127.0.0.1:6379> hexists myhash field1
(integer) 1
127.0.0.1:6379> hexists myhash field2
(integer) 0
127.0.0.1:6379> hset site redis.com
(error) ERR wrong number of arguments for 'hset' command
127.0.0.1:6379> hset site redis redis.com
(integer) 1
127.0.0.1:6379> hget site redis
"redis.com"
127.0.0.1:6379> hget site mysql
(nil)
127.0.0.1:6379> hset myhash field1 'hello'
(integer) 0
127.0.0.1:6379> hset myhash field2 'world'
(integer) 1
127.0.0.1:6379> hgetall myhash
1) "field1"
2) "hello"
3) "field2"
4) "world"
127.0.0.1:6379> hset myhash field 5
(integer) 1
127.0.0.1:6379> hincrby myhash field 1
(integer) 6
127.0.0.1:6379> hincrby myhash field -1
(integer) 5
127.0.0.1:6379> hincrby myhash field -10
(integer) -5
127.0.0.1:6379> hset mykey field 10.50
(integer) 1
127.0.0.1:6379> hincrbyfloat mykey field 0.1
"10.6"
127.0.0.1:6379> hincrbyfloat mykey field -5
"5.6"
127.0.0.1:6379> hset mykey field 5.0e3
(integer) 0
127.0.0.1:6379> hincrbyfloat mykey field 2.0e2
"5200"
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hset myhash field2 'bar'
(integer) 0
127.0.0.1:6379> hkeys myhash 
1) "field1"
2) "field2"
3) "field"
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hset myhash field2 'bar'
(integer) 0
127.0.0.1:6379> hlen myhash
(integer) 3
127.0.0.1:6379> hkeys myhash 
1) "field1"
2) "field2"
3) "field"
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hset myhash field2 'bar'
(integer) 0
127.0.0.1:6379> hmget myhash field1 field2 nofield
1) "foo"
2) "bar"
3) (nil)
127.0.0.1:6379> hmset myhash field1 'hello' field2 'world'
OK
127.0.0.1:6379> hget myhash field1
"hello"
127.0.0.1:6379> hget myhash field2
"world"
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hget myhash field1
"foo"
127.0.0.1:6379> hset website goole 'www.g.com'
(integer) 1
127.0.0.1:6379> hset website google 'www.google.com'
(integer) 1
127.0.0.1:6379> hsetnx myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hsetnx myhash field1 'bar'
(integer) 0
127.0.0.1:6379> hget myhash field1
"foo"
127.0.0.1:6379> hsetnx nosql key-value-store redis
(integer) 1
127.0.0.1:6379> hsetnx nosql key-value-store redis
(integer) 0
127.0.0.1:6379> hset myhash field1 'foo'
(integer) 0
127.0.0.1:6379> hset myhash field2 'bar'
(integer) 0
127.0.0.1:6379> hvals myhash
1) "foo"
2) "bar"
3) "-5"
127.0.0.1:6379> exists not_exists
(integer) 0
127.0.0.1:6379> hvals not_exists
(empty list or set)

```