# Chapter 8: Option, Try, Either - 안전한 에러 처리 (Safe Error Handling)

## 학습 목표
- Option을 활용한 Null 안전성 확보
- Try를 사용한 예외 처리의 함수형 접근
- Either로 풍부한 에러 정보 전달
- for-comprehension을 통한 에러 처리 체이닝

---

## 8.1 Option: Null 안전성 (Null Safety with Option)

### 8.1.1 Option의 필요성

**Java의 Null 문제**:
```java
// Java: NullPointerException 위험
public String getUserEmail(int userId) {
    User user = findUserById(userId);  // null 가능
    return user.getEmail().toLowerCase();  // NPE 발생 가능!
}
```

**Scala의 Option**:
```scala
// Scala: 컴파일 타임에 null 가능성 명시
def findUserById(userId: Int): Option[User] = {
  // DB 조회 시뮬레이션
  if (userId > 0) Some(User(userId, "alice@example.com"))
  else None
}

def getUserEmail(userId: Int): Option[String] = {
  findUserById(userId).map(_.email.toLowerCase)
}

println(getUserEmail(1))   // Some(alice@example.com)
println(getUserEmail(-1))  // None
```

### 8.1.2 Option 기본 연산

```scala
// Some과 None
val some: Option[Int] = Some(42)
val none: Option[Int] = None

// getOrElse: 기본값 제공
println(some.getOrElse(0))  // 42
println(none.getOrElse(0))  // 0

// map: 값 변환
println(some.map(_ * 2))  // Some(84)
println(none.map(_ * 2))  // None

// flatMap: 중첩 Option 평탄화
def divide(a: Int, b: Int): Option[Int] = {
  if (b != 0) Some(a / b) else None
}

println(some.flatMap(x => divide(x, 2)))  // Some(21)
println(none.flatMap(x => divide(x, 2)))  // None

// filter: 조건 필터링
println(some.filter(_ > 40))  // Some(42)
println(some.filter(_ > 50))  // None

// fold: 패턴 매칭 대안
val result = some.fold("없음")(x => s"값: $x")
println(result)  // "값: 42"
```

```java
// Java Optional 비교
import java.util.Optional;

public class OptionExample {
    public static void main(String[] args) {
        Optional<Integer> some = Optional.of(42);
        Optional<Integer> none = Optional.empty();

        // getOrElse → orElse
        System.out.println(some.orElse(0));  // 42

        // map
        System.out.println(some.map(x -> x * 2));  // Optional[84]

        // flatMap
        System.out.println(some.flatMap(x -> divide(x, 2)));  // Optional[21]

        // filter
        System.out.println(some.filter(x -> x > 40));  // Optional[42]
    }

    static Optional<Integer> divide(int a, int b) {
        return b != 0 ? Optional.of(a / b) : Optional.empty();
    }
}
```

### 8.1.3 Option 체이닝

```scala
case class Address(city: String, zipCode: String)
case class Company(name: String, address: Option[Address])
case class User(id: Int, name: String, company: Option[Company])

val users = Map(
  1 -> User(1, "Alice", Some(Company("TechCorp", Some(Address("Seoul", "12345"))))),
  2 -> User(2, "Bob", Some(Company("StartupInc", None))),
  3 -> User(3, "Charlie", None)
)

// 중첩 Option 추출
def getZipCode(userId: Int): Option[String] = {
  users.get(userId)
    .flatMap(_.company)
    .flatMap(_.address)
    .map(_.zipCode)
}

println(getZipCode(1))  // Some(12345)
println(getZipCode(2))  // None (회사는 있지만 주소 없음)
println(getZipCode(3))  // None (회사 없음)
println(getZipCode(4))  // None (사용자 없음)
```

### 8.1.4 for-comprehension과 Option

```scala
// flatMap 체이닝 대신 for-comprehension
def getZipCodeFor(userId: Int): Option[String] = for {
  user <- users.get(userId)
  company <- user.company
  address <- company.address
} yield address.zipCode

println(getZipCodeFor(1))  // Some(12345)

// 여러 Option 조합
def calculateDiscount(userId: Int, productId: Int): Option[Double] = for {
  user <- findUserById(userId)
  product <- findProductById(productId)
  discount <- getUserDiscount(user, product)
} yield discount

// 중간에 하나라도 None이면 전체 결과가 None
```

---

## 8.2 Try: 예외 처리 (Exception Handling with Try)

### 8.2.1 Try의 필요성

```scala
import scala.util.{Try, Success, Failure}

// 전통적인 try-catch
def parseIntOld(s: String): Int = {
  try {
    s.toInt
  } catch {
    case _: NumberFormatException => 0
  }
}

// Try를 사용한 함수형 접근
def parseInt(s: String): Try[Int] = Try(s.toInt)

println(parseInt("42"))    // Success(42)
println(parseInt("abc"))   // Failure(java.lang.NumberFormatException)

// Try 연산
println(parseInt("10").map(_ * 2))          // Success(20)
println(parseInt("abc").map(_ * 2))         // Failure(...)
println(parseInt("10").getOrElse(0))        // 10
println(parseInt("abc").getOrElse(0))       // 0
```

```java
// Java에는 Try가 없음 (예외를 Optional로 변환)
import java.util.Optional;

public class TryExample {
    static Optional<Integer> parseInt(String s) {
        try {
            return Optional.of(Integer.parseInt(s));
        } catch (NumberFormatException e) {
            return Optional.empty();
        }
    }

    public static void main(String[] args) {
        System.out.println(parseInt("42"));   // Optional[42]
        System.out.println(parseInt("abc"));  // Optional.empty
    }
}
```

### 8.2.2 Try 연산과 에러 처리

```scala
// Try 체이닝
def divide(a: Int, b: Int): Try[Int] = Try(a / b)

val result = for {
  x <- parseInt("10")
  y <- parseInt("2")
  z <- divide(x, y)
} yield z

println(result)  // Success(5)

// 에러 처리
val result2 = for {
  x <- parseInt("10")
  y <- parseInt("0")
  z <- divide(x, y)
} yield z

println(result2)  // Failure(java.lang.ArithmeticException: / by zero)

// recover: 특정 예외 복구
val recovered = result2.recover {
  case _: ArithmeticException => 0
}
println(recovered)  // Success(0)

// recoverWith: 다른 Try로 복구
val recoveredWith = result2.recoverWith {
  case _: ArithmeticException => Success(0)
}
println(recoveredWith)  // Success(0)

// orElse: 실패 시 대안 시도
val result3 = parseInt("abc") orElse parseInt("42")
println(result3)  // Success(42)
```

### 8.2.3 실전 예제: 파일 읽기

```scala
import scala.io.Source
import scala.util.{Try, Using}

// Try로 파일 읽기
def readFile(path: String): Try[String] = Try {
  val source = Source.fromFile(path)
  try source.mkString finally source.close()
}

// Scala 2.13+ Using으로 리소스 관리
def readFileSafe(path: String): Try[String] = {
  Using(Source.fromFile(path)) { source =>
    source.mkString
  }
}

// 사용 예제
readFileSafe("/path/to/file.txt") match {
  case Success(content) => println(s"Content: $content")
  case Failure(exception) => println(s"Error: ${exception.getMessage}")
}

// 여러 파일 읽기
def readMultipleFiles(paths: List[String]): List[Try[String]] = {
  paths.map(readFileSafe)
}

val files = List("file1.txt", "file2.txt", "file3.txt")
val results = readMultipleFiles(files)

// 성공과 실패 분리
val (successes, failures) = results.partition(_.isSuccess)
println(s"성공: ${successes.size}, 실패: ${failures.size}")
```

---

## 8.3 Either: 풍부한 에러 정보 (Rich Error Information with Either)

### 8.3.1 Either의 필요성

**Option과 Try의 한계**:
- `Option`: 에러 정보 없음 (Some/None만 구분)
- `Try`: 예외 객체만 제공 (커스텀 에러 타입 불가)

**Either의 장점**:
- 왼쪽(Left): 에러 정보 (커스텀 타입 가능)
- 오른쪽(Right): 성공 값

```scala
// Either 기본 사용
def divide(a: Int, b: Int): Either[String, Int] = {
  if (b == 0) Left("Division by zero")
  else Right(a / b)
}

println(divide(10, 2))  // Right(5)
println(divide(10, 0))  // Left(Division by zero)

// map, flatMap (Right 편향)
println(divide(10, 2).map(_ * 2))  // Right(10)
println(divide(10, 0).map(_ * 2))  // Left(Division by zero)

// getOrElse
println(divide(10, 2).getOrElse(0))  // 5
println(divide(10, 0).getOrElse(0))  // 0
```

### 8.3.2 커스텀 에러 타입

```scala
// 에러 타입 정의
sealed trait ValidationError
case class InvalidEmail(email: String) extends ValidationError
case class InvalidAge(age: Int) extends ValidationError
case class UserNotFound(userId: Int) extends ValidationError

case class User(id: Int, email: String, age: Int)

// 검증 함수
def validateEmail(email: String): Either[InvalidEmail, String] = {
  if (email.contains("@")) Right(email)
  else Left(InvalidEmail(email))
}

def validateAge(age: Int): Either[InvalidAge, Int] = {
  if (age >= 18 && age <= 120) Right(age)
  else Left(InvalidAge(age))
}

// for-comprehension으로 검증 체이닝
def createUser(id: Int, email: String, age: Int): Either[ValidationError, User] = for {
  validEmail <- validateEmail(email)
  validAge <- validateAge(age)
} yield User(id, validEmail, validAge)

println(createUser(1, "alice@example.com", 25))
// Right(User(1,alice@example.com,25))

println(createUser(2, "invalid-email", 25))
// Left(InvalidEmail(invalid-email))

println(createUser(3, "bob@example.com", 15))
// Left(InvalidAge(15))
```

### 8.3.3 Either 패턴 매칭

```scala
// Either 결과 처리
createUser(1, "alice@example.com", 25) match {
  case Right(user) => println(s"User created: $user")
  case Left(InvalidEmail(email)) => println(s"Invalid email: $email")
  case Left(InvalidAge(age)) => println(s"Invalid age: $age")
  case Left(error) => println(s"Error: $error")
}

// fold: 양쪽 모두 처리
val message = createUser(1, "alice@example.com", 25).fold(
  error => s"Error: $error",
  user => s"Success: ${user.email}"
)
println(message)
```

### 8.3.4 여러 에러 누적하기

```scala
// Validated 타입 (cats 라이브러리)
// Either는 첫 번째 에러에서 중단되지만, Validated는 모든 에러 수집 가능

// Either의 한계
def validateUserEither(email: String, age: Int): Either[List[ValidationError], (String, Int)] = {
  for {
    e <- validateEmail(email).left.map(List(_))
    a <- validateAge(age).left.map(List(_))
  } yield (e, a)
}

println(validateUserEither("invalid", 15))
// Left(List(InvalidEmail(invalid)))  // 첫 번째 에러만 반환

// 수동으로 모든 에러 수집
def validateUserAll(email: String, age: Int): Either[List[ValidationError], (String, Int)] = {
  val emailResult = validateEmail(email)
  val ageResult = validateAge(age)

  (emailResult, ageResult) match {
    case (Right(e), Right(a)) => Right((e, a))
    case (Left(e1), Left(e2)) => Left(List(e1, e2))
    case (Left(e), _) => Left(List(e))
    case (_, Left(e)) => Left(List(e))
  }
}

println(validateUserAll("invalid", 15))
// Left(List(InvalidEmail(invalid), InvalidAge(15)))
```

---

## 8.4 Option, Try, Either 선택 가이드

| 상황 | 추천 타입 | 이유 |
|------|----------|------|
| 값이 없을 수 있음 (에러 아님) | `Option` | 단순 존재 여부만 표현 |
| 예외 발생 가능 (Java 라이브러리 호출) | `Try` | 예외를 함수형으로 처리 |
| 커스텀 에러 타입 필요 | `Either` | 풍부한 에러 정보 전달 |
| 여러 에러 누적 필요 | `Validated` (cats) | Either는 첫 에러에서 중단 |

```scala
// 실전 예제: API 응답 처리
sealed trait ApiError
case class NetworkError(message: String) extends ApiError
case class ParseError(message: String) extends ApiError
case class NotFoundError(id: Int) extends ApiError

case class ApiResponse(data: String)

// Option: 캐시 조회
def getFromCache(key: String): Option[ApiResponse] = {
  // 캐시에 없으면 None (에러 아님)
  None
}

// Try: 네트워크 요청 (예외 발생 가능)
def fetchFromNetwork(url: String): Try[String] = {
  Try {
    // 네트워크 요청 시뮬레이션
    scala.io.Source.fromURL(url).mkString
  }
}

// Either: 파싱 (커스텀 에러)
def parseResponse(json: String): Either[ParseError, ApiResponse] = {
  if (json.startsWith("{")) Right(ApiResponse(json))
  else Left(ParseError("Invalid JSON format"))
}

// 조합
def getData(id: Int): Either[ApiError, ApiResponse] = {
  getFromCache(s"user:$id") match {
    case Some(response) => Right(response)
    case None =>
      fetchFromNetwork(s"https://api.example.com/users/$id")
        .toEither
        .left.map(e => NetworkError(e.getMessage))
        .flatMap(parseResponse)
  }
}
```

---

## 8.5 실전 예제: 사용자 등록 시스템

```scala
// 도메인 모델
case class Email(value: String) extends AnyVal
case class Age(value: Int) extends AnyVal
case class UserId(value: Int) extends AnyVal
case class RegisteredUser(id: UserId, email: Email, age: Age)

// 에러 타입
sealed trait RegistrationError
case class InvalidEmailFormat(email: String) extends RegistrationError
case class InvalidAgeRange(age: Int) extends RegistrationError
case class EmailAlreadyExists(email: String) extends RegistrationError
case class DatabaseError(message: String) extends RegistrationError

// 검증 로직
object Validators {
  def validateEmail(email: String): Either[InvalidEmailFormat, Email] = {
    val emailRegex = """^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$""".r
    emailRegex.findFirstIn(email) match {
      case Some(_) => Right(Email(email))
      case None => Left(InvalidEmailFormat(email))
    }
  }

  def validateAge(age: Int): Either[InvalidAgeRange, Age] = {
    if (age >= 18 && age <= 120) Right(Age(age))
    else Left(InvalidAgeRange(age))
  }
}

// 데이터베이스 접근 (시뮬레이션)
object UserDatabase {
  private var users = Map[Email, RegisteredUser]()
  private var nextId = 1

  def emailExists(email: Email): Boolean = users.contains(email)

  def save(email: Email, age: Age): Try[RegisteredUser] = Try {
    val user = RegisteredUser(UserId(nextId), email, age)
    users += (email -> user)
    nextId += 1
    user
  }
}

// 등록 서비스
object RegistrationService {
  import Validators._

  def register(email: String, age: Int): Either[RegistrationError, RegisteredUser] = {
    for {
      validEmail <- validateEmail(email)
      validAge <- validateAge(age)
      _ <- checkEmailAvailability(validEmail)
      user <- saveUser(validEmail, validAge)
    } yield user
  }

  private def checkEmailAvailability(email: Email): Either[EmailAlreadyExists, Email] = {
    if (UserDatabase.emailExists(email)) Left(EmailAlreadyExists(email.value))
    else Right(email)
  }

  private def saveUser(email: Email, age: Age): Either[DatabaseError, RegisteredUser] = {
    UserDatabase.save(email, age).toEither.left.map(e => DatabaseError(e.getMessage))
  }
}

// 사용 예제
def handleRegistration(email: String, age: Int): Unit = {
  RegistrationService.register(email, age) match {
    case Right(user) =>
      println(s"✅ User registered: ${user.email.value} (ID: ${user.id.value})")
    case Left(InvalidEmailFormat(e)) =>
      println(s"❌ Invalid email format: $e")
    case Left(InvalidAgeRange(a)) =>
      println(s"❌ Invalid age: $a (must be 18-120)")
    case Left(EmailAlreadyExists(e)) =>
      println(s"❌ Email already registered: $e")
    case Left(DatabaseError(msg)) =>
      println(s"❌ Database error: $msg")
  }
}

// 테스트
handleRegistration("alice@example.com", 25)
// ✅ User registered: alice@example.com (ID: 1)

handleRegistration("invalid-email", 25)
// ❌ Invalid email format: invalid-email

handleRegistration("bob@example.com", 15)
// ❌ Invalid age: 15 (must be 18-120)

handleRegistration("alice@example.com", 30)
// ❌ Email already registered: alice@example.com
```

---

## 8.6 Java 개발자를 위한 팁

### 8.6.1 Java Optional vs Scala Option

| 기능 | Java Optional | Scala Option |
|------|--------------|--------------|
| 생성 | `Optional.of(x)` | `Some(x)` |
| 빈 값 | `Optional.empty()` | `None` |
| 기본값 | `orElse(default)` | `getOrElse(default)` |
| 변환 | `map(f)` | `map(f)` |
| 평탄화 | `flatMap(f)` | `flatMap(f)` |
| 필터 | `filter(p)` | `filter(p)` |
| 패턴 매칭 | ❌ | ✅ |

### 8.6.2 Try vs try-catch

```scala
// Scala: Try로 함수형 처리
def parseIntSafe(s: String): Try[Int] = Try(s.toInt)

val numbers = List("1", "2", "abc", "3")
val results = numbers.map(parseIntSafe)
val validNumbers = results.collect { case Success(n) => n }
println(validNumbers)  // List(1, 2, 3)
```

```java
// Java: try-catch로 명령형 처리
List<String> numbers = List.of("1", "2", "abc", "3");
List<Integer> validNumbers = new ArrayList<>();

for (String s : numbers) {
    try {
        validNumbers.add(Integer.parseInt(s));
    } catch (NumberFormatException e) {
        // 무시
    }
}
System.out.println(validNumbers);  // [1, 2, 3]
```

### 8.6.3 혼동 포인트

1. **Option.get()은 피해야 함**
   ```scala
   // ❌ 나쁜 예: None.get() 시 예외 발생
   val value = someOption.get

   // ✅ 좋은 예: 안전한 추출
   val value = someOption.getOrElse(defaultValue)
   ```

2. **Try vs Either 선택**
   - `Try`: Java 예외를 다룰 때
   - `Either`: 커스텀 에러 타입이 필요할 때

3. **for-comprehension 단축 평가**
   ```scala
   // 첫 번째 None/Left에서 전체가 중단됨
   val result = for {
     a <- option1  // None이면 여기서 중단
     b <- option2  // 실행 안 됨
   } yield a + b
   ```

---

## 8.7 실습 과제

### 과제 8-1: 설정 파일 파서

설정 파일을 읽어 파싱하는 함수를 작성하세요:

```scala
case class Config(host: String, port: Int, timeout: Int)

sealed trait ConfigError
case class FileNotFound(path: String) extends ConfigError
case class ParseError(key: String, value: String) extends ConfigError
case class MissingKey(key: String) extends ConfigError

// 구현할 함수
def loadConfig(path: String): Either[ConfigError, Config] = ???

// 테스트 (파일 내용: host=localhost\nport=8080\ntimeout=30)
assert(loadConfig("config.txt") == Right(Config("localhost", 8080, 30)))
assert(loadConfig("missing.txt").isLeft)
```

### 과제 8-2: 사용자 인증 시스템

Option, Try, Either를 조합하여 인증 로직을 구현하세요:

```scala
case class Credentials(username: String, password: String)
case class AuthToken(value: String)

sealed trait AuthError
case object InvalidCredentials extends AuthError
case object UserLocked extends AuthError
case class DatabaseError(message: String) extends AuthError

// 구현할 함수
def authenticate(creds: Credentials): Either[AuthError, AuthToken] = ???

// 테스트
assert(authenticate(Credentials("admin", "secret")).isRight)
assert(authenticate(Credentials("admin", "wrong")) == Left(InvalidCredentials))
```

### 과제 8-3: 데이터 변환 파이프라인

여러 단계의 변환을 거치는 파이프라인을 작성하세요:

```scala
// JSON 문자열 → 파싱 → 검증 → 도메인 객체
case class Product(id: Int, name: String, price: Double)

sealed trait PipelineError
case class JsonParseError(msg: String) extends PipelineError
case class ValidationError(msg: String) extends PipelineError

// 구현할 함수
def processProduct(json: String): Either[PipelineError, Product] = ???

// 테스트
val validJson = """{"id":1,"name":"Laptop","price":1200.0}"""
assert(processProduct(validJson).isRight)
```

---

## 8.8 요약

이번 챕터에서 학습한 내용:

1. **Option**: Null 대신 Some/None으로 값의 존재 여부 표현
2. **Try**: 예외를 Success/Failure로 함수형 처리
3. **Either**: Left(에러)/Right(성공)로 풍부한 에러 정보 전달
4. **for-comprehension**: 여러 Option/Try/Either를 우아하게 체이닝
5. **타입 선택**: 상황에 따라 Option/Try/Either 적절히 선택

**다음 챕터 예고**: Chapter 9에서는 Implicit을 활용한 암시적 변환과 타입 클래스 패턴을 학습합니다.

---

## Scala 3 주요 차이점

- **Union Types**: `Either[A, B]` 대신 `A | B` 사용 가능
- **Improved Type Inference**: for-comprehension에서 타입 추론 개선
- **Extension Methods**: Option/Try/Either에 커스텀 메서드 추가 문법 변경
