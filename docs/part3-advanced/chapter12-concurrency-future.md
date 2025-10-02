# Chapter 12: 동시성과 Future (Concurrency and Futures)

## 학습 목표
- Future와 Promise 이해 및 활용
- ExecutionContext 설정 및 관리
- 비동기 에러 처리 패턴
- Future 조합 및 체이닝
- 병렬 처리 최적화

---

## 12.1 Future 기초

### 12.1.1 동기 vs 비동기

```scala
import scala.concurrent.{Future, ExecutionContext}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.util.{Success, Failure}

// 동기 실행 (블로킹)
def fetchUserSync(id: Int): User = {
  Thread.sleep(1000)  // 네트워크 요청 시뮬레이션
  User(id, s"User$id")
}

val start = System.currentTimeMillis()
val user1 = fetchUserSync(1)
val user2 = fetchUserSync(2)
val end = System.currentTimeMillis()
println(s"Time: ${end - start}ms")  // ~2000ms

// 비동기 실행 (논블로킹)
def fetchUserAsync(id: Int): Future[User] = Future {
  Thread.sleep(1000)
  User(id, s"User$id")
}

val startAsync = System.currentTimeMillis()
val future1 = fetchUserAsync(1)
val future2 = fetchUserAsync(2)

// 결과 대기
Future.sequence(List(future1, future2)).onComplete {
  case Success(users) =>
    val endAsync = System.currentTimeMillis()
    println(s"Time: ${endAsync - startAsync}ms")  // ~1000ms (병렬 실행)
  case Failure(ex) => println(s"Error: $ex")
}

case class User(id: Int, name: String)
```

```java
// Java CompletableFuture와 비교
import java.util.concurrent.CompletableFuture;

CompletableFuture<User> future1 = CompletableFuture.supplyAsync(() -> fetchUser(1));
CompletableFuture<User> future2 = CompletableFuture.supplyAsync(() -> fetchUser(2));

CompletableFuture.allOf(future1, future2).thenRun(() -> {
    // 완료 처리
});
```

### 12.1.2 Future 콜백

```scala
val future: Future[Int] = Future {
  Thread.sleep(500)
  42
}

// onComplete: Success/Failure 모두 처리
future.onComplete {
  case Success(value) => println(s"Result: $value")
  case Failure(ex) => println(s"Error: ${ex.getMessage}")
}

// foreach: 성공 시만 처리
future.foreach(value => println(s"Success: $value"))

// failed: 실패 시만 처리
future.failed.foreach(ex => println(s"Failed: ${ex.getMessage}"))
```

---

## 12.2 Future 변환 및 체이닝

### 12.2.1 map과 flatMap

```scala
// map: 결과 변환
val future1: Future[Int] = Future(42)
val future2: Future[String] = future1.map(_.toString)
future2.foreach(println)  // "42"

// flatMap: Future 체이닝
def getUser(id: Int): Future[User] = Future(User(id, s"User$id"))
def getOrders(userId: Int): Future[List[Order]] = Future(List(Order(1, userId, 100)))

val result: Future[List[Order]] = for {
  user <- getUser(1)
  orders <- getOrders(user.id)
} yield orders

result.foreach(println)

case class Order(id: Int, userId: Int, amount: Int)
```

### 12.2.2 filter와 collect

```scala
val future: Future[Int] = Future(42)

// filter: 조건 불만족 시 NoSuchElementException
val filtered = future.filter(_ > 40)
filtered.foreach(println)  // 42

// collect: 부분 함수 적용
val collected = future.collect {
  case x if x > 40 => s"Large: $x"
}
collected.foreach(println)  // "Large: 42"
```

### 12.2.3 recover와 recoverWith

```scala
def riskyOperation(): Future[Int] = Future {
  if (scala.util.Random.nextBoolean()) throw new Exception("Random failure")
  else 42
}

// recover: 동기 복구
val recovered1 = riskyOperation().recover {
  case _: Exception => 0
}

// recoverWith: 비동기 복구
val recovered2 = riskyOperation().recoverWith {
  case _: Exception => Future(0)
}

// fallbackTo: 대체 Future
val primary = riskyOperation()
val fallback = Future(0)
val result = primary.fallbackTo(fallback)
```

---

## 12.3 ExecutionContext

### 12.3.1 글로벌 ExecutionContext

```scala
// 기본 ExecutionContext
import scala.concurrent.ExecutionContext.Implicits.global

val future = Future {
  println(s"Running on: ${Thread.currentThread().getName}")
  42
}
```

### 12.3.2 커스텀 ExecutionContext

```scala
import java.util.concurrent.Executors
import scala.concurrent.ExecutionContext

// 고정 크기 스레드 풀
val fixedPool: ExecutionContext = ExecutionContext.fromExecutor(
  Executors.newFixedThreadPool(4)
)

// 캐시된 스레드 풀
val cachedPool: ExecutionContext = ExecutionContext.fromExecutor(
  Executors.newCachedThreadPool()
)

// 단일 스레드
val singleThread: ExecutionContext = ExecutionContext.fromExecutor(
  Executors.newSingleThreadExecutor()
)

// 사용
implicit val ec: ExecutionContext = fixedPool

val future = Future {
  // fixedPool에서 실행
  42
}
```

### 12.3.3 블로킹 작업 분리

```scala
import scala.concurrent.{Future, blocking, ExecutionContext}

// 블로킹 작업용 ExecutionContext
implicit val blockingEC: ExecutionContext = ExecutionContext.fromExecutor(
  Executors.newCachedThreadPool()
)

def readFile(path: String): Future[String] = Future {
  blocking {  // 블로킹 작업 명시
    scala.io.Source.fromFile(path).mkString
  }
}(blockingEC)

// CPU 집약적 작업은 별도 ExecutionContext 사용
implicit val cpuIntensiveEC: ExecutionContext = ExecutionContext.fromExecutor(
  Executors.newFixedThreadPool(Runtime.getRuntime.availableProcessors())
)
```

---

## 12.4 Future 조합

### 12.4.1 sequence와 traverse

```scala
// sequence: List[Future[A]] => Future[List[A]]
val futures: List[Future[Int]] = List(
  Future(1),
  Future(2),
  Future(3)
)

val combined: Future[List[Int]] = Future.sequence(futures)
combined.foreach(println)  // List(1, 2, 3)

// traverse: List[A] => Future[List[B]]
val ids = List(1, 2, 3)
val users: Future[List[User]] = Future.traverse(ids)(id => getUser(id))
```

### 12.4.2 zip과 zipWith

```scala
val future1 = Future(10)
val future2 = Future(20)

// zip: Future[(A, B)]
val zipped: Future[(Int, Int)] = future1.zip(future2)
zipped.foreach { case (a, b) => println(s"$a, $b") }  // 10, 20

// zipWith: 결과 변환
val sum: Future[Int] = future1.zipWith(future2)(_ + _)
sum.foreach(println)  // 30
```

### 12.4.3 firstCompletedOf와 find

```scala
// firstCompletedOf: 가장 먼저 완료된 Future
val futures = List(
  Future { Thread.sleep(100); 1 },
  Future { Thread.sleep(50); 2 },
  Future { Thread.sleep(200); 3 }
)

val first = Future.firstCompletedOf(futures)
first.foreach(println)  // 2 (가장 빠름)

// find: 조건을 만족하는 첫 번째 결과
val found = Future.find(futures)(_ > 1)
found.foreach(println)  // Some(2)
```

---

## 12.5 Promise

### 12.5.1 Promise 기본

```scala
import scala.concurrent.Promise

// Promise 생성
val promise = Promise[Int]()
val future = promise.future

// 완료 처리
future.foreach(v => println(s"Result: $v"))

// Promise 완료
promise.success(42)
// 또는 실패
// promise.failure(new Exception("Failed"))

// 한 번만 완료 가능
// promise.success(100)  // IllegalStateException
```

### 12.5.2 실전 예제: 비동기 타임아웃

```scala
import scala.concurrent.duration._

def withTimeout[T](future: Future[T], timeout: FiniteDuration): Future[T] = {
  val promise = Promise[T]()

  // 원본 Future 결과 전달
  future.onComplete(promise.tryComplete)

  // 타임아웃 스케줄링
  val scheduler = Executors.newScheduledThreadPool(1)
  scheduler.schedule(new Runnable {
    def run(): Unit = promise.tryFailure(new TimeoutException(s"Timeout after $timeout"))
  }, timeout.length, timeout.unit)

  promise.future
}

// 사용
val slowOperation = Future {
  Thread.sleep(5000)
  42
}

val result = withTimeout(slowOperation, 2.seconds)
result.onComplete {
  case Success(v) => println(s"Result: $v")
  case Failure(ex) => println(s"Failed: ${ex.getMessage}")  // Timeout after 2 seconds
}
```

---

## 12.6 실전 예제: 병렬 API 호출

```scala
case class UserProfile(id: Int, name: String, email: String)
case class UserStats(userId: Int, posts: Int, followers: Int)
case class UserSettings(userId: Int, theme: String, notifications: Boolean)

// 개별 API 호출
def fetchProfile(id: Int): Future[UserProfile] = Future {
  Thread.sleep(100)
  UserProfile(id, s"User$id", s"user$id@example.com")
}

def fetchStats(id: Int): Future[UserStats] = Future {
  Thread.sleep(100)
  UserStats(id, 42, 100)
}

def fetchSettings(id: Int): Future[UserSettings] = Future {
  Thread.sleep(100)
  UserSettings(id, "dark", true)
}

// 병렬 조합
case class CompleteUser(
  profile: UserProfile,
  stats: UserStats,
  settings: UserSettings
)

def fetchCompleteUser(id: Int): Future[CompleteUser] = {
  val profileF = fetchProfile(id)
  val statsF = fetchStats(id)
  val settingsF = fetchSettings(id)

  for {
    profile <- profileF
    stats <- statsF
    settings <- settingsF
  } yield CompleteUser(profile, stats, settings)
}

// 사용
val start = System.currentTimeMillis()
fetchCompleteUser(1).onComplete {
  case Success(user) =>
    val end = System.currentTimeMillis()
    println(s"Fetched user in ${end - start}ms")  // ~100ms (병렬)
    println(user)
  case Failure(ex) => println(s"Error: ${ex.getMessage}")
}
```

---

## 12.7 에러 처리 패턴

### 12.7.1 에러 전파

```scala
def step1(): Future[Int] = Future(10)
def step2(x: Int): Future[Int] = Future {
  if (x < 20) throw new Exception("Too small")
  else x * 2
}
def step3(x: Int): Future[String] = Future(s"Result: $x")

// 중간 에러는 자동 전파
val pipeline: Future[String] = for {
  a <- step1()
  b <- step2(a)  // 에러 발생
  c <- step3(b)  // 실행 안 됨
} yield c

pipeline.recover {
  case ex => s"Error: ${ex.getMessage}"
}.foreach(println)  // "Error: Too small"
```

### 12.7.2 개별 에러 처리

```scala
def fetchMultipleUsers(ids: List[Int]): Future[List[Either[Throwable, User]]] = {
  val futures = ids.map { id =>
    getUser(id)
      .map(Right(_): Either[Throwable, User])
      .recover { case ex => Left(ex) }
  }
  Future.sequence(futures)
}

// 사용
fetchMultipleUsers(List(1, 2, 999)).foreach { results =>
  val (errors, users) = results.partition(_.isLeft)
  println(s"Success: ${users.size}, Errors: ${errors.size}")
}
```

---

## 12.8 성능 최적화

### 12.8.1 Await 사용 (테스트용)

```scala
import scala.concurrent.Await
import scala.concurrent.duration._

val future = Future(42)

// 블로킹 대기 (프로덕션에서는 피해야 함)
val result = Await.result(future, 5.seconds)
println(result)  // 42

// 완료 여부만 확인
val ready = Await.ready(future, 5.seconds)
```

### 12.8.2 배치 처리

```scala
// 대량 작업을 배치로 분할
def processBatch[A, B](items: List[A], batchSize: Int)(f: A => Future[B]): Future[List[B]] = {
  items.grouped(batchSize).foldLeft(Future.successful(List.empty[B])) { (accF, batch) =>
    for {
      acc <- accF
      results <- Future.sequence(batch.map(f))
    } yield acc ++ results
  }
}

// 사용: 100개씩 배치 처리
val ids = (1 to 1000).toList
processBatch(ids, 100)(getUser).foreach { users =>
  println(s"Processed ${users.size} users")
}
```

---

## 12.9 Java 개발자를 위한 팁

### 12.9.1 CompletableFuture와 비교

| 기능 | Scala Future | Java CompletableFuture |
|------|--------------|------------------------|
| 생성 | `Future { ... }` | `CompletableFuture.supplyAsync` |
| 변환 | `map`, `flatMap` | `thenApply`, `thenCompose` |
| 조합 | `zip`, `sequence` | `thenCombine`, `allOf` |
| 에러 처리 | `recover`, `recoverWith` | `exceptionally`, `handle` |
| 콜백 | `onComplete`, `foreach` | `thenAccept`, `whenComplete` |

### 12.9.2 혼동 포인트

1. **Future는 즉시 시작**
   ```scala
   val future = Future { ... }  // 즉시 실행 시작
   ```

2. **ExecutionContext는 implicit**
   ```scala
   def myFunc()(implicit ec: ExecutionContext): Future[Int] = Future(42)
   ```

3. **Await는 블로킹**
   - 프로덕션 코드에서 피하고 콜백 사용

---

## 12.10 실습 과제

### 과제 12-1: 병렬 파일 처리

여러 파일을 병렬로 읽고 처리하세요:

```scala
def processFiles(paths: List[String]): Future[Map[String, Int]] = ???

// 각 파일의 단어 수 계산
// 결과: Map(파일명 -> 단어수)
```

### 과제 12-2: 재시도 로직

실패 시 재시도하는 Future 래퍼:

```scala
def retry[T](times: Int)(f: => Future[T]): Future[T] = ???

// 최대 3회 재시도
val result = retry(3) {
  fetchFromUnreliableAPI()
}
```

### 과제 12-3: 캐시 구현

Future 결과를 캐싱하는 시스템:

```scala
class AsyncCache[K, V] {
  def get(key: K)(compute: => Future[V]): Future[V] = ???
}

// 같은 키의 중복 요청 방지
val cache = new AsyncCache[Int, User]()
val user1 = cache.get(1)(fetchUser(1))
val user2 = cache.get(1)(fetchUser(1))  // 캐시에서 반환
```

---

## 12.11 요약

이번 챕터에서 학습한 내용:

1. **Future**: 비동기 연산의 미래 결과
2. **ExecutionContext**: 스레드 풀 관리
3. **조합 패턴**: sequence, traverse, zip
4. **Promise**: 수동 Future 완료
5. **에러 처리**: recover, fallbackTo

**다음 챕터 예고**: Chapter 13에서는 매크로와 리플렉션을 활용한 메타프로그래밍을 학습합니다.

---

## Scala 3 주요 차이점

- **Async/Await**: 실험적 기능으로 제공
- **Structured Concurrency**: 새로운 동시성 모델 논의 중
- Future API는 대부분 동일
