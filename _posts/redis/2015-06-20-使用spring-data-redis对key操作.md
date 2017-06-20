---
layout: post
title:  "使用spring-data-redis对key操作"
date:   2017-06-20 11:43:01 +0800
categories: redis
tag: redis 原创
sid: 1497947228
---

###   使用spring-data-redis对key操作

> 平台主要使用java进行开发,这里主要细说redis相关操作

**主要使用spring-data-redis组件进行开发,没有直接使用jedis开发**

在pom.xml添加如下依赖(当前最新版本):

~~~xml
<!-- https://mvnrepository.com/artifact/org.springframework.data/spring-data-redis -->
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.0.0.M2</version>
</dependency>
~~~

spring-data-redis对key进行操作的接口为:RedisKeyCommands,源码如下
~~~java
public interface RedisKeyCommands {

	/**
	 * Determine if given {@code key} exists.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/exists">Redis Documentation: EXISTS</a>
	 */
	Boolean exists(byte[] key);

	/**
	 * Delete given {@code keys}.
	 *
	 * @param keys must not be {@literal null}.
	 * @return The number of keys that were removed.
	 * @see <a href="http://redis.io/commands/del">Redis Documentation: DEL</a>
	 */
	Long del(byte[]... keys);

	/**
	 * Determine the type stored at {@code key}.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/type">Redis Documentation: TYPE</a>
	 */
	DataType type(byte[] key);

	/**
	 * Find all keys matching the given {@code pattern}.
	 *
	 * @param pattern must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/keys">Redis Documentation: KEYS</a>
	 */
	Set<byte[]> keys(byte[] pattern);

	/**
	 * Use a {@link Cursor} to iterate over keys.
	 *
	 * @param options must not be {@literal null}.
	 * @return
	 * @since 1.4
	 * @see <a href="http://redis.io/commands/scan">Redis Documentation: SCAN</a>
	 */
	Cursor<byte[]> scan(ScanOptions options);

	/**
	 * Return a random key from the keyspace.
	 *
	 * @return
	 * @see <a href="http://redis.io/commands/randomkey">Redis Documentation: RANDOMKEY</a>
	 */
	byte[] randomKey();

	/**
	 * Rename key {@code oldName} to {@code newName}.
	 *
	 * @param oldName must not be {@literal null}.
	 * @param newName must not be {@literal null}.
	 * @see <a href="http://redis.io/commands/rename">Redis Documentation: RENAME</a>
	 */
	void rename(byte[] oldName, byte[] newName);

	/**
	 * Rename key {@code oldName} to {@code newName} only if {@code newName} does not exist.
	 *
	 * @param oldName must not be {@literal null}.
	 * @param newName must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/renamenx">Redis Documentation: RENAMENX</a>
	 */
	Boolean renameNX(byte[] oldName, byte[] newName);

	/**
	 * Set time to live for given {@code key} in seconds.
	 *
	 * @param key must not be {@literal null}.
	 * @param seconds
	 * @return
	 * @see <a href="http://redis.io/commands/expire">Redis Documentation: EXPIRE</a>
	 */
	Boolean expire(byte[] key, long seconds);

	/**
	 * Set time to live for given {@code key} in milliseconds.
	 *
	 * @param key must not be {@literal null}.
	 * @param millis
	 * @return
	 * @see <a href="http://redis.io/commands/pexpire">Redis Documentation: PEXPIRE</a>
	 */
	Boolean pExpire(byte[] key, long millis);

	/**
	 * Set the expiration for given {@code key} as a {@literal UNIX} timestamp.
	 *
	 * @param key must not be {@literal null}.
	 * @param unixTime
	 * @return
	 * @see <a href="http://redis.io/commands/expireat">Redis Documentation: EXPIREAT</a>
	 */
	Boolean expireAt(byte[] key, long unixTime);

	/**
	 * Set the expiration for given {@code key} as a {@literal UNIX} timestamp in milliseconds.
	 *
	 * @param key must not be {@literal null}.
	 * @param unixTimeInMillis
	 * @return
	 * @see <a href="http://redis.io/commands/pexpireat">Redis Documentation: PEXPIREAT</a>
	 */
	Boolean pExpireAt(byte[] key, long unixTimeInMillis);

	/**
	 * Remove the expiration from given {@code key}.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/persist">Redis Documentation: PERSIST</a>
	 */
	Boolean persist(byte[] key);

	/**
	 * Move given {@code key} to database with {@code index}.
	 *
	 * @param key must not be {@literal null}.
	 * @param dbIndex
	 * @return
	 * @see <a href="http://redis.io/commands/move">Redis Documentation: MOVE</a>
	 */
	Boolean move(byte[] key, int dbIndex);

	/**
	 * Get the time to live for {@code key} in seconds.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/ttl">Redis Documentation: TTL</a>
	 */
	Long ttl(byte[] key);

	/**
	 * Get the time to live for {@code key} in and convert it to the given {@link TimeUnit}.
	 *
	 * @param key must not be {@literal null}.
	 * @param timeUnit must not be {@literal null}.
	 * @return
	 * @since 1.8
	 * @see <a href="http://redis.io/commands/ttl">Redis Documentation: TTL</a>
	 */
	Long ttl(byte[] key, TimeUnit timeUnit);

	/**
	 * Get the precise time to live for {@code key} in milliseconds.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/pttl">Redis Documentation: PTTL</a>
	 */
	Long pTtl(byte[] key);

	/**
	 * Get the precise time to live for {@code key} in and convert it to the given {@link TimeUnit}.
	 *
	 * @param key must not be {@literal null}.
	 * @param timeUnit must not be {@literal null}.
	 * @return
	 * @since 1.8
	 * @see <a href="http://redis.io/commands/pttl">Redis Documentation: PTTL</a>
	 */
	Long pTtl(byte[] key, TimeUnit timeUnit);

	/**
	 * Sort the elements for {@code key}.
	 *
	 * @param key must not be {@literal null}.
	 * @param params must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/sort">Redis Documentation: SORT</a>
	 */
	List<byte[]> sort(byte[] key, SortParameters params);

	/**
	 * Sort the elements for {@code key} and store result in {@code storeKey}.
	 *
	 * @param key must not be {@literal null}.
	 * @param params must not be {@literal null}.
	 * @param storeKey must not be {@literal null}.
	 * @return number of values.
	 * @see <a href="http://redis.io/commands/sort">Redis Documentation: SORT</a>
	 */
	Long sort(byte[] key, SortParameters params, byte[] storeKey);

	/**
	 * Retrieve serialized version of the value stored at {@code key}.
	 *
	 * @param key must not be {@literal null}.
	 * @return
	 * @see <a href="http://redis.io/commands/dump">Redis Documentation: DUMP</a>
	 */
	byte[] dump(byte[] key);

	/**
	 * Create {@code key} using the {@code serializedValue}, previously obtained using {@link #dump(byte[])}.
	 *
	 * @param key must not be {@literal null}.
	 * @param ttlInMillis
	 * @param serializedValue must not be {@literal null}.
	 * @see <a href="http://redis.io/commands/restore">Redis Documentation: RESTORE</a>
	 */
	void restore(byte[] key, long ttlInMillis, byte[] serializedValue);
}
~~~
基本覆盖了redis官网对key操作的命令(redis官方对keys进行操作的所有命令地址:[命令](https://redis.io/commands#generic)),如下:
- DEL                 删除某个指定的key, [官网说明](https://redis.io/commands/del)
- DUMP             序列化给定 key ,并返回被序列化的值 [官网说明](https://redis.io/commands/dump)
- EXISTS            检查给定 key 是否存在 [官网说明](https://redis.io/commands/exists)
- EXPIRE           为给定 key 设置生存时间,当 key 过期时,它会被自动删除  [官网说明](https://redis.io/commands/expire)
- EXPIREAT      接受时间戳来设置key的过期时间 [官网说明](https://redis.io/commands/expireat)
- KEYS                查找所有符合给定模式 pattern 的 key [官网说明](https://redis.io/commands/keys)
- MIGRATE       将 key 原子性地从当前实例传送到目标实例的指定数据库上，一旦传送成功， key 保证会出现在目标实例上，而当前实例上的 key 会被删除 [官网说明](https://redis.io/commands/migrate)
- MOVE              将当前数据库的 key 移动到给定的数据库 db    [官网说明](https://redis.io/commands/move)
- OBJECT           OBJECT 命令允许从内部察看给定 key 的 Redis 对象 [官网说明](https://redis.io/commands/object)
- PERSIST          将这个 key 从『易失的』(带生存时间 key )转换成『持久的』[官网说明](https://redis.io/commands/persist)
- PEXPIRE          这个命令和 EXPIRE 命令的作用类似，但是它以毫秒为单位设置 key 的生存时间，而不像 EXPIRE 命令那样，以秒为单位 [官网说明](https://redis.io/commands/pexpire)
- PEXPIREAT     这个命令和 EXPIREAT 命令类似，但它以毫秒为单位设置 key 的过期 unix 时间戳，而不是像 EXPIREAT 那样，以秒为单位 [官网说明](https://redis.io/commands/pexpireat)
- PTTL                 这个命令类似于 TTL 命令，但它以毫秒为单位返回 key 的剩余生存时间，而不是像 TTL 命令那样，以秒为单位 [官网说明](https://redis.io/commands/pttl)
- RANDOMKEY 从当前数据库中随机返回(不删除)一个 key [官网说明](https://redis.io/commands/randomkey)
- RENAME          将 key 改名为 newkey [官网说明](https://redis.io/commands/rename)
- RENAMENX    当且仅当 newkey 不存在时，将 key 改名为 newkey [官网说明](https://redis.io/commands/renamenx)
- RESTORE        反序列化给定的序列化值，并将它和给定的 key 关联    [官网说明](https://redis.io/commands/restore)
- SCAN                SCAN 命令是一个基于游标的迭代器（cursor based iterator）： SCAN 命令每次被调用之后， 都会向用户返回一个新的游标， 用户在下次迭代时需要使用这个新游标作为 SCAN 命令的游标参数， 以此来延续之前的迭代过程 [官网说明](https://redis.io/commands/scan)
- SORT                返回或保存给定列表、集合、有序集合 key 中经过排序的元素 [官网说明](https://redis.io/commands/sort)
- TOUCH             接收若干个key,返回这些key存在的数量 [官网说明](https://redis.io/commands/touch)
- TTL                   以秒为单位，返回给定 key 的剩余生存时间 [官网说明](https://redis.io/commands/ttl)
- TYPE                 返回 key 所储存的值的类型 [官网说明](https://redis.io/commands/type)
- UNLINK           删除给定的key [官网说明](https://redis.io/commands/unlink)
- WAIT                阻塞当前操作,让之前的写操作完成之后才能继续 [官网说明](https://redis.io/commands/wait)

