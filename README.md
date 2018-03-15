## 来自于 Wisdom 的 Guava 指南中文版
### 还有一部分代码的阅读与分析
#### 在灵感匮乏不能开始新项目的时候，阅读优秀的代码也是十分有效的提升

# Guava: Google 的 Java 核心类库

[![Build Status](https://travis-ci.org/google/guava.svg?branch=master)](https://travis-ci.org/google/guava)
[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.google.guava/guava/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.google.guava/guava)

Guava 是 Google 提供的一组 Java 核心类库，包括新的集合框架(比如
multimap 和 multiset), 不可变集合 (immutable collections), 图形库(graph library), 函数 (functional types), 内存中缓存技术(in-memory cache), 
以及用于并发的 API, I/O, 哈希算法(hashing), 原语(primitives), 
反射(reflection), 字符串处理(string processing)，以后还会有更多!

Guava 有两种风格

*   JRE 风格请使用 JDK 1.8 或者更高版本。
*   如果你使用的是 JDK 1.7 版本或者 Android 的话 ，请使用 Android 版本。你可以在 [`android` directory] 中找到 Guava 安卓版.

[`android` directory]: https://github.com/google/guava/tree/master/android

## 最新版本

最新的版本是 [Guava 24.1][current release], 创建于 2018-03-14.

Guava 在 Maven 仓库中的 id 为`com.google.guava`, 块ID(artifact ID) 是 `guava`. 
JRE 请使用`24.1-jre` 版本, Android 请使用 `24.1-android` 版本。

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

1. 在类或者方法上添加`@Beta` 注解的 API可能会发生变化，
他们可以随意在任何时间以任何方式被修改，或者可能被移除。
如果您的代码和一个类库(即，他可以被其他用户在不受你控制的路径下使用),
您就不应使用带有`@Beta` 注解的 API, 除非你 [重新打包](https://github.com/google/guava/wiki/UseGuavaInYourBuild#what-if-i-want-to-use-beta-apis-from-a-library-that-people-use-as-a-dependency) 他们.
**如果您的代码是一个类库，我们强烈建议您使用[Guava Beta Checker] 工具去检查您没有使用带有 `@Beta` 注解的 API**

2. 没有被`@Beta`注解标记的API在未来是会被兼容的(Previously, we sometimes removed such APIs after a deprecation period.
The last release to remove non-`@Beta` APIs was Guava 21.0.) Even `@Deprecated`
APIs will remain (again, unless they are `@Beta`). We have no plans to start
removing things again, but officially, we're leaving our options open in case
of surprises (like, say, a serious security problem).

3. Serialized forms of ALL objects are subject to change unless noted
otherwise. Do not persist these and assume they can be read by a
future version of the library.

4. Our classes are not designed to protect against a malicious caller.
You should not use them for communication between trusted and
untrusted code.

5. For the mainline flavor, we unit-test the libraries using only OpenJDK 1.8 on
Linux. Some features, especially in `com.google.common.io`, may not work
correctly in other environments. For the Android flavor, our unit tests run on
API level 15 (Ice Cream Sandwich).

[current release]: https://github.com/google/guava/releases/tag/v24.1
[guava-snapshot-api-docs]: http://google.github.io/guava/releases/snapshot-jre/api/docs/
[guava-snapshot-api-diffs]: http://google.github.io/guava/releases/snapshot-jre/api/diffs/
[Guava Explained]: https://github.com/google/guava/wiki/Home
[Guava Beta Checker]: https://github.com/google/guava-beta-checker

<!-- References -->

[Using Guava in your build]: https://github.com/google/guava/wiki/UseGuavaInYourBuild
[repackage]: https://github.com/google/guava/wiki/UseGuavaInYourBuild#what-if-i-want-to-use-beta-apis-from-a-library-that-people-use-as-a-dependency

