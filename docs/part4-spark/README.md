# Part 4: Apache Spark

> Spark 기반 실전 빅데이터 처리 프로젝트

## 📚 학습 목표

- Apache Spark 핵심 개념 (RDD, DataFrame, Dataset) 이해
- Spark SQL을 활용한 대규모 데이터 처리
- 실전 프로젝트를 통한 실무 역량 배양
- 성능 최적화 기법 습득

## 📖 챕터 목록

### [Chapter 15: Apache Spark 기초 및 실전 프로젝트](chapter15-apache-spark.md)

#### Spark 기초
- SparkSession 생성 및 설정
- DataFrame API (필터링, 그룹화, 집계)
- Dataset API와 타입 안전성
- Spark SQL 쿼리

#### 실전 프로젝트 1: 로그 분석 시스템
- 웹 서버 액세스 로그 파싱
- 트래픽 분석 (시간대별, 엔드포인트별)
- 에러 로그 분석
- 결과 저장 (Parquet, CSV)

#### 실전 프로젝트 2: 실시간 스트리밍 처리
- Structured Streaming 기초
- 윈도우 기반 집계
- 실시간 로그 모니터링
- 스트림 처리 최적화

#### 실전 프로젝트 3: 머신러닝 파이프라인
- 이메일 스팸 분류 시스템
- TF-IDF 피처 엔지니어링
- Logistic Regression 모델 학습
- 모델 평가 및 예측

#### 성능 최적화
- 파티셔닝 전략
- 캐싱 및 영속화
- 브로드캐스트 조인
- 리소스 튜닝

## ⏱️ 예상 학습 시간

**총 3-4주** (주당 10-15시간 투자 기준)

- Spark 기초: 1주
- 프로젝트 1-2: 1-2주
- 프로젝트 3: 1주

## 🎯 학습 성과

Part 4를 완료하면 다음을 할 수 있습니다:

✅ Spark로 대규모 데이터 배치 처리
✅ 실시간 스트리밍 데이터 처리
✅ 머신러닝 파이프라인 구축
✅ Spark 성능 최적화

## 🔧 실습 환경

**필수 요구사항:**
- Scala 2.12.x
- Apache Spark 3.5.x
- 최소 8GB RAM (권장)
- IntelliJ IDEA + Scala Plugin

**데이터셋:**
- 공개 데이터셋 (Kaggle, UCI ML Repository)
- 합성 데이터 (프로젝트 제공)

---

**[← Part 3: 고급 주제](../part3-advanced/README.md)** | **[메인 목차](../README.md)** | **[다음: Part 5 생태계와 도구 →](../part5-ecosystem/README.md)**
