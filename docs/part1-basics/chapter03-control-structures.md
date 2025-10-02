# Chapter 3: 제어 구조와 표현식

## 학습 목표

- Scala에서 모든 제어 구조가 표현식(값을 반환)임을 이해한다
- `if-else`, `match`, `for`, `while`의 Java와의 차이점을 파악한다
- 패턴 매칭의 기본 개념을 이해한다
- for-comprehension을 활용할 수 있다
- 표현식 지향 프로그래밍 스타일을 익힌다

## 선행 지식

- Chapter 1: Scala 시작하기
- Chapter 2: 변수와 타입 시스템

---

## 3.1 표현식 vs 문장 (Expression vs Statement)

### 3.1.1 핵심 개념

**Java:** 대부분의 제어 구조는 **문장**(값을 반환하지 않음)
**Scala:** 모든 것이 **표현식**(값을 반환)

```scala
// Scala: if는 표현식
val max = if (a > b) a else b

// Java: if는 문장
// int max;
// if (a > b) {
//     max = a;
// } else {
//     max = b;
// }
```

**장점:**
- 코드가 간결해짐
- 중간 변수(`var`) 필요 없음
- 함수형 프로그래밍에 적합

---

## 3.2 if-else 표현식

### 3.2.1 기본 사용법

```scala
val age = 20

// if-else 표현식
val status = if (age >= 18) "Adult" else "Minor"
println(status)  // Adult

// 블록 사용
val grade = if (score >= 90) {
  println("Excellent!")
  "A"
} else if (score >= 80) {
  println("Good!")
  "B"
} else {
  println("Need improvement")
  "C"
}
// 마지막 표현식이 반환값
```

**Java 비교:**
```java
// Java: 삼항 연산자로 유사하게 표현
String status = (age >= 18) ? "Adult" : "Minor";

// 복잡한 경우 if-else 문장 필요
String grade;
if (score >= 90) {
    System.out.println("Excellent!");
    grade = "A";
} else if (score >= 80) {
    System.out.println("Good!");
    grade = "B";
} else {
    System.out.println("Need improvement");
    grade = "C";
}
```

### 3.2.2 else 생략 시 Unit 반환

```scala
// else 생략
val result = if (x > 0) "positive"
// result의 타입: Any (String 또는 Unit)

// 명시적 else
val result2 = if (x > 0) "positive" else ()
// result2의 타입: Any
```

**권장:** 값을 사용할 거라면 항상 `else` 포함

---

## 3.3 match 표현식 (패턴 매칭)

### 3.3.1 기본 match 표현식

```scala
val dayOfWeek = 3

val dayName = dayOfWeek match {
  case 1 => "Monday"
  case 2 => "Tuesday"
  case 3 => "Wednesday"
  case 4 => "Thursday"
  case 5 => "Friday"
  case 6 | 7 => "Weekend"  // OR 패턴
  case _ => "Invalid day"  // 와일드카드 (default)
}

println(dayName)  // Wednesday
```

**Java 비교:**
```java
// Java: switch 문장
String dayName;
switch (dayOfWeek) {
    case 1:
        dayName = "Monday";
        break;
    case 2:
        dayName = "Tuesday";
        break;
    // ...
    case 6:
    case 7:
        dayName = "Weekend";
        break;
    default:
        dayName = "Invalid day";
}

// Java 14+: switch 표현식
String dayName = switch (dayOfWeek) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    case 3 -> "Wednesday";
    case 4 -> "Thursday";
    case 5 -> "Friday";
    case 6, 7 -> "Weekend";
    default -> "Invalid day";
};
```

### 3.3.2 타입 매칭

```scala
def describe(x: Any): String = x match {
  case i: Int => s"Integer: $i"
  case s: String => s"String: $s"
  case d: Double => f"Double: $d%.2f"
  case _ => "Unknown type"
}

println(describe(42))        // Integer: 42
println(describe("Scala"))   // String: Scala
println(describe(3.14159))   // Double: 3.14
```

**Java 비교:**
```java
// Java: instanceof 사용
String describe(Object x) {
    if (x instanceof Integer) {
        return "Integer: " + x;
    } else if (x instanceof String) {
        return "String: " + x;
    } else if (x instanceof Double) {
        return String.format("Double: %.2f", x);
    } else {
        return "Unknown type";
    }
}
```

### 3.3.3 가드 조건 (Guard)

```scala
def categorize(x: Int): String = x match {
  case n if n < 0 => "Negative"
  case 0 => "Zero"
  case n if n > 0 && n < 10 => "Small positive"
  case n if n >= 10 => "Large positive"
}

println(categorize(-5))   // Negative
println(categorize(5))    // Small positive
println(categorize(15))   // Large positive
```

### 3.3.4 케이스 클래스 매칭 (미리보기)

```scala
// 케이스 클래스 정의 (Chapter 5에서 자세히)
sealed trait Shape
case class Circle(radius: Double) extends Shape
case class Rectangle(width: Double, height: Double) extends Shape

def area(shape: Shape): Double = shape match {
  case Circle(r) => Math.PI * r * r
  case Rectangle(w, h) => w * h
}

val circle = Circle(5.0)
val rect = Rectangle(4.0, 6.0)

println(area(circle))   // 78.53...
println(area(rect))     // 24.0
```

---

## 3.4 for 표현식

### 3.4.1 기본 for 루프

```scala
// 범위 순회
for (i <- 1 to 5) {
  println(i)
}
// 출력: 1 2 3 4 5

// until: 끝 값 제외
for (i <- 1 until 5) {
  println(i)
}
// 출력: 1 2 3 4

// 컬렉션 순회
val names = List("Alice", "Bob", "Charlie")
for (name <- names) {
  println(name)
}
```

**Java 비교:**
```java
// Java: for 루프
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}

// Java: for-each
List<String> names = List.of("Alice", "Bob", "Charlie");
for (String name : names) {
    System.out.println(name);
}
```

### 3.4.2 for-comprehension: 값 생성

```scala
// for-yield: 새로운 컬렉션 생성
val numbers = for (i <- 1 to 5) yield i * 2
// numbers: Vector(2, 4, 6, 8, 10)

val names = List("Alice", "Bob", "Charlie")
val uppercased = for (name <- names) yield name.toUpperCase
// uppercased: List(ALICE, BOB, CHARLIE)
```

**Java 비교:**
```java
// Java: Stream 사용
List<Integer> numbers = IntStream.rangeClosed(1, 5)
    .map(i -> i * 2)
    .boxed()
    .collect(Collectors.toList());

List<String> uppercased = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 3.4.3 필터링

```scala
// if 가드
val evenNumbers = for (i <- 1 to 10 if i % 2 == 0) yield i
// evenNumbers: Vector(2, 4, 6, 8, 10)

// 여러 조건
val result = for {
  i <- 1 to 100
  if i % 3 == 0
  if i % 5 == 0
} yield i
// result: 15, 30, 45, 60, 75, 90
```

**Java 비교:**
```java
// Java: Stream filter
List<Integer> evenNumbers = IntStream.rangeClosed(1, 10)
    .filter(i -> i % 2 == 0)
    .boxed()
    .collect(Collectors.toList());

List<Integer> result = IntStream.rangeClosed(1, 100)
    .filter(i -> i % 3 == 0)
    .filter(i -> i % 5 == 0)
    .boxed()
    .collect(Collectors.toList());
```

### 3.4.4 중첩 for

```scala
// 중첩 루프
val pairs = for {
  i <- 1 to 3
  j <- 1 to 3
} yield (i, j)
// pairs: Vector((1,1), (1,2), (1,3), (2,1), (2,2), (2,3), (3,1), (3,2), (3,3))

// 구구단 생성
val multiplicationTable = for {
  i <- 2 to 9
  j <- 1 to 9
} yield s"$i x $j = ${i * j}"
```

**Java 비교:**
```java
// Java: 중첩 for 루프
List<String> pairs = new ArrayList<>();
for (int i = 1; i <= 3; i++) {
    for (int j = 1; j <= 3; j++) {
        pairs.add("(" + i + "," + j + ")");
    }
}

// Java: Stream flatMap
List<String> pairs = IntStream.rangeClosed(1, 3)
    .boxed()
    .flatMap(i -> IntStream.rangeClosed(1, 3)
        .mapToObj(j -> "(" + i + "," + j + ")"))
    .collect(Collectors.toList());
```

### 3.4.5 중간 변수 정의

```scala
val result = for {
  i <- 1 to 10
  square = i * i  // 중간 변수
  if square > 20
} yield square
// result: Vector(25, 36, 49, 64, 81, 100)
```

---

## 3.5 while과 do-while

### 3.5.1 while 루프

```scala
var i = 0
while (i < 5) {
  println(i)
  i += 1
}
// 출력: 0 1 2 3 4
```

**주의:** `while`은 **Unit을 반환**(값이 없음) → 함수형 스타일에서는 피함

**Java와 동일:**
```java
// Java
int i = 0;
while (i < 5) {
    System.out.println(i);
    i++;
}
```

### 3.5.2 do-while 루프

```scala
var count = 0
do {
  println(count)
  count += 1
} while (count < 3)
// 출력: 0 1 2
```

**Java와 동일:**
```java
// Java
int count = 0;
do {
    System.out.println(count);
    count++;
} while (count < 3);
```

### 3.5.3 함수형 대안

```scala
// Bad: while 사용
var sum = 0
var i = 1
while (i <= 10) {
  sum += i
  i += 1
}

// Good: 재귀 또는 고차 함수 사용
val sum = (1 to 10).sum

// 재귀 버전
def sumRange(from: Int, to: Int): Int = {
  if (from > to) 0
  else from + sumRange(from + 1, to)
}
```

---

## 3.6 표현식 지향 프로그래밍

### 3.6.1 명령형 vs 함수형 스타일

**명령형 스타일 (Java 스타일):**
```scala
// Bad: var 사용, 문장 중심
var result = ""
if (score >= 90) {
  result = "A"
} else if (score >= 80) {
  result = "B"
} else {
  result = "C"
}
```

**함수형 스타일 (Scala 스타일):**
```scala
// Good: val 사용, 표현식 중심
val result = if (score >= 90) "A"
             else if (score >= 80) "B"
             else "C"

// 또는 match 사용
val result = score match {
  case s if s >= 90 => "A"
  case s if s >= 80 => "B"
  case _ => "C"
}
```

### 3.6.2 복잡한 로직의 함수형 표현

```scala
// 명령형: 짝수만 필터링하고 제곱
var squares = List[Int]()
for (i <- 1 to 10) {
  if (i % 2 == 0) {
    squares = squares :+ (i * i)
  }
}

// 함수형: for-comprehension
val squares = for {
  i <- 1 to 10
  if i % 2 == 0
} yield i * i

// 또는 고차 함수 (Chapter 8에서 자세히)
val squares = (1 to 10).filter(_ % 2 == 0).map(x => x * x)
```

---

## 3.7 break와 continue

Scala에는 `break`와 `continue`가 **없습니다**.

### 3.7.1 대안: 재귀와 고차 함수

```scala
// Java 스타일 break
// int i = 0;
// while (true) {
//     if (i > 5) break;
//     System.out.println(i);
//     i++;
// }

// Scala 대안 1: takeWhile
(0 to 100).takeWhile(_ <= 5).foreach(println)

// Scala 대안 2: 재귀
def printUntil(i: Int, max: Int): Unit = {
  if (i > max) return  // early return
  println(i)
  printUntil(i + 1, max)
}
printUntil(0, 5)
```

### 3.7.2 scala.util.control.Breaks (비권장)

```scala
import scala.util.control.Breaks._

// break 사용 (함수형 스타일 아님)
breakable {
  for (i <- 1 to 10) {
    if (i > 5) break()  // 탈출
    println(i)
  }
}
// 출력: 1 2 3 4 5
```

**권장:** 고차 함수나 재귀로 해결

---

## 3.8 try-catch-finally 표현식

### 3.8.1 예외 처리

```scala
def divide(a: Int, b: Int): Int = {
  try {
    a / b
  } catch {
    case e: ArithmeticException => {
      println("Cannot divide by zero")
      0
    }
    case e: Exception => {
      println(s"Error: ${e.getMessage}")
      -1
    }
  } finally {
    println("Cleanup")
  }
}

val result = divide(10, 0)
// 출력:
// Cannot divide by zero
// Cleanup
// result = 0
```

**Java 비교:**
```java
// Java
int divide(int a, int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {
        System.out.println("Cannot divide by zero");
        return 0;
    } catch (Exception e) {
        System.out.println("Error: " + e.getMessage());
        return -1;
    } finally {
        System.out.println("Cleanup");
    }
}
```

### 3.8.2 Try 타입 (함수형 예외 처리)

```scala
import scala.util.{Try, Success, Failure}

val result: Try[Int] = Try {
  "123".toInt
}

result match {
  case Success(value) => println(s"Parsed: $value")
  case Failure(exception) => println(s"Failed: ${exception.getMessage}")
}
```

**Java 비교:**
Java에는 Try 타입 없음. Optional 사용:
```java
// Java
Optional<Integer> result = Optional.empty();
try {
    result = Optional.of(Integer.parseInt("123"));
} catch (NumberFormatException e) {
    // 실패 시 비어있음
}
```

---

## 3.9 Scala 3 변경사항

```scala
// Scala 2.12: match
val result = x match {
  case 1 => "one"
  case 2 => "two"
  case _ => "other"
}

// Scala 3: 동일하지만 들여쓰기로 중괄호 생략 가능
val result = x match
  case 1 => "one"
  case 2 => "two"
  case _ => "other"

// Scala 3: if 조건문도 들여쓰기만으로 가능
val status =
  if age >= 18 then "Adult"
  else "Minor"
```

---

## 3.10 실습 과제

### 과제 3-1: match 표현식

숫자를 받아 다음 규칙으로 분류하는 함수를 작성하세요:
- 1~10: "Small"
- 11~100: "Medium"
- 101~1000: "Large"
- 그 외: "Out of range"

```scala
def classify(n: Int): String = ???
```

### 과제 3-2: for-comprehension

1부터 20까지의 수 중에서:
- 3의 배수이면서
- 5의 배수가 아닌
- 수들의 제곱을 리스트로 반환하세요

```scala
val result = for {
  // TODO: 조건 작성
} yield ???
```

### 과제 3-3: 표현식 지향 리팩토링

다음 명령형 코드를 표현식 지향으로 변환하세요:

```scala
// 명령형
var grade = ""
if (score >= 90) {
  grade = "A"
  println("Excellent!")
} else if (score >= 80) {
  grade = "B"
  println("Good!")
} else {
  grade = "C"
  println("Try harder!")
}

// TODO: 함수형으로 변환
val grade = ???
```

---

## 3.11 요약

이 챕터에서 배운 내용:

✅ Scala의 모든 제어 구조는 표현식(값을 반환)
✅ `if-else`를 값으로 사용하기
✅ `match` 표현식을 통한 강력한 패턴 매칭
✅ `for-comprehension`으로 컬렉션 변환
✅ `while`은 Unit 반환 → 함수형 스타일에서 피함
✅ `break`/`continue` 없음 → 고차 함수나 재귀 사용
✅ 표현식 지향 프로그래밍으로 `var` 최소화

**다음 챕터 예고:**
Chapter 4에서는 함수의 정의, 매개변수, 반환 타입 등 함수 기초를 다룹니다.

---

## 참고 자료

- [Scala Control Structures](https://docs.scala-lang.org/tour/basics.html)
- [Pattern Matching](https://docs.scala-lang.org/tour/pattern-matching.html)
- [For Comprehensions](https://docs.scala-lang.org/tour/for-comprehensions.html)
