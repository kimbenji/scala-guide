# Chapter 6: 컬렉션 기초

## 학습 목표

이 챕터를 완료하면 다음을 할 수 있습니다:
- Scala 컬렉션 라이브러리의 구조를 이해한다
- List, Set, Map의 기본 연산을 수행할 수 있다
- 불변 컬렉션과 가변 컬렉션의 차이를 이해하고 선택할 수 있다
- 고차 함수(map, filter, fold 등)를 활용하여 컬렉션을 변환할 수 있다
- for-comprehension을 사용하여 컬렉션을 조작할 수 있다

## 선행 지식

- Chapter 1-5 완료
- Java Collections Framework 기본 (List, Set, Map)
- 함수형 프로그래밍 기초 (고차 함수, 람다)

---

## 6.1 Scala 컬렉션 개요

### 6.1.1 컬렉션 계층 구조

```
Traversable
    ├─ Iterable
    │   ├─ Seq (순서 있음)
    │   │   ├─ List
    │   │   ├─ Vector
    │   │   ├─ Array
    │   │   └─ Range
    │   ├─ Set (중복 없음)
    │   │   ├─ HashSet
    │   │   └─ TreeSet
    │   └─ Map (키-값 쌍)
    │       ├─ HashMap
    │       └─ TreeMap
```

### 6.1.2 불변 vs 가변

```scala
// 불변 컬렉션 (기본값, 권장)
import scala.collection.immutable._

val list1 = List(1, 2, 3)
val list2 = list1 :+ 4  // 새 리스트 반환
println(list1)  // List(1, 2, 3) - 원본 불변
println(list2)  // List(1, 2, 3, 4)

// 가변 컬렉션 (필요 시에만)
import scala.collection.mutable

val buffer = mutable.ListBuffer(1, 2, 3)
buffer += 4  // 원본 수정
println(buffer)  // ListBuffer(1, 2, 3, 4)
```

**Java 비교:**
```java
// Java: 기본적으로 가변
List<Integer> list1 = new ArrayList<>(Arrays.asList(1, 2, 3));
list1.add(4);  // 원본 수정
System.out.println(list1);  // [1, 2, 3, 4]

// Java: 불변 컬렉션 (Java 9+)
List<Integer> immutable = List.of(1, 2, 3);
// immutable.add(4);  // UnsupportedOperationException
```

---

## 6.2 List: 연결 리스트

### 6.2.1 List 생성

```scala
// 방법 1: List 생성자
val list1 = List(1, 2, 3, 4, 5)

// 방법 2: cons 연산자 (::)
val list2 = 1 :: 2 :: 3 :: Nil  // Nil = 빈 리스트
println(list2)  // List(1, 2, 3)

// 방법 3: 범위
val list3 = (1 to 5).toList
val list4 = (1 until 5).toList  // 5 미포함
println(list3)  // List(1, 2, 3, 4, 5)
println(list4)  // List(1, 2, 3, 4)

// 빈 리스트
val empty = List.empty[Int]
val empty2 = Nil
```

**Java 비교:**
```java
// Java
List<Integer> list1 = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> list2 = List.of(1, 2, 3);  // Java 9+
List<Integer> list3 = new ArrayList<>();
```

### 6.2.2 기본 연산

```scala
val list = List(1, 2, 3, 4, 5)

// 접근
println(list.head)      // 1 (첫 번째 요소)
println(list.tail)      // List(2, 3, 4, 5) (첫 번째 제외)
println(list(2))        // 3 (인덱스 접근)
println(list.last)      // 5 (마지막 요소)

// 추가 (새 리스트 반환)
val list2 = 0 :: list           // List(0, 1, 2, 3, 4, 5) - 앞에 추가
val list3 = list :+ 6           // List(1, 2, 3, 4, 5, 6) - 뒤에 추가
val list4 = list ++ List(6, 7)  // List(1, 2, 3, 4, 5, 6, 7) - 리스트 연결

// 삭제/필터
val list5 = list.drop(2)        // List(3, 4, 5) - 앞 2개 제거
val list6 = list.dropRight(2)   // List(1, 2, 3) - 뒤 2개 제거
val list7 = list.filter(_ > 2)  // List(3, 4, 5) - 조건 필터

// 크기 및 검사
println(list.length)         // 5
println(list.isEmpty)        // false
println(list.contains(3))    // true
```

**Java 비교:**
```java
// Java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

System.out.println(list.get(0));     // 1
System.out.println(list.size());     // 5
System.out.println(list.isEmpty());  // false
System.out.println(list.contains(3)); // true

// Java: 추가/삭제는 가변 리스트 필요
List<Integer> mutable = new ArrayList<>(list);
mutable.add(6);
mutable.remove(0);
```

### 6.2.3 고차 함수

```scala
val numbers = List(1, 2, 3, 4, 5)

// map: 각 요소 변환
val doubled = numbers.map(_ * 2)
println(doubled)  // List(2, 4, 6, 8, 10)

// filter: 조건 필터
val evens = numbers.filter(_ % 2 == 0)
println(evens)  // List(2, 4)

// flatMap: 변환 + 평탄화
val nested = numbers.flatMap(n => List(n, n * 10))
println(nested)  // List(1, 10, 2, 20, 3, 30, 4, 40, 5, 50)

// fold: 누적 연산
val sum = numbers.foldLeft(0)(_ + _)
println(sum)  // 15

val product = numbers.foldLeft(1)(_ * _)
println(product)  // 120

// reduce: fold의 간소화 버전 (초기값 없음)
val sum2 = numbers.reduce(_ + _)
println(sum2)  // 15

// foreach: 부수 효과 (반환값 없음)
numbers.foreach(n => println(s"Number: $n"))
```

**Java 비교 (Java 8+ Stream):**
```java
// Java Stream API
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> doubled = numbers.stream()
    .map(n -> n * 2)
    .collect(Collectors.toList());

List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);

numbers.forEach(n -> System.out.println("Number: " + n));
```

---

## 6.3 Set: 중복 없는 집합

### 6.3.1 Set 생성 및 기본 연산

```scala
// Set 생성
val set1 = Set(1, 2, 3, 4, 5)
val set2 = Set(4, 5, 6, 7, 8)

// 중복 자동 제거
val set3 = Set(1, 1, 2, 2, 3)
println(set3)  // Set(1, 2, 3)

// 기본 연산
println(set1.contains(3))  // true
println(set1(3))           // true (contains와 동일)

// 추가/삭제 (새 Set 반환)
val set4 = set1 + 6        // Set(1, 2, 3, 4, 5, 6)
val set5 = set1 - 3        // Set(1, 2, 4, 5)

// 집합 연산
val union = set1 | set2         // 합집합: Set(1, 2, 3, 4, 5, 6, 7, 8)
val intersection = set1 & set2  // 교집합: Set(4, 5)
val diff = set1 &~ set2         // 차집합: Set(1, 2, 3)

println(union)
println(intersection)
println(diff)
```

**Java 비교:**
```java
// Java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
Set<Integer> set2 = new HashSet<>(Arrays.asList(4, 5, 6, 7, 8));

System.out.println(set1.contains(3));  // true

// 가변 Set
set1.add(6);
set1.remove(3);

// 집합 연산 (수동 구현)
Set<Integer> union = new HashSet<>(set1);
union.addAll(set2);

Set<Integer> intersection = new HashSet<>(set1);
intersection.retainAll(set2);

Set<Integer> diff = new HashSet<>(set1);
diff.removeAll(set2);
```

### 6.3.2 TreeSet: 정렬된 Set

```scala
// TreeSet: 자동 정렬
val treeSet = scala.collection.immutable.TreeSet(5, 1, 3, 2, 4)
println(treeSet)  // TreeSet(1, 2, 3, 4, 5)

// 범위 조회
println(treeSet.range(2, 5))  // TreeSet(2, 3, 4) - 2 이상 5 미만
```

---

## 6.4 Map: 키-값 쌍

### 6.4.1 Map 생성 및 기본 연산

```scala
// Map 생성
val map1 = Map("a" -> 1, "b" -> 2, "c" -> 3)
val map2 = Map(("a", 1), ("b", 2), ("c", 3))  // 동일한 표현

// 조회
println(map1("a"))           // 1
println(map1.get("a"))       // Some(1) - Option 타입
println(map1.get("z"))       // None
println(map1.getOrElse("z", 0))  // 0 - 기본값

// 추가/업데이트 (새 Map 반환)
val map3 = map1 + ("d" -> 4)              // 추가
val map4 = map1 + ("a" -> 10)             // 업데이트
val map5 = map1 ++ Map("d" -> 4, "e" -> 5) // 병합

// 삭제
val map6 = map1 - "b"
println(map6)  // Map(a -> 1, c -> 3)

// 키/값 확인
println(map1.contains("a"))  // true
println(map1.keys)           // Set(a, b, c)
println(map1.values)         // Iterable(1, 2, 3)
```

**Java 비교:**
```java
// Java
Map<String, Integer> map1 = new HashMap<>();
map1.put("a", 1);
map1.put("b", 2);
map1.put("c", 3);

// Java 9+
Map<String, Integer> map2 = Map.of("a", 1, "b", 2, "c", 3);

System.out.println(map1.get("a"));           // 1
System.out.println(map1.getOrDefault("z", 0)); // 0
System.out.println(map1.containsKey("a"));   // true
System.out.println(map1.keySet());           // [a, b, c]
System.out.println(map1.values());           // [1, 2, 3]
```

### 6.4.2 Map 순회 및 변환

```scala
val scores = Map("Alice" -> 90, "Bob" -> 85, "Charlie" -> 95)

// foreach로 순회
scores.foreach { case (name, score) =>
  println(s"$name: $score")
}

// map으로 변환
val adjusted = scores.map { case (name, score) => (name, score + 5) }
println(adjusted)  // Map(Alice -> 95, Bob -> 90, Charlie -> 100)

// filter
val highScores = scores.filter { case (_, score) => score >= 90 }
println(highScores)  // Map(Alice -> 90, Charlie -> 95)

// mapValues (값만 변환)
val doubled = scores.mapValues(_ * 2)
println(doubled)  // Map(Alice -> 180, Bob -> 170, Charlie -> 190)
```

**Java 비교 (Java 8+):**
```java
// Java Stream
Map<String, Integer> scores = Map.of("Alice", 90, "Bob", 85, "Charlie", 95);

scores.forEach((name, score) ->
    System.out.println(name + ": " + score)
);

Map<String, Integer> adjusted = scores.entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        e -> e.getValue() + 5
    ));

Map<String, Integer> highScores = scores.entrySet().stream()
    .filter(e -> e.getValue() >= 90)
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

---

## 6.5 Vector와 Array

### 6.5.1 Vector: 효율적인 불변 시퀀스

```scala
// Vector: 랜덤 액세스에 최적화 (List는 순차 액세스에 최적화)
val vec = Vector(1, 2, 3, 4, 5)

// List와 동일한 API
println(vec.head)       // 1
println(vec.tail)       // Vector(2, 3, 4, 5)
println(vec(2))         // 3 - O(log n) 성능
println(vec.updated(2, 10))  // Vector(1, 2, 10, 4, 5)

// 성능 비교
val largeList = (1 to 1000000).toList
val largeVec = (1 to 1000000).toVector

// List(999999) - 느림 (O(n))
// largeList(999999)

// Vector(999999) - 빠름 (O(log n))
// largeVec(999999)
```

**사용 권장사항:**
- **List**: 순차 접근, 앞에 추가/삭제가 많은 경우
- **Vector**: 랜덤 액세스, 인덱스 접근이 많은 경우

### 6.5.2 Array: 가변 배열

```scala
// Array: Java 배열과 동일 (가변)
val arr = Array(1, 2, 3, 4, 5)

// 인덱스 접근 및 수정
println(arr(0))  // 1
arr(0) = 10      // 원본 수정
println(arr(0))  // 10

// Array와 List 변환
val list = arr.toList
val arr2 = list.toArray

// 다차원 배열
val matrix = Array.ofDim[Int](3, 3)
matrix(0)(0) = 1
matrix(1)(1) = 5
matrix(2)(2) = 9
```

**Java 비교:**
```java
// Java 배열
int[] arr = {1, 2, 3, 4, 5};
System.out.println(arr[0]);  // 1
arr[0] = 10;

// 다차원 배열
int[][] matrix = new int[3][3];
matrix[0][0] = 1;
```

---

## 6.6 for-comprehension

### 6.6.1 기본 for-comprehension

```scala
val numbers = List(1, 2, 3, 4, 5)

// for-yield: 새 컬렉션 생성
val doubled = for (n <- numbers) yield n * 2
println(doubled)  // List(2, 4, 6, 8, 10)

// for-loop: 부수 효과 (yield 없음)
for (n <- numbers) {
  println(n)
}

// 필터링
val evens = for {
  n <- numbers
  if n % 2 == 0
} yield n
println(evens)  // List(2, 4)

// 여러 조건
val result = for {
  n <- numbers
  if n > 2
  if n < 5
} yield n * 10
println(result)  // List(30, 40)
```

**Java 비교:**
```java
// Java: 전통적 for-loop
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

List<Integer> doubled = new ArrayList<>();
for (int n : numbers) {
    doubled.add(n * 2);
}

// Java 8+ Stream
List<Integer> evens = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

### 6.6.2 중첩 for-comprehension

```scala
val list1 = List(1, 2, 3)
val list2 = List(10, 20)

// 중첩 루프
val pairs = for {
  x <- list1
  y <- list2
} yield (x, y)

println(pairs)
// List((1,10), (1,20), (2,10), (2,20), (3,10), (3,20))

// flatMap + map과 동일
val pairs2 = list1.flatMap(x => list2.map(y => (x, y)))
println(pairs2)  // 동일한 결과
```

### 6.6.3 Map에서 for-comprehension

```scala
val scores = Map("Alice" -> 90, "Bob" -> 85, "Charlie" -> 95)

// Map 순회
for ((name, score) <- scores) {
  println(s"$name: $score")
}

// Map 변환
val adjusted = for ((name, score) <- scores) yield {
  (name, score + 5)
}
println(adjusted)  // Map(Alice -> 95, Bob -> 90, Charlie -> 100)

// 필터 + 변환
val highScores = for {
  (name, score) <- scores
  if score >= 90
} yield (name, score)
println(highScores)  // Map(Alice -> 90, Charlie -> 95)
```

---

## 6.7 실전 예제: 데이터 처리

### 6.7.1 CSV 데이터 파싱

```scala
case class Student(name: String, age: Int, score: Int)

val csvData = List(
  "Alice,20,90",
  "Bob,21,85",
  "Charlie,22,95",
  "David,20,88"
)

// CSV 파싱
val students = csvData.map { line =>
  val parts = line.split(",")
  Student(parts(0), parts(1).toInt, parts(2).toInt)
}

// 통계 계산
val avgScore = students.map(_.score).sum.toDouble / students.size
println(f"Average score: $avgScore%.2f")  // Average score: 89.50

// 필터링
val topStudents = students.filter(_.score >= 90)
topStudents.foreach(s => println(s"${s.name}: ${s.score}"))
// Alice: 90
// Charlie: 95

// 그룹화
val byAge = students.groupBy(_.age)
println(byAge)
// Map(20 -> List(Student(Alice,20,90), Student(David,20,88)),
//     21 -> List(Student(Bob,21,85)),
//     22 -> List(Student(Charlie,22,95)))
```

### 6.7.2 단어 빈도수 계산

```scala
val text = "Scala is great Scala is powerful Scala is concise"

// 단어 분리
val words = text.toLowerCase.split("\\s+").toList

// 빈도수 계산
val wordCount = words.groupBy(identity).mapValues(_.size)
println(wordCount)
// Map(scala -> 3, is -> 3, great -> 1, powerful -> 1, concise -> 1)

// 정렬
val sortedByCount = wordCount.toList.sortBy(-_._2)
sortedByCount.foreach { case (word, count) =>
  println(s"$word: $count")
}
// scala: 3
// is: 3
// great: 1
// powerful: 1
// concise: 1
```

---

## 6.8 성능 고려사항

### 6.8.1 컬렉션 선택 가이드

| 작업 | List | Vector | Array | Set | Map |
|------|------|--------|-------|-----|-----|
| **앞에 추가** | O(1) | O(log n) | O(n) | - | - |
| **뒤에 추가** | O(n) | O(log n) | O(1) | O(1) | O(1) |
| **인덱스 접근** | O(n) | O(log n) | O(1) | - | - |
| **검색** | O(n) | O(n) | O(n) | O(1) | O(1) |
| **중복 허용** | ✅ | ✅ | ✅ | ❌ | ❌(키) |
| **순서 유지** | ✅ | ✅ | ✅ | ❌ | ❌ |

### 6.8.2 불변 vs 가변

```scala
// 불변: 함수형 프로그래밍, 스레드 안전
val immutableList = List(1, 2, 3)
val newList = immutableList :+ 4  // 새 리스트 생성

// 가변: 성능 최적화, 많은 수정 필요 시
val mutableBuffer = scala.collection.mutable.ListBuffer(1, 2, 3)
mutableBuffer += 4  // 원본 수정
```

**권장사항:**
- 기본적으로 **불변 컬렉션** 사용
- 성능 병목이 확인된 경우에만 **가변 컬렉션** 고려
- 가변 컬렉션 사용 시 스코프를 좁게 유지

---

## 6.9 실습 과제

### 과제 6-1: 학생 성적 관리

다음 요구사항을 만족하는 프로그램을 작성하세요:

1. `Student` case class 정의 (name, scores: List[Int])
2. 평균 점수 계산 메서드
3. 상위 N명 학생 반환 메서드
4. 과목별 평균 점수 계산

**힌트:**
```scala
case class Student(name: String, scores: List[Int]) {
  def average: Double = ???
}

object StudentAnalyzer {
  def topN(students: List[Student], n: Int): List[Student] = ???

  def subjectAverages(students: List[Student]): List[Double] = ???
}
```

### 과제 6-2: 전화번호부

Map을 사용하여 전화번호부를 구현하세요:

```scala
class PhoneBook {
  private var contacts: Map[String, String] = Map.empty

  def add(name: String, phone: String): Unit = ???
  def remove(name: String): Unit = ???
  def lookup(name: String): Option[String] = ???
  def all: List[(String, String)] = ???
}
```

---

## 6.10 요약

이 챕터에서 배운 내용:

✅ Scala 컬렉션 계층 구조 (Seq, Set, Map)
✅ 불변 vs 가변 컬렉션
✅ List, Vector, Array의 특징과 성능
✅ Set과 Map의 기본 연산
✅ 고차 함수 (map, filter, fold, reduce)
✅ for-comprehension을 활용한 컬렉션 조작
✅ 실전 데이터 처리 예제

**Java와 주요 차이점:**
- Scala: 불변 컬렉션이 기본 (Java: 가변이 기본)
- Scala: 풍부한 고차 함수 API (Java: Stream API 필요)
- Scala: for-comprehension (Java: 전통적 for-loop 또는 Stream)

**Part 1 완료!**
다음 Part 2에서는 고급 함수형 프로그래밍을 다룹니다.

---

## Scala 3 차이점

```scala
// Scala 3: extension methods로 컬렉션 확장
extension [T](list: List[T])
  def second: T = list.tail.head

println(List(1, 2, 3).second)  // 2

// Scala 3: 더 간결한 for-comprehension
val result = for
  x <- List(1, 2, 3)
  y <- List(10, 20)
  if x > 1
yield x * y
```

---

## 참고 자료

- [Scala Collections](https://docs.scala-lang.org/overviews/collections-2.13/introduction.html)
- [Scala Standard Library API](https://www.scala-lang.org/api/current/)
- [Scala Collections Performance](https://docs.scala-lang.org/overviews/collections-2.13/performance-characteristics.html)
