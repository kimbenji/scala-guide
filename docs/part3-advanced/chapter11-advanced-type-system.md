# Chapter 11: 타입 시스템 고급 (Advanced Type System Deep Dive)

## 학습 목표
- Higher-Kinded Types 이해 및 활용
- Path-Dependent Types 패턴 학습
- Self Types와 Cake Pattern 이해
- Type Lambda와 Kind Projector 사용
- Existential Types 개념 이해

---

## 11.1 Higher-Kinded Types (고차 타입)

### 11.1.1 개념 이해

**Higher-Kinded Type**: 타입을 추상화하는 타입 생성자

```scala
// Kind-0: 구체적인 타입
val x: Int = 42
val y: String = "hello"

// Kind-1: 타입 생성자 (타입 파라미터 1개)
val list: List[Int] = List(1, 2, 3)
val opt: Option[String] = Some("hello")

// Kind-2: 고차 타입 생성자 (타입 파라미터를 받는 타입 생성자)
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// List, Option 등의 Functor 인스턴스
implicit val listFunctor: Functor[List] = new Functor[List] {
  def map[A, B](fa: List[A])(f: A => B): List[B] = fa.map(f)
}

implicit val optionFunctor: Functor[Option] = new Functor[Option] {
  def map[A, B](fa: Option[A])(f: A => B): Option[B] = fa.map(f)
}
```

```java
// Java에는 Higher-Kinded Types가 없음
// 인터페이스로 유사하게 구현 가능하지만 제약적
interface Functor<F> {
    <A, B> F map(F fa, Function<A, B> f);
}
// 하지만 F의 타입 파라미터를 표현할 방법이 없음
```

### 11.1.2 실전 예제: Monad 타입 클래스

```scala
// Monad 타입 클래스 정의
trait Monad[M[_]] {
  def pure[A](a: A): M[A]
  def flatMap[A, B](ma: M[A])(f: A => M[B]): M[B]

  // map은 flatMap으로 구현 가능
  def map[A, B](ma: M[A])(f: A => B): M[B] =
    flatMap(ma)(a => pure(f(a)))
}

// Option Monad
implicit val optionMonad: Monad[Option] = new Monad[Option] {
  def pure[A](a: A): Option[A] = Some(a)
  def flatMap[A, B](ma: Option[A])(f: A => Option[B]): Option[B] = ma.flatMap(f)
}

// List Monad
implicit val listMonad: Monad[List] = new Monad[List] {
  def pure[A](a: A): List[A] = List(a)
  def flatMap[A, B](ma: List[A])(f: A => List[B]): List[B] = ma.flatMap(f)
}

// Monad를 사용하는 제네릭 함수
def sequence[M[_]: Monad, A](list: List[M[A]]): M[List[A]] = {
  val monad = implicitly[Monad[M]]
  list.foldRight(monad.pure(List.empty[A])) { (ma, acc) =>
    monad.flatMap(ma)(a => monad.map(acc)(a :: _))
  }
}

// 사용 예제
val opts = List(Some(1), Some(2), Some(3))
println(sequence(opts))  // Some(List(1, 2, 3))

val optsWithNone = List(Some(1), None, Some(3))
println(sequence(optsWithNone))  // None
```

### 11.1.3 Functor Laws

```scala
// Functor는 다음 법칙을 만족해야 함
trait FunctorLaws[F[_]] {
  def functor: Functor[F]

  // Law 1: Identity
  // map(fa)(x => x) == fa
  def identityLaw[A](fa: F[A]): Boolean = {
    functor.map(fa)(identity) == fa
  }

  // Law 2: Composition
  // map(map(fa)(f))(g) == map(fa)(f andThen g)
  def compositionLaw[A, B, C](fa: F[A], f: A => B, g: B => C): Boolean = {
    functor.map(functor.map(fa)(f))(g) == functor.map(fa)(f andThen g)
  }
}
```

---

## 11.2 Path-Dependent Types (경로 의존 타입)

### 11.2.1 Inner Classes

```scala
// 외부 클래스 인스턴스에 의존하는 내부 타입
class Graph {
  class Node(val id: Int) {
    def connectTo(other: Node): Unit = {
      println(s"Connecting $id to ${other.id}")
    }
  }

  val nodes = scala.collection.mutable.Set[Node]()

  def newNode(id: Int): Node = {
    val node = new Node(id)
    nodes += node
    node
  }
}

val graph1 = new Graph
val graph2 = new Graph

val node1 = graph1.newNode(1)
val node2 = graph1.newNode(2)
val node3 = graph2.newNode(3)

node1.connectTo(node2)  // ✅ 같은 Graph의 Node
// node1.connectTo(node3)  // ❌ 컴파일 에러: 다른 Graph의 Node
```

```java
// Java의 Inner Class와 차이
public class Graph {
    class Node {
        int id;
        Node(int id) { this.id = id; }

        void connectTo(Node other) {
            // Java는 다른 Graph의 Node도 허용 (타입 안전성 낮음)
        }
    }
}
```

### 11.2.2 Type Projection

```scala
// Type Projection: 외부 인스턴스 독립적인 타입
class Outer {
  class Inner
}

val outer1 = new Outer
val outer2 = new Outer

// Path-dependent type
val inner1: outer1.Inner = new outer1.Inner
// val inner2: outer1.Inner = new outer2.Inner  // ❌ 타입 불일치

// Type projection
val inner3: Outer#Inner = new outer1.Inner
val inner4: Outer#Inner = new outer2.Inner  // ✅ 가능
```

### 11.2.3 실전 예제: 타입 안전한 데이터베이스 스키마

```scala
// 데이터베이스 스키마를 타입으로 표현
trait Database {
  type Table
  type Column

  def createTable(name: String): Table
  def addColumn(table: Table, name: String): Column
}

class PostgreSQL extends Database {
  case class PGTable(name: String)
  case class PGColumn(table: PGTable, name: String)

  type Table = PGTable
  type Column = PGColumn

  def createTable(name: String): Table = PGTable(name)
  def addColumn(table: Table, name: String): Column = PGColumn(table, name)
}

class MySQL extends Database {
  case class MySQLTable(name: String)
  case class MySQLColumn(table: MySQLTable, name: String)

  type Table = MySQLTable
  type Column = MySQLColumn

  def createTable(name: String): Table = MySQLTable(name)
  def addColumn(table: Table, name: String): Column = MySQLColumn(table, name)
}

// 타입 안전성 보장
val pg = new PostgreSQL
val mysql = new MySQL

val pgTable = pg.createTable("users")
val pgColumn = pg.addColumn(pgTable, "id")  // ✅ 가능

// val mixedColumn = mysql.addColumn(pgTable, "id")  // ❌ 컴파일 에러
```

---

## 11.3 Self Types (자기 타입)

### 11.3.1 기본 사용법

```scala
// Self Type: 믹스인될 타입 제약
trait User {
  def username: String
}

trait Tweeter {
  this: User =>  // Self type annotation

  def tweet(msg: String): Unit = {
    println(s"$username tweets: $msg")
  }
}

// ✅ User와 함께 믹스인
class TwitterUser(val username: String) extends User with Tweeter

val user = new TwitterUser("alice")
user.tweet("Hello Scala!")  // alice tweets: Hello Scala!

// ❌ User 없이 Tweeter만 사용 불가
// class InvalidTweeter extends Tweeter  // 컴파일 에러
```

### 11.3.2 Cake Pattern (의존성 주입)

```scala
// 컴포넌트 정의
trait UserRepositoryComponent {
  val userRepository: UserRepository

  trait UserRepository {
    def findById(id: Int): Option[User]
  }
}

trait UserServiceComponent {
  this: UserRepositoryComponent =>  // Self type

  val userService: UserService

  trait UserService {
    def getUser(id: Int): Option[String] = {
      userRepository.findById(id).map(_.username)
    }
  }
}

// 구현
trait UserRepositoryComponentImpl extends UserRepositoryComponent {
  val userRepository: UserRepository = new UserRepositoryImpl

  class UserRepositoryImpl extends UserRepository {
    private val db = Map(1 -> User("alice"), 2 -> User("bob"))
    def findById(id: Int): Option[User] = db.get(id)
  }
}

trait UserServiceComponentImpl extends UserServiceComponent {
  this: UserRepositoryComponent =>

  val userService: UserService = new UserServiceImpl

  class UserServiceImpl extends UserService
}

// 조합
object Application extends UserRepositoryComponentImpl with UserServiceComponentImpl {
  def main(args: Array[String]): Unit = {
    println(userService.getUser(1))  // Some(alice)
  }
}

case class User(username: String)
```

---

## 11.4 Type Lambdas (타입 람다)

### 11.4.1 문제 상황

```scala
// Either는 타입 파라미터가 2개
// Either[+A, +B]

// Functor는 타입 파라미터가 1개인 타입 생성자 필요
trait Functor[F[_]] {
  def map[A, B](fa: F[A])(f: A => B): F[B]
}

// Either를 Functor로 만들려면?
// implicit val eitherFunctor: Functor[Either] = ???  // ❌ Kind 불일치
```

### 11.4.2 Type Lambda 해법

```scala
// Type Lambda: 타입 생성자를 부분 적용
type StringOr[A] = Either[String, A]

implicit val stringOrFunctor: Functor[StringOr] = new Functor[StringOr] {
  def map[A, B](fa: StringOr[A])(f: A => B): StringOr[B] = fa.map(f)
}

// 또는 익명 타입 람다 (Scala 2.12)
implicit def eitherFunctor[E]: Functor[({type L[A] = Either[E, A]})#L] =
  new Functor[({type L[A] = Either[E, A]})#L] {
    def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
  }
```

```scala
// Scala 3: 더 간결한 Type Lambda 문법
implicit def eitherFunctor[E]: Functor[[A] =>> Either[E, A]] =
  new Functor[[A] =>> Either[E, A]] {
    def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
  }
```

### 11.4.3 Kind Projector 플러그인

```scala
// sbt 설정에 추가
// addCompilerPlugin("org.typelevel" % "kind-projector" % "0.13.2" cross CrossVersion.full)

// Kind Projector 문법
implicit def eitherFunctor[E]: Functor[Either[E, *]] =
  new Functor[Either[E, *]] {
    def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
  }

// Lambda 문법
implicit def eitherFunctor2[E]: Functor[Lambda[A => Either[E, A]]] =
  new Functor[Lambda[A => Either[E, A]]] {
    def map[A, B](fa: Either[E, A])(f: A => B): Either[E, B] = fa.map(f)
  }
```

---

## 11.5 Existential Types (존재 타입)

### 11.5.1 Wildcard Types

```scala
// Java의 와일드카드와 유사
def printList(list: List[_]): Unit = {
  println(s"List with ${list.size} elements")
}

printList(List(1, 2, 3))       // ✅ List[Int]
printList(List("a", "b"))      // ✅ List[String]
printList(List(true, false))   // ✅ List[Boolean]

// Upper bound wildcard
def processAnimals(list: List[_ <: Animal]): Unit = {
  list.foreach(animal => println(animal.name))
}
```

### 11.5.2 Existential Types 선언

```scala
// forSome 키워드 (Scala 2.13에서 deprecated)
type IntList = List[T] forSome { type T <: Int }

// 대신 와일드카드 사용 권장
type IntList2 = List[_ <: Int]
```

### 11.5.3 실전 예제: 타입 안전한 캐시

```scala
// 다양한 타입의 값을 저장하는 캐시
class TypeSafeCache {
  private val cache = scala.collection.mutable.Map[String, Any]()

  def put[T](key: String, value: T): Unit = {
    cache(key) = value
  }

  def get[T](key: String): Option[T] = {
    cache.get(key).map(_.asInstanceOf[T])
  }
}

val cache = new TypeSafeCache
cache.put("count", 42)
cache.put("name", "Alice")

val count: Option[Int] = cache.get[Int]("count")
val name: Option[String] = cache.get[String]("name")

println(count)  // Some(42)
println(name)   // Some(Alice)
```

---

## 11.6 실전 예제: Free Monad 기초

```scala
// Free Monad: 순수한 데이터 구조로 프로그램 표현
sealed trait Free[F[_], A]
case class Pure[F[_], A](a: A) extends Free[F, A]
case class Suspend[F[_], A](fa: F[Free[F, A]]) extends Free[F, A]

// Functor 제약
implicit def freeMonad[F[_]: Functor]: Monad[Free[F, *]] = new Monad[Free[F, *]] {
  def pure[A](a: A): Free[F, A] = Pure(a)

  def flatMap[A, B](fa: Free[F, A])(f: A => Free[F, B]): Free[F, B] = fa match {
    case Pure(a) => f(a)
    case Suspend(ffa) =>
      val functor = implicitly[Functor[F]]
      Suspend(functor.map(ffa)(_.flatMap(f)))
  }
}

// 인터프리터
def interpret[F[_]: Functor, A](free: Free[F, A], interpreter: F ~> Id): A = free match {
  case Pure(a) => a
  case Suspend(ffa) =>
    val functor = implicitly[Functor[F]]
    interpret(functor.map(ffa)(identity), interpreter)
}

// Natural Transformation
trait ~>[F[_], G[_]] {
  def apply[A](fa: F[A]): G[A]
}

type Id[A] = A
```

---

## 11.7 실전 예제: Tagless Final

```scala
// Tagless Final: Higher-Kinded Types로 효과 추상화
trait Console[F[_]] {
  def putStrLn(line: String): F[Unit]
  def getStrLn: F[String]
}

trait FileSystem[F[_]] {
  def readFile(path: String): F[String]
  def writeFile(path: String, content: String): F[Unit]
}

// 프로그램 정의 (효과 추상화)
def program[F[_]: Monad](implicit C: Console[F], FS: FileSystem[F]): F[Unit] = {
  val monad = implicitly[Monad[F]]
  for {
    _ <- C.putStrLn("Enter filename:")
    filename <- C.getStrLn
    content <- FS.readFile(filename)
    _ <- C.putStrLn(s"File content: $content")
  } yield ()
}

// IO Monad 구현
case class IO[A](run: () => A) {
  def flatMap[B](f: A => IO[B]): IO[B] = IO(() => f(run()).run())
  def map[B](f: A => B): IO[B] = IO(() => f(run()))
}

implicit val ioMonad: Monad[IO] = new Monad[IO] {
  def pure[A](a: A): IO[A] = IO(() => a)
  def flatMap[A, B](fa: IO[A])(f: A => IO[B]): IO[B] = fa.flatMap(f)
}

// Console 인터프리터
implicit val consoleIO: Console[IO] = new Console[IO] {
  def putStrLn(line: String): IO[Unit] = IO(() => println(line))
  def getStrLn: IO[String] = IO(() => scala.io.StdIn.readLine())
}

// FileSystem 인터프리터
implicit val fileSystemIO: FileSystem[IO] = new FileSystem[IO] {
  def readFile(path: String): IO[String] = IO(() => scala.io.Source.fromFile(path).mkString)
  def writeFile(path: String, content: String): IO[Unit] =
    IO(() => java.nio.file.Files.writeString(java.nio.file.Paths.get(path), content))
}

// 프로그램 실행
// program[IO].run()
```

---

## 11.8 Java 개발자를 위한 팁

### 11.8.1 Scala vs Java 타입 시스템

| 기능 | Scala | Java |
|------|-------|------|
| Higher-Kinded Types | ✅ | ❌ |
| Path-Dependent Types | ✅ | ❌ (Inner Class는 있지만 제약적) |
| Self Types | ✅ | ❌ |
| Type Lambdas | ✅ | ❌ |
| Existential Types | ✅ (deprecated) | ✅ (Wildcards) |

### 11.8.2 혼동 포인트

1. **Higher-Kinded Types는 런타임 개념이 아님**
   - 컴파일 타임에만 존재
   - Type Erasure로 인해 런타임에는 제거

2. **Self Type vs extends**
   - `extends`: 상속 관계
   - Self Type: 믹스인 제약

3. **Type Lambda의 복잡성**
   - Kind Projector 플러그인 사용 권장
   - Scala 3에서 문법 개선

---

## 11.9 실습 과제

### 과제 11-1: Applicative Functor

Applicative 타입 클래스를 구현하세요:

```scala
trait Applicative[F[_]] extends Functor[F] {
  def pure[A](a: A): F[A]
  def ap[A, B](ff: F[A => B])(fa: F[A]): F[B]
}

// 인스턴스 구현
implicit val optionApplicative: Applicative[Option] = ???
implicit val listApplicative: Applicative[List] = ???

// 테스트
val f: Option[Int => String] = Some(_.toString)
val x: Option[Int] = Some(42)
assert(optionApplicative.ap(f)(x) == Some("42"))
```

### 과제 11-2: State Monad

State Monad를 구현하세요:

```scala
case class State[S, A](run: S => (S, A)) {
  def map[B](f: A => B): State[S, B] = ???
  def flatMap[B](f: A => State[S, B]): State[S, B] = ???
}

object State {
  def get[S]: State[S, S] = ???
  def put[S](s: S): State[S, Unit] = ???
  def modify[S](f: S => S): State[S, Unit] = ???
}

// 테스트: 스택 시뮬레이션
type Stack = List[Int]
val program: State[Stack, Int] = for {
  _ <- State.modify[Stack](1 :: _)
  _ <- State.modify[Stack](2 :: _)
  top <- State.get[Stack].map(_.head)
} yield top

assert(program.run(Nil) == (List(2, 1), 2))
```

---

## 11.10 요약

이번 챕터에서 학습한 내용:

1. **Higher-Kinded Types**: 타입 생성자를 추상화
2. **Path-Dependent Types**: 인스턴스별 고유 타입
3. **Self Types**: 믹스인 제약 표현
4. **Type Lambdas**: 타입 생성자 부분 적용
5. **Existential Types**: 와일드카드를 통한 타입 추상화

**다음 챕터 예고**: Chapter 12에서는 동시성과 Future를 활용한 비동기 프로그래밍을 학습합니다.

---

## Scala 3 주요 차이점

- **Type Lambda 문법**: `[A] =>> F[A]` (더 간결)
- **Match Types**: 타입 레벨 패턴 매칭
- **Polymorphic Function Types**: `[A] => F[A] => G[A]`
- **Context Functions**: `?=> T` 문법
