# Chapter 15: Apache Spark 기초 및 실전 프로젝트

## 학습 목표
- Apache Spark의 핵심 개념 이해 (RDD, DataFrame, Dataset)
- Spark SQL을 활용한 데이터 처리
- 실전 프로젝트 1: 로그 분석 시스템 구현
- 실전 프로젝트 2: 실시간 스트리밍 처리
- 실전 프로젝트 3: 머신러닝 파이프라인

---

## 15.1 Spark 기초

### 15.1.1 Spark 개요

**Apache Spark**: 대규모 데이터 처리를 위한 통합 분석 엔진

```scala
// build.sbt
libraryDependencies ++= Seq(
  "org.apache.spark" %% "spark-core" % "3.5.0",
  "org.apache.spark" %% "spark-sql" % "3.5.0",
  "org.apache.spark" %% "spark-mllib" % "3.5.0"
)
```

### 15.1.2 SparkSession 생성

```scala
import org.apache.spark.sql.SparkSession

val spark = SparkSession.builder()
  .appName("Scala Learning Guide")
  .master("local[*]")  // 로컬 모드, 모든 코어 사용
  .config("spark.driver.memory", "2g")
  .getOrElse()

try {
  // Spark 작업 수행
} finally {
  spark.stop()
}
```

---

## 15.2 DataFrame API

### 15.2.1 DataFrame 생성

```scala
import spark.implicits._

// Case class 정의
case class Person(name: String, age: Int, city: String)

// 데이터 생성
val people = Seq(
  Person("Alice", 25, "Seoul"),
  Person("Bob", 30, "Busan"),
  Person("Charlie", 35, "Seoul"),
  Person("David", 28, "Incheon")
).toDF()

people.show()
/*
+-------+---+-------+
|   name|age|   city|
+-------+---+-------+
|  Alice| 25|  Seoul|
|    Bob| 30|  Busan|
|Charlie| 35|  Seoul|
|  David| 28|Incheon|
+-------+---+-------+
*/
```

### 15.2.2 DataFrame 연산

```scala
// 필터링
people.filter($"age" > 28).show()

// 선택
people.select("name", "city").show()

// 그룹화
people.groupBy("city").count().show()

// 정렬
people.orderBy($"age".desc).show()

// 집계
import org.apache.spark.sql.functions._
people.agg(
  avg("age").as("average_age"),
  max("age").as("max_age"),
  min("age").as("min_age")
).show()
```

---

## 15.3 Spark SQL

```scala
// 임시 뷰 생성
people.createOrReplaceTempView("people")

// SQL 쿼리 실행
val result = spark.sql("""
  SELECT city, COUNT(*) as count, AVG(age) as avg_age
  FROM people
  GROUP BY city
  ORDER BY count DESC
""")

result.show()
```

---

## 15.4 실전 프로젝트 1: 로그 분석 시스템

### 15.4.1 프로젝트 개요

**목표**: 웹 서버 액세스 로그 분석하여 인사이트 추출

```scala
// 로그 데이터 Case Class
case class AccessLog(
  ip: String,
  timestamp: java.sql.Timestamp,
  method: String,
  endpoint: String,
  status: Int,
  size: Long
)

// 로그 파싱 함수
import java.sql.Timestamp
import java.text.SimpleDateFormat

def parseLog(line: String): Option[AccessLog] = {
  val pattern = """(\S+) - - \[([\w:/]+\s[+\-]\d{4})\] "(\S+) (\S+) \S+" (\d{3}) (\d+)""".r

  line match {
    case pattern(ip, timestamp, method, endpoint, status, size) =>
      try {
        val dateFormat = new SimpleDateFormat("dd/MMM/yyyy:HH:mm:ss Z")
        val ts = new Timestamp(dateFormat.parse(timestamp).getTime)
        Some(AccessLog(ip, ts, method, endpoint, status.toInt, size.toLong))
      } catch {
        case _: Exception => None
      }
    case _ => None
  }
}

// 로그 파일 로드
val logLines = spark.read.textFile("logs/access.log")
val accessLogs = logLines.flatMap(parseLog).toDF()

accessLogs.show(5)
```

### 15.4.2 분석 쿼리

```scala
// 1. 가장 많이 방문된 엔드포인트
accessLogs
  .groupBy("endpoint")
  .count()
  .orderBy($"count".desc)
  .limit(10)
  .show()

// 2. HTTP 상태 코드 분포
accessLogs
  .groupBy("status")
  .count()
  .orderBy("status")
  .show()

// 3. 시간대별 트래픽
accessLogs
  .withColumn("hour", hour($"timestamp"))
  .groupBy("hour")
  .count()
  .orderBy("hour")
  .show()

// 4. IP별 요청 수 (Top 10)
accessLogs
  .groupBy("ip")
  .agg(
    count("*").as("requests"),
    sum("size").as("total_bytes")
  )
  .orderBy($"requests".desc)
  .limit(10)
  .show()

// 5. 에러 로그 분석 (4xx, 5xx)
val errorLogs = accessLogs.filter($"status" >= 400)
errorLogs
  .groupBy("endpoint", "status")
  .count()
  .orderBy($"count".desc)
  .show()
```

### 15.4.3 결과 저장

```scala
// Parquet 형식으로 저장
accessLogs
  .write
  .mode("overwrite")
  .partitionBy("status")
  .parquet("output/access_logs")

// CSV 형식으로 저장
errorLogs
  .write
  .mode("overwrite")
  .option("header", "true")
  .csv("output/error_logs")
```

---

## 15.5 실전 프로젝트 2: 실시간 스트리밍 처리

### 15.5.1 Structured Streaming

```scala
import org.apache.spark.sql.streaming._

// 스트리밍 소스 생성 (예: 소켓)
val lines = spark.readStream
  .format("socket")
  .option("host", "localhost")
  .option("port", 9999)
  .load()

// 워드 카운트
val wordCounts = lines
  .as[String]
  .flatMap(_.split(" "))
  .groupBy("value")
  .count()

// 콘솔 출력
val query = wordCounts.writeStream
  .outputMode("complete")
  .format("console")
  .start()

query.awaitTermination()
```

### 15.5.2 실시간 로그 모니터링

```scala
// 실시간 로그 스트림
val streamingLogs = spark.readStream
  .format("text")
  .load("logs/streaming/")

val parsedLogs = streamingLogs
  .as[String]
  .flatMap(parseLog)
  .toDF()

// 윈도우 기반 집계 (5분 단위)
val windowedCounts = parsedLogs
  .groupBy(
    window($"timestamp", "5 minutes", "1 minute"),
    $"status"
  )
  .count()

// 결과 출력
val query2 = windowedCounts.writeStream
  .outputMode("update")
  .format("console")
  .option("truncate", "false")
  .start()

query2.awaitTermination()
```

---

## 15.6 실전 프로젝트 3: 머신러닝 파이프라인

### 15.6.1 데이터 준비

```scala
import org.apache.spark.ml.feature._
import org.apache.spark.ml.classification.LogisticRegression

// 예제 데이터: 이메일 스팸 분류
case class Email(text: String, label: Double)

val emails = Seq(
  Email("Get rich quick! Click here!", 1.0),
  Email("Meeting at 3pm today", 0.0),
  Email("Win a free iPhone now!", 1.0),
  Email("Project deadline reminder", 0.0),
  Email("Congratulations! You won $1000", 1.0)
).toDF()

emails.show(truncate = false)
```

### 15.6.2 피처 엔지니어링

```scala
// 1. Tokenization
val tokenizer = new Tokenizer()
  .setInputCol("text")
  .setOutputCol("words")

// 2. TF-IDF
val hashingTF = new HashingTF()
  .setInputCol("words")
  .setOutputCol("rawFeatures")
  .setNumFeatures(1000)

val idf = new IDF()
  .setInputCol("rawFeatures")
  .setOutputCol("features")

// 3. 파이프라인 구성
import org.apache.spark.ml.Pipeline

val lr = new LogisticRegression()
  .setMaxIter(10)
  .setRegParam(0.01)

val pipeline = new Pipeline()
  .setStages(Array(tokenizer, hashingTF, idf, lr))

// 4. 모델 학습
val model = pipeline.fit(emails)

// 5. 예측
val testData = Seq(
  Email("Special offer just for you!", 0.0),
  Email("Can we reschedule the meeting?", 0.0)
).toDF()

val predictions = model.transform(testData)
predictions.select("text", "prediction").show(truncate = false)
```

### 15.6.3 모델 평가

```scala
import org.apache.spark.ml.evaluation.BinaryClassificationEvaluator

// 데이터 분할
val Array(trainingData, testData) = emails.randomSplit(Array(0.8, 0.2))

// 모델 학습
val trainedModel = pipeline.fit(trainingData)

// 예측
val predictions = trainedModel.transform(testData)

// 평가
val evaluator = new BinaryClassificationEvaluator()
  .setLabelCol("label")
  .setRawPredictionCol("rawPrediction")
  .setMetricName("areaUnderROC")

val auc = evaluator.evaluate(predictions)
println(s"AUC: $auc")
```

---

## 15.7 성능 최적화

### 15.7.1 파티셔닝

```scala
// 데이터 파티션 재조정
val repartitioned = accessLogs.repartition(10)

// 특정 컬럼 기준 파티셔닝
val partitioned = accessLogs.repartition($"status")
```

### 15.7.2 캐싱

```scala
// 자주 사용하는 DataFrame 캐싱
val cachedLogs = accessLogs.cache()

// 메모리 캐시
cachedLogs.persist(org.apache.spark.storage.StorageLevel.MEMORY_AND_DISK)

// 캐시 해제
cachedLogs.unpersist()
```

### 15.7.3 브로드캐스트 조인

```scala
import org.apache.spark.sql.functions.broadcast

// 작은 테이블을 브로드캐스트
val smallTable = spark.read.parquet("small_data.parquet")
val largeTable = spark.read.parquet("large_data.parquet")

val joined = largeTable.join(broadcast(smallTable), "key")
```

---

## 15.8 Java 개발자를 위한 팁

### 15.8.1 Java vs Scala Spark API

```java
// Java Spark
Dataset<Row> df = spark.read().json("data.json");
df.filter(col("age").gt(25)).show();
```

```scala
// Scala Spark (더 간결)
val df = spark.read.json("data.json")
df.filter($"age" > 25).show()
```

### 15.8.2 Lambda vs Scala 함수

```java
// Java
dataset.map((MapFunction<String, Integer>) s -> s.length(), Encoders.INT());
```

```scala
// Scala
dataset.map(_.length)
```

---

## 15.9 실습 과제

### 과제 15-1: 사용자 행동 분석

다음 사용자 이벤트 로그를 분석하세요:

```scala
case class UserEvent(userId: Int, action: String, timestamp: Timestamp, pageId: String)

// 1. 사용자별 세션 수 계산 (30분 이상 비활동 시 새 세션)
// 2. 가장 인기있는 페이지 Top 10
// 3. 사용자별 평균 세션 길이
```

### 과제 15-2: 추천 시스템

Collaborative Filtering으로 상품 추천 시스템 구현:

```scala
import org.apache.spark.ml.recommendation.ALS

case class Rating(userId: Int, itemId: Int, rating: Float)

// ALS 모델 학습 및 추천 생성
```

### 과제 15-3: 이상 탐지

로그 데이터에서 이상 패턴 탐지:

```scala
// 1. 정상 트래픽 패턴 학습
// 2. 비정상 IP 또는 요청 패턴 식별
// 3. 알림 생성
```

---

## 15.10 요약

이번 챕터에서 학습한 내용:

1. **Spark 기초**: SparkSession, DataFrame, Dataset
2. **Spark SQL**: 선언적 데이터 처리
3. **프로젝트 1**: 로그 분석 (배치 처리)
4. **프로젝트 2**: 실시간 스트리밍
5. **프로젝트 3**: 머신러닝 파이프라인
6. **성능 최적화**: 파티셔닝, 캐싱, 브로드캐스트

**다음 챕터 예고**: Chapter 16에서는 SBT, 테스팅, Java 상호운용 등 생태계와 도구를 학습합니다.

---

## Spark 버전별 차이

- **Spark 2.x**: RDD 중심
- **Spark 3.x**: DataFrame/Dataset API 최적화
- **Spark 3.5+**: Pandas API on Spark, Adaptive Query Execution (AQE)
