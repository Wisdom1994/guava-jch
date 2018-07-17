# Using and avoiding null 使用和避免 ‘NULL’

> *"Null sucks.(Null 就是狗屎)"* -[Doug Lea(译注:JCP成员,纽约州立大学计算机系教授,JCP成员,java.concurrent包作者)]
>
> *"I call it my billion-dollar mistake(Null引用：代价十亿美元的错误)."* - [Sir C. A. R. Hoare(译注:托尼.霍尔, 图灵奖得主)]

十分随意的使用 `null` 会导致很多惊人的 bug 产生. 我们在研究 Google 代码库之后发现:
几乎 95% 的 集合类不接受一个 `null` 作为其中的值, 所以我们认为, 相比与默默的使用 `null` 
并且容忍它对我们的伤害, 不如使用快速失败操作拒绝 `null` 对开发者们来的更有帮助.

此外, `null` 具有使人十分难受的模糊语义, 如果 `null` 作为一个返回值, 它很少具有十分清晰的含义.
举个栗子: `Map.get(key)` 返回一个 `null` 值, 它可以表示这个 map 中是空的, 或者是 map 
中没有 key 对应的值. `null` 可以表示失败, 可以表示成功, 甚至是表示任何情况.
可是如果我们使用除了 `null` 之外的其他值, 将会使语义更加的清晰明确.

同时还有着另一种情况, 在一些情况下: `null` 有着合适和正确的使用场景.
在内存使用以及运行速度方面, `null` 是非常廉价、低消耗的, 同时, 它也是数组中不可避免的存在.
但是在基于底层库的应用级别的代码中, `null` 通常是导致含糊语义以及疑难bug的罪魁祸首,
就像我们之前讨论过的 `Map.get` 返回 null 值的含义问题. 最关键的是, `null` 并没有定义 null 值是什么含义.

For these reasons, many of Guava's utilities are designed to fail fast in the
presence of null rather than allow nulls to be used, so long as there is a
null-friendly workaround available. Additionally, Guava provides a number of
facilities both to make using `null` easier, when you must, and to help you
avoid using `null`.

## Specific Cases 特殊情况

If you're trying to use `null` values in a `Set` or as a key in a `Map` --
don't; it's clearer (less surprising) if you explicitly special-case `null`
during lookup operations.

If you want to use `null` as a value in a Map -- leave out that entry; keep a
separate `Set` of non-null keys (or null keys). It's very easy to mix up the
cases where a `Map` contains an entry for a key, with value `null`, and the case
where the `Map` has no entry for a key. It's much better just to keep such keys
separate, and to think about what it _means_ to your application when the value
associated with a key is `null`.

If you're using nulls in a `List` -- if the list is sparse, might you rather use
a `Map<Integer, E>`? This might actually be more efficient, and could
potentially actually match your application's needs more accurately.

Consider if there is a natural "null object" that can be used. There isn't
always. But sometimes. For example, if it's an enum, add a constant to mean
whatever you're expecting null to mean here. For example,
`java.math.RoundingMode` has an `UNNECESSARY` value to indicate "do no rounding,
and throw an exception if rounding would be necessary."

If you really need null values, and you're having problems with a null-hostile
collection implementations, use a different implementation. For example, use
`Collections.unmodifiableList(Lists.newArrayList())` instead of `ImmutableList`.

## Optional

Many of the cases where programmers use `null` is to indicate some sort of
absence: perhaps where there might have been a value, there is none, or one
could not be found. For example, `Map.get` returns `null` when no value is found
for a key.

`Optional<T>` is a way of replacing a nullable `T` reference with a non-null
value. An `Optional` may either contain a non-null `T` reference (in which case
we say the reference is "present"), or it may contain nothing (in which case we
say the reference is "absent"). It is never said to "contain null."

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // returns true
possible.get(); // returns 5
```

`Optional` is **not** intended as a direct analogue of any existing "option" or
"maybe" construct from other programming environments, though it may bear some
similarities.

We list some of the most common `Optional` operations here.

### Making an Optional

Each of these are static methods on `Optional`.

Method                       | Description
:--------------------------- | :----------
[`Optional.of(T)`]           | Make an Optional containing the given non-null value, or fail fast on null.
[`Optional.absent()`]        | Return an absent Optional of some type.
[`Optional.fromNullable(T)`] | Turn the given possibly-null reference into an Optional, treating non-null as present and null as absent.

### Query methods

Each of these are non-static methods on a particular `Optional<T>` value.

Method                  | Description
:---------------------- | :----------
[`boolean isPresent()`] | Returns `true` if this `Optional` contains a non-null instance.
[`T get()`]             | Returns the contained `T` instance, which must be present; otherwise, throws an `IllegalStateException`.
[`T or(T)`]             | Returns the present value in this `Optional`, or if there is none, returns the specified default.
[`T orNull()`]          | Returns the present value in this `Optional`, or if there is none, returns `null`. The inverse operation of `fromNullable`.
[`Set<T> asSet()`]      | Returns an immutable singleton `Set` containing the instance in this `Optional`, if there is one, or otherwise an empty immutable set.

`Optional` provides several more handy utility methods besides these; consult
the Javadoc for details.

### What's the point?

Besides the increase in readability that comes from giving `null` a _name_, the
biggest advantage of Optional is its idiot-proof-ness. It forces you to actively
think about the absent case if you want your program to compile at all, since
you have to actively unwrap the Optional and address that case. Null makes it
disturbingly easy to simply forget things, and though FindBugs helps, we don't
think it addresses the issue nearly as well.

This is especially relevant when you're **returning** values that may or may not
be "present." You (and others) are far more likely to forget that
`other.method(a, b)` could return a null value than you're likely to forget that
`a` could be null when you're implementing other.method. Returning `Optional`
makes it impossible for callers to forget that case, since they have to unwrap
the object themselves for their code to compile.

## Convenience methods

Whenever you want a `null` value to be replaced with some default value instead,
use [`MoreObjects.firstNonNull(T, T)`]. As the method name suggests, if both of
the inputs are null, it fails fast with a `NullPointerException`. If you are
using an `Optional`, there are better alternatives -- e.g. `first.or(second)`.

A couple of methods dealing with possibly-null `String` values are provided in
`Strings`. Specifically, we provide the aptly named:

*   [`emptyToNull(String)`]
*   [`isNullOrEmpty(String)`]
*   [`nullToEmpty(String)`]

We would like to emphasize that these methods are primarily for interfacing with
unpleasant APIs that equate null strings and empty strings. Every time _you_
write code that conflates null strings and empty strings, the Guava team weeps.
(If null strings and empty strings mean actively different things, that's
better, but treating them as the same thing is a disturbingly common code
smell.)

[Doug Lea]: http://en.wikipedia.org/wiki/Doug_Lea
[Sir C. A. R. Hoare]: http://en.wikipedia.org/wiki/C._A._R._Hoare
[`Optional.of(T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#of-T-
[`Optional.absent()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#absent--
[`Optional.fromNullable(T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#fromNullable-T-
[`boolean isPresent()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#isPresent--
[`T get()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#get--
[`T or(T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#or-T-
[`T orNull()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#orNull--
[`Set<T> asSet()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Optional.html#asSet--
[`MoreObjects.firstNonNull(T, T)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/MoreObjects.html#firstNonNull-T-T-
[`emptyToNull(String)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#emptyToNull-java.lang.String-
[`isNullOrEmpty(String)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#isNullOrEmpty-java.lang.String-
[`nullToEmpty(String)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Strings.html#nullToEmpty-java.lang.String-
