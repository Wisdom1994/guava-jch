# Math -- Guava的数学工具包

这个包中含有各种各样的数学工具类,比 JDK 更优化, 测试更完善

## Contents 综述

*   Guava Math 提供了为基本数据类型而设计的独立的类[`IntMath`],[`LongMath`], 
    [`DoubleMath`], 和 [`BigIntegerMath`], 这些类具有这相互平行的结构,
    他们的方法都是基于相应数据类型而进行实现.
    **请注意: 在`com.google.common.primitives`包中, 有一些函数或者类可能看起来不那么的'数学',
    比如 [`Ints`].**
*   Guava 为单个或者成对的数据集提供了各种统计计算的方法(比如求平均值，中位数等等).
    如果想使用 Guava Math 包，请先阅读这个指南[Stats] 而不是去阅读 Java DOC.
*   [`LinearTransformation`] 代表着 `y = mx + b(这是个一次函数)` 中的线性变换;
    比如英尺和米之间的换算(英尺 = 米 * 3.2808399 ), 或者开氏温度与华氏温度之间的换算(℉ = 1.8 * K - 459.67 ).

## Examples 举个栗子

``` java
int logFloor = LongMath.log2(n, FLOOR);

int mustNotOverflow = IntMath.checkedMultiply(x, y);

long quotient = LongMath.divide(knownMultipleOfThree, 3, RoundingMode.UNNECESSARY); // fail fast on non-multiple of 3

BigInteger nearestInteger = DoubleMath.roundToBigInteger(d, RoundingMode.HALF_EVEN);

BigInteger sideLength = BigIntegerMath.sqrt(area, CEILING);
```

## Why use these? 为什么使用这些工具类?

*   Guava Math的工具类为很多不常见的溢出情况都做了充分的测试. 溢出的语义也在相关的文档中进行了清晰的定义.
    如果预检查不能通过, 则快速的返回失败(异常).
*   Guava Math 已经进行了基准测试和最佳的优化, 尽管因为不同的硬件原因会造成不可避免的性能差异,
    但是 Guava Math 通常情况下的运行速度与 `Apache Commons` 的 `MathUtils` 互相媲美,
    在某些场景下, Guava Math 甚至更优.
*   这些类在设计之初就考虑到了代码的可读性、帮助养成好的编码习惯。
    比如 `IntMath.log2(x, CEILING)` 在你进行快速浏览代码的时候也能清晰快速的了解它的含义，
    而`32 - Integer.numberOfLeadingZeros(x - 1)` 在你不查看API的情况下就很难理解
    (**而它，实际上表示x-1转换为32位二进制补码之后，最高有值位前0的个数**)。

_注意: 这些类与 GWT 不兼容的，他们也不都是 GWT 的优化， 原因是他们有不同的计算溢出逻辑。_

## Math on Integral Types 整形计算

Math工具包主要处理三种整数类型值的计算： `int`, `long`, 和 `BigInteger`。 
其中的工具类被合理的命名为[`IntMath`], [`LongMath`], 和 [`BigIntegerMath`].

### Checked Arithmetic 检查方法

We provide arithmetic methods for `IntMath` and `LongMath` that fail fast on
overflow instead of silently ignoring it.
Guava Math 为 `IntMath` 和 `LongMath` 计算中可能的一些结果溢出情况提供了一些运算方法, 
这些方法将导致有结果溢出的计算**快速失败**而不是忽略掉溢出。

`IntMath`                   | `LongMath`
:-------------------------- | :---------------------------
[`IntMath.checkedAdd`]      | [`LongMath.checkedAdd`]
[`IntMath.checkedSubtract`] | [`LongMath.checkedSubtract`]
[`IntMath.checkedMultiply`] | [`LongMath.checkedMultiply`]
[`IntMath.checkedPow`]      | [`LongMath.checkedPow`]

``` java
IntMath.checkedAdd(Integer.MAX_VALUE, Integer.MAX_VALUE); // 抛出 ArithmeticException 异常.
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

我们提供了一些额外的我们所发现的十分有用的一些数学方法(函数/工具/公式).

Operation 运算                                             | `IntMath` 整形计算                               | `LongMath` 长整型计算                                   | `BigIntegerMath` 超大整形数据计算
:---------------------------------------------------- | :--------------------------------------------------- | :---------------------------------------------------- | :---------------
Greatest common divisor                               | [`gcd(int, int)`]                                    | [`gcd(long, long)`]                                   | In JDK: [`BigInteger.gcd(BigInteger)`]
Modulus (always nonnegative, -5 mod 3 is 1)           | [`mod(int, int)`]                                    | [`mod(long, long)`]                                   | In JDK: [`BigInteger.mod(BigInteger)`]
Exponentiation (may overflow)                         | [`pow(int, int)`]                                    | [`pow(long, int)`]                                    | In JDK: [`BigInteger.pow(int)`]
Power-of-two testing                                  | [`isPowerOfTwo(int)`]                                | [`isPowerOfTwo(long)`]                                | [`isPowerOfTwo(BigInteger)`]
Factorial (returns `MAX_VALUE` if input too big)      | [`factorial(int)`][`IntMath.factorial(int)`]         | [`factorial(int)`][`LongMath.factorial(int)`]         | [`factorial(int)`][`BigIntegerMath.factorial(int)`]
Binomial coefficient (returns `MAX_VALUE` if too big) | [`binomial(int, int)`][`IntMath.binomial(int, int)`] | [`binomial(int, int)`][`LongMath.binomial(int, int)`] | [`binomial(int, int)`][`BigIntegerMath.binomial(int, int)`]

## Floating-point arithmetic 浮点数方法

Guava 对于浮点数计算的实现采用的完全是 JDK 中的方法, 但是在 Guava Math 中,
我们为 [`DoubleMath`] 添加了一些十分有用的方法.

| Method 方法                           | Description  描述                         |
| :------------------------------------ | :------------------------------------ |
| [`isMathematicalInteger(double)`]     | 判断这个数是一个有限数(非无穷)并且是一个精确的整数. |
| [`roundToInt(double, RoundingMode)`]  | 围绕这个特殊的数, 将其转换成一个整形 int,如果溢出(超出int的范围)则抛出异常 |
| [`roundToLong(double, RoundingMode)`] | 围绕这个特殊的数, 将其转换为一个 long, 如果溢出, 则抛出异常. |
| [`roundToBigInteger(double, RoundingMode)`] | 将这个数转换为 `BigInteger`, 如果这个数是无限的, 则抛出异常. |
| [`log2(double, RoundingMode)`]       | 取 2 的 log 对数, 并根据 `RoundingMode` 转换为 int; 比 JDK 中的 `Math.log(double)` 计算速度要快. |

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
