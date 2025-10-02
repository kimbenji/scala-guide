# Chapter 7: 고급 함수형 프로그래밍 (Advanced Functional Programming)

## 학습 목표
- 순수 함수와 참조 투명성의 개념과 실무 적용 방법 이해
- 함수 합성을 통한 코드 재사용성 향상
- 부분 함수(PartialFunction)의 활용법 습득
- 재귀 패턴 심화 및 최적화 기법 학습

---

## 7.1 순수 함수와 참조 투명성 (Pure Functions and Referential Transparency)

### 7.1.1 순수 함수의 정의

**순수 함수(Pure Function)**는 다음 두 가지 특성을 만족하는 함수입니다:
1. **동일한 입력에 대해 항상 동일한 출력**을 반환
2. **부수 효과(Side Effect)가 없음** (외부 상태 변경, I/O 작업 등 금지)

```scala
// 순수 함수 예제
def add(a: Int, b: Int): Int = a + b

def multiply(x: Int, y: Int): Int = x * y

def isEven(n: Int): Boolean = n % 2 == 0
```

```java
// Java 비교: 순수 함수와 비순수 함수
public class PureFunctionExample {
    // 순수 함수
    public static int add(int a, int b) {
        return a + b;
    }

    // 비순수 함수 (외부 상태 변경)
    private static int counter = 0;
    public static int incrementAndGet() {
        return ++counter;  // 부수 효과 발생
    }
}
```

### 7.1.2 참조 투명성 (Referential Transparency)

**참조 투명성**이란 표현식을 그 결과값으로 치환해도 프로그램의 동작이 변하지 않는 성질입니다.

```scala
// 참조 투명한 코드
val x = add(2, 3)
val y = x + x  // 10

// 표현식 치환
val y2 = add(2, 3) + add(2, 3)  // 여전히 10

// 참조 투명하지 않은 코드
var counter = 0
def increment(): Int = {
  counter += 1  // 부수 효과
  counter
}

val a = increment()  // 1
val b = a + a        // 2
val c = increment() + increment()  // 5 (다른 결과!)
```

### 7.1.3 순수 함수의 장점

```scala
// 예제: 사용자 데이터 처리
case class User(id: Int, name: String, age: Int)

// ❌ 비순수 함수 (로깅이라는 부수 효과)
def validateUserImpure(user: User): Boolean = {
  println(s"Validating user: ${user.name}")  // 부수 효과
  user.age >= 18
}

// ✅ 순수 함수 (부수 효과 분리)
def validateUser(user: User): Boolean = {
  user.age >= 18
}

// 부수 효과는 별도로 처리
def logValidation(user: User, result: Boolean): Unit = {
  println(s"User ${user.name} validation: $result")
}

// 사용
val user = User(1, "Alice", 25)
val isValid = validateUser(user)
logValidation(user, isValid)
```

**순수 함수의 이점**:
- **테스트 용이성**: Mock 불필요, 입출력만 검증
- **병렬 처리 안전성**: 상태 공유 없어 Race Condition 회피
- **캐싱 가능**: 메모이제이션(Memoization) 적용 가능
- **추론 용이성**: 함수 시그니처만으로 동작 예측 가능

```scala
// 순수 함수의 테스트
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

class UserValidationSpec extends AnyFlatSpec with Matchers {
  "validateUser" should "return true for adult users" in {
    validateUser(User(1, "Alice", 25)) shouldBe true
  }

  it should "return false for minor users" in {
    validateUser(User(2, "Bob", 17)) shouldBe false
  }
}
```

---

## 7.2 함수 합성 (Function Composition)

### 7.2.1 compose와 andThen

Scala의 함수는 `compose`와 `andThen` 메서드를 통해 합성할 수 있습니다.

```scala
// compose: f compose g = f(g(x))
val addOne: Int => Int = _ + 1
val double: Int => Int = _ * 2

val doubleThenAddOne = addOne compose double
println(doubleThenAddOne(3))  // (3 * 2) + 1 = 7

// andThen: f andThen g = g(f(x))
val addOneThenDouble = addOne andThen double
println(addOneThenDouble(3))  // (3 + 1) * 2 = 8
```

```java
// Java 8 Function 인터페이스의 compose, andThen
import java.util.function.Function;

public class CompositionExample {
    public static void main(String[] args) {
        Function<Integer, Integer> addOne = x -> x + 1;
        Function<Integer, Integer> doubleIt = x -> x * 2;

        // compose: f.compose(g) = f(g(x))
        Function<Integer, Integer> doubleThenAddOne = addOne.compose(doubleIt);
        System.out.println(doubleThenAddOne.apply(3));  // 7

        // andThen: f.andThen(g) = g(f(x))
        Function<Integer, Integer> addOneThenDouble = addOne.andThen(doubleIt);
        System.out.println(addOneThenDouble.apply(3));  // 8
    }
}
```

### 7.2.2 실전 예제: 데이터 변환 파이프라인

```scala
// 문자열 처리 파이프라인
val trim: String => String = _.trim
val toLowerCase: String => String = _.toLowerCase
val removeSpaces: String => String = _.replaceAll("\\s+", "")

val normalize = trim andThen toLowerCase andThen removeSpaces

println(normalize("  Hello World  "))  // "helloworld"

// 리스트 처리 파이프라인
case class Product(id: Int, name: String, price: Double, category: String)

val products = List(
  Product(1, "Laptop", 1200.0, "Electronics"),
  Product(2, "Mouse", 25.0, "Electronics"),
  Product(3, "Desk", 300.0, "Furniture"),
  Product(4, "Chair", 150.0, "Furniture")
)

// 함수 정의
val filterElectronics: List[Product] => List[Product] =
  _.filter(_.category == "Electronics")

val sortByPrice: List[Product] => List[Product] =
  _.sortBy(_.price)

val extractNames: List[Product] => List[String] =
  _.map(_.name)

// 파이프라인 합성
val pipeline = filterElectronics andThen sortByPrice andThen extractNames

println(pipeline(products))  // List("Mouse", "Laptop")
```

### 7.2.3 커스텀 합성 연산자

```scala
// 직관적인 파이프라인 연산자 정의
implicit class PipelineOps[A](val value: A) extends AnyVal {
  def |>[B](f: A => B): B = f(value)
}

// 사용 예제
val result = "  Hello World  " |> trim |> toLowerCase |> removeSpaces
println(result)  // "helloworld"

// 복잡한 파이프라인
val salesData = List(
  ("Laptop", 1200.0, 5),
  ("Mouse", 25.0, 20),
  ("Keyboard", 75.0, 15)
)

val totalRevenue = salesData
  |> (_.map { case (name, price, qty) => price * qty })
  |> (_.sum)

println(s"Total Revenue: $$${totalRevenue}")  // Total Revenue: $8000.0
```

// Scala 3 차이점: Extension Methods 문법
/*
Scala 3에서는 implicit class 대신 extension을 사용합니다:

extension [A](value: A)
  def |>[B](f: A => B): B = f(value)
*/

---

## 7.3 부분 함수 (PartialFunction)

### 7.3.1 부분 함수의 개념

**부분 함수(PartialFunction)**는 입력 도메인의 일부에서만 정의된 함수입니다.

```scala
// 부분 함수 정의
val divide: PartialFunction[(Int, Int), Int] = {
  case (a, b) if b != 0 => a / b
}

// 정의 여부 확인
println(divide.isDefinedAt((10, 2)))  // true
println(divide.isDefinedAt((10, 0)))  // false

// 안전한 호출
if (divide.isDefinedAt((10, 2))) {
  println(divide((10, 2)))  // 5
}

// lift: PartialFunction을 Option 반환 함수로 변환
val safeDivide = divide.lift
println(safeDivide((10, 2)))  // Some(5)
println(safeDivide((10, 0)))  // None
```

```java
// Java에는 부분 함수 개념이 없음
// Optional과 조건 검사로 유사하게 구현
import java.util.Optional;

public class PartialFunctionExample {
    public static Optional<Integer> divide(int a, int b) {
        if (b != 0) {
            return Optional.of(a / b);
        } else {
            return Optional.empty();
        }
    }

    public static void main(String[] args) {
        divide(10, 2).ifPresent(System.out::println);  // 5
        divide(10, 0).ifPresent(System.out::println);  // (출력 없음)
    }
}
```

### 7.3.2 패턴 매칭과 부분 함수

```scala
// 실전 예제: HTTP 상태 코드 처리
val handleHttpStatus: PartialFunction[Int, String] = {
  case 200 => "OK"
  case 201 => "Created"
  case 404 => "Not Found"
  case 500 => "Internal Server Error"
}

println(handleHttpStatus(200))  // "OK"
// println(handleHttpStatus(403))  // MatchError!

// orElse로 부분 함수 합성
val handleClientError: PartialFunction[Int, String] = {
  case 400 => "Bad Request"
  case 401 => "Unauthorized"
  case 403 => "Forbidden"
}

val handleAllStatus = handleHttpStatus orElse handleClientError orElse {
  case _ => "Unknown Status"
}

println(handleAllStatus(403))  // "Forbidden"
println(handleAllStatus(999))  // "Unknown Status"
```

### 7.3.3 컬렉션과 부분 함수

```scala
// collect: 부분 함수를 사용한 필터링 + 매핑
val numbers = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

// filter + map 조합
val evenDoubled1 = numbers.filter(_ % 2 == 0).map(_ * 2)

// collect로 한 번에 처리
val evenDoubled2 = numbers.collect {
  case n if n % 2 == 0 => n * 2
}

println(evenDoubled2)  // List(4, 8, 12, 16, 20)

// 실전 예제: 로그 파싱
sealed trait LogLevel
case object DEBUG extends LogLevel
case object INFO extends LogLevel
case object WARN extends LogLevel
case object ERROR extends LogLevel

case class LogEntry(level: LogLevel, message: String)

val logs = List(
  LogEntry(DEBUG, "Debug message"),
  LogEntry(INFO, "Application started"),
  LogEntry(WARN, "Low memory warning"),
  LogEntry(ERROR, "Connection failed"),
  LogEntry(INFO, "Request processed")
)

// 에러와 경고만 추출
val criticalLogs = logs.collect {
  case LogEntry(ERROR, msg) => s"ERROR: $msg"
  case LogEntry(WARN, msg) => s"WARN: $msg"
}

println(criticalLogs)
// List("WARN: Low memory warning", "ERROR: Connection failed")
```

---

## 7.4 재귀 패턴 심화 (Advanced Recursion Patterns)

### 7.4.1 상호 재귀 (Mutual Recursion)

```scala
// 짝수/홀수 판별 (상호 재귀)
def isEven(n: Int): Boolean = {
  if (n == 0) true
  else isOdd(n - 1)
}

def isOdd(n: Int): Boolean = {
  if (n == 0) false
  else isEven(n - 1)
}

println(isEven(4))  // true
println(isOdd(5))   // true
```

### 7.4.2 누산기 패턴 (Accumulator Pattern)

```scala
// 꼬리 재귀로 리스트 합계
@scala.annotation.tailrec
def sumList(list: List[Int], acc: Int = 0): Int = list match {
  case Nil => acc
  case head :: tail => sumList(tail, acc + head)
}

println(sumList(List(1, 2, 3, 4, 5)))  // 15

// 꼬리 재귀로 리스트 역순
@scala.annotation.tailrec
def reverseList[A](list: List[A], acc: List[A] = Nil): List[A] = list match {
  case Nil => acc
  case head :: tail => reverseList(tail, head :: acc)
}

println(reverseList(List(1, 2, 3, 4, 5)))  // List(5, 4, 3, 2, 1)
```

### 7.4.3 트리 구조 재귀

```scala
// 이진 트리 정의
sealed trait Tree[+A]
case class Leaf[A](value: A) extends Tree[A]
case class Branch[A](left: Tree[A], right: Tree[A]) extends Tree[A]

// 트리 크기 계산
def size[A](tree: Tree[A]): Int = tree match {
  case Leaf(_) => 1
  case Branch(left, right) => 1 + size(left) + size(right)
}

// 트리 깊이 계산
def depth[A](tree: Tree[A]): Int = tree match {
  case Leaf(_) => 0
  case Branch(left, right) => 1 + math.max(depth(left), depth(right))
}

// 트리 맵
def map[A, B](tree: Tree[A])(f: A => B): Tree[B] = tree match {
  case Leaf(value) => Leaf(f(value))
  case Branch(left, right) => Branch(map(left)(f), map(right)(f))
}

// 사용 예제
val tree: Tree[Int] = Branch(
  Branch(Leaf(1), Leaf(2)),
  Branch(Leaf(3), Branch(Leaf(4), Leaf(5)))
)

println(size(tree))   // 9
println(depth(tree))  // 3
println(map(tree)(_ * 2))
// Branch(Branch(Leaf(2),Leaf(4)),Branch(Leaf(6),Branch(Leaf(8),Leaf(10))))
```

### 7.4.4 메모이제이션 (Memoization)

```scala
// 피보나치 수열 (비효율적)
def fib(n: Int): Int = {
  if (n <= 1) n
  else fib(n - 1) + fib(n - 2)
}

// 메모이제이션 적용
def fibMemo: Int => BigInt = {
  val cache = scala.collection.mutable.Map[Int, BigInt]()

  def fib(n: Int): BigInt = {
    if (n <= 1) n
    else cache.getOrElseUpdate(n, fib(n - 1) + fib(n - 2))
  }

  fib
}

// 성능 비교
val start1 = System.nanoTime()
println(fib(35))  // 9227465 (약 1초 소요)
val end1 = System.nanoTime()
println(s"Time: ${(end1 - start1) / 1e6} ms")

val start2 = System.nanoTime()
println(fibMemo(35))  // 9227465 (즉시 완료)
val end2 = System.nanoTime()
println(s"Time: ${(end2 - start2) / 1e6} ms")
```

---

## 7.5 실전 예제: 함수형 데이터 변환 파이프라인

### 7.5.1 CSV 데이터 처리

```scala
// CSV 파일 처리 파이프라인
case class Sale(date: String, product: String, amount: Double, quantity: Int)

// 순수 함수들로 구성
val parseLine: String => Option[Sale] = { line =>
  val parts = line.split(",")
  if (parts.length == 4) {
    try {
      Some(Sale(parts(0), parts(1), parts(2).toDouble, parts(3).toInt))
    } catch {
      case _: NumberFormatException => None
    }
  } else None
}

val filterValidSales: List[Sale] => List[Sale] =
  _.filter(_.amount > 0)

val calculateRevenue: List[Sale] => Double =
  _.map(s => s.amount * s.quantity).sum

val groupByProduct: List[Sale] => Map[String, List[Sale]] =
  _.groupBy(_.product)

// 파이프라인 실행
val csvData = List(
  "2025-01-01,Laptop,1200.0,2",
  "2025-01-02,Mouse,25.0,10",
  "2025-01-03,Keyboard,75.0,5",
  "2025-01-04,Invalid,Data"  // 무효 데이터
)

val sales = csvData.flatMap(parseLine)
val validSales = filterValidSales(sales)
val totalRevenue = calculateRevenue(validSales)
val byProduct = groupByProduct(validSales)

println(s"Total Revenue: $$${totalRevenue}")  // Total Revenue: $2875.0
println(s"Products: ${byProduct.keys.mkString(", ")}")
// Products: Laptop, Mouse, Keyboard
```

### 7.5.2 에러 처리를 포함한 파이프라인

```scala
// Either를 사용한 에러 처리
type ParseError = String
type ParseResult[A] = Either[ParseError, A]

def parseLineWithError(line: String): ParseResult[Sale] = {
  val parts = line.split(",")
  if (parts.length != 4) {
    Left(s"Invalid format: $line")
  } else {
    try {
      Right(Sale(parts(0), parts(1), parts(2).toDouble, parts(3).toInt))
    } catch {
      case e: NumberFormatException =>
        Left(s"Invalid number format: ${e.getMessage}")
    }
  }
}

val results = csvData.map(parseLineWithError)
val (errors, validSales2) = results.partition(_.isLeft)

println(s"Valid: ${validSales2.size}, Errors: ${errors.size}")
errors.foreach(e => println(s"  ${e.left.get}"))
```

---

## 7.6 Java 개발자를 위한 팁

### 7.6.1 Java Stream API와의 비교

| Scala | Java Stream API | 설명 |
|-------|----------------|------|
| `map` | `map` | 변환 |
| `filter` | `filter` | 필터링 |
| `flatMap` | `flatMap` | 평탄화 + 변환 |
| `collect` | `filter + map` | 부분 함수 활용 |
| `fold` | `reduce` | 집계 |
| `compose` | `compose` | 함수 합성 (역순) |
| `andThen` | `andThen` | 함수 합성 (순방향) |

### 7.6.2 혼동 포인트

1. **PartialFunction vs Function**
   - Java에는 부분 함수 개념이 없음
   - `Optional`과 조건 검사로 대체

2. **참조 투명성**
   - Java는 기본적으로 참조 투명성을 강제하지 않음
   - 개발자가 의식적으로 순수 함수 작성 필요

3. **메모이제이션**
   - Java에서는 Guava의 `Suppliers.memoize()` 등 라이브러리 사용
   - Scala는 클로저와 가변 Map으로 쉽게 구현

---

## 7.7 실습 과제

### 과제 7-1: 함수 합성 파이프라인

다음 요구사항을 만족하는 문자열 처리 파이프라인을 작성하세요:

1. 앞뒤 공백 제거
2. 소문자 변환
3. 특수문자 제거 (알파벳과 숫자만 남김)
4. 5자 이상의 문자열만 통과

```scala
// 구현할 함수 시그니처
val sanitize: String => Option[String] = ???

// 테스트 케이스
assert(sanitize("  Hello World!  ") == Some("helloworld"))
assert(sanitize("Hi") == None)  // 5자 미만
assert(sanitize("  Scala@2025  ") == Some("scala2025"))
```

### 과제 7-2: 부분 함수로 JSON 파싱

간단한 JSON 값 파서를 부분 함수로 구현하세요:

```scala
// JSON 값 타입
sealed trait JsonValue
case class JsonNumber(value: Double) extends JsonValue
case class JsonString(value: String) extends JsonValue
case class JsonBoolean(value: Boolean) extends JsonValue
case object JsonNull extends JsonValue

// 구현할 부분 함수
val parseJson: PartialFunction[String, JsonValue] = ???

// 테스트 케이스
assert(parseJson("123") == JsonNumber(123))
assert(parseJson("\"hello\"") == JsonString("hello"))
assert(parseJson("true") == JsonBoolean(true))
assert(parseJson("null") == JsonNull)
assert(!parseJson.isDefinedAt("{invalid}"))
```

### 과제 7-3: 재귀로 디렉토리 트리 순회

파일 시스템 트리를 재귀로 순회하여 모든 파일 크기 합계를 계산하세요:

```scala
sealed trait FileSystem
case class File(name: String, size: Long) extends FileSystem
case class Directory(name: String, contents: List[FileSystem]) extends FileSystem

// 구현할 함수
def totalSize(fs: FileSystem): Long = ???

// 테스트 케이스
val fileTree = Directory("root", List(
  File("a.txt", 100),
  Directory("subdir", List(
    File("b.txt", 200),
    File("c.txt", 300)
  )),
  File("d.txt", 400)
))

assert(totalSize(fileTree) == 1000)
```

---

## 7.8 요약

이번 챕터에서 학습한 내용:

1. **순수 함수**: 동일 입력 → 동일 출력, 부수 효과 없음
2. **참조 투명성**: 표현식을 결과값으로 치환 가능
3. **함수 합성**: `compose`, `andThen`으로 파이프라인 구축
4. **부분 함수**: 입력 도메인의 일부에서만 정의, `collect`와 함께 활용
5. **재귀 패턴**: 상호 재귀, 누산기, 트리 순회, 메모이제이션

**다음 챕터 예고**: Chapter 8에서는 `Option`, `Try`, `Either`를 활용한 안전한 에러 처리를 학습합니다.

---

## Scala 3 주요 차이점

- **Extension Methods**: `implicit class` 대신 `extension` 키워드 사용
- **Top-level Definitions**: `object` 없이 최상위에 함수 정의 가능
- **New Control Structure Syntax**: `if`, `match` 등에서 괄호 생략 가능
