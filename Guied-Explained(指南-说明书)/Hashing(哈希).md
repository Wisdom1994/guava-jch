# Hashing (哈希)

## Overview (概述)

Java 的 HashCode 的长度被限制在 32 位，并且哈希算法与他们作用的数据(类或方法)之间没有分离，
因此很难使用另外的哈希算法进行替换。同时，Java 内建的哈希算法生成的 Hash 值是十分劣质的，有一部分原因是因为他们依赖于劣质的 Hashcode 实现，其中包括很多 JDK 中的实现类。

Java 中 `Object.hashCode` 运行十分快速，但是缺乏对 **哈希碰撞** 问题的预防，同时也缺乏对散列值的期望。这种特性却让他十分适合与应用在哈希表中，因为额外的哈希碰撞只会
带来很小的性能损失，同时十分差劲的分散性可以采用二次哈希的方法解决(Java 中几乎所有
合理的哈希算法(哈希函数)都采用这种方法进行实现)。然而，在简单哈希表之外的很多哈希应用中，
`Object.hashCode`却是不足的，因此`com.google.common.hash`包被设计出来。

## Organization (组成)

通过查看包的 Java doc 文档， 我们会发现很多不同的类，但是并没有明显的说明他们是如何协同工作的。

让我们先看一下这个类库的一部分代码：

``` java
HashFunction hf = Hashing.md5();
HashCode hc = hf.newHasher()
       .putLong(id)
       .putString(name, Charsets.UTF_8)
       .putObject(person, personFunnel)
       .hash();
```

#### HashFunction (哈希方法--生成Hash值的方法)

[`HashFunction`] 是一个对引用透明的、无状态(无返回值)的方法，他把任意的数据块映射到固定长度的地址中，
尽量确保相同的输入一定产生相同的输出，不同的输入产生不同的输出。 

#### Hasher (哈希对象)

一个 `HashFunciton` 可以提供一个有状态的 [`Hasher`]，它提供了可以将输入数据十分快速的计算出哈希值的方法。
`Hasher` 可以接受任何形式的数据，byte arrays(字节数组)、slices of byte arrays(字节数组的片段)、character squences(特定字符集的字符序列)等等，
或者是任何一个提供了 `Funnel` 实现的对象。

`Hasher` 实现了 `PrimitiveSink` 接口，为 **一个接受原生数据类型的流的对象** 定义了 fluent 风格的 API。

#### Funnel

一个 [`Funnel`] 描述了如何将一个确定的对象分解为原生字段值。比如，我们有这样一个对象：

``` java
class Person {
  final int id;
  final String firstName;
  final String lastName;
  final int birthYear;
}
```

我们的 `Funnel` 可能看起来像这样

``` java
Funnel<Person> personFunnel = new Funnel<Person>() {
  @Override
  public void funnel(Person person, PrimitiveSink into) {
    into
        .putInt(person.id)
        .putString(person.firstName, Charsets.UTF_8)
        .putString(person.lastName, Charsets.UTF_8)
        .putInt(birthYear);
  }
};
```

注： `putString("abc", Charsets.UTF_8).putString("def", Charsets.UTF_8)` 等同于 `putString("ab", Charsets.UTF_8).putString("cdef",
Charsets.UTF_8)`, 因为他们生成相同的字节序列。这可能导致意料之外的 Hash冲突，
增加某种形式的分隔符有助于减少 Hash冲突。

#### HashCode

一旦 `Hasher` 拿到了所有的输入值, 他就可以调用 [`hash()`] 方法得到一个 [`HashCode`] 实例。
`HashCode` 支持平等性检测，比如 [`asInt()`], [`asLong()`], [`asBytes()`] 方法等,此外，
[`writeBytesTo(array, offset, maxLength)`] 方法支持将哈希值前 `maxLength` 长度的字节写入到数组中。

``` Java
// 注：writeBytesTo 方法在 Guava 中如此定义： 
@CanIgnoreReturnValue
public int writeBytesTo(byte[] dest, int offset, int maxLength)
/**
解释：
    将 Hash code 的字节串拷贝到目标数组中
参数:
    dest - 将被字节数组写入的位置
    offset - 数据起始位
    maxLength - 字节数组的最大写入长度
返回值:
    int 写入到目标位置的字节数
*/
```

### BloomFilter (一个过滤器)

Bloom filters 是哈希运算的一个优雅的运用, 不能简单的采用 `Object.hashCode()` 来实现。
简而言之，BloomFilter 是一个概率数据结构, 允许你测试一个对象**一定**不在这个过滤器中,
或者他**可能**已经存在与里面。对此，[BloomFilter 的维基页面](http://en.wikipedia.org/wiki/Bloom_filter) 进行了相当详细的解释,
同时这里推荐一个关于BloomFilter 的 [教程](http://llimllib.github.com/bloomfilter-tutorial/)。

Guava 的哈希类库中有个内建的 `BloomFilter` 实现, 只需要你提供一个 `Funnel` 的实现类，就能将他分解为原始类型。
你可以通过 [`create(Funnel funnel, int
expectedInsertions, double falsePositiveProbability)`] 方法来获取一个 [`BloomFilter<T>`] 对象,
缺省误检率只有 3%。 `BloomFilter<T>` 提供了 [`boolean mightContain(T)`] 和 [`void put(T)`] 方法,
他们的功能显而易见。

``` java
BloomFilter<Person> friends = BloomFilter.create(personFunnel, 500, 0.01);
for (Person friend : friendsList) {
  friends.put(friend);
}
// 很久之后... 
if (friends.mightContain(dude)) { // 如果 dude 可能包括在 friends 中
  // dude 不是一个 friend(Person 的实现) 并且还运行到这里的概率为 1%
  // 我们可以在这对 dude 进行精确检查的同时进行一些异步加载。
}
```

## Hashing

`Hashing` 提供了很多的哈希函数，和对 `HashCode` 对象进行操作运算的工具方法。

### Provided Hash Functions(提供的Hash函数)

*   [`md5()`]
*   [`murmur3_128()`] 使用零值种子返回一个 murmur3 算法实现的 128 位的哈希值
*   [`murmur3_32()`] 使用零值种子返回一个 murmur3 算法实现的 32 位的哈希值
*   [`sha1()`] 被 Google 破解的安全哈希算法
*   [`sha256()`]
*   [`sha512()`]
*   [`goodFastHash(int bits)`] 返回一个多用途的，临时使用的，非加密的 Hash Function.

### HashCode Operations 算法

方法名                                            | 描述
:------------------------------------------------ | :----------
[`HashCode combineOrdered(Iterable<HashCode>)`]   | 以有序的方式使 HashCode 结合起来，如果两个 HashCode 集合采用此种方法结合后的 HashCode 相同，那么这两个 HashCode 集合中的元素可能是顺序相等的。
[`HashCode combineUnordered(Iterable<HashCode>)`] | 以无序的方式使 HashCode 结合起来，如果两个 HashCode 集合采用此方法结合后的 HashCode 相同，那么这两个 HashCode 集合中的元素可能在某种排序方式下是顺序相等的。
[`int consistentHash(HashCode, int buckets)`]     | 以给定 buckets 的大小分配一致性哈希，最大限度的减少随 buckets 增长而进行重新映射的需要.了解详情 [一致性哈希 Wikipedia](http://en.wikipedia.org/wiki/Consistent_hashing)。

[`com.google.common.hash`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/package-summary.html
[`HashFunction`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashFunction.html
[`Hasher`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hasher.html
[`Funnel`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Funnel.html
[`hash()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hasher.html#hash--
[`HashCode`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html
[`asInt()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asInt--
[`asLong()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asLong--
[`asBytes()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#asBytes--
[`writeBytesTo(array, offset, maxLength)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/HashCode.html#writeBytesTo-byte[]-int-int-
[`BloomFilter<T>`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html
[`create(Funnel funnel, int expectedInsertions, double falsePositiveProbability)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#create-com.google.common.hash.Funnel-int-double-
[`boolean mightContain(T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#mightContain-T-
[`void put(T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/BloomFilter.html#put-T-
[`md5()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#md5--
[`murmur3_128()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#murmur3_128--
[`murmur3_32()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#murmur3_32--
[`sha1()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha1--
[`sha256()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha256--
[`sha512()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#sha512--
[`goodFastHash(int bits)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#goodFastHash-int-
[`HashCode combineOrdered(Iterable<HashCode>)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#combineOrdered-java.lang.Iterable-
[`HashCode combineUnordered(Iterable<HashCode>)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#combineUnordered-java.lang.Iterable-
[`int consistentHash(HashCode, int buckets)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/hash/Hashing.html#consistentHash-com.google.common.hash.HashCode-int-
