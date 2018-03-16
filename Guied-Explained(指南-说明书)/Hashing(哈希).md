# Hashing

## Overview

Java 的 HashCode 的长度被限制在 32 位，并且哈希算法与他们作用的数据(类或方法)之间没有分离，
因此很难使用另外的哈希算法进行替换。同时，Java 内建的哈希算法生成的 Hash 值是十分劣质的，有一部分原因是因为他们依赖于劣质的 Hashcode 实现，其中包括很多 JDK 中的实现类。

Java 中 `Object.hashCode` 运行十分快速，但是缺乏对 **哈希碰撞** 问题的预防，同时也缺乏对散列值的期望。这种特性却让他十分适合与应用在哈希表中，因为额外的哈希碰撞只会
带来很小的性能损失，同时十分差劲的分散性可以采用二次哈希的方法解决(Java 中几乎所有
合理的哈希算法(哈希函数)都采用这种方法进行实现)。然而，在简单哈希表之外的很多哈希应用中，
`Object.hashCode`却是不足的，因此`com.google.common.hash`包被设计出来。

## Organization

Looking at the package Javadoc, we see a lot of different types, but it's not
obvious how they fit together.

Let's look at a sample piece of code using this library.

``` java
HashFunction hf = Hashing.md5();
HashCode hc = hf.newHasher()
       .putLong(id)
       .putString(name, Charsets.UTF_8)
       .putObject(person, personFunnel)
       .hash();
```

#### HashFunction

[`HashFunction`] is a pure, stateless function that maps an arbitrary block of
data to a fixed number of bits, with the property that equal inputs always yield
equal outputs, and unequal inputs yield unequal outputs as often as possible.

#### Hasher

A `HashFunction` can be asked for a stateful [`Hasher`], which provides fluent
syntax to add data to the hash and then retrieve the hash value. A `Hasher` can
accept any primitive input, byte arrays, slices of byte arrays, character
sequences, character sequences in some charset, and so on, or any other
`Object`, provided with an appropriate `Funnel`.

`Hasher` implements the `PrimitiveSink` interface, which specifies a fluent API
for an object that accepts a stream of primitive values.

#### Funnel

A [`Funnel`] describes how to decompose a particular object type into primitive
field values. For example, if we had

``` java
class Person {
  final int id;
  final String firstName;
  final String lastName;
  final int birthYear;
}
```

our `Funnel` might look like

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

_Note:_ `putString("abc", Charsets.UTF_8).putString("def", Charsets.UTF_8)` is
fully equivalent to `putString("ab", Charsets.UTF_8).putString("cdef",
Charsets.UTF_8)`, because they produce the same byte sequence. This can cause
unintended hash collisions. Adding separators of some kind can help eliminate
unintended hash collisions.

#### HashCode

Once a `Hasher` has been given all its input, its [`hash()`] method can be used
to retrieve a [`HashCode`]. `HashCode` supports equality testing and such, as
well as [`asInt()`], [`asLong()`], [`asBytes()`] methods, and additionally,
[`writeBytesTo(array, offset, maxLength)`], which writes the first `maxLength`
bytes of the hash into the array.

### BloomFilter

Bloom filters are a lovely application of hashing that cannot be done simply
using `Object.hashCode()`. Briefly, Bloom filters are a probabilistic data
structure, allowing you to test if an object is _definitely_ not in the filter,
or was _probably_ added to the Bloom filter. The [Wikipedia
page](http://en.wikipedia.org/wiki/Bloom_filter) is fairly comprehensive, and we
recommend [this tutorial](http://llimllib.github.com/bloomfilter-tutorial/).

Our hashing library has a built-in Bloom filter implementation, which requires
only that you implement a `Funnel` to decompose your type into primitive types.
You can obtain a fresh [`BloomFilter<T>`] with [`create(Funnel funnel, int
expectedInsertions, double falsePositiveProbability)`], or just accept the
default false probability of 3%. `BloomFilter<T>` offers the methods [`boolean
mightContain(T)`] and [`void put(T)`], which are self-explanatory enough.

``` java
BloomFilter<Person> friends = BloomFilter.create(personFunnel, 500, 0.01);
for (Person friend : friendsList) {
  friends.put(friend);
}
// much later
if (friends.mightContain(dude)) {
  // the probability that dude reached this place if he isn't a friend is 1%
  // we might, for example, start asynchronously loading things for dude while we do a more expensive exact check
}
```

## Hashing

The `Hashing` utility class provides a number of stock hash functions and
utilities to operate on `HashCode` objects.

### Provided Hash Functions

*   [`md5()`]
*   [`murmur3_128()`]
*   [`murmur3_32()`]
*   [`sha1()`]
*   [`sha256()`]
*   [`sha512()`]
*   [`goodFastHash(int bits)`]

### HashCode Operations

Method                                            | Description
:------------------------------------------------ | :----------
[`HashCode combineOrdered(Iterable<HashCode>)`]   | Combines hash codes in an ordered fashion, so that if two hashes obtained from this method are the same, then it is likely that each was computed from the same hashes in the same order.
[`HashCode combineUnordered(Iterable<HashCode>)`] | Combines hash codes in an unordered fashion, so that if two hashes obtained from this method are the same, then it is likely that each was computed from the same hashes in some order.
[`int consistentHash(HashCode, int buckets)`]     | Assigns the hash code a consistent "bucket" which minimizes the need for remapping as the number of buckets grows. See [Wikipedia](http://en.wikipedia.org/wiki/Consistent_hashing) for details.

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
