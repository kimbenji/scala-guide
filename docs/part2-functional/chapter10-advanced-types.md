# Chapter 10: 타입 시스템 심화 (Advanced Type System)

## 학습 목표
- 제네릭(Generic Types)과 타입 파라미터 이해
- 공변성(Covariance)과 반공변성(Contravariance) 개념 학습
- 타입 경계(Type Bounds)를 활용한 타입 제약
- Context Bounds와 타입 클래스 패턴 심화
- Phantom Types와 타입 레벨 프로그래밍 기초

---

## 10.1 제네릭 타입 (Generic Types)

### 10.1.1 타입 파라미터 기초

```scala
// 타입 파라미터를 사용한 제네릭 클래스
class Box[A](val content: A) {
  def get: A = content
  def map[B](f: A => B): Box[B] = new Box(f(content))
}

val intBox = new Box(42)
println(intBox.get)  // 42

val stringBox = intBox.map(_.toString)
println(stringBox.get)  // "42"
```

```java
// Java 제네릭과 비교
class Box<A> {
    private final A content;

    public Box(A content) {
        this.content = content;
    }

    public A get() {
        return content;
    }

    public <B> Box<B> map(Function<A, B> f) {
        return new Box<>(f.apply(content));
    }
}
```

### 10.1.2 여러 타입 파라미터

```scala
// 두 개의 타입 파라미터
case class Pair[A, B](first: A, second: B) {
  def swap: Pair[B, A] = Pair(second, first)
}

val pair = Pair("Alice", 25)
println(pair)        // Pair(Alice,25)
println(pair.swap)   // Pair(25,Alice)

// Map의 제네릭 시그니처
class Map[K, V] {
  def get(key: K): Option[V] = ???
  def put(key: K, value: V): Map[K, V] = ???
}
```

### 10.1.3 타입 추론

```scala
// 컴파일러가 타입 자동 추론
val box1 = new Box(42)           // Box[Int]
val box2 = new Box("hello")      // Box[String]
val box3 = new Box(List(1, 2))   // Box[List[Int]]

// 명시적 타입 지정도 가능
val box4 = new Box[Any](42)      // Box[Any]
```

---

## 10.2 공변성과 반공변성 (Variance)

### 10.2.1 공변성 (Covariance) - `+A`

**공변성**: `A`가 `B`의 서브타입이면, `Container[A]`도 `Container[B]`의 서브타입

```scala
// 공변 컨테이너
class Box[+A](val content: A)

class Animal { def name = "Animal" }
class Dog extends Animal { override def name = "Dog" }
class Cat extends Animal { override def name = "Cat" }

// Dog는 Animal의 서브타입이므로
// Box[Dog]는 Box[Animal]의 서브타입
val dogBox: Box[Dog] = new Box(new Dog)
val animalBox: Box[Animal] = dogBox  // ✅ 공변성 덕분에 가능

println(animalBox.content.name)  // "Dog"

// List는 공변
val dogs: List[Dog] = List(new Dog, new Dog)
val animals: List[Animal] = dogs  // ✅ List[+A]이므로 가능
```

```java
// Java는 기본적으로 불변(invariant)
List<Dog> dogs = Arrays.asList(new Dog());
// List<Animal> animals = dogs;  // ❌ 컴파일 에러

// 와일드카드로 공변성 표현
List<? extends Animal> animals = dogs;  // ✅ 가능
```

### 10.2.2 반공변성 (Contravariance) - `-A`

**반공변성**: `A`가 `B`의 서브타입이면, `Container[B]`가 `Container[A]`의 서브타입

```scala
// 반공변 컨테이너 (주로 함수의 입력 타입)
trait Printer[-A] {
  def print(value: A): Unit
}

val animalPrinter: Printer[Animal] = new Printer[Animal] {
  def print(value: Animal): Unit = println(s"Animal: ${value.name}")
}

// Animal Printer는 Dog도 출력 가능
val dogPrinter: Printer[Dog] = animalPrinter  // ✅ 반공변성 덕분에 가능

dogPrinter.print(new Dog)  // "Animal: Dog"

// 함수의 파라미터는 반공변
val f1: Animal => String = (a: Animal) => a.name
val f2: Dog => String = f1  // ✅ Dog는 Animal의 서브타입
```

### 10.2.3 불변성 (Invariance)

**불변성**: 타입 관계가 유지되지 않음 (기본값)

```scala
// 불변 컨테이너
class Container[A](var content: A) {
  def set(value: A): Unit = { content = value }
  def get: A = content
}

val dogContainer: Container[Dog] = new Container(new Dog)
// val animalContainer: Container[Animal] = dogContainer  // ❌ 컴파일 에러

// 이유: animalContainer.set(new Cat)을 허용하면
// dogContainer에 Cat이 들어가는 문제 발생
```

### 10.2.4 Variance 정리

| Variance | 기호 | 설명 | 예제 |
|----------|-----|------|------|
| 공변 (Covariant) | `+A` | 생산자(Producer) | `List[+A]`, `Option[+A]` |
| 반공변 (Contravariant) | `-A` | 소비자(Consumer) | `Function1[-T, +R]` |
| 불변 (Invariant) | `A` | 생산자+소비자 | `Array[A]`, `ListBuffer[A]` |

**규칙**: "Producer는 공변, Consumer는 반공변" (PECS: Producer Extends, Consumer Super)

---

## 10.3 타입 경계 (Type Bounds)

### 10.3.1 상한 경계 (Upper Bound) - `A <: B`

`A`는 `B`의 서브타입이어야 함

```scala
// Animal 또는 그 서브타입만 허용
def printAnimal[A <: Animal](animal: A): Unit = {
  println(animal.name)
}

printAnimal(new Dog)    // ✅ Dog는 Animal의 서브타입
printAnimal(new Cat)    // ✅ Cat도 Animal의 서브타입
// printAnimal("string")  // ❌ String은 Animal의 서브타입이 아님

// 실전 예제: 정렬 가능한 리스트
def sort[A <: Comparable[A]](list: List[A]): List[A] = {
  list.sorted
}
```

```java
// Java의 Upper Bound
<T extends Animal> void printAnimal(T animal) {
    System.out.println(animal.name());
}
```

### 10.3.2 하한 경계 (Lower Bound) - `A >: B`

`A`는 `B`의 슈퍼타입이어야 함

```scala
// List의 :: 연산자 시그니처
// def ::[B >: A](elem: B): List[B]

val dogs: List[Dog] = List(new Dog)
val animals: List[Animal] = new Cat :: dogs  // ✅ Animal은 Dog의 슈퍼타입

// 실전 예제: 범용 추가 함수
class MyList[+A] {
  def prepend[B >: A](elem: B): MyList[B] = ???
}

val dogList: MyList[Dog] = ???
val animalList: MyList[Animal] = dogList.prepend(new Cat)  // ✅ 가능
```

### 10.3.3 상한과 하한 결합

```scala
// A는 B의 슈퍼타입이면서 C의 서브타입
def transform[A, B >: A, C <: A](value: A, lower: B, upper: C): A = ???

// 실전 예제: 타입 안전한 비교
def clamp[A](value: A, min: A, max: A)(implicit ord: Ordering[A]): A = {
  import ord._
  if (value < min) min
  else if (value > max) max
  else value
}

println(clamp(5, 1, 10))    // 5
println(clamp(15, 1, 10))   // 10
println(clamp(-5, 1, 10))   // 1
```

---

## 10.4 Context Bounds (컨텍스트 경계)

### 10.4.1 기본 사용법

Context Bound는 implicit parameter의 간결한 표현입니다.

```scala
// implicit parameter 사용
def sort1[A](list: List[A])(implicit ordering: Ordering[A]): List[A] = {
  list.sorted(ordering)
}

// Context Bound 사용 (더 간결)
def sort2[A: Ordering](list: List[A]): List[A] = {
  list.sorted  // implicit Ordering[A]가 자동으로 전달됨
}

// implicitly로 implicit 값 가져오기
def sort3[A: Ordering](list: List[A]): List[A] = {
  val ordering = implicitly[Ordering[A]]
  list.sorted(ordering)
}
```

### 10.4.2 여러 Context Bounds

```scala
// 여러 개의 Context Bound
def processData[A: Ordering : Numeric](data: List[A]): A = {
  val ordering = implicitly[Ordering[A]]
  val numeric = implicitly[Numeric[A]]

  data.sorted(ordering).reduce(numeric.plus)
}

println(processData(List(3, 1, 4, 1, 5)))  // 14 (정렬 후 합계)
```

### 10.4.3 타입 클래스와 Context Bounds

```scala
// 타입 클래스 정의
trait Show[A] {
  def show(value: A): String
}

object Show {
  // 인스턴스 정의
  implicit val intShow: Show[Int] = new Show[Int] {
    def show(value: Int): String = s"Int($value)"
  }

  implicit val stringShow: Show[String] = new Show[String] {
    def show(value: String): String = s"String($value)"
  }

  // 헬퍼 메서드
  def apply[A](implicit show: Show[A]): Show[A] = show

  // Show 인스턴스가 있는 타입만 허용
  def print[A: Show](value: A): Unit = {
    println(Show[A].show(value))
  }
}

// 사용
Show.print(42)        // Int(42)
Show.print("hello")   // String(hello)
// Show.print(true)   // ❌ Boolean의 Show 인스턴스 없음
```

---

## 10.5 View Bounds (뷰 경계) - Deprecated

**주의**: Scala 2.13에서 deprecated, Scala 3에서 제거됨

```scala
// View Bound (사용 금지)
def oldPrint[A <% String](value: A): Unit = {
  println(value)  // A를 String으로 암시적 변환
}

// 대신 implicit conversion 명시적 사용
def newPrint[A](value: A)(implicit converter: A => String): Unit = {
  println(converter(value))
}
```

---

## 10.6 타입 레벨 프로그래밍 기초

### 10.6.1 Phantom Types

**Phantom Type**: 런타임에 사용되지 않지만 컴파일 타임 타입 안전성을 제공하는 타입

```scala
// 상태를 타입으로 표현
sealed trait ConnectionState
sealed trait Open extends ConnectionState
sealed trait Closed extends ConnectionState

class Connection[State <: ConnectionState] private (val url: String) {
  def open(implicit ev: State =:= Closed): Connection[Open] = {
    println(s"Opening connection to $url")
    new Connection[Open](url)
  }

  def close(implicit ev: State =:= Open): Connection[Closed] = {
    println(s"Closing connection to $url")
    new Connection[Closed](url)
  }

  def execute(query: String)(implicit ev: State =:= Open): String = {
    println(s"Executing: $query")
    "Result"
  }
}

object Connection {
  def create(url: String): Connection[Closed] = new Connection[Closed](url)
}

// 사용 예제
val conn1 = Connection.create("jdbc:mysql://localhost")
// conn1.execute("SELECT * FROM users")  // ❌ 컴파일 에러 (닫힌 연결)

val conn2 = conn1.open
conn2.execute("SELECT * FROM users")  // ✅ 가능 (열린 연결)

val conn3 = conn2.close
// conn3.execute("SELECT * FROM users")  // ❌ 컴파일 에러 (닫힌 연결)
```

### 10.6.2 타입 증명 (Type Evidence)

```scala
// =:= : 타입 동등성 증명
def onlyInt[A](value: A)(implicit ev: A =:= Int): Int = {
  value  // A가 Int임이 증명됨
}

println(onlyInt(42))  // ✅ 가능
// println(onlyInt("42"))  // ❌ 컴파일 에러

// <:< : 서브타입 증명
def onlyAnimal[A](value: A)(implicit ev: A <:< Animal): Animal = {
  value
}

println(onlyAnimal(new Dog))  // ✅ Dog는 Animal의 서브타입
// println(onlyAnimal("dog"))  // ❌ 컴파일 에러
```

### 10.6.3 HList (Heterogeneous List) 개념

```scala
// 간단한 HList 구현
sealed trait HList
case class ::[+H, +T <: HList](head: H, tail: T) extends HList
sealed trait HNil extends HList
case object HNil extends HNil

// 사용 예제
val hlist = 42 :: "hello" :: true :: HNil
// 타입: Int :: String :: Boolean :: HNil

// 타입 안전한 접근
val first: Int = hlist.head           // 42
val second: String = hlist.tail.head  // "hello"
val third: Boolean = hlist.tail.tail.head  // true

// List와 비교
val list = List(42, "hello", true)  // List[Any] (타입 정보 손실)
// val x: Int = list.head  // ❌ Any 타입
```

---

## 10.7 실전 예제: 타입 안전한 빌더 패턴

```scala
// 빌더 상태를 타입으로 표현
sealed trait BuilderState
sealed trait HasName extends BuilderState
sealed trait HasEmail extends BuilderState
sealed trait NoName extends BuilderState
sealed trait NoEmail extends BuilderState

case class User(name: String, email: String)

class UserBuilder[NameState <: BuilderState, EmailState <: BuilderState] private (
  name: Option[String],
  email: Option[String]
) {
  def withName(n: String)(implicit ev: NameState =:= NoName): UserBuilder[HasName, EmailState] = {
    new UserBuilder[HasName, EmailState](Some(n), email)
  }

  def withEmail(e: String)(implicit ev: EmailState =:= NoEmail): UserBuilder[NameState, HasEmail] = {
    new UserBuilder[NameState, HasEmail](name, Some(e))
  }

  def build(implicit
    evName: NameState =:= HasName,
    evEmail: EmailState =:= HasEmail
  ): User = {
    User(name.get, email.get)
  }
}

object UserBuilder {
  def create: UserBuilder[NoName, NoEmail] = new UserBuilder[NoName, NoEmail](None, None)
}

// 사용 예제
val user = UserBuilder.create
  .withName("Alice")
  .withEmail("alice@example.com")
  .build

println(user)  // User(Alice,alice@example.com)

// 컴파일 에러 예제
// UserBuilder.create.build  // ❌ name과 email 없음
// UserBuilder.create.withName("Alice").build  // ❌ email 없음
// UserBuilder.create.withName("A").withName("B")  // ❌ name 중복 설정
```

---

## 10.8 실전 예제: 타입 안전한 단위 변환

```scala
// 물리 단위를 타입으로 표현
sealed trait Unit
sealed trait Meters extends Unit
sealed trait Kilometers extends Unit
sealed trait Seconds extends Unit

case class Quantity[U <: Unit](value: Double) {
  def +(other: Quantity[U]): Quantity[U] = Quantity(value + other.value)
  def -(other: Quantity[U]): Quantity[U] = Quantity(value - other.value)
  def *(scalar: Double): Quantity[U] = Quantity(value * scalar)
}

// 변환 함수
object Conversions {
  implicit class MetersOps(val q: Quantity[Meters]) extends AnyVal {
    def toKilometers: Quantity[Kilometers] = Quantity[Kilometers](q.value / 1000)
  }

  implicit class KilometersOps(val q: Quantity[Kilometers]) extends AnyVal {
    def toMeters: Quantity[Meters] = Quantity[Meters](q.value * 1000)
  }
}

// 사용
val distance1 = Quantity[Meters](1000)
val distance2 = Quantity[Meters](500)

val sum = distance1 + distance2  // ✅ Quantity[Meters]
println(sum.value)  // 1500.0

// val invalid = distance1 + Quantity[Kilometers](1)  // ❌ 컴파일 에러 (단위 불일치)

import Conversions._
val inKm = distance1.toKilometers
println(inKm.value)  // 1.0
```

---

## 10.9 Java 개발자를 위한 팁

### 10.9.1 Java Generics와의 차이

| 기능 | Scala | Java |
|------|-------|------|
| Variance | Declaration-site | Use-site |
| Type Erasure | ✅ | ✅ |
| Higher-kinded Types | ✅ | ❌ |
| Type Bounds | `A <: B`, `A >: B` | `T extends B`, `T super B` |
| Context Bounds | `A: TypeClass` | ❌ |

### 10.9.2 Variance 비교

```scala
// Scala: Declaration-site variance
class Box[+A](val content: A)  // 정의 시점에 공변 선언

val dogBox: Box[Dog] = ???
val animalBox: Box[Animal] = dogBox  // ✅ 자동
```

```java
// Java: Use-site variance
class Box<A> {
    private final A content;
    public Box(A content) { this.content = content; }
}

Box<Dog> dogBox = new Box<>(new Dog());
// Box<Animal> animalBox = dogBox;  // ❌ 컴파일 에러
Box<? extends Animal> animalBox = dogBox;  // ✅ 사용 시점에 지정
```

### 10.9.3 혼동 포인트

1. **타입 소거 (Type Erasure)**
   ```scala
   // 런타임에는 타입 정보 손실
   def isListOfInt(x: Any): Boolean = x match {
     case _: List[Int] => true  // 경고: unchecked
     case _ => false
   }
   ```

2. **Variance의 제약**
   ```scala
   // 공변 타입은 파라미터로 사용 불가
   // class Box[+A] {
   //   def set(value: A): Unit = ???  // ❌ 컴파일 에러
   // }

   // 해결책: Lower bound 사용
   class Box[+A] {
     def set[B >: A](value: B): Box[B] = ???  // ✅ 가능
   }
   ```

---

## 10.10 실습 과제

### 과제 10-1: 타입 안전한 스택

공변성을 활용한 스택을 구현하세요:

```scala
sealed trait Stack[+A] {
  def push[B >: A](elem: B): Stack[B]
  def pop: (A, Stack[A])
  def isEmpty: Boolean
}

case object EmptyStack extends Stack[Nothing] {
  def push[B](elem: B): Stack[B] = ???
  def pop: Nothing = throw new NoSuchElementException("Empty stack")
  def isEmpty: Boolean = true
}

case class NonEmptyStack[A](head: A, tail: Stack[A]) extends Stack[A] {
  def push[B >: A](elem: B): Stack[B] = ???
  def pop: (A, Stack[A]) = (head, tail)
  def isEmpty: Boolean = false
}

// 테스트
val stack1: Stack[Int] = EmptyStack.push(1).push(2).push(3)
val stack2: Stack[Any] = stack1.push("string")  // 공변성 덕분에 가능
```

### 과제 10-2: 타입 클래스로 직렬화

타입 클래스를 사용한 직렬화 시스템을 구현하세요:

```scala
trait Serializer[A] {
  def serialize(value: A): String
  def deserialize(s: String): Option[A]
}

// 인스턴스 구현
implicit val intSerializer: Serializer[Int] = ???
implicit val stringSerializer: Serializer[String] = ???
implicit def listSerializer[A: Serializer]: Serializer[List[A]] = ???

// 헬퍼 함수
def serialize[A: Serializer](value: A): String = ???
def deserialize[A: Serializer](s: String): Option[A] = ???

// 테스트
assert(serialize(42) == "42")
assert(deserialize[Int]("42") == Some(42))
assert(serialize(List(1, 2, 3)) == "[1,2,3]")
```

### 과제 10-3: Phantom Type으로 상태 머신

파일 핸들의 상태를 타입으로 표현하세요:

```scala
sealed trait FileState
sealed trait ReadMode extends FileState
sealed trait WriteMode extends FileState
sealed trait ClosedMode extends FileState

class FileHandle[State <: FileState] private (val path: String) {
  def openRead(implicit ev: State =:= ClosedMode): FileHandle[ReadMode] = ???
  def openWrite(implicit ev: State =:= ClosedMode): FileHandle[WriteMode] = ???
  def read(implicit ev: State =:= ReadMode): String = ???
  def write(data: String)(implicit ev: State =:= WriteMode): Unit = ???
  def close(implicit ev: State =:= ReadMode | State =:= WriteMode): FileHandle[ClosedMode] = ???
}

// 테스트: 컴파일 타임에 잘못된 사용 방지
val fh = FileHandle.create("file.txt")
// fh.read  // ❌ 닫힌 파일 읽기 불가
val fh2 = fh.openRead
fh2.read  // ✅ 가능
```

---

## 10.11 요약

이번 챕터에서 학습한 내용:

1. **제네릭**: 타입 파라미터로 재사용 가능한 코드 작성
2. **Variance**: 공변(`+A`), 반공변(`-A`), 불변(`A`)
3. **타입 경계**: 상한(`<:`), 하한(`>:`)으로 타입 제약
4. **Context Bounds**: `A: TypeClass` 문법으로 타입 클래스 활용
5. **타입 레벨 프로그래밍**: Phantom Types로 컴파일 타임 안전성 확보

**다음 챕터 예고**: Part 3에서는 동시성, 매크로, 메타프로그래밍 등 고급 주제를 다룹니다.

---

## Scala 3 주요 차이점

- **Union Types**: `A | B` 문법으로 Either 대체 가능
- **Intersection Types**: `A & B` 문법
- **Opaque Types**: 타입 별칭의 타입 안전 버전
- **Match Types**: 타입 레벨 패턴 매칭
- **Polymorphic Function Types**: 더 유연한 제네릭 함수

```scala
// Scala 3 Union Types
def handle(value: Int | String): Unit = value match {
  case i: Int => println(s"Int: $i")
  case s: String => println(s"String: $s")
}
```
