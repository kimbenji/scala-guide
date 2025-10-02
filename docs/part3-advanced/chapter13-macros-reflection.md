# Chapter 13: 매크로와 리플렉션 (Macros and Reflection)

## 학습 목표
- Scala 매크로의 개념과 사용 사례 이해
- 런타임 리플렉션 활용법 학습
- 타입 태그와 클래스 태그 사용
- 컴파일 타임 메타프로그래밍 기초
- 실무에서의 매크로 활용 패턴

---

## 13.1 리플렉션 기초

### 13.1.1 TypeTag와 ClassTag

```scala
import scala.reflect.runtime.universe._

// TypeTag: 컴파일 타임 타입 정보 유지
def printType[T: TypeTag](value: T): Unit = {
  val tpe = typeOf[T]
  println(s"Type: $tpe")
  println(s"Type arguments: ${tpe.typeArgs}")
}

printType(List(1, 2, 3))
// Type: List[Int]
// Type arguments: List(Int)

// ClassTag: 런타임 타입 정보 (Type Erasure 회피)
import scala.reflect.ClassTag

def createArray[T: ClassTag](size: Int, default: T): Array[T] = {
  Array.fill(size)(default)
}

val intArray = createArray(5, 0)
val stringArray = createArray(5, "default")

println(intArray.mkString(", "))      // 0, 0, 0, 0, 0
println(stringArray.mkString(", "))   // default, default, ...
```

```java
// Java에서는 Type Erasure로 인해 런타임에 제네릭 타입 정보 손실
// Scala의 TypeTag/ClassTag가 이를 해결
```

### 13.1.2 런타임 리플렉션

```scala
import scala.reflect.runtime.universe._

case class Person(name: String, age: Int)

// 타입 정보 추출
val tpe = typeOf[Person]
println(s"Type: ${tpe.typeSymbol.name}")

// 멤버 정보 조회
val members = tpe.members
members.foreach(m => println(s"Member: ${m.name}"))

// 생성자 찾기
val constructor = tpe.decl(termNames.CONSTRUCTOR).asMethod
println(s"Constructor: $constructor")

// 인스턴스 생성
val mirror = runtimeMirror(getClass.getClassLoader)
val classPerson = tpe.typeSymbol.asClass
val cm = mirror.reflectClass(classPerson)
val ctor = cm.reflectConstructor(constructor)
val instance = ctor("Alice", 25).asInstanceOf[Person]
println(instance)  // Person(Alice,25)
```

### 13.1.3 메서드 호출

```scala
case class Calculator(x: Int) {
  def add(y: Int): Int = x + y
  def multiply(y: Int): Int = x * y
}

val calc = Calculator(10)

// 리플렉션으로 메서드 호출
val mirror = runtimeMirror(getClass.getClassLoader)
val instanceMirror = mirror.reflect(calc)

val methodSymbol = typeOf[Calculator].decl(TermName("add")).asMethod
val methodMirror = instanceMirror.reflectMethod(methodSymbol)

val result = methodMirror(5)
println(result)  // 15
```

---

## 13.2 Scala 매크로 개요

### 13.2.1 매크로란?

**매크로(Macro)**: 컴파일 타임에 실행되는 코드 생성기

```scala
// 매크로 정의는 별도 모듈 필요 (Scala 2.x)
// build.sbt:
// libraryDependencies += "org.scala-lang" % "scala-reflect" % scalaVersion.value

import scala.language.experimental.macros
import scala.reflect.macros.blackbox

object DebugMacros {
  // 매크로 인터페이스
  def debug(expr: Any): Unit = macro debugImpl

  // 매크로 구현
  def debugImpl(c: blackbox.Context)(expr: c.Expr[Any]): c.Expr[Unit] = {
    import c.universe._
    val exprString = show(expr.tree)
    val valueExpr = expr
    reify {
      println(s"$exprString = ${valueExpr.splice}")
    }
  }
}

// 사용
import DebugMacros._

val x = 10
val y = 20
debug(x + y)  // 컴파일 타임에 "x + y = 30" 코드 생성
```

### 13.2.2 매크로 사용 사례

1. **로깅 최적화**
2. **코드 생성** (Boilerplate 제거)
3. **컴파일 타임 검증**
4. **DSL 구현**

---

## 13.3 실전 예제: JSON 직렬화 매크로

```scala
import scala.language.experimental.macros
import scala.reflect.macros.blackbox

// JSON 직렬화 타입 클래스
trait JsonWriter[T] {
  def write(value: T): String
}

object JsonWriter {
  // 매크로로 자동 생성
  implicit def materialize[T]: JsonWriter[T] = macro materializeImpl[T]

  def materializeImpl[T: c.WeakTypeTag](c: blackbox.Context): c.Expr[JsonWriter[T]] = {
    import c.universe._

    val tpe = weakTypeOf[T]
    val fields = tpe.decls.collect {
      case m: MethodSymbol if m.isCaseAccessor => m
    }

    val fieldSerialization = fields.map { field =>
      val name = field.name.toString
      val fieldType = field.returnType
      q"""${name} + ": " + value.${field.name}.toString"""
    }

    val result = q"""
      new JsonWriter[$tpe] {
        def write(value: $tpe): String = {
          "{" + List(..$fieldSerialization).mkString(", ") + "}"
        }
      }
    """

    c.Expr[JsonWriter[T]](result)
  }
}

// 사용
case class User(name: String, age: Int, email: String)

val user = User("Alice", 25, "alice@example.com")
val writer = implicitly[JsonWriter[User]]
println(writer.write(user))
// {"name": Alice, "age": 25, "email": alice@example.com"}
```

---

## 13.4 Compile-Time Validation

### 13.4.1 정규식 검증 매크로

```scala
object RegexMacros {
  def validateRegex(pattern: String): Boolean = macro validateRegexImpl

  def validateRegexImpl(c: blackbox.Context)(pattern: c.Expr[String]): c.Expr[Boolean] = {
    import c.universe._

    pattern.tree match {
      case Literal(Constant(p: String)) =>
        try {
          p.r  // 컴파일 타임에 정규식 검증
          reify(true)
        } catch {
          case e: Exception =>
            c.abort(c.enclosingPosition, s"Invalid regex: ${e.getMessage}")
        }
      case _ =>
        c.abort(c.enclosingPosition, "Pattern must be a string literal")
    }
  }
}

// 사용
validateRegex("[a-z]+")      // ✅ 컴파일 성공
// validateRegex("[a-z")     // ❌ 컴파일 에러: Invalid regex
```

### 13.4.2 SQL 쿼리 검증 (개념)

```scala
// Slick, Quill 등의 라이브러리에서 사용하는 패턴
object SqlMacros {
  def sql(query: String): Query = macro sqlImpl

  def sqlImpl(c: blackbox.Context)(query: c.Expr[String]): c.Expr[Query] = {
    // 컴파일 타임에 SQL 문법 검증
    // 실제 구현은 복잡하므로 개념만 설명
    ???
  }
}
```

---

## 13.5 실무 활용 패턴

### 13.5.1 자동 Ordering 생성

```scala
// Case class 필드 순서대로 Ordering 자동 생성
implicit def caseClassOrdering[T]: Ordering[T] = macro orderingImpl[T]

def orderingImpl[T: c.WeakTypeTag](c: blackbox.Context): c.Expr[Ordering[T]] = {
  import c.universe._

  val tpe = weakTypeOf[T]
  val fields = tpe.decls.collect {
    case m: MethodSymbol if m.isCaseAccessor => m
  }

  // 첫 번째 필드로 정렬 (단순화)
  val firstField = fields.head
  q"""
    Ordering.by[$tpe, ${firstField.returnType}](_.${firstField.name})
  """
}
```

### 13.5.2 성능 측정 매크로

```scala
object PerformanceMacros {
  def timed[T](expr: T): T = macro timedImpl[T]

  def timedImpl[T: c.WeakTypeTag](c: blackbox.Context)(expr: c.Expr[T]): c.Expr[T] = {
    import c.universe._

    reify {
      val start = System.nanoTime()
      val result = expr.splice
      val end = System.nanoTime()
      println(s"Execution time: ${(end - start) / 1e6} ms")
      result
    }
  }
}

// 사용
import PerformanceMacros._

val result = timed {
  Thread.sleep(100)
  42
}
// Execution time: ~100 ms
```

---

## 13.6 매크로 라이브러리

### 13.6.1 Shapeless

```scala
// Shapeless: 제네릭 프로그래밍 라이브러리
import shapeless._

case class User(name: String, age: Int)
case class Person(name: String, age: Int, email: String)

val user = User("Alice", 25)

// Generic Representation
val gen = Generic[User]
val repr = gen.to(user)  // String :: Int :: HNil

println(repr)  // Alice :: 25 :: HNil
```

### 13.6.2 Magnolia

```scala
// Magnolia: 타입 클래스 자동 생성
// 간단한 예제 (실제 사용은 라이브러리 참조)
import magnolia._

trait Show[T] {
  def show(value: T): String
}

object Show {
  type Typeclass[T] = Show[T]

  def combine[T](ctx: CaseClass[Show, T]): Show[T] = new Show[T] {
    def show(value: T): String = {
      ctx.parameters.map { p =>
        s"${p.label}=${p.typeclass.show(p.dereference(value))}"
      }.mkString("{", ", ", "}")
    }
  }

  implicit def gen[T]: Show[T] = macro Magnolia.gen[T]
}
```

---

## 13.7 Scala 3 Macros (Inline & Quotes)

### 13.7.1 Inline 함수

```scala
// Scala 3: inline 키워드
inline def debug(inline expr: Any): Unit = {
  println(s"${expr} = ${expr}")
}

// 사용
val x = 42
debug(x + 10)  // "x + 10 = 52"
```

### 13.7.2 Quotes와 Splices

```scala
// Scala 3 매크로 (새 문법)
import scala.quoted._

inline def show(inline expr: Any): String = ${showImpl('expr)}

def showImpl(expr: Expr[Any])(using Quotes): Expr[String] = {
  import quotes.reflect._
  val tree = expr.asTerm
  Expr(tree.show)
}
```

---

## 13.8 Java 개발자를 위한 팁

### 13.8.1 Java Reflection과 비교

| 기능 | Scala Reflection | Java Reflection |
|------|------------------|-----------------|
| 타입 정보 | TypeTag (제네릭 유지) | Class (Type Erasure) |
| 컴파일 타임 | Macros | Annotation Processing |
| 메서드 호출 | reflectMethod | invoke |
| 성능 | 상대적으로 느림 | 상대적으로 느림 |

### 13.8.2 Annotation Processing vs Macros

```java
// Java Annotation Processing (컴파일 타임)
@AutoValue
abstract class Person {
    abstract String name();
    abstract int age();
}
// 컴파일러가 구현 클래스 자동 생성
```

```scala
// Scala Macros (컴파일 타임)
implicit def jsonWriter[T]: JsonWriter[T] = macro materialize[T]
// 컴파일러가 JsonWriter 구현 자동 생성
```

---

## 13.9 실습 과제

### 과제 13-1: Enum 생성 매크로

Sealed trait의 모든 케이스를 자동으로 리스트로 반환:

```scala
sealed trait Color
case object Red extends Color
case object Green extends Color
case object Blue extends Color

def enumValues[T]: List[T] = macro enumValuesImpl[T]

// 테스트
val colors = enumValues[Color]
assert(colors == List(Red, Green, Blue))
```

### 과제 13-2: ToString 매크로

Case class의 필드를 포맷팅하는 toString 생성:

```scala
case class Person(name: String, age: Int)

def prettyPrint[T](value: T): String = macro prettyPrintImpl[T]

// 테스트
val person = Person("Alice", 25)
assert(prettyPrint(person) == "Person { name: Alice, age: 25 }")
```

---

## 13.10 요약

이번 챕터에서 학습한 내용:

1. **TypeTag/ClassTag**: 타입 정보 유지
2. **런타임 리플렉션**: 동적 메서드 호출, 인스턴스 생성
3. **매크로**: 컴파일 타임 코드 생성
4. **실무 활용**: JSON 직렬화, 검증, 성능 측정
5. **Scala 3**: Inline과 Quotes/Splices

**다음 챕터 예고**: Chapter 14에서는 DSL(Domain-Specific Language) 설계를 학습합니다.

---

## Scala 3 주요 차이점

- **Def Macro 제거**: Scala 2 스타일 매크로 deprecated
- **Inline Macros**: 새로운 매크로 시스템
- **Quotes & Splices**: `'{}`, `${}` 문법
- **더 안전하고 간결한 문법**
