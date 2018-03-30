# String utilities 字符串工具类

## Joiner

Joining together a sequence of strings with a separator can be unnecessarily
tricky -- but it shouldn't be. If your sequence contains nulls, it can be even
harder. The fluent style of [`Joiner`] makes it simple.

``` java
Joiner joiner = Joiner.on("; ").skipNulls();
return joiner.join("Harry", null, "Ron", "Hermione");
```

returns the string "Harry; Ron; Hermione". Alternately, instead of using
`skipNulls`, you may specify a string to use instead of null with
`useForNull(String)`.

You may also use `Joiner` on objects, which will be converted using their
`toString()` and then joined.

``` java
Joiner.on(",").join(Arrays.asList(1, 5, 7)); // returns "1,5,7"
```

**Warning:** joiner instances are always immutable. The joiner configuration
methods will always return a new `Joiner`, which you must use to get the desired
semantics. This makes any `Joiner` thread safe, and usable as a `static final`
constant.

## Splitter

The built in Java utilities for splitting strings can have some quirky
behaviors. For example, `String.split` silently discards trailing separators,
and `StringTokenizer` respects exactly five whitespace characters and nothing
else.

Quiz: What does `",a,,b,".split(",")` return?

1.  `"", "a", "", "b", ""`
1.  `null, "a", null, "b", null`
1.  `"a", null, "b"`
1.  `"a", "b"`
1.  None of the above

The correct answer is none of the above: `"", "a", "", "b"`. Only trailing empty
strings are skipped. What is this I don't even.

[`Splitter`] allows complete control over all this confusing behavior using a
reassuringly straightforward fluent pattern.

``` java
Splitter.on(',')
    .trimResults()
    .omitEmptyStrings()
    .split("foo,bar,,   qux");
```

returns an `Iterable<String>` containing "foo", "bar", "qux". A `Splitter` may
be set to split on any `Pattern`, `char`, `String`, or `CharMatcher`.

#### Base Factories

Method                                                     | Description                                                                                                                         | Example
:--------------------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- | :------
[`Splitter.on(char)`]                                      | Split on occurrences of a specific, individual character.                                                                           | `Splitter.on(';')`
[`Splitter.on(CharMatcher)`]                               | Split on occurrences of any character in some category.                                                                             | `Splitter.on(CharMatcher.BREAKING_WHITESPACE)`<br>`Splitter.on(CharMatcher.anyOf(";,."))`
[`Splitter.on(String)`]                                    | Split on a literal `String`.                                                                                                        | `Splitter.on(", ")`
[`Splitter.on(Pattern)`]<br>[`Splitter.onPattern(String)`] | Split on a regular expression.                                                                                                      | `Splitter.onPattern("\r?\n")`
[`Splitter.fixedLength(int)`]                              | Splits strings into substrings of the specified fixed length. The last piece can be smaller than `length`, but will never be empty. | `Splitter.fixedLength(3)`

#### Modifiers

Method                       | Description                                                                             | Example
:--------------------------- | :-------------------------------------------------------------------------------------- | :------
[`omitEmptyStrings()`]       | Automatically omits empty strings from the result.                                      | `Splitter.on(',').omitEmptyStrings().split("a,,c,d")` returns `"a", "c", "d"`
[`trimResults()`]            | Trims whitespace from the results; equivalent to `trimResults(CharMatcher.WHITESPACE)`. | `Splitter.on(',').trimResults().split("a, b, c, d")` returns `"a", "b", "c", "d"`
[`trimResults(CharMatcher)`] | Trims characters matching the specified `CharMatcher` from results.                     | `Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")` returns `"a ", "b_ ", "c"`.
[`limit(int)`]               | Stops splitting after the specified number of strings have been returned.               | `Splitter.on(',').limit(3).split("a,b,c,d")` returns `"a", "b", "c,d"`

TODO: Map splitters

If you wish to get a `List`, just use
`Lists.newArrayList(splitter.split(string))` or the like.

**Warning:** splitter instances are always immutable. The splitter configuration
methods will always return a new `Splitter`, which you must use to get the
desired semantics. This makes any `Splitter` thread safe, and usable as a
`static final` constant.

<!--
<a href='Hidden comment:
= Escaper =
Escaping strings correctly -- converting them into a format safe for inclusion in e.g. an XML document or a Java source file -- can be a tricky business, and critical for security reasons.  Guava provides a flexible API for escaping text, and a number of built-in escapers, in the com.google.common.escape package.

All escapers in Guava extend the [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/Escaper.html Escaper] abstract class, and support the method String escape(String).  Built-in Escaper instances can be found in several classes, depending on your needs: [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/html/HtmlEscapers.html HtmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/xml/XmlEscapers.html XmlEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/escape/SourceCodeEscapers.html SourceCodeEscapers], [http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/net/UriEscapers.html UriEscapers], or you can build your own with [http://google.github.io/guava/releases/snapshot/api/docs/ an Escapers.Builder].  To inspect an Escaper, you can use Escapers.computeReplacement to find the replacement string for a given character.
'></a>
-->

## CharMatcher

In olden times, our `StringUtil` class grew unchecked, and had many methods like
these:

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

They represent a partial cross product of two notions:

1.  what constitutes a "matching" character?
1.  what to do with those "matching" characters?

To simplify this morass, we developed `CharMatcher`.

Intuitively, you can think of a `CharMatcher` as representing a particular class
of characters, like digits or whitespace. Practically speaking, a `CharMatcher`
is just a boolean predicate on characters -- indeed, `CharMatcher` implements
[`Predicate<Character>`] -- but because it
is so common to refer to "all whitespace characters" or "all lowercase letters,"
Guava provides this specialized syntax and API for characters.

But the utility of a `CharMatcher` is in the _operations_ it lets you perform on
occurrences of the specified class of characters: trimming, collapsing,
removing, retaining, and much more. An object of type `CharMatcher` represents
notion 1: what constitutes a matching character? It then provides many
operations answering notion 2: what to do with those matching characters? The
result is that API complexity increases linearly for quadratically increasing
flexibility and power. Yay!

``` java
String noControl = CharMatcher.javaIsoControl().removeFrom(string); // remove control characters
String theDigits = CharMatcher.digit().retainFrom(string); // only the digits
String spaced = CharMatcher.whitespace().trimAndCollapseFrom(string, ' ');
  // trim whitespace at ends, and replace/collapse whitespace into single spaces
String noDigits = CharMatcher.javaDigit().replaceFrom(string, "*"); // star out all digits
String lowerAndDigit = CharMatcher.javaDigit().or(CharMatcher.javaLowerCase()).retainFrom(string);
  // eliminate all characters that aren't digits or lowercase
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
