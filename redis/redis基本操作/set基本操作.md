# set基本操作
```
127.0.0.1:6379> sadd runoobkey redis
(integer) 1
127.0.0.1:6379> sadd runoobkey mongodb
(integer) 1
127.0.0.1:6379> sadd runoobkey mysql
(integer) 1
127.0.0.1:6379> sadd runoobkey mysql
(integer) 0
127.0.0.1:6379> smembers runoobkey
1) "mysql"
2) "redis"
3) "mongodb"
127.0.0.1:6379> sadd myset 'hello'
(integer) 1
127.0.0.1:6379> sadd myset 'foo'
(integer) 1
127.0.0.1:6379> sadd myset 'hello'
(integer) 0
127.0.0.1:6379> smembers myset
1) "hello"
2) "foo"
127.0.0.1:6379> sadd myset 'hello'
(integer) 0
127.0.0.1:6379> sadd myset 'foo'
(integer) 0
127.0.0.1:6379> sadd myset 'hello'
(integer) 0
127.0.0.1:6379> scard myset 
(integer) 2
127.0.0.1:6379> sadd key1 'a'
(integer) 1
127.0.0.1:6379> sadd key 'b'
(integer) 1
127.0.0.1:6379> sadd key1 'b'
(integer) 1
127.0.0.1:6379> sadd key1 'c'
(integer) 1
127.0.0.1:6379> sadd key2 'c'
(integer) 1
127.0.0.1:6379> sadd key2 'd'
(integer) 1
127.0.0.1:6379> sadd key2 'e'
(integer) 1
127.0.0.1:6379> sdiff key1 key2
1) "b"
2) "a"
127.0.0.1:6379> sadd myset 'hello'
(integer) 0
127.0.0.1:6379> sadd myset 'foo'
(integer) 0
127.0.0.1:6379> sadd myset 'bar'
(integer) 1
127.0.0.1:6379> sadd myset2 'hello'
(integer) 1
127.0.0.1:6379> sadd myset2 'world'
(integer) 1
127.0.0.1:6379> sdiffstore distset myset myset2
(integer) 2
127.0.0.1:6379> smembers distset
1) "foo"
2) "bar"
127.0.0.1:6379> sadd myset 'hello'
(integer) 0
127.0.0.1:6379> sadd myset 'foo'
(integer) 0
127.0.0.1:6379> sadd myset 'bar'
(integer) 0
127.0.0.1:6379> sadd myset2 'hello'
(integer) 0
127.0.0.1:6379> sadd myset2 'world'
(integer) 0
127.0.0.1:6379> sinter myset myset2
1) "hello"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 1
127.0.0.1:6379> sadd myset1 'foo'
(integer) 1
127.0.0.1:6379> sadd myset1 'bar'
(integer) 1
127.0.0.1:6379> sadd myset2 'hello'
(integer) 0
127.0.0.1:6379> sadd myset2 'world'
(integer) 0
127.0.0.1:6379> sinterstore myset myset1 myset2
(integer) 1
127.0.0.1:6379> smembers myset 
1) "hello"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 0
127.0.0.1:6379> sismember myset1 'hello'
(integer) 1
127.0.0.1:6379> sismember myset1 'world'
(integer) 0
127.0.0.1:6379> smembers myset 1
(error) ERR wrong number of arguments for 'smembers' command
127.0.0.1:6379> smembers myset1
1) "hello"
2) "bar"
3) "foo"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 0
127.0.0.1:6379> sadd myset1 'world'
(integer) 1
127.0.0.1:6379> sadd myset1 'bar'
(integer) 0
127.0.0.1:6379> sadd myset2 'foo'
(integer) 1
127.0.0.1:6379> smove myset1 myset2 'bar'
(integer) 1
127.0.0.1:6379> smembers myset1
1) "world"
2) "foo"
3) "hello"
127.0.0.1:6379> smembers myset2
1) "hello"
2) "bar"
3) "foo"
4) "world"
127.0.0.1:6379> sadd myset 'one'
(integer) 1
127.0.0.1:6379> sadd myset 'two'
(integer) 1
127.0.0.1:6379> sadd myset 'three'
(integer) 1
127.0.0.1:6379> spop myset
"two"
127.0.0.1:6379> smembers myset
1) "hello"
2) "three"
3) "one"
127.0.0.1:6379> sadd myset 'four'
(integer) 1
127.0.0.1:6379> sadd myset 'five'
(integer) 1
127.0.0.1:6379> spop myset 3
1) "hello"
2) "one"
3) "four"
127.0.0.1:6379> smembers myset 
1) "three"
2) "five"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 0
127.0.0.1:6379> sadd myset1 'world'
(integer) 0
127.0.0.1:6379> sadd myset 'bar'
(integer) 1
127.0.0.1:6379> srandmember myset1
"hello"
127.0.0.1:6379> srandmember myset1 2
1) "foo"
2) "world"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 0
127.0.0.1:6379> sadd myset1 'world'
(integer) 0
127.0.0.1:6379> sadd myset1 'bar'
(integer) 1
127.0.0.1:6379> srem myset1 'hello'
(integer) 1
127.0.0.1:6379> srem myset1 'foo'
(integer) 1
127.0.0.1:6379> smembers myset1
1) "bar"
2) "world"
127.0.0.1:6379> sadd key1 'a'
(integer) 0
127.0.0.1:6379> sadd key2 'b'
(integer) 1
127.0.0.1:6379> sadd key1 'b'
(integer) 0
127.0.0.1:6379> sadd key1 'c'
(integer) 0
127.0.0.1:6379> sadd key2 'c'
(integer) 0
127.0.0.1:6379> sadd key2 'd'
(integer) 0
127.0.0.1:6379> sadd key2 'e'
(integer) 0
127.0.0.1:6379> sunion key1 key2
1) "d"
2) "b"
3) "c"
4) "a"
5) "e"
127.0.0.1:6379> sadd myset1 'hello'
(integer) 1
127.0.0.1:6379> sadd myset1 'world'
(integer) 0
127.0.0.1:6379> sadd myset1 'bar'
(integer) 0
127.0.0.1:6379> sadd myset2 'hello'
(integer) 0
127.0.0.1:6379> sadd myset2 'bar'
(integer) 0
127.0.0.1:6379> sunionstore myset myset1 myset2
(integer) 4
127.0.0.1:6379> smembers myset
1) "foo"
2) "world"
3) "bar"
4) "hello"

```