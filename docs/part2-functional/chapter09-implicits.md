# Chapter 9: 암시적 변환 (Implicits)

## 학습 목표
- implicit parameter를 통한 컨텍스트 전달 이해
- implicit conversion의 활용과 주의사항 학습
- implicit class로 기존 타입 확장하기
- 타입 클래스 패턴(Type Class Pattern) 기초 습득

---

## 9.1 Implicit의 필요성

**문제 상황**: 많은 함수에 동일한 설정 객체를 반복 전달

```scala
// ❌ 반복적인 파라미터 전달
case class Config(timeout: Int, maxRetries: Int)

def fetchUser(id: Int, config: Config): User = ???
def fetchOrders(userId: Int, config: Config): List[Order] = ???
def processPayment(orderId: Int, config: Config): Payment = ???

val config = Config(3000, 3)
val user = fetchUser(1, config)
val orders = fetchOrders(user.id, config)
val payment = processPayment(orders.head.id, config)
```

**해결책**: Implicit parameter

```scala
// ✅ implicit으로 자동 전달
def fetchUser(id: Int)(implicit config: Config): User = ???
def fetchOrders(userId: Int)(implicit config: Config): List[Order] = ???
def processPayment(orderId: Int)(implicit config: Config): Payment = ???

implicit val config: Config = Config(3000, 3)

// config 명시 불필요
val user = fetchUser(1)
val orders = fetchOrders(user.id)
val payment = processPayment(orders.head.id)
```

---

## 9.2 Implicit Parameters (암시적 매개변수)

### 9.2.1 기본 사용법

```scala
// 실행 컨텍스트 전달
import scala.concurrent.{Future, ExecutionContext}

// ❌ 명시적 전달
def asyncFetch(url: String)(ec: ExecutionContext): Future[String] = {
  Future {
    // HTTP 요청 시뮬레이션
    s"Content from $url"
  }(ec)
}

// ✅ implicit 사용
def asyncFetchImplicit(url: String)(implicit ec: ExecutionContext): Future[String] = {
  Future {
    s"Content from $url"
  }  // ec가 자동으로 전달됨
}

// 사용
implicit val ec: ExecutionContext = ExecutionContext.global
val result = asyncFetchImplicit("https://example.com")
```

```java
// Java에는 implicit 개념 없음
// ThreadLocal이나 Dependency Injection으로 유사하게 구현
public class ConfigContext {
    private static final ThreadLocal<Config> context = new ThreadLocal<>();

    public static void setConfig(Config config) {
        context.set(config);
    }

    public static Config getConfig() {
        return context.get();
    }
}

// 사용
ConfigContext.setConfig(new Config(3000, 3));
User user = fetchUser(1);  // 내부에서 ConfigContext.getConfig() 호출
```

### 9.2.2 Implicit 해결 규칙

Scala 컴파일러는 다음 순서로 implicit을 찾습니다:

1. **현재 스코프**: 로컬 변수, 파라미터, 임포트된 implicit
2. **동반 객체(Companion Object)**: 타입의 동반 객체에 정의된 implicit

```scala
// 우선순위 예제
case class Currency(symbol: String)

object Currency {
  // 동반 객체의 implicit (낮은 우선순위)
  implicit val defaultCurrency: Currency = Currency("USD")
}

def formatPrice(amount: Double)(implicit currency: Currency): String = {
  s"${currency.symbol} $amount"
}

// 케이스 1: 동반 객체의 implicit 사용
println(formatPrice(100.0))  // USD 100.0

// 케이스 2: 로컬 implicit이 우선
{
  implicit val localCurrency: Currency = Currency("EUR")
  println(formatPrice(100.0))  // EUR 100.0
}
```

### 9.2.3 Multiple Implicit Parameters

```scala
// 여러 개의 implicit 파라미터
case class User(name: String)
case class Logger(level: String)

def log(message: String)(implicit user: User, logger: Logger): Unit = {
  println(s"[${logger.level}] ${user.name}: $message")
}

implicit val currentUser: User = User("Alice")
implicit val systemLogger: Logger = Logger("INFO")

log("Operation completed")  // [INFO] Alice: Operation completed
```

### 9.2.4 implicitly 함수

```scala
// implicit 값을 명시적으로 가져오기
def showImplicit[T](implicit value: T): T = value

// 또는 Predef.implicitly 사용
implicit val config: Config = Config(3000, 3)

val retrievedConfig = implicitly[Config]
println(retrievedConfig)  // Config(3000, 3)

// 사용 사례: 타입 증명
def printSorted[T: Ordering](list: List[T]): Unit = {
  val ordering = implicitly[Ordering[T]]
  println(list.sorted(ordering))
}

printSorted(List(3, 1, 2))  // List(1, 2, 3)
```

---

## 9.3 Implicit Conversion (암시적 변환)

### 9.3.1 타입 변환

```scala
// 자동 타입 변환
case class RichInt(value: Int) {
  def times(f: => Unit): Unit = {
    (1 to value).foreach(_ => f)
  }
}

implicit def intToRichInt(x: Int): RichInt = RichInt(x)

// 사용: Int가 자동으로 RichInt로 변환
3.times {
  println("Hello!")
}
// 출력:
// Hello!
// Hello!
// Hello!
```

**⚠️ 주의**: Implicit conversion은 코드 가독성을 해칠 수 있으므로 신중히 사용해야 합니다.

### 9.3.2 실전 예제: 단위 변환

```scala
// 거리 단위 변환
case class Meters(value: Double) {
  def toKilometers: Kilometers = Kilometers(value / 1000)
  def toMiles: Miles = Miles(value / 1609.34)
}

case class Kilometers(value: Double) {
  def toMeters: Meters = Meters(value * 1000)
}

case class Miles(value: Double) {
  def toMeters: Meters = Meters(value * 1609.34)
}

// Implicit conversion 정의
implicit def doubleToMeters(d: Double): Meters = Meters(d)

// 사용
val distance: Meters = 1000.0  // 암시적 변환
println(distance.toKilometers)  // Kilometers(1.0)

// DSL 스타일
object DistanceUnits {
  implicit class DistanceOps(val value: Double) extends AnyVal {
    def meters: Meters = Meters(value)
    def km: Kilometers = Kilometers(value)
    def miles: Miles = Miles(value)
  }
}

import DistanceUnits._
val distance2 = 5.km
println(distance2.toMeters)  // Meters(5000.0)
```

### 9.3.3 주의사항

```scala
// ❌ 나쁜 예: 암시적 변환 남용
implicit def stringToInt(s: String): Int = s.toInt

val x: Int = "42"  // 작동하지만 혼란스러움
val y = "100" + 200  // "100200" 또는 300?

// ✅ 좋은 예: 명시적 변환
def stringToInt(s: String): Int = s.toInt
val x = stringToInt("42")
```

// Scala 3 차이점: Implicit conversion 제한
/*
Scala 3에서는 implicit conversion이 더 엄격하게 제한됩니다:
- given/using 문법으로 대체
- 명시적으로 Conversion 타입 클래스 사용
*/

---

## 9.4 Implicit Class (Extension Methods)

### 9.4.1 기본 사용법

```scala
// 기존 타입에 메서드 추가
implicit class StringOps(val s: String) extends AnyVal {
  def isEmail: Boolean = s.contains("@") && s.contains(".")
  def capitalize: String = s.headOption.map(_.toUpper).getOrElse("") + s.drop(1)
  def repeat(n: Int): String = s * n
}

println("alice@example.com".isEmail)  // true
println("hello".capitalize)           // Hello
println("abc".repeat(3))              // abcabcabc
```

```java
// Java: 유틸리티 클래스로 구현
public class StringUtils {
    public static boolean isEmail(String s) {
        return s.contains("@") && s.contains(".");
    }

    public static String capitalize(String s) {
        if (s.isEmpty()) return "";
        return s.substring(0, 1).toUpperCase() + s.substring(1);
    }
}

// 사용 (메서드 체이닝 불가)
boolean valid = StringUtils.isEmail("alice@example.com");
```

### 9.4.2 AnyVal을 사용한 성능 최적화

```scala
// AnyVal 상속으로 박싱 오버헤드 제거
implicit class RichInt(val value: Int) extends AnyVal {
  def isEven: Boolean = value % 2 == 0
  def squared: Int = value * value
}

println(5.isEven)    // false
println(5.squared)   // 25

// AnyVal 없이 구현하면 매번 RichInt 객체 생성
// AnyVal 사용 시 컴파일 타임에 최적화되어 객체 생성 없음
```

### 9.4.3 실전 예제: 컬렉션 확장

```scala
// List 확장
implicit class ListOps[A](val list: List[A]) extends AnyVal {
  def second: Option[A] = list.drop(1).headOption

  def safeHead: Option[A] = list.headOption

  def partitionBy[K](f: A => K): Map[K, List[A]] = list.groupBy(f)
}

val numbers = List(1, 2, 3, 4, 5)
println(numbers.second)  // Some(2)
println(numbers.safeHead)  // Some(1)

case class Person(name: String, age: Int)
val people = List(Person("Alice", 25), Person("Bob", 30), Person("Charlie", 25))
val byAge = people.partitionBy(_.age)
println(byAge)
// Map(25 -> List(Person(Alice,25), Person(Charlie,25)), 30 -> List(Person(Bob,30)))
```

### 9.4.4 Context Bounds와의 조합

```scala
// Ordering을 implicit class와 함께 사용
implicit class ComparableOps[A: Ordering](val value: A) {
  def <(other: A): Boolean = implicitly[Ordering[A]].lt(value, other)
  def >(other: A): Boolean = implicitly[Ordering[A]].gt(value, other)
  def between(min: A, max: A): Boolean = value > min && value < max
}

println(5.between(1, 10))  // true
println("b".between("a", "c"))  // true
```

---

## 9.5 타입 클래스 패턴 (Type Class Pattern)

### 9.5.1 타입 클래스의 개념

**타입 클래스**는 기존 타입을 수정하지 않고 새로운 기능을 추가하는 디자인 패턴입니다.

```scala
// 1단계: 타입 클래스 정의
trait JsonSerializer[A] {
  def toJson(value: A): String
}

// 2단계: 타입 클래스 인스턴스 정의
object JsonSerializer {
  implicit val intSerializer: JsonSerializer[Int] = new JsonSerializer[Int] {
    def toJson(value: Int): String = value.toString
  }

  implicit val stringSerializer: JsonSerializer[String] = new JsonSerializer[String] {
    def toJson(value: String): String = s""""$value""""
  }

  implicit val booleanSerializer: JsonSerializer[Boolean] = new JsonSerializer[Boolean] {
    def toJson(value: Boolean): String = value.toString
  }

  // 리스트 시리얼라이저 (재귀적 타입 클래스)
  implicit def listSerializer[A](implicit serializer: JsonSerializer[A]): JsonSerializer[List[A]] =
    new JsonSerializer[List[A]] {
      def toJson(value: List[A]): String = {
        val elements = value.map(serializer.toJson).mkString(",")
        s"[$elements]"
      }
    }
}

// 3단계: 타입 클래스 사용
def toJson[A](value: A)(implicit serializer: JsonSerializer[A]): String = {
  serializer.toJson(value)
}

// 또는 context bound 사용
def toJsonShort[A: JsonSerializer](value: A): String = {
  implicitly[JsonSerializer[A]].toJson(value)
}

// 사용 예제
println(toJson(42))          // 42
println(toJson("hello"))     // "hello"
println(toJson(true))        // true
println(toJson(List(1, 2, 3)))  // [1,2,3]
```

### 9.5.2 커스텀 타입의 타입 클래스 인스턴스

```scala
case class Person(name: String, age: Int)

// Person의 JsonSerializer 인스턴스
implicit val personSerializer: JsonSerializer[Person] = new JsonSerializer[Person] {
  def toJson(value: Person): String = {
    s"""{"name":"${value.name}","age":${value.age}}"""
  }
}

val alice = Person("Alice", 25)
println(toJson(alice))  // {"name":"Alice","age":25}

// 리스트도 자동으로 작동
val people = List(Person("Alice", 25), Person("Bob", 30))
println(toJson(people))
// [{"name":"Alice","age":25},{"name":"Bob","age":30}]
```

### 9.5.3 실전 예제: Ordering 타입 클래스

```scala
// Scala 표준 라이브러리의 Ordering 타입 클래스 활용
case class Employee(name: String, salary: Int, department: String)

// 다양한 정렬 기준
object Employee {
  implicit val byName: Ordering[Employee] = Ordering.by(_.name)

  val bySalary: Ordering[Employee] = Ordering.by(_.salary)

  val byDepartment: Ordering[Employee] = Ordering.by(_.department)
}

val employees = List(
  Employee("Alice", 80000, "Engineering"),
  Employee("Bob", 70000, "Sales"),
  Employee("Charlie", 90000, "Engineering")
)

// implicit Ordering 사용
println(employees.sorted)  // 이름순 (암시적)

// 명시적 Ordering 지정
println(employees.sorted(Employee.bySalary))  // 급여순
println(employees.sorted(Employee.byDepartment))  // 부서순
```

### 9.5.4 타입 클래스의 장점

```scala
// 기존 타입을 수정하지 않고 새로운 동작 추가
// 예: 써드파티 라이브러리 타입 확장

// 외부 라이브러리 타입 (수정 불가)
case class ExternalUser(id: Int, email: String)

// 우리만의 JsonSerializer 추가
implicit val externalUserSerializer: JsonSerializer[ExternalUser] =
  new JsonSerializer[ExternalUser] {
    def toJson(value: ExternalUser): String = {
      s"""{"id":${value.id},"email":"${value.email}"}"""
    }
  }

val user = ExternalUser(1, "user@example.com")
println(toJson(user))  // {"id":1,"email":"user@example.com"}
```

---

## 9.6 실전 예제: 데이터 검증 타입 클래스

```scala
// 검증 타입 클래스
trait Validator[A] {
  def validate(value: A): Either[String, A]
}

object Validator {
  // 검증 인스턴스
  implicit val emailValidator: Validator[String] = new Validator[String] {
    def validate(value: String): Either[String, String] = {
      if (value.contains("@")) Right(value)
      else Left(s"Invalid email: $value")
    }
  }

  implicit val ageValidator: Validator[Int] = new Validator[Int] {
    def validate(value: Int): Either[String, Int] = {
      if (value >= 0 && value <= 120) Right(value)
      else Left(s"Invalid age: $value")
    }
  }

  // 헬퍼 함수
  def validate[A](value: A)(implicit validator: Validator[A]): Either[String, A] = {
    validator.validate(value)
  }
}

// 사용 예제
import Validator._

println(validate("alice@example.com"))  // Right(alice@example.com)
println(validate("invalid-email"))      // Left(Invalid email: invalid-email)
println(validate(25))                   // Right(25)
println(validate(150))                  // Left(Invalid age: 150)

// 커스텀 타입 검증
case class User(email: String, age: Int)

implicit val userValidator: Validator[User] = new Validator[User] {
  def validate(user: User): Either[String, User] = {
    for {
      validEmail <- Validator.validate(user.email)
      validAge <- Validator.validate(user.age)
    } yield User(validEmail, validAge)
  }
}

val user1 = User("alice@example.com", 25)
val user2 = User("invalid", 25)

println(validate(user1))  // Right(User(alice@example.com,25))
println(validate(user2))  // Left(Invalid email: invalid)
```

---

## 9.7 Implicit 사용 모범 사례

### 9.7.1 DO's (권장사항)

1. **명확한 이름 사용**
   ```scala
   implicit val defaultTimeout: Timeout = Timeout(30.seconds)
   implicit val jsonConfig: JsonConfig = JsonConfig.default
   ```

2. **동반 객체에 배치**
   ```scala
   case class Money(amount: Double, currency: String)

   object Money {
     implicit val ordering: Ordering[Money] = Ordering.by(_.amount)
   }
   ```

3. **타입 클래스 패턴 활용**
   ```scala
   trait Show[A] {
     def show(value: A): String
   }
   ```

### 9.7.2 DON'Ts (피해야 할 사항)

1. **❌ 여러 개의 동일 타입 implicit**
   ```scala
   // 컴파일 에러: ambiguous implicit values
   implicit val timeout1: Int = 3000
   implicit val timeout2: Int = 5000

   def fetch(url: String)(implicit timeout: Int): String = ???
   ```

2. **❌ 암시적 변환 남용**
   ```scala
   // 혼란스러움
   implicit def intToString(i: Int): String = i.toString
   implicit def stringToInt(s: String): Int = s.toInt
   ```

3. **❌ 너무 광범위한 implicit scope**
   ```scala
   // 전역 스코프 오염
   implicit val config: Config = ???
   ```

---

## 9.8 Java 개발자를 위한 팁

### 9.8.1 Java의 대안

| Scala | Java | 설명 |
|-------|------|------|
| implicit parameter | Dependency Injection | Spring, Guice 등 |
| implicit conversion | 메서드 오버로딩 | 제한적 |
| implicit class | 유틸리티 클래스 | 정적 메서드 |
| 타입 클래스 | 인터페이스 + 구현 | 상속 기반 |

### 9.8.2 혼동 포인트

1. **Implicit resolution은 컴파일 타임**
   - 런타임 오버헤드 없음
   - 컴파일러가 자동으로 코드 삽입

2. **Implicit은 자동 임포트되지 않음**
   - 명시적 import 필요
   - 또는 동반 객체에 정의

3. **Implicit 우선순위**
   - 로컬 > 임포트 > 동반 객체
   - 같은 우선순위에서 여러 개 발견 시 컴파일 에러

---

## 9.9 실습 과제

### 과제 9-1: 로깅 타입 클래스

다양한 타입의 로깅을 지원하는 타입 클래스를 구현하세요:

```scala
trait Logger[A] {
  def log(value: A): String
}

// 구현할 인스턴스
implicit val intLogger: Logger[Int] = ???
implicit val stringLogger: Logger[String] = ???
implicit def listLogger[A: Logger]: Logger[List[A]] = ???

// 헬퍼 함수
def log[A: Logger](value: A): Unit = println(implicitly[Logger[A]].log(value))

// 테스트
log(42)          // "[INT] 42"
log("hello")     // "[STRING] hello"
log(List(1, 2))  // "[LIST] [INT] 1, [INT] 2"
```

### 과제 9-2: 거리 단위 DSL

거리 단위 변환을 지원하는 DSL을 구현하세요:

```scala
case class Distance(meters: Double)

// 구현할 implicit class
implicit class DistanceSyntax(val value: Double) extends AnyVal {
  def meters: Distance = ???
  def km: Distance = ???
  def miles: Distance = ???
}

// 테스트
val d1 = 1000.meters
val d2 = 1.km
val d3 = 0.621371.miles
assert(d1 == d2)
```

### 과제 9-3: Monoid 타입 클래스

Monoid 타입 클래스를 구현하고 활용하세요:

```scala
trait Monoid[A] {
  def empty: A
  def combine(x: A, y: A): A
}

// 구현할 인스턴스
implicit val intMonoid: Monoid[Int] = ???
implicit val stringMonoid: Monoid[String] = ???
implicit def listMonoid[A]: Monoid[List[A]] = ???

// 헬퍼 함수
def combineAll[A: Monoid](list: List[A]): A = ???

// 테스트
assert(combineAll(List(1, 2, 3)) == 6)
assert(combineAll(List("a", "b", "c")) == "abc")
assert(combineAll(List(List(1), List(2), List(3))) == List(1, 2, 3))
```

---

## 9.10 요약

이번 챕터에서 학습한 내용:

1. **Implicit Parameters**: 컨텍스트 자동 전달
2. **Implicit Conversion**: 타입 자동 변환 (신중히 사용)
3. **Implicit Class**: 기존 타입에 메서드 추가 (Extension Methods)
4. **타입 클래스 패턴**: 기존 타입을 수정하지 않고 새로운 동작 추가
5. **모범 사례**: 명확한 이름, 동반 객체 배치, 남용 금지

**다음 챕터 예고**: Chapter 10에서는 제네릭, 공변성/반공변성 등 타입 시스템 심화를 학습합니다.

---

## Scala 3 주요 차이점

- **given/using 문법**: `implicit def` → `given`/`using` 키워드
- **Extension Methods**: `implicit class` → `extension` 키워드
- **Context Functions**: `implicit` 파라미터를 위한 새로운 함수 타입
- **제한된 Implicit Conversion**: 명시적 `Conversion` 타입 클래스 필요

```scala
// Scala 3 예제
given config: Config = Config(3000, 3)

def fetchUser(id: Int)(using config: Config): User = ???

extension (s: String)
  def isEmail: Boolean = s.contains("@")
```
