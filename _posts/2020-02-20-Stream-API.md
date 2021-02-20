---
title:  "Stream API"
categories: programming
tags: JAVA
classes: wide
---

## Stream(스트림)이란?

Stream(스트림)은 요소들의 스퀀스를 처리하기위한 클래스다. 이 스트림은 컬렉션과 여러 가지면에서 차이가 있다. 스트림은 다음과 같은 특징을 가진다.

- No storage: 컬렉션은 데이터의 물리적인 집합이지만, 스트림은 여러 개의 연산을 적용하기 위한 논리적인 뷰다. 즉, 컬렉션은 데이터에 관한 것이지만, 스트림은 계산에 관한 것이다.

- Functional in nature: 스트림에 대한 작업은 결과를 생성하지만 원본을 수정하지는 않다. 예를 들어 스트림에서 필터링을 호출하면 원래 컬렉션에서 제거하지 않고 새 스트림을 반환한다.

- Lazyness execution: 불필요한 작업을 피하기 위해 연산을 최대한 뒤로 미룬다. 예를들어 스트림에서 처음 3개의 홀수를 찾기 위해 전체 데이터 세트를 거치지않고, 3개의 값이 발견되면 실행을 중지한다.

- Possibily unbounded: 컬렉션과 다르게 스트림은 크기가 제한되어 있지 않다. 따라서 무한 스트림에 대한 계산을 유한 시간 내에 완료할 수 있다.

- Consumable: 스트림의 요소는 한 번만 사용이 가능하다. 동일한 요소를 다시 사용하려면 새 스트림을 생성해야된다.

## 생성 하기

- 빈 스트림

```java
Stream<String> streamEmpty = Stream.empty();
```

- 컬렉션의 스트림

```java
List<String> collection = Arrays.asList("a", "b", "c");
Stream<String> streamOfCollection = collection.stream();
```

- 어레이의 스트림

```java
Stream<String> streamOfArray = Stream.of("a", "b", "c");
```

또는

```java
String[] arr = new String[]{"a", "b", "c"};
Stream<String> streamOfArrayFull = Arrays.stream(arr);
Stream<String> streamOfArrayPart = Arrays.stream(arr, 1, 3);
```

- Stream.builder()

```java
Stream<String> streamBuilder =　Stream.<String>builder().add("a").add("b").add("c").build();
```

- Stream.generate()

무한한 스트림을 생성하므로 원하는 크기를 잘라주는 작업이 필요하다.

```java
Stream<String> streamGenerated = Stream.generate(() -> "element").limit(10);
```

- Stream.iterate()

무한 스트림을 만드는 또 다른 방법으로, 첫 번째 인자는 첫 번째로 사용할 매개 변수이다. 다음 요소에서는 이전에 생성한 요소를 매개 변수로 만들어진다.

```java
Stream<Integer> streamIterated = Stream.iterate(40, n -> n + 2).limit(20);
```

- 원시 스트림

```java
IntStream intStream = IntStream.range(1, 3);
LongStream longStream = LongStream.rangeClosed(1, 3);
```

`Random` 클래스도 원시 스트림을 위한 메서드를 제공하고 있다.

```java
Random random = new Random();
DoubleStream doubleStream = random.doubles(3);
```

- String 스트림

`char()` 메서드를 이용하여 문자열의 스트림을 만들 수 있다. 따로 인터페이스가 없기 때문에 `IntStream`을 사용한다.

```java
IntStream streamOfChars = "abc".chars();
```

정귝식에 따라 문자열을 잘라서 사용할 수도 있다.

```java
Stream<String> streamOfString = Pattern.compile(", ").splitAsStream("a, b, c");
```

- 파일 스트림

```java
Path path = Paths.get("C:\\file.txt");
Stream<String> streamOfStrings = Files.lines(path);
Stream<String> streamWithCharset = 
  Files.lines(path, Charset.forName("UTF-8"));
```

## Stream API

- Filtering

조건이 참인 요소를 걸러낸다.

`Stream<T> filter(Predicate<? super T> p)`

- Trucating Stream

스트림을 필요한 개수만큼 잘라낸다.

`Stream<T> limit(long maxSize)`: maxSize보다 길지않도록 잘라낸다.

`Stream<T> skip(long n)`: 처음 n개의 값을 잘라낸다.

- Consuming Stream

각 요소를 인자로 받는 Consumer를 사용할 때 쓴다.

`Stream<T> peek(Consumer<? super T> action)`: 스트림이 끝나지 않고 중간 연산일 때 사용

`void forEach(Consumer<? super T> action)`: 스트림이 끝나는 최종 연산에서 사용

- Mapping

각 요소를 다른 요소로 변형할 때 사용한다.

`<R> Stream <R> map (Function <? super T,? extends R> mapper)`

원시 타입으로 변형하기 위해서는 `mapToInt`, `mapToDouble`, `mapToLong`을 써야된다.

각 요소가 스트림이거나 2중 배열일 때, `flatMap`으로 단일 스트림으로 변형시킬 수 있다.

- Matching

스트림의 요소들이 조건을 만족하는지 확인하 때 사용한다.

`anyMatch`: 하나라도 조건을 만족하는 요소가 있으면 `true` 반환

`allMatch`: 모든 요소가 조건을 만족하면 `true` 반환

`noneMatch`: 조건을 만족하는 요소가 없으면 `true` 반환

- Finding element

조건을 만족하는 요소를 하나 찾을 때 사용한다.

`Optional<T> findFirst()`: 첫번째로 조건을 만족하는 요소 반환

`Optional<T> findAny()`: 조건을 만족하는 요소 하나를 자유롭게 반환하기 때문에 비결정적

반환값이 없을 수도 있다는 것을 나타내는 Optional 이기 때문에, `orElseGet` 또는 `orElseThrow`로 처리해줘야된다.

- Stream Reduction

스트림의 요소를 바탕으로 하나의 값을 도출할 때 사용할 수 있다.

`T reduce(T identity, BinaryOperator<T> accumulator)`: identity는 accumulator에 처음에 대입할 인자의 값이다.

- Sort

스트림의 요소들을 정렬할 때 사용할 수 있다.

`Stream<T> sorted(Comparator<? super T> comparator)`: comparator는 이미 존재하는 API를 사용하거나, 직접 커스텀하여 만들 수도 있다. `comparingXXX()`나 `thenComparing()`가 있다.

## Collecting

- Collecting as collections

스트림을 컬렉션으로 변환할 수 있는 방법이 있다.

```java
Collector<T, ?, List<T>> toList()
Collector<T, ?, Set<T>> toSet()
Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)
Collector<T, ?, Map<K,U>> toMap(Function<T, K> keyMapper, Function<T, U> valueMapper)
```

- Strings joining

스트림에 있는 요소들을 하나의 String으로 표현하게 할 수 있다.

```java
joining(CharSequence delimiter)
joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)
```

- Grouping elements

스트림의 요소들을 그룹에따라 나누어 Map으로 만들 수 있다.

```java
groupingBy(Function<T, K> classifier, Collector<T, A, D> downstream
```

- Partitioning elements

`TRUE`, `FALSE`에따라 그룹을 나눌 수도 있다

예시

```java
Map<Boolean, List<Trade>> map2 = trades.stream().collect(Collectors.partitioningBy(t -> "USD".equals(t.getCurrency())));
```

- Arthmetic & Summerizing

스트림의 요소들의 최대값이나 최소값, 합, 평균을 구할 수 있다.

```java
Collector<T, ?, Optional<T>> minBy(Comparator<T> comparator)
Collector<T, ?, Optional<T>> maxBy(Comparator<T> comparator)
Collector<T, ?, XXX> summingXXX(ToXXXFunction<T> mapper)
Collector<T, ?, XXX> averagingXXX(ToXXXFunction<T> mapper)
```

- 기타 기능

`collectingAndThen(Collector<T,A,R> downstream, Function<R,RR> finisher)`: 컬렉션을 만들고, 이를 바탕으로 Function의 인자로 사용할 수 있다
`mapping(Function<T,U> mapper, Collector<U, A, R> downstream)`: 반대로 Function을 적용하고, 컬렉션을 만들 수 있다.

## 관련 사이트

<https://www.baeldung.com/java-8-streams-introduction>

<https://www.baeldung.com/java-8-streams>

<https://java-8-tips.readthedocs.io/en/stable/streams.html>