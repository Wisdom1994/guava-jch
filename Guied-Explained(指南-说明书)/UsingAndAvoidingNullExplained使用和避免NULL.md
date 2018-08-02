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

基于以上原因, Guava 的很多工具类都定义了对 `null` 值的快速失败, 同时, 
除非工具类本身对 NULL 定义了确定的含义(值), 否则不允许 null 值被使用.
并且, Guava 也提供了大量的方法, 在你有需要的时候, 能帮助你使用特定的值来代替结果集中的 NULL.

## Specific Cases 特殊情况

不要尝试着将 `null` 作为 `Set` 中的一个值或者 `Map` 中的一个键！ 在查找操作中, 
使用一个特殊的确定的值来代替 `null` 会使语义毫无惊喜 (less surprising) 的清晰起来.

如果你想使用 `null` 作为一个 Map 中的值, 不如把它们整理出来单独作为一个集合来进行维护,
比如将你的 `Set` 分为两个, 不含 `null` 和只含 `null` 的. 在一个 Map 中, **‘一个确定的键对应了的值是NULL’** 和
 **‘Map中确定的键没有与之对应的值’** 是十分容易混淆的. 所以, 最好是将值为 null 的键从这个 map 中分离出来,
 并且需要你仔细想想：`null` 在你的应用程序中到底代表着什么含义.

如果你想在一个十分"稀疏"的列表中使用"NULL", 还不如用一个 `Map<Integer, E>` 来的更好一点,
`Map` 可能会有更高效的性能, 并能满足您应用程序中潜在的需求.

我们还需要思考一个特殊情况, 我们可能需要一个特殊的的 "null值对象", 并不总是能用到,但是有时候我们真的需要有。
举个栗子, 比如有个枚举类需要添加一个特殊的枚举值为 null, 就像 `java.math.RoundingMode` 中提供了一个 `UNNECESSARY` 来表示 "不做任何取整的操作,
如果有任何取整的行为( _译注:除非计算结果为整数,不然都抛出异常_ ), 就快速抛出一个异常".

如果你真的需要一个空值, 可是 Guava 中的"非空"集合类却不能够满足你的要求,
那么你应该使用集合类另外的实现, 比如使用 JDK 中的 `Collections.unmodifiableList(Lists.newArrayList())`
来代替 Guava 的 `ImmutableList`.

## Optional 对象

在大多数的情况中, 程序开发者们使用 `null` 来表示一种 "**缺失**" 的情境 ———— 
在某个地方可能表示默认的值， 也可能是表示那个地方没有值， 或者是那个值找不到了。
举个简单的例子：就像 `Map.get` 方法，当它找不到 key 所对应的 value 的时候, 就会返回一个 `null`.

基于这个情况, Guava 设计了 `Optional<T>` 类，用以接收 `null` 或者 `no-null` 类型的对象 **T**.
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
