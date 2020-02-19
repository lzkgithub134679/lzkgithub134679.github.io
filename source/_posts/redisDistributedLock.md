---
title: redis分布式锁
date: 2020-01-15 09:24:48
header-img: "Demo.png"
tags:
- Show All
- redis
---

## 什么是分布式锁

在以往的单机系统中，对一些并发场景读取公共资源时如扣库存，卖车票之类的需求可以简单的使用同步或者是加锁就可以实现。
但是应用分布式了之后系统由以前的**单进程多线程**的程序变为了**多进程多线程**，这时使用以上的解决方案明显就不够了。
因此因此业界常用的解决方案通常是借助于一个第三方组件并利用它自身的排他性来达到多进程的互斥。如：

- 基于数据库的乐观锁
- 基于ZooKeeper的分布式锁
- 基于Redis的分布式锁
  这里主要基于 Redis 进行讨论。

## Redis分布式锁应该满足哪些条件

首先，为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

- 互斥性。在任意时刻，只有一个客户端能持有锁。
- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
- 具有容错性。只要大部分的Redis节点正常运行，客户端就可以加锁和解锁。
- 解铃还须系铃人。加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了。

## Redis命令

### SETNX key value

只在键 key 不存在的情况下， 将键 key 的值设置为 value 。
若键 key 已经存在， 则 SETNX 命令不做任何动作。
SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。

- 返回值
  命令在设置成功时返回 1 ， 设置失败时返回 0 。

### Jedis evel()

eval()方法是将Lua代码交给Redis服务端执行
EVAL script numkeys key [key …] arg [arg …]

- numkeys 参数用于指定键名参数的个数。
- 键名参数 key [key …] 从 EVAL 的第三个参数开始算起，表示在脚本中所用到的那些 Redis 键(key)，
- 这些键名参数可以在 Lua 中通过全局变量 KEYS 数组，用 1 为基址的形式访问( KEYS[1] ， KEYS[2] ，以此类推)。
- 在命令的最后，那些不是键名参数的附加参数 arg [arg …] ，可以在 Lua 中通过全局变量 ARGV 数组访问，
- 访问的形式和 KEYS 变量类似( ARGV[1] 、 ARGV[2] ，诸如此类)。
  简单来说，就是在eval命令执行Lua代码的时候，Lua代码将被当成一个命令去执行，并且直到eval命令执行完成，Redis才会执行其他命令。确保原子性。

### 在 Lua 脚本中，可以使用两个不同函数来执行 Redis 命令，它们分别是：redis.call(),redis.pcall()

这两个函数的唯一区别在于它们使用不同的方式处理执行命令所产生的错误。
redis.call()
当 redis.call() 在执行命令的过程中发生错误时，脚本会停止执行，并返回一个脚本错误，错误的输出信息会说明错误造成的原因。
redis.pcall()
redis.pcall() 出错时并不引发(raise)错误，而是返回一个带 err 域的 Lua 表(table)，用于表示错误

## 代码实现

### 加锁

```java
public class RedisTool {
    private static final String LOCK_SUCCESS = "OK";
    private static final String SET_IF_NOT_EXIST = "NX";
    private static final String SET_WITH_EXPIRE_TIME = "PX";
    /**
     * 尝试获取分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @param expireTime 超期时间
     * @return 是否获取成功
     */
    public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {
        String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);
        if (LOCK_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

可以看到，我们加锁就一行代码：jedis.set(String key, String value, String nxxx, String expx, int time)，这个set()方法一共有五个形参：

- 第一个为key，我们使用key来当锁，因为key是唯一的。
- 第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。
- 第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；
- 第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。
- 第五个为time，与第四个参数相呼应，代表key的过期时间。

总的来说，执行上面的set()方法就只会导致两种结果：1. 当前没有锁（key不存在），那么就进行加锁操作，并对锁设置个有效期，同时value表示加锁的客户端。2. 已有锁存在，不做任何操作。

心细的童鞋就会发现了，我们的加锁代码满足我们可靠性里描述的三个条件。首先，set()加入了NX参数，可以保证如果已有key存在，则函数不会调用成功，也就是只有一个客户端能持有锁，满足互斥性。其次，由于我们对锁设置了过期时间，即使锁的持有者后续发生崩溃而没有解锁，锁也会因为到了过期时间而自动解锁（即key被删除），不会发生死锁。最后，因为我们将value赋值为requestId，代表加锁的客户端请求标识，那么在客户端在解锁的时候就可以进行校验是否是同一个客户端。由于我们只考虑Redis单机部署的场景，所以容错性我们暂不考虑。

错误方式：

- 使用setnx() 和 expire()，但并没有线程同步或加锁保证这两个操作的原子性。
  setnx() 之后如果突然崩溃导致没有执行expire()，会导致死锁。

## 解锁

```java
public class RedisTool {
    private static final Long RELEASE_SUCCESS = 1L;
    /**
     * 释放分布式锁
     * @param jedis Redis客户端
     * @param lockKey 锁
     * @param requestId 请求标识
     * @return 是否释放成功
     */
    public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {
        String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
        Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));
        if (RELEASE_SUCCESS.equals(result)) {
            return true;
        }
        return false;
    }
}
```

## 我们的实现还有什么问题么？

考虑由于A服务器执行时间过长，导致锁失效，从而使得B服务器获取到了锁，对同一个key执行了相同的逻辑。
首先，需要确认的是，以上的情况发生概率很低，如果你的系统并发量不大，业务逻辑不复杂的话，基本上很难遇到这个者A、B服务器都对同一个key执行业务逻辑的问题。