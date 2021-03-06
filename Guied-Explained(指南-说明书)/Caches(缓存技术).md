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
`getAll(Iterable<? extends K>)`方法可以用来进行批量查询操作，默认情况下，`getAll`一般采用 `CacheLoader.load`
来对每一个key加载缓存项.你可以重写 [`CacheLoader.loadAll`] 去使一个批量加载的效率高于多个单独加载.
`getAll(Iterable)` 的性能也会相应的提高。

注：你可以用一个 `CacheLoader.loadAll` 来实现为“没有明确请求的键”加载缓存值的功能。例如,
你要计算一个可以提供所有键值的组中的任意键的值, 采用`loadAll` 就可以同时获取到组中的其他键值。

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
使用 `Cache.asMap()` 提供的任何 `ConcurrentMap` 方法也能修改缓存。*注：`asMap` 中的任何方法都不能使缓存项自动的加载到缓存中*.
进一步来说，视图的原子运算是在缓存的自动加载范围之外的, 所以使用`CacheLoader` 或者 `Callable` 加载缓存项的时候,
`Cache.get(K, Callable<V>)` 总是优先于 `Cache.asMap().putIfAbsent` 被使用。

## Eviction -- 内存回收

现在有个非常残酷的现实：那就是肯定没有足够的内存来缓存我们需要缓存的内容.
你必须要决定某个项什么时候不需要保留了? Guava Cache 提供了三种内存回收的方式:
基于内存大小的回收、超时回收和基于引用的回收。

### Size-based Eviction -- 基于内存大小的回收(超出你设置的内存大小)

你可以使用 [`CacheBuilder.maximumSize(long)`] 来确定缓存空间的大小.
缓存会尝试清除最近没有使用或者不常使用的缓存项.**警告：** 在缓存值达到你设定的限额之前,
缓存也可能会开始清除操作——这通常发生在缓存量接近设定值的时候.

另一种方式来说，不同的缓存项拥有不同的“权重”. 举个栗子来说：如果你的缓存值占据着不同的内存空间,
你可以使用 [`CacheBuilder.weigher(Weigher)`] 来指定一个权重函数, 同时用  [`CacheBuilder.maximumWeight(long)`]
来指定缓存总量大小. 在有着权重限定的场景中，除了数量接近限定值会开始内存回收之外，还要注意到权重的计算,
计算结果临近限定值时也会开始内存回收。

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

`CacheBuilder` 提供了超时回收的两种方式：

*   [`expireAfterAccess(long, TimeUnit)`] 缓存项在限制时间内没有读/写操作, 它就将被回收.注:这种方式下的回收顺序与基于大小的回收 [size-based eviction] 相同。
*   [`expireAfterWrite(long, TimeUnit)`] 缓存项在限制时间内没有进行写访问(创建/覆盖键值)则回收。如果缓存项被认为在一段时间后变得陈旧不可用，就可以采用此种方式。

就像讨论中的那样, 超时回收周期性的在写操作中执行, 偶尔也在读操作中执行。

#### Testing Timed Eviction -- 超时回收的测试
对于超时回收的测试, 不是一定需要痛苦的等待的...也就是说, 并不是一个两秒的超时回收就一定要等待两秒钟去进行测试.
使用 [Ticker] 接口和 [`CacheBuilder.ticker(Ticker)`] 方法在你的缓存中自定义一个时间源, 以此来代替系统时钟。

### Reference-based Eviction -- 基于引用的回收

通过使用弱引用 [weak references] 来标记键和值、使用软引用 [soft references] 来标记值,
Guava 允许你将缓存设置为垃圾回收(将你的缓存项存入垃圾集合).

*   [`CacheBuilder.weakKeys()`] 将键标记为弱引用. 当这个键没有其他的引用方式(强引用或软引用)时,缓存项可以被垃圾回收.
    因为垃圾回收依赖于强一致(恒等), 所以这导致了这些 "弱引用键的缓存" 采用 `==` 而不是 `equals()` 来比较键。
*   [`CacheBuilder.weakValues()`] 将值标记为弱引用. 当这个值没有其他的引用方式(强引用或软引用)时,缓存项可以被垃圾回收.
    因为垃圾回收依赖于强一致(恒等), 所以这导致了这些 "弱引用键的缓存" 采用 `==` 而不是 `equals()` 来比较值。
*   [`CacheBuilder.softValues()`] 将值标记为软引用. 软引用只是在需要释放内存时才进行垃圾回收, 而且是选择全局最近最少使用的缓存项.
    考虑到使用软引用时导致的一些性能影响, 我们建议采用更有预测性的方法如 **设定缓存最大值**(基于大小的回收策略)
    来进行一些限定。`softValues()` 同样采用 `==` 而不是 `equals()` 来比较值。

### Explicit Removals -- 显示移除

在任何时间, 你都可以指定移除某一缓存项, 而不是等待它被系统回收,你可以采用以下几个方法:

*   单项移除, 使用 [`Cache.invalidate(key)`] 方法
*   部分移除, 使用 [`Cache.invalidateAll(keys)`] 方法
*   全部移除, 使用 [`Cache.invalidateAll()`] 方法

### Removal Listeners -- 移除时监听器

你可以通过 [`CacheBuilder.removalListener(RemovalListener)`] 声明一个 **移除时监听器**,
这可以让你在移除一个缓存项的时候做点其他的事。通过 [`RemovalNotification`] 可以获取一个 [`RemovalListener`],
需要一个[`RemovalCause`]移除原因、key和value, 如下面的代码:
*注：`RemovalListener` 抛出的任何异常，都会在记录到日志中(使用 `Logger` 持久化)后被丢弃(swallowed)*

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
  // 超时回收 2分钟
  .expireAfterWrite(2, TimeUnit.MINUTES)
  .removalListener(removalListener)
  .build(loader);
```

**警告**:默认情况下,移除监听器的触发是和缓存项移除同步进行的, 此时, 性能开销巨大的监听器会拉低缓存效率!
而此时, 你应该使用 [`RemovalListeners.asynchronous(RemovalListener, Executor)`] 来将监听器 `RemovalListener` 装饰为异步操作。

### 清理(内存释放)会在什么时候发生？

使用 `CacheBuilder` 构建的缓存**不会**自动的进行清理或者回收值的操作, 也不会在超时后立即处理, 也没有如上所说的清理机制.
相反的是, 它会在执行写操作的时候进行一小部分的维护工作, 如果写操作实在太少, 那么它也会偶尔在读操作的时候这样做.

这样做的原因在于: 如果我们想不断的对缓存进行维护, 我们需要创建一个线程, 这个线程会和用户操作(读/写)竞争共享锁.
此外, 在某些环境下会限制线程的创建, 那在这样的环境中 `CacheBuilder` 就不能用了。

对此, 我们将选择权交到你的手里. 如果你的缓存是高吞吐量, 那么你完全不用考虑超时清理等类似的维护工作;
如果你的缓存只有一些小量的写操作而你又不希望维护线程阻碍你的读操作, 你就可以创建一个你自己的维护线程定期调用 [`Cache.cleanUp()`] 来进行维护.

如果你想要你的维护线程只在少量的写操作时执行规定的缓存维护任务, [`ScheduledExecutorService`] 会给你提供有效的帮助.

### Refresh -- 刷新

刷新操作并不是与回收操作同步进行的. 正如 [`LoadingCache.refresh(K)`] 中指定声明的那样: 刷新指的是为一个 key 加载新的 value,
可能是异步执行的.刷新过程中, 老的 value 仍然可以被返回(从缓冲中获取), 直到刷新完成; 不像回收, 读取缓存值必须要等回收结束.

如果在刷新过程中抛出了一个异常, 那么旧值会被继续保留, 异常在记录到日志中后被丢弃。

`CacheLoader` 支持开发者重写 [`CacheLoader.reload(K, V)`] 时加入个性化的操作, 比如允许你在计算新值时采用旧值数据。

``` java
// 有些键不需要刷新 所以我们希望刷新是异步操作的。
LoadingCache<Key, Graph> graphs = CacheBuilder.newBuilder()
       .maximumSize(1000)
       .refreshAfterWrite(1, TimeUnit.MINUTES)
       .build(
           new CacheLoader<Key, Graph>() {
             public Graph load(Key key) { // 没有检查异常
               return getGraphFromDatabase(key);
             }

             public ListenableFuture<Graph> reload(final Key key, Graph prevGraph) {
               if (neverNeedsRefresh(key)) {
                 return Futures.immediateFuture(prevGraph);
               } else {
                 // 异步操作！！！+-
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
缓存定时自动刷新可以添加到一个 [`CacheBuilder.refreshAfterWrite(long, TimeUnit)`] 中。
相比于 `expireAfterWrite`, `refreshAfterWrite`通过定时刷新使缓存项在一定时间内可用, 但是缓存项只有在 key 
被检索时才会真正的刷新(如果 `CacheLoader.reload` 已经实现异步, 那么检索性能/速度不会因为刷新而减慢).
所以，你可以在同一个缓存上同时使用 `refreshAfterWrite` 和 `expireAfterWrite` , 缓存项不会因为触发刷新而盲目的重置,
因为如果缓存项没有被检索, 那么刷新就不会真正的生效, 缓存项在过期之后也可以被回收。

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
