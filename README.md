## 来自于 Wisdom 的 Guava 指南中文版
### 还有一部分代码的阅读与分析
#### 在灵感匮乏不能开始新项目的时候，阅读优秀的代码也是十分有效的提升

# Guava: Google 的 Java 核心类库

[![Build Status](https://travis-ci.org/google/guava.svg?branch=master)](https://travis-ci.org/google/guava)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.google.guava/guava/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.google.guava/guava)

Guava 是 Google 提供的一组 Java 核心类库，包括新的集合框架(比如 multimap 和 multiset), 不可变集合 (immutable collections), 图形库(graph library), 函数 (functional types), 内存中缓存技术(in-memory cache), 以及用于并发的 API, I/O, 哈希算法(hashing), 原语(primitives), 反射(reflection), 字符串处理(string processing)，以后还会有更多!

Guava 有两种风格

*   JRE 风格请使用 JDK 1.8 或者更高版本。
*   如果你使用的是 JDK 1.7 版本或者 Android 的话 ，请使用 Android 版本。你可以在 [`android` directory] 中找到 Guava 安卓版.

[`android` directory]: https://github.com/google/guava/tree/master/android

## 最新版本

最新的版本是 [Guava 24.1][current release], 创建于 2018-03-14.

Guava 在 Maven 仓库中的 id 为`com.google.guava`, 块ID(artifact ID) 是 `guava`. JRE 请使用`24.1-jre` 版本, Android 请使用 `24.1-android` 版本。

想要通过 Maven 将 Guava 加入到你的项目，请在你的 pom.xml 中加入以下配置：

```xml
<dependency>
  <groupId>com.google.guava</groupId>
  <artifactId>guava</artifactId>
  <version>24.1-jre</version>
  <!-- or, for Android: -->
  <version>24.1-android</version>
</dependency>
```

通过 Gradle 将 Guava 加入到你的项目:

```
dependencies {
  compile 'com.google.guava:guava:24.1-jre'
  // or, for Android:
  compile 'com.google.guava:guava:24.1-android'
}
```

想了解更多将 Guava 引入的方式，请参见 [Using Guava in your build].

## 快照

Guava 的最新的快照是通过 Maven 构建的、基于 `master` 分支的 `HEAD-jre-SNAPSHOT`, 或者是应用于 Android 的 `HEAD-android-SNAPSHOT` 分支。

- 快照-API 文档: [guava][guava-snapshot-api-docs]
- 快照-版本更新: [guava][guava-snapshot-api-diffs]

## 关于 Guava

- 官方指南 : [Guava Explained]
- 一个优秀的、有用的网站的集合 ：[www.tfinico.com](http://www.tfnico.com/presentations/google-guava)

## 链接

- [Github项目地址 https://github.com/google/guava](https://github.com/google/guava)
- [Issue tracker: 提出 bug 或请求新特性](https://github.com/google/guava/issues/new)
- [StackOverflow: 询问有关于"如何使用(how-to)" 和 "为什么Guava不工作了(why-didn't-it-work)"等相关问题](https://stackoverflow.com/questions/ask?tags=guava+java)
- [guava-discuss: 开放式问答与讨论](http://groups.google.com/group/guava-discuss)

## 警告

1. 在类或者方法上添加`@Beta` 注解的 API可能会发生变化，他们可以随意在任何时间以任何方式被修改，或者可能被移除。如果您的代码和一个类库(即，他可以被其他用户在不受你控制的路径下使用),
您就不应使用带有`@Beta` 注解的 API, 除非你 [重新打包](https://github.com/google/guava/wiki/UseGuavaInYourBuild#what-if-i-want-to-use-beta-apis-from-a-library-that-people-use-as-a-dependency) 他们.
**如果您的代码是一个类库，我们强烈建议您使用[Guava Beta Checker] 工具去检查您没有使用带有 `@Beta` 注解的 API**

2. 没有被`@Beta`注解标记的API在不确定的将来会保持二进制兼容(在之前，我们有时会在弃用期结束之后移除掉这些API。最后一次移除 “非`@Beta`”的API是在 21.0 版本。)
然而`@Deprecated(已经弃用)`的API仍然存在(除非他们是`@Beta`)。我们没有再次开始删除这些东西的计划，但是从官方的角度来说，我们可能在出现意外时(比如说发生了严重的安全问题的)再次的开启这个计划(删除 API)。
[注：我也没太搞懂他说的什么意思...可能是我翻译的有问题?]

3. 除非另有说明，所有的对象序列化的方式都有可能被更改。对此你不应该特别的关注，你只要相信这些对象在未来版本的库中都能被序列化就可以了。

4. 我们的类在设计之初就没有设计成可以防范非法调用的功能，所以你不应该将他们用在“可信”与“不可信”的代码之间的通信上。

5. 对于主流版本，我们只在 Linux 上使用 OpenJDK_1.8 来进行单元测试。对于某些功能来说，尤其是 `com.google.common.io`可能在其他的环境下无法正常的工作。对于 Android 版本，我们的单元测试运行在 Lv15 的等级上。

# 这里之下可以看作目录
## Guava 用户指南 中文
### Hashing 哈希
* [Hashing 哈希](https://github.com/Wisdom1994/guava-jch/blob/master/Guied-Explained(%E6%8C%87%E5%8D%97-%E8%AF%B4%E6%98%8E%E4%B9%A6)/Hashing(%E5%93%88%E5%B8%8C).md)
* [Caches 缓存](https://github.com/Wisdom1994/guava-jch/blob/master/Guied-Explained(%E6%8C%87%E5%8D%97-%E8%AF%B4%E6%98%8E%E4%B9%A6)/Caches(%E7%BC%93%E5%AD%98%E6%8A%80%E6%9C%AF).md)
## Guava 代码解析
<!-- References -->
[current release]: https://github.com/google/guava/releases/tag/v24.1
[guava-snapshot-api-docs]: http://google.github.io/guava/releases/snapshot-jre/api/docs/
[guava-snapshot-api-diffs]: http://google.github.io/guava/releases/snapshot-jre/api/diffs/
[Guava Explained]: https://github.com/google/guava/wiki/Home
[Guava Beta Checker]: https://github.com/google/guava-beta-checker

[Using Guava in your build]: https://github.com/google/guava/wiki/UseGuavaInYourBuild
[repackage]: https://github.com/google/guava/wiki/UseGuavaInYourBuild#what-if-i-want-to-use-beta-apis-from-a-library-that-people-use-as-a-dependency

