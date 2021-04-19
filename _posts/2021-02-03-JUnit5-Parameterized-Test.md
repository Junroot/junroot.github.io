---
title:  "JUnit5 Parameterized Test"
categories: programming
tags: JUnit
---

## Parameterized Test란?

테스트 코드 작성시 각 다른 매개변수들로 한 테스트 메소드를 여러 번 실행해야되는 상황이 발생할 수 있다. 이 때 중복 코드를 발생시키지 않고 여러번 테스트를 구현가능 하도록해주는 것이 Parameterized Test 이다.

## 시작하기

기존의 테스트 코드에서 `@ParameterizedTest` annotation을 추가하면 사용할 수 있다.

예시:

```java
@ParameterizedTest
@ValueSource(ints = {1, 3, 5, -3, 15, Integer.MAX_VALUE}) // six numbers
void isOdd_ShouldReturnTrueForOddNumbers(int number) {
    assertTrue(Numbers.isOdd(number));
}
```

## 인자 넘기기

테스트 케이스로 사용할 인자를 넘기는 방법에는 여러가지 방법이 있다.

### @ValueSource

간단한 값들을 테스트 메소드에 넘길 때 사용할 수 있다.

예시:

```java
@ParameterizedTest
@ValueSource(strings = {"", "  "})
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

매개 변수로 사용 아래의 자료형만 사용가능하다.

- *short* (with the *`shorts`* attribute)
- *byte* (with the *`bytes`* attribute)
- *int* (with the *`ints`* attribute)
- *long* (with the *`longs`* attribute)
- *float* (with the *`floats`* attribute)
- *double* (with the *`doubles`* attribute)
- *char* (with the *`chars`* attribute)
- *java.lang.String* (with the *`strings`* attribute)
- *java.lang.Class* (with the *`classes`* attribute)

### @NullSource / @EmptySource

`null` 값이나 빈 값을 매개 변수로 사용할 수 있다. `String`과 `Collection` 에서 사용이 가능하다.

```java
@ParameterizedTest
@NullSource
void isBlank_ShouldReturnTrueForNullInputs(String input) {
    assertTrue(Strings.isBlank(input));
}
```

```java
@ParameterizedTest
@EmptySource
void isBlank_ShouldReturnTrueForEmptyStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

또한 이 둘을 합쳐 `@NullAndEmptySource`도 사용이 가능하다.

```java
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = {"  ", "\t", "\n"})
void isBlank_ShouldReturnTrueForAllTypesOfBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

### @Enum

Enumeration에 있는 모든 값을 테스트할 때 사용할 수 있다.

```java
@ParameterizedTest
@EnumSource(Month.class) // passing all 12 months
void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
    int monthNumber = month.getValue();
    assertTrue(monthNumber >= 1 && monthNumber <= 12);
}
```

또는, `names` 속성을 이용하여 필요한 값만 테스트를 할 수 있다.

```java
@ParameterizedTest
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

`names` 속성 외의 값을 테스트하고 싶다면, `mode` 속성을 `EXCLUDE`로 지정하면 된다.

```java
@ParameterizedTest
@EnumSource(
  value = Month.class,
  names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER", "FEBRUARY"},
  mode = EnumSource.Mode.EXCLUDE)
void exceptFourMonths_OthersAre31DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(31, month.length(isALeapYear));
}
```

정규식을 사용하여 테스트 할 값을 지정할 수도 있다.

```java
@ParameterizedTest
@EnumSource(value = Month.class, names = ".+BER", mode = EnumSource.Mode.MATCH_ANY)
void fourMonths_AreEndingWithBer(Month month) {
    EnumSet<Month> months =
      EnumSet.of(Month.SEPTEMBER, Month.OCTOBER, Month.NOVEMBER, Month.DECEMBER);
    assertTrue(months.contains(month));
}
```

### @CsvSource

입력 값과 예상하는기대 값을 모두 매개 변수로 사용해야되는 경우 CSV파일 형식으로 사용할 수도 있다.

예시:

```java
@ParameterizedTest
@CsvSource({"test,TEST", "tEst,TEST", "Java,JAVA"})
void toUpperCase_ShouldGenerateTheExpectedUppercaseValue(String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

코드 상에 CSV을 적는 대신, `@CsvFileSource`를 통해 CSV 파일을 사용할 수도 있다. 이 Annotation에는 다음의 속성을 가진다.

- `numLinesToSkip`: CSV 파일을 읽을 때 넘어갈 라인의 수. 헤더 라인을 넘길 때 유용하게 사용할 수 있다.

- `lineSeparator`: 라인 구분 기호를 설정할 수 있다.(default: newline)

- `encoding`: 파일 인코딩을 설정할 수 있다.(default: UTF-8)

```java
@ParameterizedTest
@CsvFileSource(resources = "/data.csv", numLinesToSkip = 1)
void toUpperCase_ShouldGenerateTheExpectedUppercaseValueCSVFile(
  String input, String expected) {
    String actualValue = input.toUpperCase();
    assertEquals(expected, actualValue);
}
```

### @MethodSource

메서드를 이용하여 복잡한 인자를 넘길 수 있다.

```java
@ParameterizedTest
@MethodSource("provideStringsForIsBlank")
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input, boolean expected) {
    assertEquals(expected, Strings.isBlank(input));
}

private static Stream<Arguments> provideStringsForIsBlank() {
    return Stream.of(
      Arguments.of(null, true),
      Arguments.of("", true),
      Arguments.of("  ", true),
      Arguments.of("not blank", false)
    );
}
```

`@MethodSource`에 이름을 따로 지정하지 않으면, 테스트 메서드와 같은 이름을 가지는 메서드를 실행한다

```java
@ParameterizedTest
@MethodSource // hmm, no method name ...
void isBlank_ShouldReturnTrueForNullOrBlankStringsOneArgument(String input) {
    assertTrue(Strings.isBlank(input));
}

private static Stream<String> isBlank_ShouldReturnTrueForNullOrBlankStringsOneArgument() {
    return Stream.of(null, "", "  ");
}
```

### @ArgumentsSource

`ArgumentsProvider` 인터페이스를 구현하여 인자를 넘기는 방법도 있다.

```java
class BlankStringsArgumentsProvider implements ArgumentsProvider {

    @Override
    public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
        return Stream.of(
          Arguments.of((String) null), 
          Arguments.of(""), 
          Arguments.of("   ") 
        );
    }
}
```

위와 같이 인터페이스를 구현하였다고 가정한다.

```java
@ParameterizedTest
@ArgumentsSource(BlankStringsArgumentsProvider.class)
void isBlank_ShouldReturnTrueForNullOrBlankStringsArgProvider(String input) {
    assertTrue(Strings.isBlank(input));
}
```

## Argument Accessor

기본적으로 테스트 메서드에 제공되는 각 인자들은 하나의 메서드 매개 변수에 해당한다. 이로인해, 테스트 메서드가 너무 지저분해진다.

`ArgumentsAccessor`의 인스턴스를 통하여, 메서드의 모든 인수를 캡슐화하여 사용할 수 있다.

```java
class Person {

    String firstName;
    String middleName;
    String lastName;
    
    // constructor

    public String fullName() {
        if (middleName == null || middleName.trim().isEmpty()) {
            return String.format("%s %s", firstName, lastName);
        }

        return String.format("%s %s %s", firstName, middleName, lastName);
    }
}

@ParameterizedTest
@CsvSource({"Isaac,,Newton,Isaac Newton", "Charles,Robert,Darwin,Charles Robert Darwin"})
void fullName_ShouldGenerateTheExpectedFullName(ArgumentsAccessor argumentsAccessor) {
    String firstName = argumentsAccessor.getString(0);
    String middleName = (String) argumentsAccessor.get(1);
    String lastName = argumentsAccessor.get(2, String.class);
    String expectedFullName = argumentsAccessor.getString(3);

    Person person = new Person(firstName, middleName, lastName);
    assertEquals(expectedFullName, person.fullName());
}
```

ArgumentsAccessor에 접근할 수 있는 메서드는 다음들이 있다.

- `getString(index)`: `index` 번째에 위치한 요소를 `String`으로 바꾼다.
- `get(index)`: `index` 번째에 위치한 요소를 `Object`로 바꾼다.
- `get(index, type)`: `index` 번째에 위치한 요소를 `type`으로 바꾼다.

## Argument Aggregator

하지만 Argument Accessor를 사용한 것도 코드의 가독성을 떨어뜨릴 수 있다. 이를 위해 `ArgumentsAggregator` 인터페이스를 구현한다.

```java
class PersonAggregator implements ArgumentsAggregator {

    @Override
    public Object aggregateArguments(ArgumentsAccessor accessor, ParameterContext context)
      throws ArgumentsAggregationException {
        return new Person(
          accessor.getString(1), accessor.getString(2), accessor.getString(3));
    }
}

@ParameterizedTest
@CsvSource({"Isaac Newton,Isaac,,Newton", "Charles Robert Darwin,Charles,Robert,Darwin"})
void fullName_ShouldGenerateTheExpectedFullName(
  String expectedFullName,
  @AggregateWith(PersonAggregator.class) Person person) {

    assertEquals(expectedFullName, person.fullName());
}
```

## Display Names 커스터마이징

`@ParameterizedTest`의 `name` 속성을 통해 Display Name을 커스터마이징할 수 있다.

```java
@ParameterizedTest(name = "{index} {0} is 30 days long")
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

아래와 같은 결과가 나온다.

```text
├─ someMonths_Are30DaysLong(Month)
│     │  ├─ 1 APRIL is 30 days long
│     │  ├─ 2 JUNE is 30 days long
│     │  ├─ 3 SEPTEMBER is 30 days long
│     │  └─ 4 NOVEMBER is 30 days long
```

- `{index}`: 호출에대한 인덱스
- `{arguments}`: 쉼표로 구분 된 전체 인수 목록.
- `{0},{1},...`: 개별 인자에 대한 자리 표시자.

## 관련 사이트

<https://www.baeldung.com/parameterized-tests-junit-5>