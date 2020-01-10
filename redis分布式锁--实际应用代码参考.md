# 分布式锁原理
**文章后面会给出实际应用代码参考**：

**分布式锁原理**  

 分布式锁，是控制分布式系统之间同步访问共享资源的一种方式。在分布式系统中，常常需要协调他们的动作。如果不同的系统或是同一个系统的不同主机之间共享了一个或一组资源，那么访问这些资源的时候，往往需要互斥来防止彼此干扰来保证一致性，在这种情况下，便需要使用到分布式锁。

**使用setnx、getset、expire、del这4个redis命令实现**

## 1. setnx 命令
SETNX key value

将 key 的值设为 value ，当且仅当 key 不存在。

若给定的 key 已经存在，则 SETNX 不做任何动作。

SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。

可用版本：>= 1.0.0

时间复杂度：O(1)   

返回值：
设置成功，返回 1 

设置失败，返回 0 

```
redis> EXISTS job                # job 不存在
(integer) 0

redis> SETNX job "programmer"    # job 设置成功
(integer) 1

redis> SETNX job "code-farmer"   # 尝试覆盖 job ，失败
(integer) 0

redis> GET job                   # 没有被覆盖
"programmer"
```

## 2.getset命令
GETSET key value

将给定 key 的值设为 value ，并返回 key 的旧值(old value)。

当 key 存在但不是字符串类型时，返回一个错误。

可用版本：>= 1.0.0

时间复杂度：O(1)

返回值：
返回给定 key 的旧值。
当 key 没有旧值时，也即是， key 不存在时，返回 nil 。

```
redis> GETSET db mongodb    # 没有旧值，返回 nil
(nil)

redis> GET db
"mongodb"

redis> GETSET db redis      # 返回旧值 mongodb
"mongodb"

redis> GET db
"redis"
```
## 3.expire命令
EXPIRE key seconds

为给定 key 设置生存时间，当 key 过期时(生存时间为 0 )，它会被自动删除。

在 Redis 中，带有生存时间的 key 被称为『易失的』(volatile)。

生存时间可以通过使用 DEL 命令来删除整个 key 来移除，或者被 SET 和 GETSET 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 key 的值而不是用一个新的 key 值来代替(replace)它的话，那么生存时间不会被改变。

比如说，对一个 key 执行 INCR 命令，对一个列表进行 LPUSH 命令，或者对一个哈希表执行 HSET 命令，这类操作都不会修改 key 本身的生存时间。

另一方面，如果使用 RENAME 对一个 key 进行改名，那么改名后的 key 的生存时间和改名前一样。

RENAME 命令的另一种可能是，尝试将一个带生存时间的 key 改名成另一个带生存时间的 another_key ，这时旧的 another_key (以及它的生存时间)会被删除，然后旧的 key 会改名为 another_key ，因此，新的 another_key 的生存时间也和原本的 key 一样。

使用 PERSIST 命令可以在不删除 key 的情况下，移除 key 的生存时间，让 key 重新成为一个『持久的』(persistent) key 。

更新生存时间

可以对一个已经带有生存时间的 key 执行 EXPIRE 命令，新指定的生存时间会取代旧的生存时间。

过期时间的精确度

在 Redis 2.4 版本中，过期时间的延迟在 1 秒钟之内 —— 也即是，就算 key 已经过期，但它还是可能在过期之后一秒钟之内被访问到，而在新的 Redis 2.6 版本中，延迟被降低到 1 毫秒之内。

Redis 2.1.3 之前的不同之处

在 Redis 2.1.3 之前的版本中，修改一个带有生存时间的 key 会导致整个 key 被删除，这一行为是受当时复制(replication)层的限制而作出的，现在这一限制已经被修复。

可用版本：>= 1.0.0

时间复杂度：O(1)

返回值：  

1. 设置成功返回 1。  
2. 当 key 不存在或者不能为 key 设置生存时间时(比如在低于 2.1.3 版本的 Redis 中你尝试更新 key 的生存时间)，返回 0 。

```
redis> SET cache_page "www.google.com"
OK

redis> EXPIRE cache_page 30  # 设置过期时间为 30 秒
(integer) 1

redis> TTL cache_page    # 查看剩余生存时间
(integer) 23

redis> EXPIRE cache_page 30000   # 更新过期时间
(integer) 1

redis> TTL cache_page
(integer) 29996
```
## 4 DEL命令
DEL key [key ...]

删除给定的一个或多个 key 。

不存在的 key 会被忽略。

可用版本：>= 1.0.0
时间复杂度：O(N)， N 为被删除的 key 的数量。

删除单个字符串类型的 key ，时间复杂度为O(1)。

删除单个列表、集合、有序集合或哈希表类型的 key ，时间复杂度为O(M)， M 为以上数据结构内的元素数量。

返回值：
被删除 key 的数量。

```
#  删除单个 key

redis> SET name huangz
OK

redis> DEL name
(integer) 1


# 删除一个不存在的 key

redis> EXISTS phone
(integer) 0

redis> DEL phone # 失败，没有 key 被删除
(integer) 0


# 同时删除多个 key

redis> SET name "redis"
OK

redis> SET type "key-value store"
OK

redis> SET website "redis.com"
OK

redis> DEL name type website
(integer) 3
```

参考代码：

根据实际情况修改一下就可以拿去应用：

```
const int LOCK_TIMEOUT = 10000000;
const int LOCK_SLEEPTIME = 1000;

std::vector<std::pair<std::string, int> > g_sentinels;
std::string g_master_name;
std::string g_redis_password;
int g_redis_database = 0;

redisContext *g_redis_context = NULL;

inline uint64_t now_microseconds(void) {
  struct timeval tv;
  gettimeofday(&tv, NULL);
  return (uint64_t) tv.tv_sec * 1000000 + (uint64_t) tv.tv_usec;
}

inline void microsleep(int usec) {
  struct timespec req;
  req.tv_sec = 0;
  req.tv_nsec = 1000 * usec;
  nanosleep(&req, NULL);
}

void CloseRedis() {
    if (!g_redis_context)
        return;

    redisFree(g_redis_context);
    g_redis_context = NULL;
}

void ReconnectRedis() {
    CloseRedis();
    g_redis_context = SentinelRedisConnect(g_sentinels, g_master_name, g_redis_password, g_redis_database);
    if (g_redis_context == NULL) {
        LOG(INFO) << "lock redis connect failed!";
        exit(1);
    }
}

bool InitLock(std::vector<std::pair<std::string, int> > sentinels, const char* master_name, const char * password, int database) {
    assert(g_redis_context == NULL);

    g_sentinels = sentinels;
    g_master_name = master_name;
    g_redis_password = password;
    g_redis_database = database;

    ReconnectRedis();
    return g_redis_context != NULL;
}

bool Try(std::string key) {
    key = "lock." + key;

    uint64_t now = now_microseconds();
    uint64_t expired = 0;
    int acquired = 0;

    redisReply* reply;
    reply = (redisReply*)redisCommand(g_redis_context, "SETNX %s %ld", key.c_str(), now + LOCK_TIMEOUT + 1);
    if (reply == NULL || reply->type == REDIS_REPLY_ERROR) {
        if (reply) {
            LOG(ERROR) << "Command error: " << reply->str;
            freeReplyObject(reply);
        } else {
            LOG(ERROR) << "Connection error: " << g_redis_context->errstr;
            ReconnectRedis();
        }
        return false;
    } else if (reply->type == REDIS_REPLY_INTEGER) {
        acquired = reply->integer;
    } else {
        return false;
    }
    freeReplyObject(reply);
    
    if (acquired == 1) {  // acquired the lock
        return true;
    }
    
    reply = (redisReply*)redisCommand(g_redis_context, "GET %s", key.c_str());
    if (reply == NULL || reply->type == REDIS_REPLY_ERROR) {
        if (reply) {
            LOG(ERROR) << "Command error: " << reply->str;
            freeReplyObject(reply);
        } else {
            LOG(ERROR) << "Connection error: " << g_redis_context->errstr;
            ReconnectRedis();
        }
        return false;
    } else if (reply->type == REDIS_REPLY_STRING) {
        expired = atol(reply->str);
    } else if (reply->type == REDIS_REPLY_NIL) {
        expired = 0;
    } else {
        return false;
    }
    freeReplyObject(reply);

    if (expired >= now) {  // not expired
        return false;
    }
        
    reply = (redisReply*)redisCommand(g_redis_context, "GETSET %s %ld", key.c_str(), now + LOCK_TIMEOUT + 1);
    if (reply == NULL || reply->type == REDIS_REPLY_ERROR) {
        if (reply) {
            LOG(ERROR) << "Command error: " << reply->str;
            freeReplyObject(reply);
        } else {
            LOG(ERROR) << "Connection error: " << g_redis_context->errstr;
            ReconnectRedis();
        }
        return false;
    } else if (reply->type == REDIS_REPLY_STRING) { 
        expired = atol(reply->str);
    } else if (reply->type == REDIS_REPLY_NIL) {
        expired = 0;
    } else {
        return false;
    }
    freeReplyObject(reply);

    if (expired >= now) {  // not expired
        return false;
    }

    return true;
}

void Lock(std::string key) {
    int loop = 0;
    for (;;) {
        loop++;
        if (Try(key))
            break;

        microsleep(1000);
    }
    if (loop > 1)
        LOG(INFO) << "Lock: " << key << " , try count: " << loop;
}

void Unlock(std::string key) {
    key = "lock." + key;

    uint64_t now = now_microseconds();
    uint64_t expired = 0;

    redisReply* reply;
    reply = (redisReply*)redisCommand(g_redis_context, "GET %s", key.c_str());
    if (reply == NULL || reply->type == REDIS_REPLY_ERROR) {
        if (reply) {
            LOG(ERROR) << "Command error: " << reply->str;
            freeReplyObject(reply);
        } else {
            LOG(ERROR) << "Connection error: " << g_redis_context->errstr;
            ReconnectRedis();
        }
        return;
    } else if (reply->type == REDIS_REPLY_STRING) {
        expired = atol(reply->str);
    } else if (reply->type == REDIS_REPLY_NIL) {
        expired = 0;
    } else {
        return;
    }
    freeReplyObject(reply);

    if (expired < now) {  // expired
        return;
    }

    reply = (redisReply*)redisCommand(g_redis_context, "DEL %s", key.c_str());
    if (reply == NULL || reply->type == REDIS_REPLY_ERROR) {
        if (reply) {
            LOG(ERROR) << "Command error: " << reply->str;
            freeReplyObject(reply);
        } else {
            LOG(ERROR) << "Connection error: " << g_redis_context->errstr;
            ReconnectRedis();
        }
        return;
    }
    freeReplyObject(reply);
}


```
参考资料：  

https://blog.csdn.net/dazou1/article/details/88088223  

