# Chapter 16: Scala 생태계와 도구

## 학습 목표
- SBT 빌드 시스템 심화
- ScalaTest를 활용한 효과적인 테스팅
- Scala-Java 상호운용성 이해
- 주요 라이브러리 및 프레임워크 소개
- 프로덕션 배포 베스트 프랙티스

---

## 16.1 SBT (Simple Build Tool)

### 16.1.1 build.sbt 구조

```scala
// build.sbt
name := "scala-learning-guide"
version := "1.0.0"
scalaVersion := "2.12.18"

// 의존성 설정
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % "3.5.0",
  "org.apache.spark" %% "spark-sql" % "3.5.0",
  "org.scalatest" %% "scalatest" % "3.2.15" % Test,
  "com.typesafe" % "config" % "1.4.2",
  "ch.qos.logback" % "logback-classic" % "1.4.7"
)

// 컴파일러 옵션
scalacOptions ++= Seq(
  "-deprecation",
  "-feature",
  "-unchecked",
  "-Xlint",
  "-Ywarn-unused"
)

// Java 버전
javacOptions ++= Seq("-source", "11", "-target", "11")
```

### 16.1.2 멀티 프로젝트 설정

```scala
// build.sbt (루트)
lazy val root = (project in file("."))
  .aggregate(core, api, worker)

lazy val core = (project in file("core"))
  .settings(
    name := "scala-guide-core",
    libraryDependencies ++= commonDependencies
  )

lazy val api = (project in file("api"))
  .dependsOn(core)
  .settings(
    name := "scala-guide-api",
    libraryDependencies ++= apiDependencies
  )

lazy val worker = (project in file("worker"))
  .dependsOn(core)
  .settings(
    name := "scala-guide-worker",
    libraryDependencies ++= workerDependencies
  )

// 공통 의존성
val commonDependencies = Seq(
  "com.typesafe.scala-logging" %% "scala-logging" % "3.9.5",
  "org.scalatest" %% "scalatest" % "3.2.15" % Test
)

val apiDependencies = Seq(
  "com.typesafe.akka" %% "akka-http" % "10.5.0",
  "com.typesafe.akka" %% "akka-stream" % "2.8.0"
)

val workerDependencies = Seq(
  "org.apache.kafka" %% "kafka" % "3.4.0"
)
```

### 16.1.3 SBT 플러그인

```scala
// project/plugins.sbt
addSbtPlugin("org.scalameta" % "sbt-scalafmt" % "2.5.0")  // 코드 포매팅
addSbtPlugin("org.scoverage" % "sbt-scoverage" % "2.0.7") // 코드 커버리지
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "2.1.1")   // Fat JAR 생성
addSbtPlugin("com.typesafe.sbt" % "sbt-native-packager" % "1.9.16") // 패키징
```

### 16.1.4 SBT 명령어

```bash
# 컴파일
sbt compile

# 테스트 실행
sbt test

# 특정 테스트만 실행
sbt "testOnly *UserServiceSpec"

# 지속적 컴파일 (파일 변경 감지)
sbt ~compile

# 의존성 트리 보기
sbt dependencyTree

# 패키지 생성
sbt package

# Fat JAR 생성
sbt assembly

# 프로젝트 클린
sbt clean

# REPL 시작
sbt console
```

---

## 16.2 ScalaTest

### 16.2.1 테스트 스타일

```scala
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

// FlatSpec: BDD 스타일
class CalculatorSpec extends AnyFlatSpec with Matchers {
  "A Calculator" should "add two numbers" in {
    val result = Calculator.add(2, 3)
    result should be(5)
  }

  it should "subtract two numbers" in {
    val result = Calculator.subtract(5, 3)
    result should be(2)
  }

  it should "multiply two numbers" in {
    val result = Calculator.multiply(2, 3)
    result should be(6)
  }

  it should "handle division by zero" in {
    an[ArithmeticException] should be thrownBy {
      Calculator.divide(1, 0)
    }
  }
}

object Calculator {
  def add(a: Int, b: Int): Int = a + b
  def subtract(a: Int, b: Int): Int = a - b
  def multiply(a: Int, b: Int): Int = a * b
  def divide(a: Int, b: Int): Int = a / b
}
```

### 16.2.2 Matchers

```scala
import org.scalatest.matchers.should.Matchers._

// 동등성
result should be(expected)
result shouldBe expected
result should equal(expected)

// 부정
result should not be expected
result shouldNot be(expected)

// 컬렉션
list should contain("element")
list should have size 3
list should be(empty)
list should contain allOf ("a", "b", "c")
list should contain oneOf ("a", "b", "c")

// Option
option should be(defined)
option should be(empty)
option should contain(value)

// 예외
an[Exception] should be thrownBy { /* code */ }
the[Exception] thrownBy { /* code */ } should have message "error"

// 문자열
string should startWith("prefix")
string should endWith("suffix")
string should include("substring")
string should fullyMatch regex """[a-z]+""".r
```

### 16.2.3 비동기 테스팅

```scala
import org.scalatest.concurrent.ScalaFutures
import scala.concurrent.Future
import scala.concurrent.ExecutionContext.Implicits.global

class AsyncServiceSpec extends AnyFlatSpec with Matchers with ScalaFutures {
  "AsyncService" should "fetch user data" in {
    val future = AsyncService.getUser(1)

    whenReady(future) { user =>
      user.id should be(1)
      user.name should not be empty
    }
  }

  it should "handle errors gracefully" in {
    val future = AsyncService.getUser(-1)

    whenReady(future.failed) { ex =>
      ex shouldBe a[IllegalArgumentException]
    }
  }
}

object AsyncService {
  def getUser(id: Int): Future[User] = Future {
    if (id > 0) User(id, s"User$id")
    else throw new IllegalArgumentException("Invalid ID")
  }
}

case class User(id: Int, name: String)
```

### 16.2.4 속성 기반 테스팅 (Property-Based Testing)

```scala
import org.scalatest.propspec.AnyPropSpec
import org.scalatestplus.scalacheck.ScalaCheckPropertyChecks

class StringUtilsSpec extends AnyPropSpec with ScalaCheckPropertyChecks {
  property("reversing a string twice gives original string") {
    forAll { (s: String) =>
      val reversed = s.reverse.reverse
      assert(reversed == s)
    }
  }

  property("string length equals char count") {
    forAll { (s: String) =>
      assert(s.length == s.toList.size)
    }
  }
}
```

---

## 16.3 Scala-Java 상호운용

### 16.3.1 Java 클래스를 Scala에서 사용

```scala
// Java 클래스
import java.util.{List => JList, ArrayList}
import java.util.HashMap

// Java 컬렉션을 Scala로 변환
import scala.jdk.CollectionConverters._

val javaList: JList[String] = new ArrayList[String]()
javaList.add("Java")
javaList.add("Scala")

// Java List → Scala List
val scalaList: List[String] = javaList.asScala.toList
println(scalaList)  // List(Java, Scala)

// Scala List → Java List
val backToJava: JList[String] = scalaList.asJava
```

### 16.3.2 Scala 클래스를 Java에서 사용

```scala
// Scala 클래스 (Java 친화적으로 작성)
import scala.beans.BeanProperty

class ScalaUser(@BeanProperty var name: String, @BeanProperty var age: Int) {
  def this() = this("", 0)  // 기본 생성자 (Java 호환)

  def greet(): String = s"Hello, $name"
}

// @BeanProperty는 Java 스타일 getter/setter 생성
// getName(), setName(String), getAge(), setAge(int)
```

```java
// Java에서 사용
public class JavaClient {
    public static void main(String[] args) {
        ScalaUser user = new ScalaUser("Alice", 25);
        System.out.println(user.getName());  // Alice
        user.setAge(26);
        System.out.println(user.greet());    // Hello, Alice
    }
}
```

### 16.3.3 주의사항

```scala
// ❌ Java에서 사용 불가: 기본 파라미터
def greet(name: String = "World"): String = s"Hello, $name"

// ✅ Java에서 사용 가능: 오버로딩
def greet(name: String): String = s"Hello, $name"
def greet(): String = greet("World")

// ❌ Java에서 사용 불가: Option
def findUser(id: Int): Option[User] = ???

// ✅ Java에서 사용 가능: null 허용 또는 Optional
import java.util.Optional
def findUser(id: Int): Optional[User] = ???
```

---

## 16.4 주요 라이브러리 및 프레임워크

### 16.4.1 Cats (함수형 프로그래밍)

```scala
// build.sbt
libraryDependencies += "org.typelevel" %% "cats-core" % "2.9.0"

import cats._
import cats.implicits._

// Semigroup
val result1 = 1 |+| 2 |+| 3  // 6
val result2 = "Hello" |+| " " |+| "World"  // "Hello World"

// Functor
val option = Some(5)
val mapped = option.map(_ * 2)  // Some(10)

// Monad
val result3 = for {
  a <- Some(1)
  b <- Some(2)
  c <- Some(3)
} yield a + b + c  // Some(6)
```

### 16.4.2 Akka (액터 모델)

```scala
// build.sbt
libraryDependencies ++= Seq(
  "com.typesafe.akka" %% "akka-actor-typed" % "2.8.0",
  "com.typesafe.akka" %% "akka-stream" % "2.8.0"
)

import akka.actor.typed.ActorSystem
import akka.actor.typed.scaladsl.Behaviors

object HelloWorld {
  def apply(): Behaviors.Receive[String] = Behaviors.receive { (context, message) =>
    context.log.info(s"Received: $message")
    Behaviors.same
  }
}

// 사용
val system = ActorSystem(HelloWorld(), "hello-world")
system ! "Hello"
system ! "World"
```

### 16.4.3 Play Framework (웹 프레임워크)

```scala
// build.sbt
libraryDependencies += "com.typesafe.play" %% "play" % "2.9.0"

// app/controllers/HomeController.scala
package controllers

import javax.inject._
import play.api.mvc._

@Singleton
class HomeController @Inject()(val controllerComponents: ControllerComponents) extends BaseController {
  def index() = Action { implicit request: Request[AnyContent] =>
    Ok("Hello from Play Framework!")
  }

  def user(id: Int) = Action {
    Ok(s"User ID: $id")
  }
}
```

### 16.4.4 Slick (데이터베이스 액세스)

```scala
// build.sbt
libraryDependencies ++= Seq(
  "com.typesafe.slick" %% "slick" % "3.4.1",
  "org.postgresql" % "postgresql" % "42.5.4"
)

import slick.jdbc.PostgresProfile.api._
import scala.concurrent.ExecutionContext.Implicits.global

// 테이블 정의
class Users(tag: Tag) extends Table[(Int, String, Int)](tag, "users") {
  def id = column[Int]("id", O.PrimaryKey, O.AutoInc)
  def name = column[String]("name")
  def age = column[Int]("age")
  def * = (id, name, age)
}

val users = TableQuery[Users]

// 쿼리
val db = Database.forConfig("mydb")
val query = users.filter(_.age > 18).result
val future = db.run(query)
```

---

## 16.5 코드 품질 도구

### 16.5.1 Scalafmt (코드 포매팅)

```scala
// .scalafmt.conf
version = "3.7.3"
runner.dialect = scala212

maxColumn = 100
indent.main = 2
indent.defnSite = 2

rewrite.rules = [
  RedundantBraces,
  RedundantParens,
  SortImports
]
```

```bash
# 포매팅 적용
sbt scalafmt

# 포매팅 검증
sbt scalafmtCheck
```

### 16.5.2 Scoverage (코드 커버리지)

```bash
# 커버리지 리포트 생성
sbt clean coverage test coverageReport
```

### 16.5.3 WartRemover (정적 분석)

```scala
// build.sbt
addCompilerPlugin("org.wartremover" %% "wartremover" % "3.0.9" cross CrossVersion.full)

wartremoverErrors ++= Warts.unsafe
```

---

## 16.6 프로덕션 배포

### 16.6.1 Fat JAR 생성

```scala
// build.sbt
// sbt-assembly 플러그인 사용

assembly / assemblyJarName := "app.jar"

assembly / assemblyMergeStrategy := {
  case PathList("META-INF", xs @ _*) => MergeStrategy.discard
  case x => MergeStrategy.first
}
```

```bash
sbt assembly
java -jar target/scala-2.12/app.jar
```

### 16.6.2 Docker 이미지 생성

```scala
// build.sbt
enablePlugins(JavaAppPackaging, DockerPlugin)

dockerBaseImage := "openjdk:11-jre-slim"
dockerExposedPorts := Seq(8080)
```

```bash
sbt docker:publishLocal
docker run -p 8080:8080 app:1.0.0
```

---

## 16.7 Java 개발자를 위한 팁

### 16.7.1 Maven vs SBT

| 기능 | Maven | SBT |
|------|-------|-----|
| 설정 파일 | XML (pom.xml) | Scala (build.sbt) |
| 의존성 | `<dependency>` | `libraryDependencies` |
| 플러그인 | XML 설정 | Scala 코드 |
| 멀티 모듈 | `<modules>` | `aggregate`, `dependsOn` |
| 빌드 | `mvn package` | `sbt package` |

### 16.7.2 JUnit vs ScalaTest

```java
// JUnit
@Test
public void testAddition() {
    assertEquals(5, Calculator.add(2, 3));
}
```

```scala
// ScalaTest (더 표현력 좋음)
"Calculator" should "add two numbers" in {
  Calculator.add(2, 3) should be(5)
}
```

---

## 16.8 실습 과제

### 과제 16-1: 멀티 모듈 프로젝트

다음 구조의 프로젝트를 SBT로 구성하세요:

```
root
├── core (공통 모델과 유틸리티)
├── api (REST API)
└── worker (백그라운드 작업)
```

### 과제 16-2: 통합 테스트

데이터베이스를 사용하는 통합 테스트 작성:

```scala
class UserRepositoryIntegrationSpec extends AnyFlatSpec with Matchers with BeforeAndAfterAll {
  var db: Database = _

  override def beforeAll(): Unit = {
    // 테스트 DB 초기화
  }

  override def afterAll(): Unit = {
    // 테스트 DB 정리
  }

  "UserRepository" should "save and retrieve users" in {
    // 테스트 구현
  }
}
```

### 과제 16-3: Java 라이브러리 래핑

Java 라이브러리를 Scala 친화적으로 래핑:

```scala
// Java의 java.util.concurrent.ExecutorService를 Scala Future로 래핑
```

---

## 16.9 요약

이번 챕터에서 학습한 내용:

1. **SBT**: 빌드 설정, 멀티 프로젝트, 플러그인
2. **ScalaTest**: 다양한 테스트 스타일, Matchers, 비동기 테스팅
3. **Scala-Java 상호운용**: 컬렉션 변환, BeanProperty, 주의사항
4. **주요 라이브러리**: Cats, Akka, Play, Slick
5. **프로덕션 배포**: Fat JAR, Docker

**축하합니다!** Scala 학습 가이드의 모든 챕터를 완료하셨습니다.

---

## 다음 단계

1. **실전 프로젝트 구현**: 배운 내용을 종합하여 실무 프로젝트 개발
2. **Scala 3 학습**: 새로운 기능과 문법 탐구
3. **커뮤니티 참여**: Scala Days, 로컬 밋업 참가
4. **오픈소스 기여**: Scala 라이브러리 개선에 참여

**추천 자료**:
- Scala 공식 문서: https://docs.scala-lang.org
- Scala Exercises: https://www.scala-exercises.org
- Coursera: Functional Programming in Scala
- Twitter Scala School
