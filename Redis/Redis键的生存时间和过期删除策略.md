# Redis键的生存时间和过期删除策略

## 简介

Redis中的键是可以设置生存时间的，在经过指定的时间后，Redis服务器会自动删除生存时间为0的键。本文将介绍Redis中键的生存时间是如何保存的，以及过期键是如何删除的。

## 1.键的生存时间

### 1.1 设置生存时间

Redis有四个不同的命令可以用于设置键的生存时间（键可以存在多久）或过期时间（键什么时候会被删除）：

1. EXPIRE &lt;key&gt; &lt;ttl&gt; 命令用于将键key的生存时间设置为ttl秒。

2. PEXPIRE &lt;key&gt; &lt;ttl&gt; 命令用于将键key的生存时间设置为ttl毫秒。

3. EXPIREAT &lt;key&gt; &lt;timestamp&gt; 命令用于将key的过期时间设置为timestamp所指定的秒数时间戳。

4. PEXPIREAT &lt;key&gt; &lt;timestamp&gt; 命令用于将key的过期时间设置为timestamp所指定的毫秒数时间戳。

虽然有多种不同单位和不同形式的设置命令，但实际上EXPIRE、PEXPIRE、EXPIREAT三个命令都是使用PEXPIREAT命令来实现的：无论客户端执行的是以上四个命令中的哪一个，都将最终转换为PEXPIREAT命令来执行。

首选，EXPIRE命令可以转换为PEXPIRE命令：

```
def EXPIRE(key, ttl_in_sec):
    # 将TTL从秒转换成毫秒
    ttl_in_ms = sec_to_ms(ttl_in_sec)

    # 再调用PEXPIRE命令
    PEXPIRE(key, ttl_in_ms)
```

接着，PEXPIRE命令又可以转换成PEXPIREAT命令：

```
def PEXPIRE(key, ttl_in_ms):
    # 获取以毫秒计算的当前UNIX时间戳
    now_ms = get_current_unix_timestamp_in_ms()

    # 当前时间加上TTL，得出毫秒格式的过期时间
    # 再调用PEXPIREAT命令
    PEXPIREAT(key, now_ms + ttl_in_ms)
```

并且，EXPIREAT命令也可以转换成PEXPIREAT命令：

```
def EXPIREAT(key, expire_time_in_sec):
    # 将过期时间从秒转换成毫秒
    expire_time_in_ms = sec_to_ms(expire_time_in_sec)

    # 再调用PEXPIREAT命令
    PEXPIREAT(key, expire_time_in_ms)
```

最终，EXPIRE、PEXPIRE、EXPIREAT三个命令都会转换成PEXPIREAT命令来执行。

### 1.2 保存过期时间

在redisDb结构的expire字典保存了数据库中所有键的过期时间，我们称这个字典为过期时间字典：

1. 过期字典的键是一个指针，这个指针指向键空间中的某个键对象（也即是某个数据库键）。

2. 过期字典的值是一个long long类型的整数，这个整数保存了键所指向的数据库键的过期时间——一个毫秒精度的UNIX时间戳。

具体结构如下：

```
typedef struct redisDb {
    // ...

    // 过期字典，保存着键的过期时间
    dict *expires;

    // ...
} redisDb;
```

当客户端执行PEXPIREAT命令（或者其他三个会转换成PEXPIREAT命令的命令）为一个数据库键设置过期时间时，服务器会在数据库的过期字典中关联给定的数据库键和过期时间。其中键空间的键和过期字典的键都是指向同一个键对象，所以不会出现任何重复对象，也不会浪费任何空间。

以下是PEXPIREAT命令的伪代码定义：

```
def PEXPIREAT(key, expire_time_in_ms):
    # 如果给定的键不存在于键空间，那么不能设置过期时间
    if key not in redisDb.dict:
        return 0
    
    # 在过期字典中关联键和过期时间
    redisDb.expires[key] = expire_time_in_ms

    # 过期时间设置成功
    return 1
```

### 1.3 移除过期时间

PERSIST命令可以移除一个键的过期时间，PERSIST命令就是PEXPIREAT命令的反操作：PERSIST命令在过期字典中查找给定的键，并解除键和值（过期时间）在过期字典中的关联。

以下是PERSIST命令的伪代码定义：

```
def PERSIST(key):
    # 如果键不存在，或者键没有设置过期时间，那么直接返回
    if key not in redisDb.expires:
        return 0

    # 移除过期字典中给定键的键值对关联
    redisDb.expires.remove(key)

    # 键的过期时间移除成功
    return 1
```

### 1.4 计算并返回剩余生存时间

TTL命令以秒为单位返回键的剩余生存时间，而PTTL命令则以毫秒为单位返回键的剩余生存时间。

以下是TTL和PTTL两个命令的伪代码实现：

```
def PTTL(key):
    # 键不存在于数据库
    if key not in redisDb.dict:
        return -2
    
    # 尝试获得键的过期时间
    # 如果键没有设置过期时间，那么expire_time_in_ms将为None
    expire_time_in_ms = redisDb.expires.get(key)

    # 键没有设置过期时间
    if expire_time_in_ms is None:
        return -1

    # 获得当前时间
    now_ms = get_current_unix_timestamp_in_ms()

    # 过期时间减去当前时间，得出的差就是键剩余生存时间
    return (expire_time_in_ms - now_ms)

def TTL(key):
    ## 获取当前已毫秒为单位的剩余生存时间
    ttl_in_ms = PTTL(key)

    if ttl_in_ms < 0:
        # 处理返回值为-1和-2的情况
        return ttl_in_ms
    else:
        # 将毫秒转换为秒
        return ms_to_sec(ttl_in_ms)
```

### 1.5 过期键的判定

通过过期字典，程序可以用以下步骤检查一下给定键是否过期：

1. 检查给定键是否存在于过期字典：如果存在，那么取得键的过期时间。

2. 检查当前UNIX时间戳是否大于键的过期时间：如果是的话，那么键已经过期；否则的话，键未过期。

可以用一段伪代码来描述这一过程：

```
def is_expired(key):
    # 取得键的过期时间
    expire_time_in_ms = redisDb.expires.get(key)

    # 键没有设置过期时间
    if expire_time_in_ms is None:
        return False
    
    # 获得当前时间
    now_ms = get_current_unix_timestamp_in_ms()

    # 检查当前时间是否大于键的过期时间
    if now_ms > expire_time_in_ms:
        # 是，键已过期
        return True
    else:
        # 否，键未过期
        return False
```

## 2.过期键删除策略

如果一个键过期了，这里提供了三种不同的删除策略去删除过期的键：

1. 定时删除：在设置键的过期时间的同时，创建一个定时器（timer），让定时器在键的过期时间来临时，立即执行对键过期的删除操作。

2. 惰性删除：放任键过期不管，但是每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键；否则，返回该键。

3. 定期删除：每隔一段时间，程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

### 2.1 定时删除

定时删除策略是对内存最友好的：通过使用定时器，定时删除策略可以保证过期键会尽可能快地被删除，并释放过期键所占用的内存。

另一方面，定时删除策略的缺点是，它对CPU时间是最不友好的：在过期键比较多的情况下，删除过期键这一行为可能会占用相当一部分的CPU时间，在内存不紧张但是CPU时间非常紧张的情况下，将CPU时间用在删除和当前任务无关的过期键上，无疑会对服务器的响应时间和吞吐量造成影响。

### 2.2 惰性删除

惰性删除策略对CPU时间来说是最友好的：程序只会在取出键时才对键进行过期检查，这可以保证删除过期键的操作只会在非做不可的情况下进行，并且删除的目标仅限于当前处理的键，这个策略不会在删除其他无关的过期键上花费任何CPU时间。

惰性删除策略的缺点是：它对内存是最不友好的：如果一个键已经过期，而这个键又仍然保留在数据库中，那么只要这个过期键不被删除，它所占用的内存就不会被释放。

### 2.3 定期删除

从上面来看，定时删除和惰性删除在单一使用时都有明显的缺陷：

1. 定时删除占用太多CPU时间，影响服务器的响应时间和吞吐量。

2. 惰性删除浪费太多内存，有内存泄露的危险。

定期删除策略是前两种方式的整合和折中：

1. 定期删除策略每隔一段时间执行一次删除过期键的操作，并通过限制删除操作执行的时长和频率来减少删除操作对CPU的影响。

2. 除此之外，通过定期删除过期键，定期删除策略有效地减少了因为过期键而带来的内存浪费。

定期删除策略的难点是确定删除操作执行的时长和频率：

1. 如果删除操作执行得太频繁，或者执行的时间太长，定期删除策略就会退化成定时删除策略，以至于将CPU时间过多地消耗在删除过期键上面。

2. 如果删除操作执行得太少，或者执行的时间太短，定期删除策略又会和惰性删除策略一样，出现浪费内存的情况。

## 3.Redis的过期删除策略

Redis服务器实际使用的是惰性删除和定期删除两种策略：通过配合使用这两种删除策略，服务器可以很好地在合理使用CPU时间和避免浪费内存空间之间取得平衡。

### 3.1 惰性删除策略的实现

过期键的惰性删除策略由db.c/expireIfNeeded函数实现，所有读写数据库的Redis命令在执行之前都会调用expireIfNeeded函数对输入键进行检查：

1. 如果输入键已经过期，那么expireIfNeeded函数对输入键从数据库中删除。

2. 如果输入键未过期，那么expireIfNeeded函数不做动作。

### 3.2 定期删除策略的实现

过期键的定期删除策略由redis.c/activeExpireCycle函数实现，每当Redis的服务器周期性操作redis.c/serverCron函数执行时，activeExpireCycle函数就会被调用，它在规定时间内，分多次遍历服务器中的各个数据库，从数据库expires字典中随记检查一部分键的过期时间，并删除其中的过期键。

activeExpireCycle函数的工作模式可以总结如下：

1. 函数每次运行时，都从一定数量的数据库中取出一定数量的随机键进行检查，并删除其中的过期键。

2. 全局变量current_db会记录当前activeExpireCycle函数检查的进度，并在下一次activeExpireCycle函数调用时，接着上一次的进度进行处理。

3. 随着activeExpireCycle函数的不断执行，服务器中的所有数据库都会被检查一遍，这时函数将current_db变量重置为0，然后再次开始新一轮的检查工作。

> 本文摘自《Redis的设计与实现》