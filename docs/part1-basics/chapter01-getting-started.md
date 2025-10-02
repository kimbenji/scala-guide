# Chapter 1: Scala 시작하기

## 학습 목표

이 챕터를 완료하면 다음을 할 수 있습니다:
- Scala 개발 환경을 설정하고 첫 프로그램을 실행할 수 있다
- REPL(Read-Eval-Print-Loop)을 사용하여 Scala 코드를 대화식으로 실행할 수 있다
- Scala와 Java의 기본 차이점을 이해한다
- SBT를 사용하여 간단한 프로젝트를 생성하고 빌드할 수 있다

## 선행 지식

- Java 기본 문법 (변수, 제어문, 클래스)
- 터미널/커맨드라인 기본 사용법
- Git 기본 명령어 (선택 사항)

---

## 1.1 Scala란 무엇인가?

Scala(Scalable Language)는 객체지향과 함수형 프로그래밍을 결합한 JVM 언어입니다.

### Scala의 주요 특징

1. **JVM 기반**: Java 라이브러리를 그대로 사용 가능
2. **정적 타입**: 컴파일 타임에 타입 오류 발견
3. **타입 추론**: 명시적 타입 선언 최소화
4. **함수형 + 객체지향**: 두 패러다임의 장점 결합
5. **간결한 문법**: Java 대비 코드량 감소

### Java vs Scala 기본 비교

```scala
// Scala: Hello World
object HelloWorld extends App {
  println("Hello, Scala!")
}

// Java와 비교:
// public class HelloWorld {
//     public static void main(String[] args) {
//         System.out.println("Hello, Java!");
//     }
// }
```

**주요 차이점:**
- `object`: Java의 `public static` 메서드를 가진 싱글톤 클래스
- `extends App`: `main` 메서드 자동 생성
- `println`: `System.out.println` 축약형
- 세미콜론(`;`) 생략 가능

---

## 1.2 개발 환경 설정

### 1.2.1 필수 소프트웨어 설치

#### Java 11 설치 확인

```bash
java -version
# 출력 예시: openjdk version "11.0.x"
```

Java가 없다면 [AdoptOpenJDK](https://adoptopenjdk.net/) 또는 [Oracle JDK](https://www.oracle.com/java/technologies/downloads/)에서 설치하세요.

#### SBT 설치

**macOS (Homebrew):**
```bash
brew install sbt
```

**Linux (Ubuntu/Debian):**
```bash
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | sudo apt-key add
sudo apt-get update
sudo apt-get install sbt
```

**Windows:**
[SBT 공식 사이트](https://www.scala-sbt.org/download.html)에서 설치 파일 다운로드

**설치 확인:**
```bash
sbt sbtVersion
# 출력: [info] 1.9.x
```

### 1.2.2 IntelliJ IDEA 설정

1. [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 설치 (Community 무료 버전도 가능)
2. Scala Plugin 설치:
   - `Settings/Preferences` → `Plugins`
   - "Scala" 검색 후 설치
   - IDE 재시작

---

## 1.3 첫 번째 Scala 프로그램

### 1.3.1 SBT 프로젝트 생성

```bash
mkdir scala-hello
cd scala-hello
```

`build.sbt` 파일 생성:
```scala
name := "scala-hello"
version := "0.1.0"
scalaVersion := "2.12.18"
```

디렉토리 구조 생성:
```bash
mkdir -p src/main/scala
```

### 1.3.2 Hello World 작성

`src/main/scala/HelloWorld.scala` 파일 생성:

```scala
object HelloWorld extends App {
  println("Hello, Scala!")

  // 변수 선언
  val message = "Scala is awesome!"
  println(message)

  // 간단한 연산
  val sum = 1 + 2 + 3
  println(s"Sum: $sum")  // 문자열 인터폴레이션
}
```

**코드 설명:**
- `val message`: 불변 변수 (Java의 `final`)
- `s"Sum: $sum"`: 문자열 인터폴레이션 (Java의 `String.format` 대체)

**Java 비교:**
```java
// Java 동등 코드
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");

        final String message = "Java is cool!";
        System.out.println(message);

        final int sum = 1 + 2 + 3;
        System.out.println("Sum: " + sum);
    }
}
```

### 1.3.3 프로그램 실행

```bash
sbt run
```

**출력:**
```
Hello, Scala!
Scala is awesome!
Sum: 6
```

---

## 1.4 REPL: 대화식 Scala

REPL(Read-Eval-Print-Loop)은 코드를 즉시 실험할 수 있는 대화식 환경입니다.

### 1.4.1 REPL 시작

```bash
sbt console
```

또는 Scala만 설치한 경우:
```bash
scala
```

### 1.4.2 REPL 기본 사용법

```scala
scala> 1 + 2
res0: Int = 3

scala> val name = "Scala"
name: String = Scala

scala> s"Hello, $name!"
res1: String = Hello, Scala!

scala> def square(x: Int) = x * x
square: (x: Int)Int

scala> square(5)
res2: Int = 25

scala> :quit  // REPL 종료
```

**주요 명령어:**
- `:help` - 도움말
- `:type <expression>` - 표현식 타입 확인
- `:load <file>` - 파일 로드
- `:quit` - 종료

**Java와 비교:**
Java에는 기본 REPL이 없었으나, Java 9부터 `jshell` 추가:
```java
// Java 9+ jshell
jshell> int sum = 1 + 2 + 3
sum ==> 6
```

---

## 1.5 Scala의 기본 문법 맛보기

### 1.5.1 변수 선언: val vs var

```scala
val immutable = 42        // 불변 (권장)
var mutable = 10          // 가변 (필요 시에만)
mutable = 20              // OK
// immutable = 50         // 컴파일 오류!
```

**Java 비교:**
```java
// Java
final int immutable = 42;    // 불변
int mutable = 10;            // 가변
```

### 1.5.2 타입 추론

```scala
// Scala: 타입 명시 생략 가능
val count = 10                    // Int로 추론
val name = "Scala"                // String으로 추론
val numbers = List(1, 2, 3)       // List[Int]로 추론

// 타입 명시도 가능
val explicitCount: Int = 10
val explicitName: String = "Scala"
```

**Java 비교:**
```java
// Java: 타입 명시 필수 (Java 10+ var 가능)
int count = 10;
String name = "Java";
List<Integer> numbers = List.of(1, 2, 3);

// Java 10+
var count = 10;  // int로 추론
```

### 1.5.3 함수 정의

```scala
// Scala
def add(a: Int, b: Int): Int = {
  a + b
}

// 한 줄이면 중괄호 생략 가능
def multiply(a: Int, b: Int): Int = a * b

// 반환 타입 추론
def subtract(a: Int, b: Int) = a - b

println(add(3, 5))       // 8
println(multiply(4, 7))  // 28
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

### 1.5.4 표현식 vs 문장

Scala의 모든 것은 **표현식**(값을 반환)입니다. Java의 `if`, `try-catch` 등은 **문장**(값 반환 안 함)입니다.

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

// Scala: try-catch도 표현식
val result = try {
  someRiskyOperation()
} catch {
  case e: Exception => "error"
}
```

---

## 1.6 SBT 기초

### 1.6.1 build.sbt 구조

```scala
// 프로젝트 정보
name := "my-scala-project"
version := "1.0.0"
scalaVersion := "2.12.18"

// 의존성
libraryDependencies ++= Seq(
  "org.scalatest" %% "scalatest" % "3.2.15" % Test
)

// 컴파일 옵션
scalacOptions ++= Seq(
  "-deprecation",      // 사용 중단 경고
  "-feature",          // 명시적 import 필요 기능 경고
  "-unchecked"         // 타입 소거 경고
)
```

### 1.6.2 주요 SBT 명령어

```bash
sbt compile         # 컴파일
sbt run            # 실행
sbt test           # 테스트 실행
sbt console        # REPL 시작
sbt clean          # 빌드 결과 삭제
sbt package        # JAR 파일 생성
```

**Maven/Gradle과 비교:**
- `sbt compile` ≈ `mvn compile` / `gradle compileScala`
- `sbt test` ≈ `mvn test` / `gradle test`
- `sbt package` ≈ `mvn package` / `gradle jar`

---

## 1.7 Scala 2.12 vs Scala 3 주요 차이

이 가이드는 Scala 2.12를 기준으로 하지만, Scala 3의 주요 변경사항을 알아둘 필요가 있습니다.

| 기능 | Scala 2.12 | Scala 3 |
|------|------------|---------|
| **메인 메서드** | `object O extends App` | `@main def main() = ...` |
| **암시적 변환** | `implicit val/def` | `given`/`using` |
| **Enum** | 없음 (sealed trait 사용) | `enum Color { case Red, Blue }` |
| **중괄호** | 필수 | 선택 (들여쓰기로 대체 가능) |
| **타입 람다** | `({ type L[A] = F[G[A]] })#L` | `[A] =>> F[G[A]]` |

**예시:**
```scala
// Scala 2.12
object HelloScala extends App {
  println("Hello")
}

// Scala 3
@main def hello() =
  println("Hello")
```

---

## 1.8 실습 과제

### 과제 1-1: 계산기 프로그램

다음 요구사항을 만족하는 `Calculator.scala`를 작성하세요:

1. `add`, `subtract`, `multiply`, `divide` 함수 구현
2. `main` 메서드에서 각 함수 테스트
3. 0으로 나누기 예외 처리

**힌트:**
```scala
object Calculator extends App {
  def add(a: Int, b: Int): Int = ???  // 구현하세요

  // 테스트
  println(s"3 + 5 = ${add(3, 5)}")
}
```

### 과제 1-2: REPL 탐색

REPL에서 다음을 수행하고 결과를 관찰하세요:

```scala
scala> val x = 10
scala> :type x
scala> val doubled = x * 2
scala> def factorial(n: Int): Int = if (n <= 1) 1 else n * factorial(n - 1)
scala> factorial(5)
```

---

## 1.9 요약

이 챕터에서 배운 내용:

✅ Scala의 특징과 Java와의 차이점
✅ 개발 환경 설정 (Java, SBT, IntelliJ IDEA)
✅ 첫 Scala 프로그램 작성 및 실행
✅ REPL을 사용한 대화식 코딩
✅ 기본 문법: `val`/`var`, 함수, 타입 추론
✅ SBT 프로젝트 구조와 명령어

**다음 챕터 예고:**
Chapter 2에서는 Scala의 변수와 타입 시스템을 깊이 있게 다룹니다.

---

## 참고 자료

- [Scala 공식 문서](https://docs.scala-lang.org/)
- [SBT 공식 문서](https://www.scala-sbt.org/)
- [Scala Exercises](https://www.scala-exercises.org/)
