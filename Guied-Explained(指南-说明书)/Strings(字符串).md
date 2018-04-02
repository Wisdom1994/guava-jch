# String utilities 字符串工具类

## Joiner

将被分隔符分割的字符串序列串联起来，可能会产生些不必要的麻烦, 如果你的字符串序列中包含几个 **null**,
那连接操作将会更为困难, 那么 Fluent 风格的 [`Joiner`] 类就可以将连接操作变得非常容易.

``` java
// skipNulls() 跳过 null , 忽略 null 在字符串中发挥的作用
Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```

代码返回了 "Harry; Ron; Hermione" 字符串, 另外, 使用 `useForNull(String)` 方法,
你可以使用任意字符串来代替 null.

``` java
// 使用 useForNull(String) 方法，使用自定义字符串代替 null
Joiner joiner = Joiner.on(";").useForNull("Hi");
return joiner.join("Harry", null, "Ron", "Hermione");
// 代码将会返回 Harry", "Hi", "Ron", "Hermione"
```

你也可以在对象中使用 `Joiner`, 这时, Joiner 会将对象的 `toString()` 值连接起来.

``` java
Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

**警告:** joiner 的实例总是不可变的(具体请参考joiner的构造以及具体方法),
也就是使用 joiner 的构造方法总会返回一个新的 `Joiner` 对象, 这使 `Joiner` 是线程安全的,
所以你可以将其定义为一个 `static final` 常量.

## Splitter 分割(隔)

在 Java 内建的工具类中，字符串拆分工具总有着一些莫名奇妙的特性. 举个栗子,
`String.split` 偷偷抛弃了结尾的分隔符, `StringTokenizer` 则只关心五种空白字符(" ","\t","\n","\r","\f") 不管其他.

*问: 执行`",a,,b,".split(",")` 之后, 将会返回什么?*

1.  `"", "a", "", "b", ""`
1.  `null, "a", null, "b", null`
1.  `"a", null, "b"`
1.  `"a", "b"`
1.  以上都不对

正确答案是第五项: 以上都不对. `"", "a", "", "b"` 才是最终的返回结果,只有结尾的空字符串被忽略了.
[`Splitter`] 通过流畅、直白的代码风格对以上混乱的特性达到了完全的掌控。

``` java
Splitter.on(',')
    .trimResults()
    .omitEmptyStrings()
    .split("foo,bar,,   qux");
```
返回一个迭代器 `Iterable<String>` 包含 "foo", "bar", "qux".
`Splitter` 被设置成可以按照任何的 `Pattern(模式)`, `char(字符)`,
`String(字符串)` 或者 `CharMatcher(字符匹配器)` 来对字符串进行拆分

#### Base Factories 拆分器工厂

Method 方法                                                    | Description 描述                                         | Example 举例                                                                           | Example 举例
:--------------------------------------------------------- | :----------------------------------------------------------| :------
[`Splitter.on(char)`]                                      | 按单个字符拆分                                           | `Splitter.on(';')`
[`Splitter.on(CharMatcher)`]                               | 按字符匹配器拆分                                        | `Splitter.on(CharMatcher.BREAKING_WHITESPACE)`<br>`Splitter.on(CharMatcher.anyOf(";,."))`
[`Splitter.on(String)`]                                    | 按照字符串拆分                                          | `Splitter.on(", ")`
[`Splitter.on(Pattern)`]<br>[`Splitter.onPattern(String)`] | 按照正则表达式拆分                                   | `Splitter.onPattern("\r?\n")`
[`Splitter.fixedLength(int)`]                              | 按照固定长度拆分,最后一段可能比指定长度要短, 但是不为空 | `Splitter.fixedLength(3)`

#### Modifiers 调节器/修饰器

Method 方法                  | Description   描述                                                                      | Example 举个栗子
:--------------------------- | :-------------------------------------------------------------------------------------- | :------
[`omitEmptyStrings()`]       | 从结果中自动忽略空字符串                                                              | `Splitter.on(',').omitEmptyStrings().split("a,,c,d")` 返回 `"a", "c", "d"`
[`trimResults()`]            | 移除掉结果字符串中的前/后空格; 等效于 `trimResults(CharMatcher.WHITESPACE)`方法       | `Splitter.on(',').trimResults().split("a, b, c, d")` 返回 `"a", "b", "c", "d"`
[`trimResults(CharMatcher)`] | 移除掉结果字符串中的匹配“字符串匹配器”的字符                                       | `Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")` 返回 `"a ", "b_ ", "c"`.
[`limit(int)`]               | 限制拆分出的字符串数量                                                                 | `Splitter.on(',').limit(3).split("a,b,c,d")` 返回 `"a", "b", "c,d"`

备忘录: Map splitters Map 拆分器

如果你希望返回结果是一个 `List`, 只需要用类似 `Lists.newArrayList(splitter.split(string))` 的方法就好

**警告:** splitter 的实例也是不可变的(参考 joiner). 调用确定语义的构造方法生成一个 splitter 总是返回一个新的 `Splitter` 对象,
这使得 `Splitter` 是线程安全的, 可以看作为 `static final` 常量.

<!--
<a href='Hidden comment:
= Escaper =
Escaping strings correctly -- converting them into a format safe for inclusion in e.g. an XML document or a Java source file -- can be a tricky business, and critical for security reasons.  Guava provides a flexible API for escaping text, and a number of built-in escapers, in the com.google.common.escape package.

All escapers in Guava extend the [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/Escaper.html Escaper] abstract class, and support the method String escape(String).  Built-in Escaper instances can be found in several classes, depending on your needs: [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/html/HtmlEscapers.html HtmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/xml/XmlEscapers.html XmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/SourceCodeEscapers.html SourceCodeEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/net/UriEscapers.html UriEscapers], or you can build your own with [http://google.github.io/guava/releases/snapshot/api/docs/ an Escapers.Builder].  To inspect an Escaper, you can use Escapers.computeReplacement to find the replacement string for a given character.
'></a>
-->

## CharMatcher 字符匹配器

在以前的版本中, Guava 的 `StringUtil` 类疯狂的扩大, 包含了很多类似下面的方法...

*   `allAscii`
*   `collapse`
*   `collapseControlChars`
*   `collapseWhitespace`
*   `lastIndexNotOf`
*   `numSharedChars`
*   `removeChars`
*   `removeCrLf`
*   `retainAllChars`
*   `strip`
*   `stripAndCollapse`
*   `stripNonDigits`

他们代表着两种设计概念：

1.  什么才是匹配字符?
1.  我们应该如何处理匹配字符?

为了走出如上困境, 我们设计了 `CharMatcher`

直观的来看, 你可以将 `CharMatcher` 看作某种字符的具体描述, 比如 *数字* 或者 *空格字符*.
实际来说, `CharMatcher` 就是对一个字符的 boolean 类型的判断 —— `CharMatcher` 的确实现了[`Predicate<Character>`],
但是“所有的空白字符”或者“所有的小写字符”这样的工作实在是太过普遍, 所以我们为此提供了专门的语法和API.

但是在具体工作中, `CharMatcher` 中最实用的是对字符进行特定操作的方法：trimming(修剪:一般是前后空格),
collapsing(折叠), removing(移除), retaining(保留) 等等更多... `CharMatcher` 的实例正是代表了以上设计概念: 
1: 什么是匹配字符? 2: 如何处理匹配字符?  这使得难复杂度线性增长的 API 具有了越来越优秀的灵活性与功能性.
耶O(∩_∩)O...

``` java
String noControl = CharMatcher.javaIsoControl().removeFrom(string); // 移除 control 字符.
String theDigits = CharMatcher.digit().retainFrom(string); // 仅保留数字
String spaced = CharMatcher.whitespace().trimAndCollapseFrom(string, ' '); // 去掉两端空格, 中间多个空格变为一个
String noDigits = CharMatcher.javaDigit().replaceFrom(string, "*"); // 用 * 替换所有的数字
String lowerAndDigit = CharMatcher.javaDigit().or(CharMatcher.javaLowerCase()).retainFrom(string);
  // 只保留小写字母和数字
```

**Note:** `CharMatcher` deals only with `char` values; it does not understand
supplementary Unicode code points in the range 0x10000 to 0x10FFFF. Such logical
characters are encoded into a `String` using surrogate pairs, and a
`CharMatcher` treats these just as two separate characters.

### Obtaining CharMatchers

Many needs can be satisfied by the provided `CharMatcher` factory methods:

*   [`any()`]
*   [`none()`]
*   [`whitespace()`]
*   [`breakingWhitespace()`]
*   [`invisible()`]
*   [`digit()`]
*   [`javaLetter()`]
*   [`javaDigit()`]
*   [`javaLetterOrDigit()`]
*   [`javaIsoControl()`]
*   [`javaLowerCase()`]
*   [`javaUpperCase()`]
*   [`ascii()`]
*   [`singleWidth()`]

Other common ways to obtain a `CharMatcher` include:

Method                  | Description
:---------------------- | :----------
[`anyOf(CharSequence)`] | Specify all the characters you wish matched. For example, `CharMatcher.anyOf("aeiou")` matches lowercase English vowels.
[`is(char)`]            | Specify exactly one character to match.
[`inRange(char, char)`] | Specify a range of characters to match, e.g. `CharMatcher.inRange('a', 'z')`.

Additionally, `CharMatcher` has [`negate()`], [`and(CharMatcher)`], and
[`or(CharMatcher)`]. These provide simple boolean operations on `CharMatcher`.

### Using CharMatchers

`CharMatcher` provides a [wide variety] of methods to operate on occurrences of
the specified characters in any `CharSequence`. There are more methods provided
than we can list here, but some of the most commonly used are:

Method                                      | Description
:------------------------------------------ | :----------
[`collapseFrom(CharSequence, char)`]        | Replace each group of consecutive matched characters with the specified character. For example, `WHITESPACE.collapseFrom(string, ' ')` collapses whitespaces down to a single space.
[`matchesAllOf(CharSequence)`]              | Test if this matcher matches all characters in the sequence. For example, `ASCII.matchesAllOf(string)` tests if all characters in the string are ASCII.
[`removeFrom(CharSequence)`]                | Removes matching characters from the sequence.
[`retainFrom(CharSequence)`]                | Removes all non-matching characters from the sequence.
[`trimFrom(CharSequence)`]                  | Removes leading and trailing matching characters.
[`replaceFrom(CharSequence, CharSequence)`] | Replace matching characters with a given sequence.

(Note: all of these methods return a `String`, except for `matchesAllOf`, which
returns a `boolean`.)

## Charsets

Don't do this:

``` java
try {
  bytes = string.getBytes("UTF-8");
} catch (UnsupportedEncodingException e) {
  // how can this possibly happen?
  throw new AssertionError(e);
}
```

Do this instead:

``` java
bytes = string.getBytes(Charsets.UTF_8);
```

[`Charsets`] provides constant references to the six standard `Charset`
implementations guaranteed to be supported by all Java platform implementations.
Use them instead of referring to charsets by their names.

TODO: an explanation of charsets and when to use them

(Note: If you're using JDK7, you should use the constants in
[`StandardCharsets`]

## CaseFormat

`CaseFormat` is a handy little class for converting between ASCII case
conventions &mdash; like, for example, naming conventions for programming
languages. Supported formats include:

Format               | Example
:------------------- | :-----------------
[`LOWER_CAMEL`]      | `lowerCamel`
[`LOWER_HYPHEN`]     | `lower-hyphen`
[`LOWER_UNDERSCORE`] | `lower_underscore`
[`UPPER_CAMEL`]      | `UpperCamel`
[`UPPER_UNDERSCORE`] | `UPPER_UNDERSCORE`

Using it is relatively straightforward:

``` java
CaseFormat.UPPER_UNDERSCORE.to(CaseFormat.LOWER_CAMEL, "CONSTANT_NAME")); // returns "constantName"
```

We find this especially useful, for example, when writing programs that generate
other programs.

[`Joiner`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Joiner.html
[`Splitter`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html
[`Splitter.on(char)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-char-
[`Splitter.on(CharMatcher)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-com.google.common.base.CharMatcher-
[`Splitter.on(String)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-java.lang.String-
[`Splitter.on(Pattern)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#on-java.util.regex.Pattern-
[`Splitter.onPattern(String)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#onPattern-java.lang.String-
[`Splitter.fixedLength(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#fixedLength-int-
[`omitEmptyStrings()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#omitEmptyStrings--
[`trimResults()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults--
[`trimResults(CharMatcher)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#trimResults-com.google.common.base.CharMatcher-
[`limit(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Splitter.html#limit-int-
[`any()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#any--
[`none()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#none--
[`whitespace()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#whitespace--
[`breakingWhitespace()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#breakingWhitespace--
[`invisible()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#invisible--
[`digit()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#digit--
[`javaLetter()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLetter--
[`javaDigit()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaDigit--
[`javaLetterOrDigit()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLetterOrDigit--
[`javaIsoControl()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaIsoControl--
[`javaLowerCase()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaLowerCase--
[`javaUpperCase()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#javaUpperCase--
[`ascii()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#ascii--
[`singleWidth()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#singleWidth--
[`anyOf(CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#anyOf-java.lang.CharSequence-
[`is(char)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#is-char-
[`inRange(char, char)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#inRange-char-char-
[`negate()`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#negate--
[`and(CharMatcher)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#and-com.google.common.base.CharMatcher-
[`or(CharMatcher)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#or-com.google.common.base.CharMatcher-
[wide variety]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#method_summary
[`collapseFrom(CharSequence, char)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#collapseFrom-java.lang.CharSequence-char-
[`matchesAllOf(CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#matchesAllOf-java.lang.CharSequence-
[`removeFrom(CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#removeFrom-java.lang.CharSequence-
[`retainFrom(CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#retainFrom-java.lang.CharSequence-
[`trimFrom(CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#trimFrom-java.lang.CharSequence-
[`replaceFrom(CharSequence, CharSequence)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CharMatcher.html#replaceFrom-java.lang.CharSequence-java.lang.CharSequence-
[`Charsets`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/Charsets.html
[`StandardCharsets`]: http://docs.oracle.com/javase/7/docs/api/java/nio/charset/StandardCharsets.html
[`LOWER_CAMEL`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_CAMEL
[`LOWER_HYPHEN`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_HYPHEN
[`LOWER_UNDERSCORE`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#LOWER_UNDERSCORE
[`UPPER_CAMEL`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_CAMEL
[`UPPER_UNDERSCORE`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/base/CaseFormat.html#UPPER_UNDERSCORE
[`Predicate\<Character>`]: FunctionalExplained#predicate
