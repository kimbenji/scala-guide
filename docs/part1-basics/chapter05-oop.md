# Chapter 5: 객체지향 프로그래밍

## 학습 목표

이 챕터를 완료하면 다음을 할 수 있습니다:
- 클래스와 객체의 차이를 이해하고 활용할 수 있다
- Trait를 사용하여 다중 상속과 믹스인 패턴을 구현할 수 있다
- Case Class를 활용하여 불변 데이터 모델을 작성할 수 있다
- Companion Object를 사용하여 팩토리 메서드를 구현할 수 있다
- 상속과 다형성을 Scala 방식으로 활용할 수 있다

## 선행 지식

- Chapter 1-4 완료
- Java 클래스와 인터페이스 개념
- 객체지향 프로그래밍 기본 원리 (캡슐화, 상속, 다형성)

---

## 5.1 클래스(Class) 기초

### 5.1.1 기본 클래스 정의

```scala
// Scala: 기본 클래스
class Person(val name: String, val age: Int) {
  def greet(): String = s"Hello, I'm $name"

  def isAdult: Boolean = age >= 18  // 메서드 괄호 생략 가능
}

val person = new Person("Alice", 25)
println(person.name)       // Alice
println(person.greet())    // Hello, I'm Alice
println(person.isAdult)    // true
```

**Java 비교:**
```java
// Java 동등 코드
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() { return name; }
    public int getAge() { return age; }

    public String greet() {
        return "Hello, I'm " + name;
    }

    public boolean isAdult() {
        return age >= 18;
    }
}
```

**주요 차이점:**
- Scala: 생성자 매개변수에 `val`/`var` 키워드로 필드 자동 생성
- Java: 필드, 생성자, getter를 명시적으로 작성
- Scala: getter/setter 자동 생성 (`val` = getter만, `var` = getter + setter)

### 5.1.2 생성자

```scala
// 주 생성자 (Primary Constructor)
class Rectangle(val width: Int, val height: Int) {
  // 클래스 본문은 주 생성자의 일부
  println(s"Creating rectangle: $width x $height")

  // 보조 생성자 (Auxiliary Constructor)
  def this(side: Int) = {
    this(side, side)  // 주 생성자 호출 필수
    println("Creating square")
  }

  def area: Int = width * height
}

val rect1 = new Rectangle(10, 20)
// 출력: Creating rectangle: 10 x 20

val square = new Rectangle(10)
// 출력: Creating rectangle: 10 x 10
//       Creating square
```

**Java 비교:**
```java
// Java
public class Rectangle {
    private final int width;
    private final int height;

    // 주 생성자
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
        System.out.println("Creating rectangle: " + width + " x " + height);
    }

    // 오버로드 생성자
    public Rectangle(int side) {
        this(side, side);
        System.out.println("Creating square");
    }

    public int area() {
        return width * height;
    }
}
```

### 5.1.3 접근 제어자

```scala
class BankAccount(private var balance: Double) {
  // public (기본값): 어디서나 접근 가능
  def getBalance: Double = balance

  // private: 클래스 내부에서만 접근
  private def log(msg: String): Unit = {
    println(s"[LOG] $msg")
  }

  // protected: 하위 클래스에서도 접근 가능
  protected def validate(amount: Double): Boolean = {
    amount > 0 && amount <= balance
  }

  def withdraw(amount: Double): Boolean = {
    if (validate(amount)) {
      balance -= amount
      log(s"Withdrawn: $amount")
      true
    } else {
      false
    }
  }
}

val account = new BankAccount(1000)
println(account.getBalance)  // 1000
account.withdraw(100)        // true
// account.balance           // 컴파일 오류! private
// account.log("test")       // 컴파일 오류! private
```

**Java 비교:**
- `public`, `private`, `protected` 동일하게 작동
- Scala 기본값: `public` (Java와 동일)
- Scala 추가 기능: `private[this]` (인스턴스 레벨 private)

---

## 5.2 객체(Object)와 싱글톤

### 5.2.1 Object 정의

```scala
// Scala: Object = 싱글톤 (단 하나의 인스턴스)
object MathUtils {
  val PI = 3.14159

  def square(x: Int): Int = x * x

  def cube(x: Int): Int = x * x * x
}

// new 키워드 없이 직접 접근
println(MathUtils.PI)         // 3.14159
println(MathUtils.square(5))  // 25
```

**Java 비교:**
```java
// Java: static 메서드/필드로 구현
public class MathUtils {
    public static final double PI = 3.14159;

    public static int square(int x) {
        return x * x;
    }

    public static int cube(int x) {
        return x * x * x;
    }

    // 인스턴스 생성 방지
    private MathUtils() {}
}
```

### 5.2.2 Companion Object

```scala
// Class와 같은 파일에 같은 이름의 Object를 정의
class User(val id: Int, val name: String)

object User {
  // 팩토리 메서드
  def apply(id: Int, name: String): User = {
    new User(id, name)
  }

  // 정적 메서드
  def fromCsv(csv: String): User = {
    val parts = csv.split(",")
    new User(parts(0).toInt, parts(1))
  }
}

// apply 메서드 덕분에 new 생략 가능
val user1 = User(1, "Alice")  // User.apply(1, "Alice")
val user2 = User.fromCsv("2,Bob")

println(user1.name)  // Alice
println(user2.id)    // 2
```

**Java 비교:**
```java
// Java: 정적 팩토리 메서드
public class User {
    private final int id;
    private final String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    // 정적 팩토리 메서드
    public static User of(int id, String name) {
        return new User(id, name);
    }

    public static User fromCsv(String csv) {
        String[] parts = csv.split(",");
        return new User(Integer.parseInt(parts[0]), parts[1]);
    }
}

// 사용
User user1 = User.of(1, "Alice");
User user2 = User.fromCsv("2,Bob");
```

---

## 5.3 Case Class: 불변 데이터 모델

### 5.3.1 Case Class 기본

```scala
// Scala: case class 자동 기능
case class Point(x: Int, y: Int)

val p1 = Point(10, 20)  // new 생략 가능
val p2 = Point(10, 20)

// 1. 자동 toString
println(p1)  // Point(10,20)

// 2. 자동 equals/hashCode
println(p1 == p2)        // true (구조적 동등성)
println(p1.equals(p2))   // true

// 3. copy 메서드 (불변 업데이트)
val p3 = p1.copy(x = 30)
println(p3)  // Point(30,20)

// 4. 패턴 매칭 지원
p1 match {
  case Point(x, y) => println(s"x=$x, y=$y")
}
```

**Java 비교 (Java 14+ Record):**
```java
// Java 14+: Record (case class와 유사)
public record Point(int x, int y) {}

Point p1 = new Point(10, 20);
Point p2 = new Point(10, 20);

System.out.println(p1);         // Point[x=10, y=20]
System.out.println(p1.equals(p2)); // true

// Java Record는 copy 메서드 없음 (수동 구현 필요)
Point p3 = new Point(30, p1.y());
```

**Java 비교 (Java 13 이하):**
```java
// Java 13 이하: 수동 구현
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public String toString() {
        return "Point(" + x + "," + y + ")";
    }

    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Point)) return false;
        Point p = (Point) obj;
        return this.x == p.x && this.y == p.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }

    // copy 메서드 수동 구현
    public Point copy(int x, int y) {
        return new Point(x, y);
    }
}
```

### 5.3.2 Case Class 활용

```scala
case class Employee(
  id: Int,
  name: String,
  department: String,
  salary: Double
)

object Employee {
  // 커스텀 팩토리 메서드
  def intern(name: String): Employee = {
    Employee(0, name, "Intern", 0.0)
  }
}

val emp1 = Employee(1, "Alice", "Engineering", 80000)
val emp2 = emp1.copy(salary = 85000)  // 연봉 인상

// 패턴 매칭으로 데이터 추출
def describeSalary(emp: Employee): String = emp match {
  case Employee(_, _, _, s) if s > 100000 => "High earner"
  case Employee(_, _, _, s) if s > 50000  => "Average"
  case _                                   => "Entry level"
}

println(describeSalary(emp1))  // Average
println(describeSalary(emp2))  // Average
```

---

## 5.4 Trait: 다중 상속과 믹스인

### 5.4.1 Trait 기본

```scala
// Scala: Trait = Java의 Interface + 구현 메서드
trait Greeter {
  def greet(name: String): Unit = {
    println(s"Hello, $name!")
  }
}

trait Logger {
  def log(message: String): Unit = {
    println(s"[LOG] $message")
  }
}

// 다중 Trait 상속 (with 키워드)
class Service extends Greeter with Logger {
  def process(name: String): Unit = {
    log("Starting process")
    greet(name)
    log("Process complete")
  }
}

val service = new Service
service.process("Alice")
// 출력:
// [LOG] Starting process
// Hello, Alice!
// [LOG] Process complete
```

**Java 비교 (Java 8+):**
```java
// Java 8+: Interface with default methods
public interface Greeter {
    default void greet(String name) {
        System.out.println("Hello, " + name + "!");
    }
}

public interface Logger {
    default void log(String message) {
        System.out.println("[LOG] " + message);
    }
}

// 다중 인터페이스 구현
public class Service implements Greeter, Logger {
    public void process(String name) {
        log("Starting process");
        greet(name);
        log("Process complete");
    }
}
```

### 5.4.2 Trait with 추상 메서드

```scala
trait Animal {
  // 추상 메서드
  def makeSound(): String

  // 구현 메서드
  def describe(): Unit = {
    println(s"This animal says: ${makeSound()}")
  }
}

class Dog extends Animal {
  override def makeSound(): String = "Woof!"
}

class Cat extends Animal {
  override def makeSound(): String = "Meow!"
}

val dog = new Dog
val cat = new Cat

dog.describe()  // This animal says: Woof!
cat.describe()  // This animal says: Meow!
```

### 5.4.3 Trait 믹스인 패턴

```scala
trait Timestamped {
  val timestamp: Long = System.currentTimeMillis()
}

trait Versioned {
  var version: Int = 1

  def incrementVersion(): Unit = {
    version += 1
  }
}

// 런타임에 Trait 믹스인
class Document(val content: String)

val doc1 = new Document("Hello") with Timestamped with Versioned

println(doc1.content)      // Hello
println(doc1.timestamp)    // 1234567890 (현재 시간)
println(doc1.version)      // 1
doc1.incrementVersion()
println(doc1.version)      // 2
```

**Java 비교:**
Java에는 런타임 믹스인이 없음 (Decorator 패턴으로 유사하게 구현 가능)

---

## 5.5 상속과 다형성

### 5.5.1 클래스 상속

```scala
// 추상 클래스
abstract class Shape {
  def area: Double          // 추상 메서드
  def perimeter: Double     // 추상 메서드

  // 구현 메서드
  def description: String = {
    s"Area: $area, Perimeter: $perimeter"
  }
}

class Circle(val radius: Double) extends Shape {
  override def area: Double = Math.PI * radius * radius
  override def perimeter: Double = 2 * Math.PI * radius
}

class Rectangle(val width: Double, val height: Double) extends Shape {
  override def area: Double = width * height
  override def perimeter: Double = 2 * (width + height)
}

val shapes: List[Shape] = List(
  new Circle(5),
  new Rectangle(4, 6)
)

shapes.foreach(s => println(s.description))
// 출력:
// Area: 78.53981633974483, Perimeter: 31.41592653589793
// Area: 24.0, Perimeter: 20.0
```

**Java 비교:**
```java
// Java 동등 코드
public abstract class Shape {
    public abstract double area();
    public abstract double perimeter();

    public String description() {
        return "Area: " + area() + ", Perimeter: " + perimeter();
    }
}

public class Circle extends Shape {
    private final double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }

    @Override
    public double perimeter() {
        return 2 * Math.PI * radius;
    }
}
```

### 5.5.2 타입 체크와 캐스팅

```scala
val shape: Shape = new Circle(5)

// 타입 체크
if (shape.isInstanceOf[Circle]) {
  val circle = shape.asInstanceOf[Circle]
  println(circle.radius)  // 5.0
}

// 패턴 매칭 (더 안전한 방법)
shape match {
  case c: Circle => println(s"Circle with radius ${c.radius}")
  case r: Rectangle => println(s"Rectangle ${r.width}x${r.height}")
  case _ => println("Unknown shape")
}
```

**Java 비교:**
```java
// Java: instanceof와 캐스팅
Shape shape = new Circle(5);

if (shape instanceof Circle) {
    Circle circle = (Circle) shape;
    System.out.println(circle.getRadius());
}

// Java 16+: Pattern Matching for instanceof
if (shape instanceof Circle c) {
    System.out.println("Circle with radius " + c.getRadius());
}
```

### 5.5.3 Sealed Trait (봉인된 계층 구조)

```scala
// sealed: 같은 파일 내에서만 상속 가능
sealed trait Result

case class Success(value: String) extends Result
case class Failure(error: String) extends Result

def handleResult(result: Result): Unit = result match {
  case Success(v) => println(s"Success: $v")
  case Failure(e) => println(s"Error: $e")
  // 컴파일러가 모든 케이스 확인 (exhaustiveness check)
}

val res1: Result = Success("Data loaded")
val res2: Result = Failure("Network error")

handleResult(res1)  // Success: Data loaded
handleResult(res2)  // Error: Network error
```

**Java 비교 (Java 17+):**
```java
// Java 17+: Sealed Classes
public sealed interface Result permits Success, Failure {}

public final class Success implements Result {
    private final String value;
    public Success(String value) { this.value = value; }
    public String getValue() { return value; }
}

public final class Failure implements Result {
    private final String error;
    public Failure(String error) { this.error = error; }
    public String getError() { return error; }
}
```

---

## 5.6 실전 예제: 타입 안전한 DSL

### 5.6.1 Builder 패턴 with Trait

```scala
trait QueryBuilder {
  def select(fields: String*): SelectClause = {
    new SelectClause(fields.toList)
  }
}

class SelectClause(val fields: List[String]) {
  def from(table: String): FromClause = {
    new FromClause(fields, table)
  }
}

class FromClause(val fields: List[String], val table: String) {
  def where(condition: String): WhereClause = {
    new WhereClause(fields, table, condition)
  }

  def build(): String = {
    s"SELECT ${fields.mkString(", ")} FROM $table"
  }
}

class WhereClause(
  val fields: List[String],
  val table: String,
  val condition: String
) {
  def build(): String = {
    s"SELECT ${fields.mkString(", ")} FROM $table WHERE $condition"
  }
}

object SQL extends QueryBuilder

// 사용 예시
val query1 = SQL.select("name", "age")
  .from("users")
  .build()

val query2 = SQL.select("*")
  .from("products")
  .where("price > 100")
  .build()

println(query1)  // SELECT name, age FROM users
println(query2)  // SELECT * FROM products WHERE price > 100
```

---

## 5.7 실습 과제

### 과제 5-1: 도형 계산기

다음 요구사항을 만족하는 프로그램을 작성하세요:

1. `Shape` trait 정의 (area, perimeter 추상 메서드)
2. `Circle`, `Rectangle`, `Triangle` case class 구현
3. `ShapeCalculator` object에 총 면적/둘레 계산 메서드 작성

**힌트:**
```scala
trait Shape {
  def area: Double
  def perimeter: Double
}

case class Circle(radius: Double) extends Shape {
  // 구현하세요
}

object ShapeCalculator {
  def totalArea(shapes: List[Shape]): Double = ???
}
```

### 과제 5-2: 로깅 시스템

다음 Trait를 믹스인하여 로깅 시스템을 구현하세요:

```scala
trait Logger {
  def log(level: String, message: String): Unit
}

trait ConsoleLogger extends Logger {
  // 콘솔 출력 구현
}

trait FileLogger extends Logger {
  // 파일 출력 구현 (간단히 println으로 시뮬레이션)
}

class Service // ConsoleLogger와 FileLogger 믹스인
```

---

## 5.8 요약

이 챕터에서 배운 내용:

✅ 클래스와 생성자 (주 생성자, 보조 생성자)
✅ Object와 Companion Object (싱글톤, 팩토리 메서드)
✅ Case Class (불변 데이터 모델, copy, 패턴 매칭)
✅ Trait (다중 상속, 믹스인 패턴)
✅ 상속과 다형성 (추상 클래스, sealed trait)

**Java와 주요 차이점:**
- Scala: 생성자 매개변수로 필드 자동 생성
- Scala: Case Class로 보일러플레이트 제거
- Scala: Trait로 다중 상속 및 런타임 믹스인
- Scala: Object로 싱글톤 패턴 내장

**다음 챕터 예고:**
Chapter 6에서는 Scala 컬렉션 프레임워크를 다룹니다.

---

## Scala 3 차이점

```scala
// Scala 3: Enum 지원 (sealed trait 대체)
enum Result:
  case Success(value: String)
  case Failure(error: String)

// Scala 3: Extension methods
extension (s: String)
  def toSnakeCase: String =
    s.replaceAll("([A-Z])", "_$1").toLowerCase

println("HelloWorld".toSnakeCase)  // hello_world
```

---

## 참고 자료

- [Scala Classes and Objects](https://docs.scala-lang.org/tour/classes.html)
- [Scala Case Classes](https://docs.scala-lang.org/tour/case-classes.html)
- [Scala Traits](https://docs.scala-lang.org/tour/traits.html)
