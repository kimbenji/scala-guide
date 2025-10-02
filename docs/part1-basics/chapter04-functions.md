# Chapter 4: 함수 기초

## 학습 목표

- 메서드와 함수의 차이를 이해한다
- 다양한 형태의 함수 정의 방법을 익힌다
- 매개변수(기본값, 가변인자, 명명된 인자)를 활용할 수 있다
- 고차 함수의 기본 개념을 이해한다
- 함수형 프로그래밍의 기초를 다진다

## 선행 지식

- Chapter 1-3: Scala 기본 문법

---

## 4.1 메서드 정의

### 4.1.1 기본 메서드

```scala
// 기본 형식
def add(a: Int, b: Int): Int = {
  a + b
}

// 한 줄이면 중괄호 생략
def multiply(a: Int, b: Int): Int = a * b

// 반환 타입 추론 (비권장: 공개 API에서는 명시)
def subtract(a: Int, b: Int) = a - b

println(add(3, 5))        // 8
println(multiply(4, 7))   // 28
```

**Java 비교:**
```java
// Java
public static int add(int a, int b) {
    return a + b;
}

public static int multiply(int a, int b) {
    return a * b;
}
```

### 4.1.2 Unit 반환 (값이 없는 메서드)

```scala
def printSum(a: Int, b: Int): Unit = {
  println(s"Sum: ${a + b}")
}

// return 생략 (마지막 표현식이 Unit)
def greet(name: String): Unit = {
  println(s"Hello, $name!")
}

printSum(3, 5)     // Sum: 8
greet("Scala")     // Hello, Scala!
```

**Java 비교:**
```java
// Java: void
public static void printSum(int a, int b) {
    System.out.println("Sum: " + (a + b));
}
```

### 4.1.3 매개변수 없는 메서드

```scala
// 괄호 있음
def getCurrentTime(): Long = System.currentTimeMillis()

// 괄호 없음 (부수 효과 없을 때 권장)
def pi: Double = 3.14159

println(getCurrentTime())  // 1234567890123
println(pi)                // 3.14159
```

**컨벤션:**
- 부수 효과 있으면 괄호 사용 (`getCurrentTime()`)
- 부수 효과 없으면 괄호 생략 (`pi`)

---

## 4.2 함수 (Function) vs 메서드 (Method)

### 4.2.1 메서드: 클래스/객체에 속함

```scala
object Calculator {
  def add(a: Int, b: Int): Int = a + b  // 메서드
}

Calculator.add(3, 5)  // 8
```

### 4.2.2 함수: 일급 객체 (First-Class)

```scala
// 함수 값 (Function Value)
val addFunc: (Int, Int) => Int = (a, b) => a + b

// 함수 호출
println(addFunc(3, 5))  // 8

// 함수를 변수에 할당
val f = addFunc
println(f(10, 20))      // 30
```

**Java 비교:**
```java
// Java 8+: 람다와 함수형 인터페이스
BiFunction<Integer, Integer, Integer> addFunc = (a, b) -> a + b;
System.out.println(addFunc.apply(3, 5));  // 8
```

### 4.2.3 메서드를 함수로 변환 (Eta Expansion)

```scala
def addMethod(a: Int, b: Int): Int = a + b

// 메서드를 함수로 변환
val addFunc = addMethod _  // 언더스코어로 변환

// 또는 자동 변환 (컨텍스트에 따라)
val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(addMethod)  // 컴파일 오류: 메서드는 직접 전달 불가
// val doubled = numbers.map(addMethod _)  // OK: 함수로 변환
```

---

## 4.3 매개변수

### 4.3.1 기본 매개변수 (Default Parameters)

```scala
def greet(name: String, greeting: String = "Hello"): String = {
  s"$greeting, $name!"
}

println(greet("Alice"))             // Hello, Alice!
println(greet("Bob", "Hi"))         // Hi, Bob!
println(greet("Charlie", "Hey"))    // Hey, Charlie!
```

**Java 비교:**
Java는 기본 매개변수 없음. 오버로딩으로 구현:
```java
// Java: 메서드 오버로딩
String greet(String name) {
    return greet(name, "Hello");
}

String greet(String name, String greeting) {
    return greeting + ", " + name + "!";
}
```

### 4.3.2 명명된 인자 (Named Arguments)

```scala
def createUser(name: String, age: Int, city: String): String = {
  s"$name, $age years old, from $city"
}

// 위치 기반
createUser("Alice", 30, "Seoul")

// 명명된 인자
createUser(name = "Bob", age = 25, city = "Busan")

// 순서 변경 가능
createUser(city = "Incheon", name = "Charlie", age = 35)

// 혼합 사용
createUser("David", age = 40, city = "Daegu")
```

**Java 비교:**
Java는 명명된 인자 없음. Builder 패턴으로 대체:
```java
// Java: Builder 패턴
User user = new UserBuilder()
    .name("Alice")
    .age(30)
    .city("Seoul")
    .build();
```

### 4.3.3 가변 인자 (Variable Arguments)

```scala
def sum(numbers: Int*): Int = {
  numbers.sum
}

println(sum(1, 2, 3))          // 6
println(sum(1, 2, 3, 4, 5))    // 15
println(sum())                 // 0

// 시퀀스를 가변 인자로 전달
val nums = List(1, 2, 3, 4, 5)
println(sum(nums: _*))  // 15 (언더스코어 필요)
```

**Java 비교:**
```java
// Java: varargs
int sum(int... numbers) {
    return Arrays.stream(numbers).sum();
}

sum(1, 2, 3);          // 6
sum(1, 2, 3, 4, 5);    // 15
```

---

## 4.4 반환 값

### 4.4.1 마지막 표현식이 반환값

```scala
def max(a: Int, b: Int): Int = {
  if (a > b) a else b  // 마지막 표현식
}

def abs(x: Int): Int = {
  val result = if (x < 0) -x else x
  result  // 마지막 표현식
}
```

### 4.4.2 명시적 return (비권장)

```scala
// 명시적 return (비권장)
def findFirst(numbers: List[Int], target: Int): Int = {
  for (i <- numbers.indices) {
    if (numbers(i) == target) {
      return i  // early return
    }
  }
  -1  // not found
}

// 함수형 스타일 (권장)
def findFirstFunctional(numbers: List[Int], target: Int): Int = {
  numbers.indexOf(target)
}
```

**주의:** `return`은 함수의 흐름을 복잡하게 만듦 → 표현식으로 해결

---

## 4.5 재귀 함수

### 4.5.1 기본 재귀

```scala
def factorial(n: Int): Int = {
  if (n <= 1) 1
  else n * factorial(n - 1)
}

println(factorial(5))  // 120

// 패턴 매칭 버전
def factorialMatch(n: Int): Int = n match {
  case 0 | 1 => 1
  case _ => n * factorialMatch(n - 1)
}
```

**Java 비교:**
```java
// Java
int factorial(int n) {
    if (n <= 1) return 1;
    else return n * factorial(n - 1);
}
```

### 4.5.2 꼬리 재귀 (Tail Recursion)

```scala
import scala.annotation.tailrec

// 일반 재귀: 스택 오버플로우 위험
def factorial(n: Int): Int = {
  if (n <= 1) 1
  else n * factorial(n - 1)
}

// 꼬리 재귀: 최적화 가능
@tailrec
def factorialTail(n: Int, acc: Int = 1): Int = {
  if (n <= 1) acc
  else factorialTail(n - 1, n * acc)
}

println(factorialTail(5))       // 120
println(factorialTail(10000))   // 스택 오버플로우 없음
```

**@tailrec 어노테이션:**
- 꼬리 재귀가 아니면 컴파일 오류
- 컴파일러가 while 루프로 최적화

**Java 비교:**
Java는 꼬리 재귀 최적화 없음:
```java
// Java: 꼬리 재귀 최적화 안 됨
int factorialTail(int n, int acc) {
    if (n <= 1) return acc;
    else return factorialTail(n - 1, n * acc);
}
```

---

## 4.6 고차 함수 (Higher-Order Functions)

### 4.6.1 함수를 매개변수로 받기

```scala
// 함수를 인자로 받는 함수
def applyTwice(f: Int => Int, x: Int): Int = {
  f(f(x))
}

def double(x: Int): Int = x * 2

println(applyTwice(double, 3))  // double(double(3)) = 12

// 익명 함수 (람다)
println(applyTwice(x => x + 1, 5))  // 7
```

**Java 비교:**
```java
// Java 8+: Function 인터페이스
int applyTwice(Function<Integer, Integer> f, int x) {
    return f.apply(f.apply(x));
}

applyTwice(x -> x * 2, 3);   // 12
applyTwice(x -> x + 1, 5);   // 7
```

### 4.6.2 함수를 반환하기

```scala
// 함수를 반환하는 함수
def multiplier(factor: Int): Int => Int = {
  (x: Int) => x * factor
}

val double = multiplier(2)
val triple = multiplier(3)

println(double(5))  // 10
println(triple(5))  // 15

// 간결한 형태
def multiplierShort(factor: Int): Int => Int = _ * factor
```

**Java 비교:**
```java
// Java 8+
Function<Integer, Integer> multiplier(int factor) {
    return x -> x * factor;
}

Function<Integer, Integer> doubler = multiplier(2);
System.out.println(doubler.apply(5));  // 10
```

---

## 4.7 익명 함수 (람다)

### 4.7.1 기본 형태

```scala
// 전체 형태
val add = (a: Int, b: Int) => a + b

// 타입 추론 가능하면 생략
val numbers = List(1, 2, 3, 4, 5)
numbers.map((x: Int) => x * 2)    // List(2, 4, 6, 8, 10)
numbers.map(x => x * 2)           // 타입 추론
numbers.map(_ * 2)                // 플레이스홀더
```

**Java 비교:**
```java
// Java 8+
List<Integer> numbers = List.of(1, 2, 3, 4, 5);
numbers.stream().map(x -> x * 2).collect(Collectors.toList());
```

### 4.7.2 플레이스홀더 (_)

```scala
val numbers = List(1, 2, 3, 4, 5)

// 한 번만 사용하는 매개변수
numbers.map(_ * 2)           // x => x * 2
numbers.filter(_ > 2)        // x => x > 2

// 여러 매개변수
val pairs = List((1, 2), (3, 4), (5, 6))
pairs.map(_ + _)             // (a, b) => a + b
// List(3, 7, 11)

// 재사용하면 플레이스홀더 불가
// numbers.map(_ + _)  // 컴파일 오류: 각 _는 다른 매개변수
numbers.map(x => x + x)  // OK
```

---

## 4.8 커링 (Currying)

### 4.8.1 다중 매개변수 리스트

```scala
// 일반 함수
def add(a: Int, b: Int): Int = a + b

// 커링된 함수
def addCurried(a: Int)(b: Int): Int = a + b

println(add(3, 5))            // 8
println(addCurried(3)(5))     // 8

// 부분 적용 (Partial Application)
val add3 = addCurried(3) _    // Int => Int
println(add3(5))              // 8
println(add3(10))             // 13
```

**Java 비교:**
Java는 커링 미지원. 함수를 반환하는 함수로 구현:
```java
// Java
Function<Integer, Function<Integer, Integer>> addCurried =
    a -> b -> a + b;

Function<Integer, Integer> add3 = addCurried.apply(3);
System.out.println(add3.apply(5));   // 8
```

### 4.8.2 커링의 활용

```scala
// DSL 스타일 API
def withResource[T](resource: String)(action: String => T): T = {
  println(s"Opening $resource")
  val result = action(resource)
  println(s"Closing $resource")
  result
}

withResource("database.db") { db =>
  println(s"Using $db")
  42
}
// 출력:
// Opening database.db
// Using database.db
// Closing database.db
```

---

## 4.9 Scala 3 변경사항

```scala
// Scala 2.12
def add(a: Int, b: Int): Int = a + b

// Scala 3: 동일하지만 새로운 문법도 가능
def add(a: Int, b: Int): Int = a + b

// Scala 3: 중괄호 대신 들여쓰기
def factorial(n: Int): Int =
  if n <= 1 then 1
  else n * factorial(n - 1)
```

---

## 4.10 실습 과제

### 과제 4-1: 재귀 함수

피보나치 수를 계산하는 두 가지 버전을 작성하세요:
1. 일반 재귀
2. 꼬리 재귀

```scala
def fibonacci(n: Int): Int = ???

@tailrec
def fibonacciTail(n: Int, a: Int = 0, b: Int = 1): Int = ???
```

### 과제 4-2: 고차 함수

다음 고차 함수를 구현하세요:

```scala
// 리스트의 모든 요소에 함수 적용
def mapList(list: List[Int], f: Int => Int): List[Int] = ???

// 조건을 만족하는 요소만 필터링
def filterList(list: List[Int], predicate: Int => Boolean): List[Int] = ???

// 테스트
val numbers = List(1, 2, 3, 4, 5)
println(mapList(numbers, _ * 2))       // List(2, 4, 6, 8, 10)
println(filterList(numbers, _ > 2))    // List(3, 4, 5)
```

### 과제 4-3: 커링

섭씨를 화씨로 변환하는 커링된 함수를 작성하세요:

```scala
def convertTemperature(conversionType: String)(value: Double): Double = ???

val celsiusToFahrenheit = convertTemperature("C2F") _
val fahrenheitToCelsius = convertTemperature("F2C") _

println(celsiusToFahrenheit(0))   // 32.0
println(fahrenheitToCelsius(32))  // 0.0
```

---

## 4.11 요약

이 챕터에서 배운 내용:

✅ 메서드 정의와 다양한 형태
✅ 메서드 vs 함수 (일급 객체)
✅ 매개변수 (기본값, 명명된 인자, 가변 인자)
✅ 재귀와 꼬리 재귀 최적화
✅ 고차 함수 (함수를 인자/반환값으로)
✅ 익명 함수(람다)와 플레이스홀더
✅ 커링과 부분 적용

**다음 챕터 예고:**
Chapter 5에서는 Scala의 객체지향 프로그래밍(클래스, 트레이트, 케이스 클래스)을 다룹니다.

---

## 참고 자료

- [Scala Functions](https://docs.scala-lang.org/tour/basics.html)
- [Higher-Order Functions](https://docs.scala-lang.org/tour/higher-order-functions.html)
