# Chapter 2: 변수와 타입 시스템

## 학습 목표

- `val`과 `var`의 차이를 이해하고 적절히 사용할 수 있다
- Scala의 기본 타입과 Java 타입의 관계를 이해한다
- 타입 추론의 원리와 한계를 파악한다
- 문자열 인터폴레이션을 활용할 수 있다
- 타입 별칭(Type Alias)과 타입 파라미터를 이해한다

## 선행 지식

- Chapter 1: Scala 시작하기

---

## 2.1 변수 선언: val vs var

### 2.1.1 val: 불변 변수 (Immutable)

```scala
val pi = 3.14159
val name = "Scala"
val count = 42

// 재할당 불가능
// pi = 3.14  // 컴파일 오류: reassignment to val
```

**Java 비교:**
```java
// Java
final double pi = 3.14159;
final String name = "Java";
final int count = 42;

// pi = 3.14;  // 컴파일 오류
```

**핵심:** `val`은 Java의 `final` 변수와 동일하지만, Scala에서는 **기본 선택**입니다.

### 2.1.2 var: 가변 변수 (Mutable)

```scala
var score = 0
score = 10       // OK
score = score + 5  // OK

var message = "Hello"
message = "Hi"   // OK
```

**Java 비교:**
```java
// Java
int score = 0;
score = 10;
score = score + 5;

String message = "Hello";
message = "Hi";
```

### 2.1.3 val vs var 선택 기준

**권장 순서:**
1. **기본적으로 `val` 사용** (불변성)
2. 값이 변경되어야 한다면 `var` 사용
3. `var`가 많아지면 설계 재검토

**예시: 함수형 스타일 (권장)**
```scala
// Good: val 사용
val numbers = List(1, 2, 3, 4, 5)
val doubled = numbers.map(_ * 2)
```

**명령형 스타일 (비권장)**
```scala
// Bad: var 사용
var result = List[Int]()
for (i <- 1 to 5) {
  result = result :+ (i * 2)
}
```

---

## 2.2 Scala의 기본 타입

### 2.2.1 숫자 타입

| Scala 타입 | Java 타입 | 크기 | 범위 |
|-----------|-----------|------|------|
| `Byte` | `byte` | 8bit | -128 ~ 127 |
| `Short` | `short` | 16bit | -32,768 ~ 32,767 |
| `Int` | `int` | 32bit | -2^31 ~ 2^31-1 |
| `Long` | `long` | 64bit | -2^63 ~ 2^63-1 |
| `Float` | `float` | 32bit | IEEE 754 single |
| `Double` | `double` | 64bit | IEEE 754 double |

```scala
val byteVal: Byte = 127
val shortVal: Short = 32000
val intVal: Int = 2147483647
val longVal: Long = 9223372036854775807L  // L 접미사

val floatVal: Float = 3.14f  // f 접미사
val doubleVal: Double = 3.14159
```

**Java 비교:**
```java
// Java: 기본 타입은 소문자
byte byteVal = 127;
short shortVal = 32000;
int intVal = 2147483647;
long longVal = 9223372036854775807L;

float floatVal = 3.14f;
double doubleVal = 3.14159;
```

**중요한 차이점:**
- Scala의 모든 타입은 **객체** (Java의 Primitive가 없음)
- 하지만 컴파일 시 성능을 위해 Java Primitive로 변환됨

### 2.2.2 Boolean과 Char

```scala
val isScala: Boolean = true
val firstLetter: Char = 'S'

// Boolean 연산
val result = (5 > 3) && (10 < 20)  // true
val negation = !result              // false
```

**Java와 동일:**
```java
// Java
boolean isJava = true;
char firstLetter = 'J';

boolean result = (5 > 3) && (10 < 20);
boolean negation = !result;
```

### 2.2.3 String

```scala
val greeting: String = "Hello, Scala"
val multiline = """
  |This is a
  |multiline
  |string
  """.stripMargin

// 문자열 연산
val name = "Scala"
val version = "2.12"
val fullName = name + " " + version  // "Scala 2.12"
```

**Java 비교:**
```java
// Java
String greeting = "Hello, Java";

// Java 15+ Text Blocks
String multiline = """
    This is a
    multiline
    string
    """;

String fullName = name + " " + version;
```

---

## 2.3 타입 추론 (Type Inference)

### 2.3.1 기본 타입 추론

```scala
// 타입 명시 생략
val x = 42                  // Int로 추론
val y = 3.14                // Double로 추론
val name = "Scala"          // String으로 추론
val isReady = true          // Boolean으로 추론

// 컬렉션 타입 추론
val numbers = List(1, 2, 3)           // List[Int]
val mixed = List(1, "two", 3.0)       // List[Any]
val pairs = List((1, "one"), (2, "two"))  // List[(Int, String)]
```

### 2.3.2 타입 명시가 필요한 경우

```scala
// 1. 재귀 함수 반환 타입
def factorial(n: Int): Int = {  // 반환 타입 필수
  if (n <= 1) 1
  else n * factorial(n - 1)
}

// 2. 공개 API 메서드
class Calculator {
  def add(a: Int, b: Int): Int = a + b  // 반환 타입 명시 권장
}

// 3. 복잡한 제네릭 타입
val complexMap: Map[String, List[Int]] = Map(
  "numbers" -> List(1, 2, 3)
)
```

### 2.3.3 타입 확인: :type 명령

```scala
scala> :type 42
Int

scala> :type "Hello"
String

scala> :type List(1, 2, 3)
List[Int]

scala> :type (x: Int) => x * 2
Int => Int
```

---

## 2.4 문자열 인터폴레이션 (String Interpolation)

### 2.4.1 s-interpolator: 변수 삽입

```scala
val name = "Scala"
val version = 2.12

// s-interpolator
val message = s"$name version $version"
// "Scala version 2.12"

// 표현식 삽입
val result = s"5 + 3 = ${5 + 3}"
// "5 + 3 = 8"
```

**Java 비교:**
```java
// Java
String name = "Java";
int version = 11;

// Java 15+ 이전
String message = String.format("%s version %d", name, version);

// Java 15+
String message = name + " version " + version;
```

### 2.4.2 f-interpolator: 포맷 지정

```scala
val pi = 3.14159

// 소수점 2자리
val formatted = f"Pi is approximately $pi%.2f"
// "Pi is approximately 3.14"

// 정수 포맷
val count = 42
val padded = f"Count: $count%05d"
// "Count: 00042"
```

**Java 비교:**
```java
// Java
double pi = 3.14159;
String formatted = String.format("Pi is approximately %.2f", pi);

int count = 42;
String padded = String.format("Count: %05d", count);
```

### 2.4.3 raw-interpolator: 이스케이프 무시

```scala
// s-interpolator: \n이 개행으로 처리
val s = s"Line1\nLine2"
// Line1
// Line2

// raw-interpolator: \n이 그대로 출력
val raw = raw"Line1\nLine2"
// Line1\nLine2
```

---

## 2.5 타입 변환 (Type Conversion)

### 2.5.1 자동 타입 승격 (Widening)

```scala
val intVal: Int = 42
val longVal: Long = intVal     // Int -> Long 자동 변환
val doubleVal: Double = intVal  // Int -> Double 자동 변환
```

**Java와 동일:**
```java
// Java
int intVal = 42;
long longVal = intVal;
double doubleVal = intVal;
```

### 2.5.2 명시적 타입 변환

```scala
val doubleVal = 3.14
val intVal = doubleVal.toInt      // 3 (소수점 버림)

val stringVal = "123"
val number = stringVal.toInt      // 123

// 변환 메서드들
val x = 42
x.toDouble   // 42.0
x.toString   // "42"
x.toByte     // 42: Byte
x.toLong     // 42L
```

**Java 비교:**
```java
// Java
double doubleVal = 3.14;
int intVal = (int) doubleVal;  // 명시적 캐스팅

String stringVal = "123";
int number = Integer.parseInt(stringVal);

// Wrapper 클래스 사용
int x = 42;
double d = (double) x;
String s = String.valueOf(x);
```

---

## 2.6 Unit, Null, Nothing, Any

### 2.6.1 Unit: 값이 없음

```scala
// Unit: 의미 있는 값을 반환하지 않는 함수
def printMessage(msg: String): Unit = {
  println(msg)
}

val result: Unit = printMessage("Hello")
// result는 () 값을 가짐
```

**Java 비교:**
```java
// Java: void
public void printMessage(String msg) {
    System.out.println(msg);
}
```

### 2.6.2 Null과 null

```scala
// null 사용 (비권장)
val s: String = null  // 가능하지만 피해야 함

// 권장: Option 사용
val optionalName: Option[String] = None
val presentName: Option[String] = Some("Scala")
```

**Java 비교:**
```java
// Java: null 사용
String s = null;

// Java 8+: Optional
Optional<String> optionalName = Optional.empty();
Optional<String> presentName = Optional.of("Java");
```

### 2.6.3 타입 계층 구조

```
          Any
         /   \
    AnyVal  AnyRef (= Java Object)
      |        |
   Int, Double, Boolean...  String, List, 등 모든 참조 타입
      |        |
    Nothing  Null
```

```scala
val anyInt: Any = 42
val anyString: Any = "Hello"
val anyList: Any = List(1, 2, 3)

// Nothing: 모든 타입의 하위 타입
def error(msg: String): Nothing = {
  throw new Exception(msg)
}
```

---

## 2.7 타입 별칭 (Type Alias)

### 2.7.1 기본 타입 별칭

```scala
type UserId = Int
type UserName = String

val id: UserId = 123
val name: UserName = "Alice"

// 가독성 향상
type Point = (Double, Double)
val origin: Point = (0.0, 0.0)
```

### 2.7.2 복잡한 타입 별칭

```scala
type UserMap = Map[String, List[Int]]

val userScores: UserMap = Map(
  "Alice" -> List(95, 88, 92),
  "Bob" -> List(78, 85, 90)
)

// 함수 타입 별칭
type IntTransformer = Int => Int

val doubler: IntTransformer = x => x * 2
val tripler: IntTransformer = x => x * 3
```

**Java 비교:**
Java에는 타입 별칭이 없음. 대신 Wrapper 클래스나 주석 사용:
```java
// Java: 타입 별칭 없음
// @UserId  // 커스텀 어노테이션으로 표현
// int id = 123;

Map<String, List<Integer>> userScores = new HashMap<>();
```

---

## 2.8 Lazy Evaluation

### 2.8.1 lazy val

```scala
// 일반 val: 즉시 평가
val eagerVal = {
  println("Evaluating eagerVal")
  42
}
// 출력: Evaluating eagerVal

// lazy val: 필요 시 평가
lazy val lazyVal = {
  println("Evaluating lazyVal")
  42
}
// 출력 없음 (아직 평가 안 됨)

println(lazyVal)  // 여기서 평가
// 출력:
// Evaluating lazyVal
// 42

println(lazyVal)  // 이미 평가됨, 재평가 안 함
// 출력: 42
```

**사용 사례:**
```scala
class DatabaseConnection {
  // 실제로 사용할 때만 연결
  lazy val connection = {
    println("Connecting to database...")
    // 실제 DB 연결 로직
    "DB Connection"
  }
}

val db = new DatabaseConnection()
// 아직 연결 안 됨

db.connection  // 여기서 연결
```

**Java 비교:**
Java는 lazy 키워드 없음. 패턴으로 구현:
```java
// Java: Lazy Initialization 패턴
class DatabaseConnection {
    private String connection;

    public String getConnection() {
        if (connection == null) {
            System.out.println("Connecting to database...");
            connection = "DB Connection";
        }
        return connection;
    }
}
```

---

## 2.9 Scala 3 변경사항

```scala
// Scala 2.12: 별도 변경 없음
val x = 42
var y = 10

// Scala 3: 동일하지만 개선된 타입 추론
// Scala 3에서는 더 복잡한 경우도 타입 추론 가능
```

**주요 차이:**
- Scala 3의 타입 추론이 더 강력함
- `given`/`using`으로 implicit 개선 (다른 챕터에서 다룸)

---

## 2.10 실습 과제

### 과제 2-1: 타입 변환 연습

다음 코드의 출력을 예측하고, REPL에서 확인하세요:

```scala
val x = 10
val y = 3.5
val sum = x + y
println(sum)         // ?
println(sum.getClass)  // ?

val intDiv = x / 3
val doubleDiv = x / 3.0
println(intDiv)      // ?
println(doubleDiv)   // ?
```

### 과제 2-2: 문자열 인터폴레이션

BMI 계산기를 작성하세요:

```scala
object BMICalculator extends App {
  val name = "Alice"
  val weight = 70.0  // kg
  val height = 1.75  // m

  val bmi = weight / (height * height)

  // TODO: s-interpolator와 f-interpolator를 사용하여
  // "Alice's BMI is 22.86" 형식으로 출력하세요
  // BMI는 소수점 2자리까지 표시
}
```

### 과제 2-3: lazy val 실험

다음 코드를 실행하고 출력 순서를 설명하세요:

```scala
object LazyTest extends App {
  val eager = {
    println("Eager evaluated")
    42
  }

  lazy val lazyVal = {
    println("Lazy evaluated")
    100
  }

  println("Starting main")
  println(s"Eager: $eager")
  println(s"Lazy: $lazyVal")
  println(s"Lazy again: $lazyVal")
}
```

---

## 2.11 요약

이 챕터에서 배운 내용:

✅ `val`(불변)과 `var`(가변)의 차이와 선택 기준
✅ Scala의 기본 타입과 Java와의 관계
✅ 타입 추론의 원리와 명시가 필요한 경우
✅ 문자열 인터폴레이션 (s, f, raw)
✅ 타입 변환과 계층 구조 (Any, AnyVal, AnyRef)
✅ 타입 별칭으로 가독성 향상
✅ `lazy val`을 통한 지연 평가

**다음 챕터 예고:**
Chapter 3에서는 Scala의 제어 구조와 표현식을 다룹니다. `if`, `match`, `for` 등이 Java와 어떻게 다른지 배웁니다.

---

## 참고 자료

- [Scala Type System](https://docs.scala-lang.org/tour/unified-types.html)
- [String Interpolation](https://docs.scala-lang.org/overviews/core/string-interpolation.html)
