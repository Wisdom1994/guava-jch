# Caches(缓存技术)

## Example -- 举个栗子

``` java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .expireAfterWrite(10, TimeUnit.MINUTES)
       .removalListener(MY_LISTENER)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
```

## Applicability -- 适用于

缓存在很多场景下都非常有用.比如,当计算或者搜索一个值的开销特别大、或者需要多次获取同一个输出而产生的值的时候.

`Cache` 有些类似于 Java 集合类中的 `ConcurrentMap`, 但又不是完全相同.
最基本的不同之处在于 `ConcurrentMap` 会保存所有的元素直到这些元素被显示的移除,
而 `Cache` 为了限制内存占用通常会被设置为自动清理元素.在某些情况下,
尽管`LoadingCache` 从不回收元素,它也是很有用的,它会在必要的时候自动加载缓存. 

总的来说, Guava 缓存适用于以下几个方面：

*   你愿意使用更多的内存来提升速度(空间换时间).
*   你预料到某些键将会被使用一次以上.
*   缓存数据量不会超过你的内存大小.(Guava caches 是你应用程序上单线程的本地缓存, 数据不会存储到文件或者外部服务器中.
    如果这满足不了你的需求,请考虑一下比如 [Memcached](http://memcached.org/) 的工具.(注：[Redis](https://redis.io) 也可以))

如果你的应用场景符合上边的每一条,Guava Caches 就非常满足你的要求.

如同示例一样,`Cache` 可以通过 `CacheBuilder` 来生成,但是自定义你的 `Cache` 才是最有趣的一部分. 

注：如果你不需要 `Cache` 的一些特性,`ConcurrentHashMap` 有更优的内存效率,
但是通过 `ConcurrentMap` 来复制(实现) `Cache` 的一些特性却是极其困难或者说是不可能的.

## Population -- 成员

关于你的缓存,你首先应该问自己一个问题：有没有一个 **合理、默认** 的方法去加载或者计算一个与键关联的值?
如果有,你应该采用 `CacheLoader`,如果没有,或者你想要重写默认的 **加载-计算** 方法,而且希望保有 **获取缓存-若没有-进行计算**
[get-if-absent-compute] 的原始语义(实现思路), 你应该在调用`get` 方法时传入(pass)一个`Callable`实例.
我们可以将元素通过 `Cache.put` 方法直接插入,但是采用自动加载仍然是首选方案,因为它可以更容易的推断缓存内容的一致性.

#### From a CacheLoader -- Cache加载器
`LoadingCache` 是附带 [`CacheLoader`] 构建的一个缓存实现.构建一个 `CacheLoader` 非常容易,
你只需要实现 `V load(K key) throws Exception` 方法.举个简单的栗子,你可以模仿下面的代码创建一个`LoadingCache`.

``` java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) throws AnyException {
               return createExpensiveGraph(key);
             }
           });
...
try {
  return graphs.get(key);
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

获取一个 `LoadingCache` 的标准方式是使用[`get(K)`]方法，这个方法要么返回一个准备好的(已缓存)的值,
要么使用缓存的`CacheLoader` **原子性** 的向缓存中加载新的值。因为 `CacheLoader` 可能抛出 `Exception`,
`LoadingCache.get(K)` 也会抛出一个`ExecutionException`(如果缓存加载器抛出一个**非检查时异常**,
那么`get(K)`就会返回一个包装过的`UncheckedExecutionException` 异常)。
如果你的 `CacheLoader` 没有声明任何检查时异常, 你可以使用 `getUnchecked(K)` (包装了所有的`UncheckedExecutionException`).
如果它声明了检查时异常，那么这就会造成一些令人惊讶的(难以解决)的问题(不能使用 `getUnchecked(K)`);

``` java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .expireAfterAccess(10, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });

...
return graphs.getUnchecked(key);
```

Bulk lookups can be performed with the method `getAll(Iterable<? extends K>)`.
By default, `getAll` will issue a a separate call to `CacheLoader.load` for each
key which is absent from the cache. When bulk retrieval is more efficient than
many individual lookups, you can override [`CacheLoader.loadAll`] to exploit
this. The performance of `getAll(Iterable)` will improve accordingly.

Note that you can write a `CacheLoader.loadAll` implementation that loads values
for keys that were not specifically requested. For example, if computing the
value of any key from some group gives you the value for all keys in the group,
`loadAll` might load the rest of the group at the same time.

#### From a Callable -- 回调

所有的 Guava caches, 不管有没有自动加载,都支持 [`get(K, Callable<V>)`] 方法.
这个方法返回了缓存中相对应的值,或者对特定`Callable` 返回的值进行计算,并将结果添加到缓存中.
在加载完成之间,缓存的可观察状态都不会改变,这个方法简单的实现了“如果缓存,返回；否则计算、缓存然后返回”.

``` java
Cache<Key, Value> cache = CacheBuilder.newBuilder()
    .maximumSize(1000)
    .build(); // look Ma, no CacheLoader
...
try {
  // If the key wasn't in the "easy to compute" group, we need to
  // do things the hard way.
  cache.get(key, new Callable<Value>() {
    @Override
    public Value call() throws AnyException {
      return doThingsTheHardWay(key);
    }
  });
} catch (ExecutionException e) {
  throw new OtherException(e.getCause());
}
```

#### Inserted Directly -- 显示插入

值可以通过 [`cache.put(key, value)`] 方法显示的插入到缓存中.这会覆盖掉这个key以前所映射的任何的值.
. Changes
can also be made to a cache using any of the `ConcurrentMap` methods exposed by
the `Cache.asMap()` view. Note that no method on the `asMap` view will ever
cause entries to be automatically loaded into the cache. Further, the atomic
operations on that view operate outside the scope of automatic cache loading, so
`Cache.get(K, Callable<V>)` should always be preferred over
`Cache.asMap().putIfAbsent` in caches which load values using either
`CacheLoader` or `Callable`.

## Eviction -- 内存回收

The cold hard reality is that we almost _certainly_ don't have enough memory to
cache everything we could cache. You must decide: when is it not worth keeping a
cache entry? Guava provides three basic types of eviction: size-based eviction,
time-based eviction, and reference-based eviction.

### Size-based Eviction -- 基于内存大小的回收(超出你设置的内存大小)

If your cache should not grow beyond a certain size, just use
[`CacheBuilder.maximumSize(long)`]. The cache will try to evict entries that
haven't been used recently or very often. _Warning_: the cache may evict entries
before this limit is exceeded -- typically when the cache size is approaching
the limit.

Alternately, if different cache entries have different "weights" -- for example,
if your cache values have radically different memory footprints -- you may
specify a weight function with [`CacheBuilder.weigher(Weigher)`] and a maximum
cache weight with [`CacheBuilder.maximumWeight(long)`]. In addition to the same
caveats as `maximumSize` requires, be aware that weights are computed at entry
creation time, and are static thereafter.

``` java
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumWeight(100000)
       .weigher(new Weigher<Key, Graph>() {
          public int weigh(Key k, Graph g) {
            return g.vertices().size();
          }
        })
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return createExpensiveGraph(key);
             }
           });
```

### Timed Eviction -- 超时回收

`CacheBuilder` provides two approaches to timed eviction:

*   [`expireAfterAccess(long, TimeUnit)`] Only expire entries after the
    specified duration has passed since the entry was last accessed by a read or
    a write. Note that the order in which entries are evicted will be similar to
    that of [size-based eviction].
*   [`expireAfterWrite(long, TimeUnit)`] Expire entries after the specified
    duration has passed since the entry was created, or the most recent
    replacement of the value. This could be desirable if cached data grows stale
    after a certain amount of time.

Timed expiration is performed with periodic maintenance during writes and
occasionally during reads, as discussed below.

#### Testing Timed Eviction -- 测试定时回收

Testing timed eviction doesn't have to be painful...and doesn't actually have to
take you two seconds to test a two-second expiration. Use the [Ticker] interface
and the [`CacheBuilder.ticker(Ticker)`] method to specify a time source in your
cache builder, rather than having to wait for the system clock.

### Reference-based Eviction -- 基于引用的回收

Guava allows you to set up your cache to allow the garbage collection of
entries, by using [weak references] for keys or values, and by using [soft
references] for values.

*   [`CacheBuilder.weakKeys()`] stores keys using weak references. This allows
    entries to be garbage-collected if there are no other (strong or soft)
    references to the keys. Since garbage collection depends only on identity
    equality, this causes the whole cache to use identity (`==`) equality to
    compare keys, instead of `equals()`.
*   [`CacheBuilder.weakValues()`] stores values using weak references. This
    allows entries to be garbage-collected if there are no other (strong or
    soft) references to the values. Since garbage collection depends only on
    identity equality, this causes the whole cache to use identity (`==`)
    equality to compare values, instead of `equals()`.
*   [`CacheBuilder.softValues()`] wraps values in soft references. Softly
    referenced objects are garbage-collected in a globally least-recently-used
    manner, _in response to memory demand_. Because of the performance
    implications of using soft references, we generally recommend using the more
    predictable [maximum cache size][size-based eviction] instead. Use of `softValues()` will cause
    values to be compared using identity (`==`) equality instead of `equals()`.

### Explicit Removals -- 显示移除

At any time, you may explicitly invalidate cache entries rather than waiting for
entries to be evicted. This can be done:

*   individually, using [`Cache.invalidate(key)`]
*   in bulk, using [`Cache.invalidateAll(keys)`]
*   to all entries, using [`Cache.invalidateAll()`]

### Removal Listeners -- 移除时监听器

You may specify a removal listener for your cache to perform some operation when
an entry is removed, via [`CacheBuilder.removalListener(RemovalListener)`]. The
[`RemovalListener`] gets passed a [`RemovalNotification`], which specifies the
[`RemovalCause`], key, and value.

Note that any exceptions thrown by the `RemovalListener` are logged (using
`Logger`) and swallowed.

``` java
CacheLoader<Key, DatabaseConnection> loader = new CacheLoader<Key, DatabaseConnection> () {
  public DatabaseConnection load(Key key) throws Exception {
    return openConnection(key);
  }
};
RemovalListener<Key, DatabaseConnection> removalListener = new RemovalListener<Key, DatabaseConnection>() {
  public void onRemoval(RemovalNotification<Key, DatabaseConnection> removal) {
    DatabaseConnection conn = removal.getValue();
    conn.close(); // tear down properly
  }
};

return CacheBuilder.newBuilder()
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

**警告**: removal listener operations are executed synchronously by default,
and since cache maintenance is normally performed during normal cache
operations, expensive removal listeners can slow down normal cache function! If
you have an expensive removal listener, use
[`RemovalListeners.asynchronous(RemovalListener, Executor)`] to decorate a
`RemovalListener` to operate asynchronously.

### 清理(内存释放)会在什么时候发生？

Caches built with `CacheBuilder` do _not_ perform cleanup and evict values
"automatically," or instantly after a value expires, or anything of the sort.
Instead, it performs small amounts of maintenance during write operations, or
during occasional read operations if writes are rare.

The reason for this is as follows: if we wanted to perform `Cache` maintenance
continuously, we would need to create a thread, and its operations would be
competing with user operations for shared locks. Additionally, some environments
restrict the creation of threads, which would make `CacheBuilder` unusable in
that environment.

Instead, we put the choice in your hands. If your cache is high-throughput, then
you don't have to worry about performing cache maintenance to clean up expired
entries and the like. If your cache does writes only rarely and you don't want
cleanup to block cache reads, you may wish to create your own maintenance thread
that calls [`Cache.cleanUp()`] at regular intervals.

If you want to schedule regular cache maintenance for a cache which only rarely
has writes, just schedule the maintenance using [`ScheduledExecutorService`].

### Refresh -- 刷新

Refreshing is not quite the same as eviction. As specified in
[`LoadingCache.refresh(K)`], refreshing a key loads a new value for the key,
possibly asynchronously. The old value (if any) is still returned while the key
is being refreshed, in contrast to eviction, which forces retrievals to wait
until the value is loaded anew.

If an exception is thrown while refreshing, the old value is kept, and the
exception is logged and swallowed.

A `CacheLoader` may specify smart behavior to use on a refresh by overriding
[`CacheLoader.reload(K, V)`], which allows you to use the old value in computing
the new value.

``` java
// Some keys don't need refreshing, and we want refreshes to be done asynchronously.
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .refreshAfterWrite(1, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // no checked exception
               return getGraphFromDatabase(key);
             }

             public ListenableFuture<Graph> reload(final Key key, Graph prevGraph) {
               if (neverNeedsRefresh(key)) {
                 return Futures.immediateFuture(prevGraph);
               } else {
                 // asynchronous!
                 ListenableFutureTask<Graph> task = ListenableFutureTask.create(new Callable<Graph>() {
                   public Graph call() {
                     return getGraphFromDatabase(key);
                   }
                 });
                 executor.execute(task);
                 return task;
               }
             }
           });
```

Automatically timed refreshing can be added to a cache using
[`CacheBuilder.refreshAfterWrite(long, TimeUnit)`]. In contrast to
`expireAfterWrite`, `refreshAfterWrite` will make a key _eligible_ for refresh
after the specified duration, but a refresh will only be actually initiated when
the entry is queried. (If `CacheLoader.reload` is implemented to be
asynchronous, then the query will not be slowed down by the refresh.) So, for
example, you can specify both `refreshAfterWrite` and `expireAfterWrite` on the
same cache, so that the expiration timer on an entry isn't blindly reset
whenever an entry becomes eligible for a refresh, so if an entry isn't queried
after it comes eligible for refreshing, it is allowed to expire.

## Features -- 特性

### Statistics -- 统计(对Caches工作状态的统计)

通过使用 [`CacheBuilder.recordStats()`] 方法开启 Guava cache 的统计功能。统计开启后,
[`Cache.stats()`]方法返回一个[`CacheStats`]对象，里面提供了如下的统计信息：

*   [`hitRate()`],返回缓存的命中率
*   [`averageLoadPenalty()`], 返回加载新值的平均时间，单位为纳秒(nanoseconds)
*   [`evictionCount()`], 被回收的缓存数量。

还有很多其他的统计信息，这些信息对于调整缓存设置至关重要，在一些特别要求性能的应用中，我们应该对此保持密切关注。

### `asMap` -- asMap视图

你可以使用 `ConcurrentMap` 的 `asMap` 来创建一个缓存视图，但是对于 `asMap` 视图和缓存的交互机制，这里要作一些解释：

*   `cache.asMap()` 包含所有加载到缓存中的项,比如`cache.asMap().keySet()` 包含了所有已经加载到缓存中的 key 值。
*   `asMap().get(key)` 本质上相当于 `cache.getIfPresent(key)`，而且不会引起缓存项得加载, 这与 `Map` 语义约定一致。
*   所有缓存项的读写操作都会重置相关缓存项的读取时间(包括`Cache.asMap().get(Object)` 和 `Cache.asMap().put(K, V)`),
但是 `containsKey(Object)` 方法不包括在其中，同样也不包含 `Cache.asMap()` 方法在集合视图上的操作.
比如遍历`cache.asMap().entrySet()` 就不会重置缓存项的读取时间。

## Interruption -- 中断

缓存加载方法(比如 `get` )从来不会抛出 `InterruptedException` 异常.我们应该设计一些方法去支持 `InterruptedException`,
但是这种支持通常是不完善的,强迫增加所有使用者的开销,可是却只有少部分获益.

`get` 请求到未缓存的值时通常是因为两个方面：一个是 **当前线程加载值**；二是 **等待另一个加载值的线程**.
所以我们支持中断的两种方式是不一样的.**等待另一个加载值的线程** 是较为简单的一种情况：这里我们可以加入一个中断等待；
**当前线程加载值** 的中断就比较困难：线程运行在用户提供的`CacheLoader`中,如果它是可中断的,我们就可以实现对中断的支持,
如果不可中断,那么就不行.

所以为什么用户提供的`CacheLoader` 是可中断的,而 `get` 却不提供显示的支持? 某种意义上来说,我们提供了支持:
如果一个 `CacheLoader` 抛出了一个 `InterruptedException` 异常,`get` 方法将立刻返回 key 值(与其他的异常情况相同).
此外,在正在执行的线程中,`get` 捕捉到`InterruptedException` 后将会恢复中断,其他的线程则将 `InterruptedException` 包装成 `ExecutionException`.

原则上来说,我们可以将 `ExecutionException` 接封装为 `InterruptedException`,但是这会导致所有的 `LoadingCache` 的使用者都要处理异常,
尽管大部分的`CacheLoader` 的实现都没有抛出这个异常. 你可能认为所有的**非加载线程** 的等待都应该可以被中断,
这种想法是很有价值的. 但是很多情况下,缓存只使用在单个线程中, 它们的用户仍然需要catch(捕获)那个不可能被抛出的 `InterruptedException` 异常.
那些跨线程共享缓存的用户也只是在**有的时候**中断它们的`get`调用，这个时间取决于哪个线程先发出了请求。

我们的一个设计原则是：让缓存看上去只是在当前线程中加载值。这个原则使得 **每次将caching引入代码预先计算它的值** 变得容易实现.
如果老代码不能被中断，那么新的代码同样是不能被中断的。

所以说**在某种意义上**我们(Guava) 支持中断, 而在另一个意义上来说, 我们(Guava)又不支持, 这使得 `LoadingCache` 是一个有漏洞的抽象：
如果加载中线程被中断了, 我们将它当做其他异常一样进行处理, 这在某些情况下来说是可以的; 但是当多个`get` 线程等待加载同一个缓存项时,
就是不正确的, 这种情况下, 即使这个**计算中**线程被中断了, 其他的线程(也捕获到了`InterruptedException`异常)也不应都失败,
正确的行为是让某个线程重新加载。为此，我们记录了一个 [bug](https://github.com/google/guava/issues/1122). 然而,
逾期冒风险修复这个bug，不如花更多的精力去设计一个 `AsyncLoadingCache`(同步的), 这个实现会返回一个具有可中断行为的 `Future` 对象。

[`CacheLoader`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html
[`get(K)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#get-K-
[`CacheLoader.loadAll`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#loadAll-java.lang.Iterable-
[`get(K, Callable<V>)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#get-java.lang.Object-java.util.concurrent.Callable-
[`cache.put(key, value)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#put-K-V-
[`CacheBuilder.maximumSize(long)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumSize-long-
[`CacheBuilder.weigher(Weigher)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weigher-com.google.common.cache.Weigher-
[`CacheBuilder.maximumWeight(long)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#maximumWeight-long-
[`expireAfterAccess(long, TimeUnit)`]: https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterAccess-long-java.util.concurrent.TimeUnit-
[size-based eviction]: #Size-based-Eviction
[`expireAfterWrite(long, TimeUnit)`]: https://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#expireAfterWrite-long-java.util.concurrent.TimeUnit-
[Ticker]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Ticker.html
[`CacheBuilder.ticker(Ticker)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#ticker-com.google.common.base.Ticker-
[weak references]: http://docs.oracle.com/javase/6/docs/api/java/lang/ref/WeakReference.html
[soft references]: http://docs.oracle.com/javase/6/docs/api/java/lang/ref/SoftReference.html
[`CacheBuilder.weakKeys()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakKeys--
[`CacheBuilder.weakValues()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#weakValues--
[`CacheBuilder.softValues()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#softValues--
[`Cache.invalidate(key)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidate-java.lang.Object-
[`Cache.invalidateAll(keys)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll-java.lang.Iterable-
[`Cache.invalidateAll()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#invalidateAll--
[`CacheBuilder.removalListener(RemovalListener)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#removalListener-com.google.common.cache.RemovalListener-
[`RemovalListener`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListener.html
[`RemovalNotification`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalNotification.html
[`RemovalCause`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalCause.html
[`RemovalListeners.asynchronous(RemovalListener, Executor)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/RemovalListeners.html#asynchronous-com.google.common.cache.RemovalListener-java.util.concurrent.Executor-
[`Cache.cleanUp()`]: http://google.github.io/guava/releases/11.0.1/api/docs/com/google/common/cache/Cache.html#cleanUp--
[`ScheduledExecutorService`]: http://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html
[`LoadingCache.refresh(K)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/LoadingCache.html#refresh-K-
[`CacheLoader.reload(K, V)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheLoader.html#reload-K-V-
[`CacheBuilder.refreshAfterWrite(long, TimeUnit)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheBuilder.html#refreshAfterWrite-long-java.util.concurrent.TimeUnit-
[`CacheBuilder.recordStats()`]: http://google.github.io/guava/releases/12.0/api/docs/com/google/common/cache/CacheBuilder.html#recordStats--
[`Cache.stats()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/Cache.html#stats--
[`CacheStats`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html
[`hitRate()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#hitRate--
[`averageLoadPenalty()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#averageLoadPenalty--
[`evictionCount()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/cache/CacheStats.html#evictionCount--
