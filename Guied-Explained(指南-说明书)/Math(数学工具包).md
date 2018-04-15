# Math -- Guava的数学工具包

这个包中含有各种各样的数学工具类

## Contents 综述

*   Guava Math 提供了为基本数据类型而设计的独立的类[`IntMath`],[`LongMath`], 
    [`DoubleMath`], 和 [`BigIntegerMath`], 这些类具有这相互平行的结构,
    他们的方法都是基于相应数据类型而进行实现.
    **请注意: 在`com.google.common.primitives`包中, 有一些函数或者类可能看起来不那么的 *数学*, 比如 [`Ints`].**
*   Guava 为单个或者成对的数据集提供了各种统计计算的方法(比如求平均值，中位数等等).
    如果想使用 Guava Math 包，请先阅读这个指南[Stats] 而不是去阅读 Java DOC.
*   [`LinearTransformation`] represents a linear conversion between `double`
    values of the form `y = mx + b`; for example, a conversion between feet and
    meters, or between Kelvins and degrees Fahrenheit.

## Examples 举个栗子

``` java
int logFloor = LongMath.log2(n, FLOOR);

int mustNotOverflow = IntMath.checkedMultiply(x, y);

long quotient = LongMath.divide(knownMultipleOfThree, 3, RoundingMode.UNNECESSARY); // fail fast on non-multiple of 3

BigInteger nearestInteger = DoubleMath.roundToBigInteger(d, RoundingMode.HALF_EVEN);

BigInteger sideLength = BigIntegerMath.sqrt(area, CEILING);
```

## Why use these? 为什么使用这些工具类?

*   These utilities are already exhaustively tested for unusual overflow
    conditions. Overflow semantics, if relevant, are clearly specified in the
    associated documentation. When a precondition fails, it fails fast.
*   They have been benchmarked and optimized.
    While performance inevitably varies depending on particular hardware
    details, their speed is competitive with -- and in some cases, significantly
    better than -- analogous functions in Apache Commons `MathUtils`.
*   They are designed to encourage readable, correct
    programming habits. The meaning of `IntMath.log2(x, CEILING)` is unambiguous
    and obvious even on a casual read-through. The meaning of `32 -
    Integer.numberOfLeadingZeros(x - 1)` is not.

_Note: these utilities are not especially compatible with GWT, nor are
they optimized for GWT, due to differing overflow logic._

## Math on Integral Types 整形计算

These utilities deal primarily with three integral types: `int`, `long`,
and `BigInteger`. The math utilities on these types are conveniently named
[`IntMath`], [`LongMath`], and [`BigIntegerMath`].

### Checked Arithmetic 检查方法

We provide arithmetic methods for `IntMath` and `LongMath` that fail fast on
overflow instead of silently ignoring it.

`IntMath`                   | `LongMath`
:-------------------------- | :---------------------------
[`IntMath.checkedAdd`]      | [`LongMath.checkedAdd`]
[`IntMath.checkedSubtract`] | [`LongMath.checkedSubtract`]
[`IntMath.checkedMultiply`] | [`LongMath.checkedMultiply`]
[`IntMath.checkedPow`]      | [`LongMath.checkedPow`]

``` java
IntMath.checkedAdd(Integer.MAX_VALUE, Integer.MAX_VALUE); // throws ArithmeticException
```

## Real-valued methods 真值方法

`IntMath`, `LongMath`, and `BigIntegerMath` have support for a variety of
methods with a "precise real value," but that round their result to an integer.
These methods accept a [`java.math.RoundingMode`]. This is the same
`RoundingMode` used in the JDK, and is an enum with the following values:

*   `DOWN`: round towards 0. (This is the behavior of Java division.)
*   `UP`: round away from 0.
*   `FLOOR`: round towards negative infinity.
*   `CEILING`: round towards positive infinity.
*   `UNNECESSARY`: rounding should not be necessary; if it is, fail fast by
    throwing an `ArithmeticException`.
*   `HALF_UP`: round to the nearest half, rounding `x.5` away from 0.
*   `HALF_DOWN`: round to the nearest half, rounding `x.5` towards 0.
*   `HALF_EVEN`: round to the nearest half, rounding `x.5` to its nearest even
    neighbor.

These methods are meant to be readable when used: for example, `divide(x, 3,
CEILING)` is completely unambiguous even on a casual read-through.

Additionally, each of these functions internally use only integer arithmetic,
except in constructing initial approximations for use in `sqrt`.

| Operation         | `IntMath`          | `LongMath`      | `BigIntegerMath`     |
| :---------------- | :----------------- | :-------------- | :------------------- |
| Division          | [`divide(int, int, RoundingMode)`] | [`divide(long, long, RoundingMode)`] | [`divide(BigInteger, BigInteger, RoundingMode)`] |
| Base-2 logarithm  | [`log2(int, RoundingMode)`] | [`log2(long, RoundingMode)`] | [`log2(BigInteger, RoundingMode)`] |
| Base-10 logarithm | [`log10(int, RoundingMode)`] | [`log10(long, RoundingMode)`] | [`log10(BigInteger, RoundingMode)`] |
| Square root       | [`sqrt(int, RoundingMode)`] | [`sqrt(long, RoundingMode)`] | [`sqrt(BigInteger, RoundingMode)`] |

``` java
BigIntegerMath.sqrt(BigInteger.TEN.pow(99), RoundingMode.HALF_EVEN);
   // returns 31622776601683793319988935444327185337195551393252
```

### Additional functions 其他方法

We provide support for a few other mathematical functions we've found useful.

Operation                                             | `IntMath`                                            | `LongMath`                                            | `BigIntegerMath`
:---------------------------------------------------- | :--------------------------------------------------- | :---------------------------------------------------- | :---------------
Greatest common divisor                               | [`gcd(int, int)`]                                    | [`gcd(long, long)`]                                   | In JDK: [`BigInteger.gcd(BigInteger)`]
Modulus (always nonnegative, -5 mod 3 is 1)           | [`mod(int, int)`]                                    | [`mod(long, long)`]                                   | In JDK: [`BigInteger.mod(BigInteger)`]
Exponentiation (may overflow)                         | [`pow(int, int)`]                                    | [`pow(long, int)`]                                    | In JDK: [`BigInteger.pow(int)`]
Power-of-two testing                                  | [`isPowerOfTwo(int)`]                                | [`isPowerOfTwo(long)`]                                | [`isPowerOfTwo(BigInteger)`]
Factorial (returns `MAX_VALUE` if input too big)      | [`factorial(int)`][`IntMath.factorial(int)`]         | [`factorial(int)`][`LongMath.factorial(int)`]         | [`factorial(int)`][`BigIntegerMath.factorial(int)`]
Binomial coefficient (returns `MAX_VALUE` if too big) | [`binomial(int, int)`][`IntMath.binomial(int, int)`] | [`binomial(int, int)`][`LongMath.binomial(int, int)`] | [`binomial(int, int)`][`BigIntegerMath.binomial(int, int)`]

## Floating-point arithmetic 浮点数方法

Floating point arithmetic is pretty thoroughly covered by the JDK, but we added
a few useful methods to [`DoubleMath`].

| Method                                | Description                           |
| :------------------------------------ | :------------------------------------ |
| [`isMathematicalInteger(double)`]     | Tests if the input is finite and an  exact integer. |
| [`roundToInt(double, RoundingMode)`]  | Rounds the specified number and casts it to an int, if it fits into an int, failing fast otherwise. |
| [`roundToLong(double, RoundingMode)`] | Rounds the specified number and casts it to a long, if it fits into a long, failing fast otherwise. |
| [`roundToBigInteger(double, RoundingMode)`] | Rounds the specified number to a `BigInteger`, if it is finite, failing fast otherwise. |
| [`log2(double, RoundingMode)`]       | Takes the base-2 logarithm, and rounds to an `int` using the specified `RoundingMode`. Faster than `Math.log(double)`. |

[`BigIntegerMath`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html
[`DoubleMath`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html
[`IntMath`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html
[`Ints`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/primitives/Ints.html
[Stats]: StatsExplained
[`LinearTransformation`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LinearTransformation.html
[`LongMath`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html
[`IntMath.checkedAdd`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedAdd-int-int-
[`LongMath.checkedAdd`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedAdd-long-long-
[`IntMath.checkedSubtract`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedSubtract-int-int-
[`LongMath.checkedSubtract`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedSubtract-long-long-
[`IntMath.checkedMultiply`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedMultiply-int-int-
[`LongMath.checkedMultiply`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedMultiply-long-long-
[`IntMath.checkedPow`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#checkedPow-int-int-
[`LongMath.checkedPow`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#checkedPow-long-long-
[`java.math.RoundingMode`]: http://docs.oracle.com/javase/8/docs/api/java/math/RoundingMode.html
[`divide(int, int, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#divide-int-int-java.math.RoundingMode-
[`divide(long, long, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#divide-long-long-java.math.RoundingMode-
[`divide(BigInteger, BigInteger, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#divide-java.math.BigInteger-java.math.BigInteger-java.math.RoundingMode-
[`log2(int, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log2-int-java.math.RoundingMode-
[`log2(long, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log2-long-java.math.RoundingMode-
[`log2(BigInteger, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log2-java.math.BigInteger-java.math.RoundingMode-
[`log10(int, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#log10-int-java.math.RoundingMode-
[`log10(long, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#log10-long-java.math.RoundingMode-
[`log10(BigInteger, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#log10-java.math.BigInteger-java.math.RoundingMode-
[`sqrt(int, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#sqrt-int-java.math.RoundingMode-
[`sqrt(long, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#sqrt-long-java.math.RoundingMode-
[`sqrt(BigInteger, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#sqrt-java.math.BigInteger-java.math.RoundingMode-
[`gcd(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#gcd-int-int-
[`gcd(long, long)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#gcd-long-long-
[`BigInteger.gcd(BigInteger)`]: http://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html#gcd-java.math.BigInteger-
[`mod(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#mod-int-int-
[`mod(long, long)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#mod-long-long-
[`BigInteger.mod(BigInteger)`]: http://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html#mod-java.math.BigInteger-
[`pow(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#pow-int-int-
[`pow(long, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#pow-long-int-
[`BigInteger.pow(int)`]: http://docs.oracle.com/javase/8/docs/api/java/math/BigInteger.html#pow-int-
[`isPowerOfTwo(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#isPowerOfTwo-int-
[`isPowerOfTwo(long)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#isPowerOfTwo-long-
[`isPowerOfTwo(BigInteger)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#isPowerOfTwo-java.math.BigInteger-
[`IntMath.factorial(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#factorial-int-
[`LongMath.factorial(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#factorial-int-
[`BigIntegerMath.factorial(int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#factorial-int-
[`IntMath.binomial(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/IntMath.html#binomial-int-int-
[`LongMath.binomial(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/LongMath.html#binomial-int-int-
[`BigIntegerMath.binomial(int, int)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/BigIntegerMath.html#binomial-int-int-
[`DoubleMath`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html
[`isMathematicalInteger(double)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#isMathematicalInteger-double-
[`roundToInt(double, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToInt-double-java.math.RoundingMode-
[`roundToLong(double, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToLong-double-java.math.RoundingMode-
[`roundToBigInteger(double, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#roundToBigInteger-double-java.math.RoundingMode-
[`log2(double, RoundingMode)`]: http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/math/DoubleMath.html#log2-double-java.math.RoundingMode-
