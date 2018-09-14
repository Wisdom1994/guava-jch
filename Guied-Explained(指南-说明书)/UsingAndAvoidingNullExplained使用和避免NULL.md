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

`Optional<T>` 是使用一个 “非空” 的值来代替一个 “可为空” 的 `T` 类型引用的一种方法, 
一个 `Optional` 实例可能包含着一个非空的 `T` 类型引用(这种情况我们称之为 **引用存在** ),
也可能什么都不包含(我们称之为 **引用缺失**), 总之, 就是不会出现引用为 `null` 的情况.

```java
Optional<Integer> possible = Optional.of(5);
possible.isPresent(); // 对象是否存在？ 返回 true
possible.get(); // 返回 5
```

`Optional` 并非刻意的模拟其他编程语言下已经存在的 "option" 或者 "maybe" 的情况,
但是总会有那么一些相似.

我们这里给出了一些常用的 `Optional` 操作.

### Making an Optional 构建一个 Optional

这列出 `Optional` 的一些静态方法.

Method                       | Description 描述
:--------------------------- | :----------
[`Optional.of(T)`]           | 创建指定引用 `T` 的 Optional 对象, 如果该引用为 `null` 则快速失败.
[`Optional.absent()`]        | 创建一个 **引用缺失** 的 Optional 对象.
[`Optional.fromNullable(T)`] | 创建一个指定引用 `T` 的 Optional 对象, `T` 可能为 null, 如果为 null, 则生成一个 **引用缺失** 的 Optional 对象.

### Optional 的查询方法

这些方法都不是静态的.

Method                  | Description 描述
:---------------------- | :----------
[`boolean isPresent()`] | 如果这个 `Optional` 对象包含的是 **非空** 引用, 返回 true.
[`T get()`]             | 返回这个 `Optional` 所包含的 `T` 类型引用,  这个引用必须 **存在**, 否则抛出一个 `IllegalStateException` 异常.
[`T or(T)`]             | 返回这个 `Optional` 所包含的 `T` 类型引用, 如果引用缺失, 返回一个指定的默认值.
[`T orNull()`]          | 返回这个 `Optional` 所包含的 `T` 类型引用, 如果引用缺失, 返回一个 `null`. 这是`fromNullable(T)` 方法的逆操作.
[`Set<T> asSet()`]      | 返回包含于这个 `Optional` 的不可变单例集合, 如果引用存在, 返回只有单一元素的集合, 如果引用缺失, 返回一个空的不可变集合

`Optional` 提供了更多有用的工具方法, 浏览 Javadoc 以获得更多信息..

### What's the point? 意义是什么？

一方面, Optional 为 `null` 赋予确定的语义增强了可阅读性, 另一方面, Optional最大的优点可以看做是提供了 **傻瓜式** 的防护.
Optional 使你将注意力集中在思考可能导致 **引用缺失** 的情况, 因为你必须 **显式** 的从 Optional 中获取引用.
`null` 很容易使人忘记或者忽略掉一些细节, 尽管你可以通过 `FindBugs` 来找到那些相关的bug, 但是我们并不认为这能准确定位到问题的根源. 

与方法入参可能为 `null` 一样, 你的方法返回值也可能具有不那么 **现实** 含义(可能为 Null).
你(或者其他人)可能会忘记别人给你提供的方法 `method(a, b)` 方法可能返回一个 null, 同样的,
你也会在你的方法 `method(a, b)` 中忽略掉入参 `a` 也可能为 null 的问题. 使用 `Optional` 作为你方法的返回值,
迫使你必须考虑到各种可能引起 `引用缺失` 的情况, 直到考虑全面才能使你的代码通过编译.

## Convenience methods 一些有用的方法

当你想要使用 `null` 来代替一些默认值得话, 请使用 [`MoreObjects.firstNonNull(T, T)`],
就像方法名所表示的那样, 如果这两者值都是 `null`, 将会抛出一个 `NullPointerException`异常.
而如果使用了一个 `Opintion`, 那就不会有这样的问题出现, 他将非常完美的代替 `null`. 比如 Optional.fromNullable(first).or(second).

我们提供了一些方法去处理那些可能为 `null` 的字符串类型的数据. 通过方法名, 我们能清晰地知道这个方法的作用是什么:

*   [`emptyToNull(String)`]
*   [`isNullOrEmpty(String)`]
*   [`nullToEmpty(String)`]

需要特别强调的是, 我们提供的这些方法主要是用于改善那些将 `null` 和 `empty String (空字符串)` 混淆使用的API.
当你写出那些 `null 字符串` 和 `empty 字符串` 混淆的代码的时候, 就是我们 Guava 团队哭泣的时候.
(如果 null 字符串 和 empty 字符串 明显代表着不同含义, 我们建议您不如直接将他们分开来,
而如果您仍要坚持将他们混为一谈的话, 那这真是个悲伤的故事)

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
