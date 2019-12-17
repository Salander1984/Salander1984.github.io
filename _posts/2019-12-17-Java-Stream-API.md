---
layout: post
title: Java Stream API
categories: Java
tags: Basic Java8 Stream
excerpt_separator: <!--more-->
---
This article is an introduction of Java Stream API.
<!--more-->
## Creation
There are so many methods to create a stream, most common way is to create from a collection or an array with the help 
of _stream()_ and _of()_. You can also use _builder()_, _generate()_ or _iterate()_ to create a stream under the 
condition without source.
### Empty Stream
Empty stream is often used to avoid returning _null_, you can create an empty stream by the _empty()_ method.
```java
Stream<String> streamEmpty = Stream.empty();
```
### Create From Values
```java
Stream<String> stringStream = Stream.of("a", "b", "c");
```
_of(T... values)_ is a static method to create stream.
### Create From An Array
```java
String[] strArray = new String[]{"a", "b", "c"};
Stream<String> stringStream = Arrays.stream(strArray);
```
Notice that above example used a static method to create a stream from an array.<br/>
When you use _Arrays.stream()_ to create a stream, you can only pick a part of the source array's elements into the 
stream by specify _startInclusive_ parameter and _endExclusive_ parameter of _Arrays.stream()_:
```java
Stream<String> stringStream = Arrays.stream(strArray, 1, 3);
stringStream.forEach(System.out::println);
```
In above example, the _startInclusive_ is 1, means the first _String_ in _stringStream_ is the element of _strArray_ 
whose index is 1; and the _endExclusive_ is 3, means the last _String_ in _stringStream_ is the element of _strArray_ 
whose index is less than 3. So the outputs of this example should be:
```
b
c
```
### Create From A List
```java
List<String> strList = new ArrayList<>();
strList.add("a");
strList.add("b");
strList.add("c");
Stream<String> StringStream = strList.stream();
```
Unlike [array](#create-from-an-array) , it used a default method in _Collection_ interface to create a stream from a list.

Of course, you can also use _of(T... values)_ to create stream from array or list.
```java
Stream<String> stream1 = Stream.of(strArray);
Stream<String> stream2 = Stream.of(strList);
```
### Create From 2 Stream
We can use _contact()_ to combine two streams into one. There are 3 tips should be mentioned here:
1. Types of the 2 source streams should be same.
2. The target stream is concatenated lazily. This means that the source 2 streams will be concatenated only if it is 
necessary for the terminal operation execution.
3. The target stream's elements are all the elements of the first stream followed by all the elements of the second stream.<br/>
Here is an example create a stream of _String_ from original two streams:
```java
Stream<String> stream1 = Stream.of("a", "b");
Stream<String> stream2 = Stream.of("d", "e");
Stream<String> stream3 = Stream.concat(stream1, stream2);
stream3.forEach(System.out::println);
```
OutPuts:
```
a
b
c
d
```
### _Stream.builder()_
Here is an example to use _builder()_ to create a stream whose elements are _String_.
```java
Stream<String> stringStream = Stream.<String>builder().add("a").add("b").add("c").build();
```
When _Stream.builder()_ is used, the desired type should be specified just before "builder()". Otherwise the builder 
doesn't know which type the target stream wants, so it'll create an instance of the ```Stream<Object>```.
### _Stream.generate()_
_Stream.generate()_ is used to create a infinity stream from a ```Supplier<T>```, in reality it's no point to do so, so 
we always use _limit()_ after calling _generate()_ to specify the size of stream. The following example creates a 
stream contains 5 random _Integer_.
```java
Stream<Integer> intStream = Stream.generate(new Random()::nextInt).limit(5);
```
Without _limit()_, the generate() method will work until it reaches the memory limit.
### _Stream.iterate()_
Another way of creating an infinite stream is by using the iterate() method.
The following example creates a stream contains 5 _Integer_, starting from 1 and increased by step of 2.
```java
Stream<Integer> intStream = Stream.iterate(1L, n -> n+2).limit(5);
intStream.forEach(System.out::println);
```
Outputs:
```
1
3
5
7
9
```
### Primitive Stream
Java 8 offers a possibility to create streams out of three primitive types: int, long and double. As ```Stream<T>``` is a 
generic interface and there is no way to use primitives as a type parameter with generics, three new special interfaces 
were created: _IntStream_, _LongStream_, _DoubleStream_.<br/>
Primitive Stream can be created by _Stream.of()_ or _Arrays.stream()_ from a primitive array.
```java
int[] nums = new int[]{1, 2, 3};
IntStream intStream1 = IntStream.of(nums);
IntStream intStream2 = Arrays.stream(nums);
IntStream intStream3 = Arrays.stream(nums, 1, 3);
```
The rules of creating a primitive stream from array is quite same as creating a ```Stream<T>``` from array, only the type 
of the result stream is _IntStream_(or _LongStream_ or _DoubleStream_) instead of ```Stream<T>```.

If you want to create a primitive stream without any sources, try _range(int startInclusive, int endExclusive)_ or
_rangeClosed(int startInclusive, int endExclusive)_.
```java
IntStream intStream = IntStream.range(1, 3);
LongStream longStream = LongStream.rangeClosed(1, 3);
```
Both _range()_ and _rangeClosed()_ create streams whose elements are incremental with the step equals to 1, and the 
first element is 1(_startInclusive_). The difference of _range()_ and _rangeClosed()_ is that the results of _range()_ 
don't include 3(_endExclusive) but _rangeClosed()_'s results do.

Another way to create a primitive stream is to use method of _Random_ class.
For example, the following code creates a DoubleStream, which has three elements:
```java
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);
```
## Operation
Here are 2 kinds of stream operations:
* **Intermediate Operation**: Intermediate operations transform a stream into a new onw, so the result of intermediate 
operations is a modified stream. Intermediate operations can be chained.
* **Terminal Operation**: As its name suggested, terminal operations are usually called after a series of intermediate 
operations and return a specific value or some actions applied to every elements of the stream. Once a terminal operation 
is called the stream becomes inaccessible.
### Intermediate Operation
**Mapping**<br/>
Mapping is used to perform a specific function on each element of the stream. The result of a mapping operation is a 
new stream whose each single element is different from the original stream, but the size of stream won't change.<br/>
There are 2 kinds of mapping:
1. _map()_<br/>
_map()_ is one of the most frequently used method when you deal with _Stream_.<br/>
Here is an example uses _map()_ to change every element of the stream to uppercase then prints out them one by one.
```java
Stream.of("a", "b", "c").map(str -> str.toUpperCase()).forEach(System.out::println);
```
2. _flatMap()_<br/>
_flatMap()_ is used to transform a stream whose every element contains its own inner elements into a larger size stream 
of all these inner elements.<br/>
The following example shows you how to use _flatMap()_.
```java
List<Integer> integers = Arrays.asList(1, 2, 3);
List<Double> doubles = Arrays.asList(1.0, 2.0, 3.0);
List<String> strings = Arrays.asList("a", "b", "c");
Stream.of(integers, doubles, strings).flatMap(list -> list.stream()).forEach(System.out::println);
```
Outputs:
```
1
2
3
1.0
2.0
3.0
a
b
c
```
**Filtering**<br/>
As its name tells, we use ```filter(Predicate<? super T> predicate)``` to filter out the elements that don't match the given 
predicate. Usually, the result is a smaller size stream than original one.
```java
Stream.of(1,2,3,4).filter(num -> num<=3).forEach(System.out::println);
```
Outputs:
```
1
2
3
```
**sorting**<br/>
_sorted()_ returns a stream consisting of the elements of this stream, sorted according to natural order.
```java
Stream.of(2, 1, 3).peek(System.out::println).sorted().forEach(System.out::println);
```
_peek()_ was used in above example to print out the middle results without changing the stream. We'll talk about 
_peek()_ in detail later.<br/>
Outputs of the above example should be:
```
2
1
3
1
2
3
```
Except for non-parameter sort, there is another ```sorted(Comparator<? super T> comparator)``` that returns a stream 
consisting of the elements of this stream, sorted according to the provided _Comparator_.
```java
Stream.of(2, 1, 3).sorted(Comparator.reverseOrder()).forEach(System.out::println);
```
Outputs:
```
3
2
1
```
**_peek()_**<br/>
Remember this: once you did any operations on a stream, no matter intermediate operation or terminal operation, the 
original stream has gone, it doesn't exist any more.<br/> 
What if you want to execute some actions on a stream but don't want to lose the original one? Try _peek()_.<br/>
_peek()_ returns the original stream who called it and performs the provided function on each element additionally. 
 _peek()_ is often used for debugging, in case you want to check the middle results of a series intermediate operations.
 ```java
Stream<String> stringStream = Stream.of("a", "b", "c").peek(str -> System.out.println(str.toUpperCase()));
stringStream.forEach(System.out::println);
```
Above example generated a stream whose elements were all lowercase, then _peek()_ transformed these lowercase into 
uppercase and printed out them but still returned a stream with all lowercase elements. So the outputs of this 
example are:
```
A
a
B
b
C
c
```
Maybe you'll ask why not:
```
A
B
C
a
b
c
```
We will talk it later in [Stream Pipeline](#stream-pipeline).
### Terminal Operation
**Matching**<br/>
```allMatch(Predicate<? super T> predicate)```: When all elements of this stream match the provided predicate returns 
true, otherwise returns false.<br/>
```anyMatch(Predicate<? super T> predicate)```: When any elements of this stream match the provided predicate returns 
 true, otherwise returns false.<br/>
```noneMatch(Predicate<? super T> predicate)```: When no elements of this stream match the provided predicate returns 
true, otherwise returns false.<br/>
Example:
```java
Stream.of(1, 2, 3).allMatch(num -> num > 2);  //false
Stream.of(1, 2, 3).anyMatch(num -> num > 2);  //true
Stream.of(1, 2, 3).noneMatch(num -> num > 2);  //false
```
**Searching**<br/>
In a non-parallel operation, results of _findAny()_ and _findFirst()_ are always same: the first element of this stream 
or an empty Optional if the stream is empty.<br/>
But in parallel operations, the result of _findAny()_ cannot be reliably determined, it may return any element of the 
stream if it's not empty.
```java
Stream.of(1, 2, 3).parallel().findAny();  //can be 1 or 2 or 3
Stream.of(1, 2, 3).parallel().findFirst();  //always 1
```
**Aggregating**<br/>
_count()_: Returns the size of the stream.
```java
Stream.of(1, 2, 3).count();  //3
```
_max()_: Returns the max element of the stream, sorted according to natural order.
```java
Stream.of(1, 2, 3).max();  //3
```
_min()_: Returns the min element of the stream, sorted according to natural order.
```java
Stream.of(1, 2, 3).min();  //1
```
By the way, you can also provide a comparator to _max()_ and _min()_.

**Reduction**<br/>
There are 3 _reduce()_ methods depending on their different parameters.<br/>
```reduce(BinaryOperator<T> accumulator)``` performs a reduction on the elements of this stream, using an associative 
accumulation function, and returns an Optional describing the reduced value, if any.<br/>
```reduce(T identity, BinaryOperator<T> accumulator)``` uses the identity as the first operand of accumulation function.<br/>
```reduce(V identity, BiFunction<V,? super T,V> accumulator, BinaryOperator<V> combiner)``` is used only in parallel mode. 
It works like performing the accumulation function with the identity on every element of stream in a parallel way, then 
using combining function to combine all the results returned from the accumulation function.<br/>
Here is an example:
```java
int result1 = Stream.of(1, 2, 3).reduce(Integer::sum).get();  // 6 = 1 + 2 + 3

int result2 = Stream.of(1, 2, 3).reduce(4, Integer::sum);  // 10 = 4 + 1 + 2 + 3

int result3 = Stream.of(1, 2, 3).parallel().reduce(4, Integer::sum, Integer::sum);  // 18 = (4 + 1) + (4 + 2) + (4 + 3)
```
**Collecting**<br/>
Actually _collect()_ method also performs reduction operation on the elements of the stream but returns a collection 
instead of a particular value. _collect()_ takes parameters that can be ```(Collector<? super T,A,R> collector)``` or 
```(Supplier<R> supplier, BiConsumer<R,? super T> accumulator, BiConsumer<R,R> combiner)```.<br/>
We use _collect()_ to transform a stream to a collection all the time:
```java
Stream<String> stringStream = Stream.of("a", "b", "c");
List<String> strList = stringStream.collect(Collectors.toList());
```