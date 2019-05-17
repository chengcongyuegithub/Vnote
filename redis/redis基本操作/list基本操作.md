# list基本操作
```
127.0.0.1:6379> lpush runoobkey redis
(integer) 1
127.0.0.1:6379> lpush runoobkey mongodb
(integer) 2
127.0.0.1:6379> lpush runoobkey mysql
(integer) 3
127.0.0.1:6379> lrange runoobkey 0 10
1) "mysql"
2) "mongodb"
3) "redis"
```
```
127.0.0.1:6379> blpop runoobkey 100
1) "runoobkey"
2) "mysql"
127.0.0.1:6379> blpop runoobkey 100
1) "runoobkey"
2) "mongodb"
127.0.0.1:6379> blpop runoobkey 100
1) "runoobkey"
2) "redis"
127.0.0.1:6379> blpop runoobkey 1
(nil)
(1.07s)
```
```
127.0.0.1:6379> brpoplpush msg reciver 1
(nil)
(1.01s)
127.0.0.1:6379> lpush msg hello
(integer) 1
127.0.0.1:6379> lpush msg world
(integer) 2
127.0.0.1:6379> lrange msg 0 10
1) "world"
2) "hello"
127.0.0.1:6379> brpoplpush msg revicer 1
"hello"
127.0.0.1:6379> brpoplpush msg revicer 1
"world"
127.0.0.1:6379> llen revicer
(integer) 2
127.0.0.1:6379> lrange revicer 0 2
1) "world"
2) "hello"
```
```
127.0.0.1:6379> lpush mylist "world"
(integer) 1
127.0.0.1:6379> lpush mylist "hello"
(integer) 2
127.0.0.1:6379> lindex mylist 0
"hello"
127.0.0.1:6379> lindex mylist -1
"world"
127.0.0.1:6379> lindex mylist 3
(nil)
```
```
127.0.0.1:6379> rpush mylist 'hello'
(integer) 3
127.0.0.1:6379> rpush mylist 'world'
(integer) 4
127.0.0.1:6379> range mylist 0 2
(error) ERR unknown command 'range'
127.0.0.1:6379> lrange mylist 0 2
1) "hello"
2) "world"
3) "hello"
127.0.0.1:6379> linsert mylist before "world" "There"
(integer) 5
127.0.0.1:6379> lrange mylist 0 -1
1) "hello"
2) "There"
3) "world"
4) "hello"
5) "world"
127.0.0.1:6379> llen list1
(integer) 0
127.0.0.1:6379> llen mylist
(integer) 5
```