# Apache Commons Collections Equivalents
# Guava Commons 与 Apache Commons

*脑残译者注：Apache 基金会的 Commons 用在了成千上万的 Java 项目中,  无论是 SE 项目，Web 项目还是物联网项目,甚至在各大框架中也都有
Apache Commons 包的身影，仅仅拿工具类库或者是基础类库来说，Apache Commons 已经可以算作 Open Sourse 的神话.*

# [CollectionUtils]([source])

`CollectionUtils`                                           | Guava
:---------------------------------------------------------- | :----
`void addAll(Collection, Enumeration)`                      | `Iterators.addAll(collection, Iterators.forEnumeration(enumeration))`
`void addAll(Collection, Iterator)`                         | `Iterators.addAll(collection, iterator)`
`void addAll(Collection, Object[])`                         | `Collections.addAll(collection, array)` (JDK)
`boolean addIgnoreNull(Collection, Object)`                 | `if (o != null) { collection.add(o); }`
`int cardinality(Object, Collection)`                       | `Iterables.frequency(collection, object)`
`Collection collect(Collection, Transformer)`               | `newArrayList(Collections2.transform(input, function))`
`Collection collect(Collection, Transformer, Collection)`   | `output.addAll(Collections2.transform(input, function))`
`Collection collect(Iterator, Transformer)`                 | `newArrayList(Iterators.transform(input, function))`
`Collection collect(Iterator, Transformer, Collection)`     | `Iterators.addAll(output, Iterators.transform(input, function))`
`boolean containsAny(Collection coll1, Collection coll2)`   | `!Collections.disjoint(coll1, coll2)` (JDK)
`int countMatches(Collection, Predicate)`                   | `Iterables.size(Iterables.filter(collection, predicate))`
`Collection disjunction(Collection, Collection)`            | `Sets.symmetricDifference(set1, set2)`
`boolean exists(Collection, Predicate)`                     | `Iterables.any(collection, predicate)`
`void filter(Collection, Predicate)`                        | `Iterables.removeIf(collection, not(predicate))` (参见 `Iterables.transform`,创建了视图而不是改变了输入)
`Object find(Collection, Predicate)`                        | `Iterables.find(collection, predicate)`
`void forAllDo(Collection, Closure)`                        | `for (Object o : collection) { closure.execute(o); }`
`Object get(Object, int)`                                   | `Iterables.get(o, index)`, 或者调用 `entrySet()`, `forEnumeration()`, 等等.
`Map getCardinalityMap(Collection)`                         | `ImmutableMultiset.copyOf(collection)`
`Object index(Object, int)`                                 | `Iterables.get(o, index)`, 或者调用 `keySet()`, `forEnumeration()`, 等等.
`Object index(Object, Object)`                              | `Iterables.get(o, index)`, s或者调用 `entrySet()`, `forEnumeration()`, 等等.
`Collection intersection(Collection, Collection)`           | `Sets/Multisets.intersection(a, b)`
`boolean isEmpty(Collection)`                               | `collection == null`
`boolean isEqualCollection(Collection, Collection)`         | 如果集合都是 `Set` 或者 `Multiset`, 请使用 `equals()`; 要不然就用 `ImmutableMultiset.copyOf(a).equals(ImmutableMultiset.copyOf(b)`
`boolean isFull(Collection)`                                | 没有等价方法--不是 `BoundedCollection` 类型.
`boolean isNotEmpty(Collection)`                            | `collection != null && !collection.isEmpty()`
`boolean isProperSubCollection(Collection, Collection)`     | 没有等价方法-- 检查`a.size() < b.size()` 然后使用下面的描述进行检查.
`boolean isSubCollection(Collection, Collection)`           | `Multisets.containsOccurrences(ImmutableMultiset.copyOf(coll1), ImmutableMultiset.copyOf(coll2))`
`int maxSize(Collection)`                                   | 没有等价方法--不是 `BoundedCollection` 类型.
`Collection predicatedCollection(Collection, Predicate)`    | `Constraints.constrainedCollection/List/Set`/等等.
`Collection removeAll(Collection, Collection)`              | `newArrayList(Iterables.filter(collection, Predicates.not(Predicates.in(remove))))`
`Collection retainAll(Collection, Collection)`              | `newArrayList(Iterables.filter(collection, Predicates.in(retain)))`
`void reverseArray(Object[])`                               | `Lists.reverse(Arrays.asList(array))` (返回一个倒序的 `List` 视图，而不是修改这个数组)
`Collection select(Collection, Predicate)`                  | `newArrayList(Iterables.filter(collection, predicate))`
`void select(Collection, Predicate, Collection)`            | `Iterables.addAll(output, Iterables.filter(input, predicate))`
`Collection selectRejected(Collection, Predicate)`          | `newArrayList(Iterables.filter(collection, Predicates.not(predicate)))`
`void selectRejected(Collection, Predicate, Collection)`    | `Iterables.addAll(output, Iterables.filter(input, Predicates.not(predicate)))`
`int size(Object)`                                          | `Collection/Map.size()`, `array.length`, `Iterables/Iterators.size` (如果有必要的话，使用 `forEnumeration()` )
`boolean sizeIsEmpty(Object)`                               | `Collection/Map.isEmpty()`, `array.length == 0`, `Iterables/Iterators.isEmpty` (如果有必要的话，使用 `forEnumeration()`)
`Collection subtract(Collection, Collection)`               | 没有等价方法--创建一个包含 `a` 的 `ArrayList` 然后调用 `remove` 方法将其中的 `a` 换成 `b`.
`Collection synchronizedCollection(Collection)`             | `Collections.synchronizedCollection(collection)` (JDK)
`void transform(Collection, Transformer)`                   | 没有等价的方法—— 转换 `Collection` 成一个 `Transformer` 并不是十分有用,最好的方式是转换成视图(`Lists/Collections2.transform`)或者是复制一份.
`Collection transformedCollection(Collection, Transformer)` | 没有等价的方法—— 转换添加到 `Collection` 中的 `Object` 对象。不过使用`ForwardingCollection` 可以轻松搞定.
`Collection typedCollection(Collection, Class)`             | `Collections.checkedCollection/Set/List`/等等. (JDK)
`Collection union(Collection, Collection)`                  | `Sets.union(a, b)`
`Collection unmodifiableCollection(Collection)`             | `Collections.unmodifiableCollection/Set/List`/等等. (JDK) 如果想要不可变的话，考虑 `ImmutableCollection` 类型的集合.

[CollectionUtils]: http://commons.apache.org/collections/apidocs/org/apache/commons/collections/CollectionUtils.html
[source]: http://svn.apache.org/viewvc/commons/proper/collections/trunk/src/java/org/apache/commons/collections/CollectionUtils.java?view=markup
