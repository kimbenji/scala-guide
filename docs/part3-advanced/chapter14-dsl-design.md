# Chapter 14: DSL 설계 (Domain-Specific Language Design)

## 학습 목표
- DSL(도메인 특화 언어)의 개념과 장점 이해
- Internal DSL vs External DSL 차이점 학습
- Scala의 DSL 설계 기법 습득
- 실전 DSL 구현 패턴 학습
- 테스트 DSL, 빌더 DSL 등 실무 예제

---

## 13.1 DSL 개념

### 14.1.1 DSL이란?

**DSL (Domain-Specific Language)**: 특정 도메인에 특화된 언어

```scala
// Internal DSL: Scala 문법 활용
val query = select("name", "age") from "users" where "age" > 18

// External DSL: 별도 파서 필요
// SELECT name, age FROM users WHERE age > 18
```

### 14.1.2 Scala의 DSL 장점

1. **읽기 쉬운 코드**: 비즈니스 로직을 자연어처럼 표현
2. **타입 안전성**: 컴파일 타임 검증
3. **IDE 지원**: 자동 완성, 리팩토링
4. **유지보수**: 도메인 전문가도 이해 가능

---

## 14.2 DSL 설계 기법

### 14.2.1 메서드 체이닝

```scala
// HTTP 클라이언트 DSL
case class HttpRequest(
  url: String,
  method: String = "GET",
  headers: Map[String, String] = Map.empty,
  body: Option[String] = None
) {
  def withMethod(m: String): HttpRequest = copy(method = m)
  def withHeader(key: String, value: String): HttpRequest =
    copy(headers = headers + (key -> value))
  def withBody(b: String): HttpRequest = copy(body = Some(b))
  def execute(): HttpResponse = {
    // 실제 HTTP 요청 실행
    HttpResponse(200, "OK")
  }
}

case class HttpResponse(status: Int, body: String)

object Http {
  def get(url: String): HttpRequest = HttpRequest(url, "GET")
  def post(url: String): HttpRequest = HttpRequest(url, "POST")
}

// 사용
val response = Http.post("https://api.example.com/users")
  .withHeader("Content-Type", "application/json")
  .withBody("""{"name":"Alice","age":25}""")
  .execute()

println(response)
```

### 14.2.2 중위 표기법 (Infix Notation)

```scala
// 수학 표현식 DSL
case class Expr(value: Int) {
  def +(other: Expr): Expr = Expr(value + other.value)
  def -(other: Expr): Expr = Expr(value - other.value)
  def *(other: Expr): Expr = Expr(value * other.value)
  def /(other: Expr): Expr = Expr(value / other.value)
}

implicit def intToExpr(i: Int): Expr = Expr(i)

// 중위 표기법 사용
val result = 10 + 20 * 3 - 5
println(result)  // Expr(65)
```

### 14.2.3 암시적 변환

```scala
// 시간 표현 DSL
case class Duration(seconds: Long)

implicit class IntDuration(val value: Int) extends AnyVal {
  def second: Duration = Duration(value)
  def seconds: Duration = Duration(value)
  def minute: Duration = Duration(value * 60)
  def minutes: Duration = Duration(value * 60)
  def hour: Duration = Duration(value * 3600)
  def hours: Duration = Duration(value * 3600)
}

// 자연어처럼 사용
val timeout1 = 30.seconds
val timeout2 = 5.minutes
val timeout3 = 2.hours

println(timeout1)  // Duration(30)
println(timeout2)  // Duration(300)
println(timeout3)  // Duration(7200)
```

### 14.2.4 By-Name 파라미터

```scala
// 테스트 DSL
class TestSuite(name: String) {
  def test(description: String)(body: => Unit): Unit = {
    try {
      body
      println(s"✓ $description")
    } catch {
      case e: AssertionError =>
        println(s"✗ $description: ${e.getMessage}")
    }
  }
}

def describe(name: String)(body: TestSuite => Unit): Unit = {
  println(s"\n$name")
  body(new TestSuite(name))
}

// 사용
describe("Calculator") { suite =>
  suite.test("should add two numbers") {
    assert(1 + 1 == 2)
  }

  suite.test("should multiply two numbers") {
    assert(2 * 3 == 6)
  }
}
```

---

## 14.3 실전 예제: SQL 쿼리 DSL

```scala
// 타입 안전한 SQL DSL
case class Column(name: String) {
  def ===(value: Any): Condition = EqualCondition(this, value)
  def >(value: Int): Condition = GreaterThanCondition(this, value)
  def <(value: Int): Condition = LessThanCondition(this, value)
}

sealed trait Condition
case class EqualCondition(column: Column, value: Any) extends Condition
case class GreaterThanCondition(column: Column, value: Int) extends Condition
case class LessThanCondition(column: Column, value: Int) extends Condition
case class AndCondition(left: Condition, right: Condition) extends Condition
case class OrCondition(left: Condition, right: Condition) extends Condition

case class SelectQuery(
  columns: List[String],
  table: String,
  whereClause: Option[Condition] = None
) {
  def toSql: String = {
    val cols = columns.mkString(", ")
    val where = whereClause.map(renderCondition).map(" WHERE " + _).getOrElse("")
    s"SELECT $cols FROM $table$where"
  }

  private def renderCondition(cond: Condition): String = cond match {
    case EqualCondition(col, value) => s"${col.name} = '$value'"
    case GreaterThanCondition(col, value) => s"${col.name} > $value"
    case LessThanCondition(col, value) => s"${col.name} < $value"
    case AndCondition(left, right) => s"${renderCondition(left)} AND ${renderCondition(right)}"
    case OrCondition(left, right) => s"${renderCondition(left)} OR ${renderCondition(right)}"
  }
}

// DSL 빌더
object SQL {
  def select(columns: String*): SelectBuilder = SelectBuilder(columns.toList)
}

case class SelectBuilder(columns: List[String]) {
  def from(table: String): FromBuilder = FromBuilder(columns, table)
}

case class FromBuilder(columns: List[String], table: String) {
  def where(condition: Condition): SelectQuery =
    SelectQuery(columns, table, Some(condition))

  def build: SelectQuery = SelectQuery(columns, table)
}

// 컬럼 정의
val name = Column("name")
val age = Column("age")
val city = Column("city")

// 사용
val query1 = SQL.select("name", "age")
  .from("users")
  .where(age > 18)

println(query1.toSql)
// SELECT name, age FROM users WHERE age > 18

val query2 = SQL.select("*")
  .from("users")
  .where(AndCondition(age > 18, EqualCondition(city, "Seoul")))

println(query2.toSql)
// SELECT * FROM users WHERE age > 18 AND city = 'Seoul'
```

---

## 14.4 실전 예제: HTML 빌더 DSL

```scala
// HTML 생성 DSL
sealed trait HtmlElement {
  def render: String
}

case class Tag(name: String, attributes: Map[String, String], children: List[HtmlElement]) extends HtmlElement {
  def render: String = {
    val attrs = if (attributes.isEmpty) ""
                else attributes.map { case (k, v) => s"""$k="$v"""" }.mkString(" ", " ", "")
    val childHtml = children.map(_.render).mkString

    if (children.isEmpty) s"<$name$attrs/>"
    else s"<$name$attrs>$childHtml</$name>"
  }
}

case class Text(content: String) extends HtmlElement {
  def render: String = content
}

// DSL 빌더
class HtmlBuilder {
  private var elements = List.empty[HtmlElement]

  def tag(name: String, attrs: (String, String)*)(body: HtmlBuilder => Unit): HtmlElement = {
    val builder = new HtmlBuilder
    body(builder)
    Tag(name, attrs.toMap, builder.elements)
  }

  def text(content: String): Unit = {
    elements = elements :+ Text(content)
  }

  def +=(element: HtmlElement): Unit = {
    elements = elements :+ element
  }

  // 편의 메서드
  def div(attrs: (String, String)*)(body: HtmlBuilder => Unit): HtmlElement =
    tag("div", attrs: _*)(body)

  def p(attrs: (String, String)*)(body: HtmlBuilder => Unit): HtmlElement =
    tag("p", attrs: _*)(body)

  def a(href: String, attrs: (String, String)*)(body: HtmlBuilder => Unit): HtmlElement =
    tag("a", ("href" -> href) +: attrs: _*)(body)

  def span(body: HtmlBuilder => Unit): HtmlElement =
    tag("span")(body)
}

object Html {
  def build(body: HtmlBuilder => Unit): String = {
    val builder = new HtmlBuilder
    body(builder)
    builder.elements.map(_.render).mkString
  }
}

// 사용
val html = Html.build { page =>
  page += page.div("class" -> "container") { container =>
    container += container.p() { p =>
      p.text("Hello, ")
      p += p.span { span =>
        span.text("World")
      }
      p.text("!")
    }
    container += container.a("https://example.com") { link =>
      link.text("Click here")
    }
  }
}

println(html)
// <div class="container"><p>Hello, <span>World</span>!</p><a href="https://example.com">Click here</a></div>
```

---

## 14.5 실전 예제: 설정 DSL

```scala
// 애플리케이션 설정 DSL
case class DatabaseConfig(
  host: String,
  port: Int,
  database: String,
  username: String,
  password: String
)

case class ServerConfig(
  host: String,
  port: Int,
  timeout: Duration
)

case class AppConfig(
  database: DatabaseConfig,
  server: ServerConfig,
  features: Map[String, Boolean]
)

// DSL 빌더
class ConfigBuilder {
  private var dbConfig: Option[DatabaseConfig] = None
  private var srvConfig: Option[ServerConfig] = None
  private var features: Map[String, Boolean] = Map.empty

  def database(body: DatabaseBuilder => Unit): Unit = {
    val builder = new DatabaseBuilder
    body(builder)
    dbConfig = Some(builder.build)
  }

  def server(body: ServerBuilder => Unit): Unit = {
    val builder = new ServerBuilder
    body(builder)
    srvConfig = Some(builder.build)
  }

  def feature(name: String, enabled: Boolean): Unit = {
    features += (name -> enabled)
  }

  def build: AppConfig = {
    AppConfig(
      dbConfig.getOrElse(throw new IllegalStateException("Database config required")),
      srvConfig.getOrElse(throw new IllegalStateException("Server config required")),
      features
    )
  }
}

class DatabaseBuilder {
  private var host: String = "localhost"
  private var port: Int = 5432
  private var database: String = _
  private var username: String = _
  private var password: String = _

  def setHost(h: String): Unit = { host = h }
  def setPort(p: Int): Unit = { port = p }
  def setDatabase(d: String): Unit = { database = d }
  def setUsername(u: String): Unit = { username = u }
  def setPassword(p: String): Unit = { password = p }

  def build: DatabaseConfig = DatabaseConfig(host, port, database, username, password)
}

class ServerBuilder {
  private var host: String = "0.0.0.0"
  private var port: Int = 8080
  private var timeout: Duration = Duration(30)

  def setHost(h: String): Unit = { host = h }
  def setPort(p: Int): Unit = { port = p }
  def setTimeout(t: Duration): Unit = { timeout = t }

  def build: ServerConfig = ServerConfig(host, port, timeout)
}

object Config {
  def apply(body: ConfigBuilder => Unit): AppConfig = {
    val builder = new ConfigBuilder
    body(builder)
    builder.build
  }
}

// 사용
val config = Config { app =>
  app.database { db =>
    db.setHost("db.example.com")
    db.setPort(5432)
    db.setDatabase("myapp")
    db.setUsername("admin")
    db.setPassword("secret")
  }

  app.server { srv =>
    srv.setHost("0.0.0.0")
    srv.setPort(8080)
    srv.setTimeout(60.seconds)
  }

  app.feature("logging", enabled = true)
  app.feature("metrics", enabled = true)
  app.feature("experimental", enabled = false)
}

println(config)
```

---

## 14.6 ScalaTest DSL

```scala
// ScalaTest의 FlatSpec DSL
import org.scalatest.flatspec.AnyFlatSpec
import org.scalatest.matchers.should.Matchers

class CalculatorSpec extends AnyFlatSpec with Matchers {
  "A Calculator" should "add two numbers" in {
    val result = 1 + 1
    result should be(2)
  }

  it should "multiply two numbers" in {
    val result = 2 * 3
    result should be(6)
  }

  it should "handle division by zero" in {
    an[ArithmeticException] should be thrownBy {
      1 / 0
    }
  }
}
```

---

## 14.7 DSL 설계 원칙

### 14.7.1 가독성 우선

```scala
// ❌ 나쁜 예: 복잡한 메서드 이름
query.selectColumnsFromTableWhereConditionIsTrue(List("name"), "users", age > 18)

// ✅ 좋은 예: 자연스러운 흐름
select("name") from "users" where (age > 18)
```

### 14.7.2 타입 안전성

```scala
// ❌ 나쁜 예: 문자열 기반 (런타임 에러)
sql"SELECT * FROM $tableName WHERE $columnName = $value"

// ✅ 좋은 예: 타입 안전 (컴파일 타임 검증)
select("*").from(Users).where(Users.name === "Alice")
```

### 14.7.3 단순성 유지

```scala
// DSL은 복잡한 기능보다 명확한 의도 표현에 집중
// 너무 많은 기능을 넣으면 오히려 혼란
```

---

## 14.8 Java 개발자를 위한 팁

### 14.8.1 Builder Pattern과 비교

```java
// Java Builder Pattern
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .header("Content-Type", "application/json")
    .POST(BodyPublishers.ofString("{\"name\":\"Alice\"}"))
    .build();
```

```scala
// Scala DSL (더 유연하고 간결)
val request = Http.post("https://example.com")
  .withHeader("Content-Type", "application/json")
  .withBody("""{"name":"Alice"}""")
```

---

## 14.9 실습 과제

### 과제 14-1: JSON 빌더 DSL

JSON 생성 DSL을 구현하세요:

```scala
val json = obj(
  "name" -> "Alice",
  "age" -> 25,
  "address" -> obj(
    "city" -> "Seoul",
    "zip" -> "12345"
  ),
  "hobbies" -> arr("reading", "coding", "music")
)

assert(json.toString == """{"name":"Alice","age":25,"address":{"city":"Seoul","zip":"12345"},"hobbies":["reading","coding","music"]}""")
```

### 과제 14-2: 날짜 연산 DSL

날짜 계산 DSL을 구현하세요:

```scala
val today = LocalDate.now()
val future = today + 3.days + 2.weeks - 1.month

implicit class DateOps(val date: LocalDate) extends AnyVal {
  def +(duration: ???): LocalDate = ???
  def -(duration: ???): LocalDate = ???
}
```

---

## 14.10 요약

이번 챕터에서 학습한 내용:

1. **DSL 개념**: 도메인 특화 언어로 가독성 향상
2. **설계 기법**: 메서드 체이닝, 중위 표기법, 암시적 변환
3. **실전 예제**: SQL, HTML, 설정 DSL
4. **설계 원칙**: 가독성, 타입 안전성, 단순성

**다음 챕터 예고**: Part 4에서 Apache Spark를 활용한 빅데이터 처리를 학습합니다.

---

## Scala 3 주요 차이점

- **Extension Methods**: implicit class 대신 extension 사용
- **Infix Operators**: 더 엄격한 규칙
- **Contextual Abstractions**: given/using으로 더 명확한 DSL 구현
