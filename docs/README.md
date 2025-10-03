# Scala 학습 가이드

> Java 백엔드 개발자를 위한 Scala 2.12 기반 실무 가이드북

## 📚 소개

이 가이드는 경력 3-5년의 Java 백엔드 개발자가 2개월 내에 Scala를 실무에 적용할 수 있도록 설계되었습니다.

**핵심 특징:**
- ✅ **Java 비교 예제**: 모든 개념을 Java와 비교하여 설명
- ✅ **실전 프로젝트**: Apache Spark 기반 3개 프로젝트 (로그 분석, 스트리밍, ML)
- ✅ **실습 과제**: 48개 실습 과제로 학습 강화
- ✅ **Scala 3 대비**: 모든 챕터에 Scala 3 차이점 명시

**학습 목표:**
- Scala 2.12 기본 문법 및 함수형 프로그래밍 습득
- Apache Spark를 활용한 빅데이터 처리 능력 개발
- 실무 프로젝트에 즉시 적용 가능한 실전 역량 배양

---

## 📖 목차

### [Part 1: Scala 기초](part1-basics/)
Java 개발자를 위한 Scala 기본 문법 및 핵심 개념

1. [Scala 시작하기](part1-basics/chapter01-getting-started.md) - 환경 설정, Hello World, REPL
2. [변수와 타입 시스템](part1-basics/chapter02-variables-types.md) - val/var, 타입 추론, 기본 타입
3. [제어 구조와 표현식](part1-basics/chapter03-control-structures.md) - if, match, for/while
4. [함수 기초](part1-basics/chapter04-functions.md) - 메서드 vs 함수, 고차 함수, 커링
5. [객체지향 프로그래밍](part1-basics/chapter05-oop.md) - 클래스, 트레이트, 케이스 클래스
6. [컬렉션 기초](part1-basics/chapter06-collections.md) - List, Set, Map, 기본 연산

### [Part 2: 함수형 프로그래밍](part2-functional/)
함수형 패러다임의 실무 활용 및 Scala 컬렉션 API 숙달

7. [고급 함수형 프로그래밍](part2-functional/chapter07-advanced-functional.md) - 순수 함수, 함수 합성, 부분 함수
8. [Option, Try, Either](part2-functional/chapter08-option-try-either.md) - 안전한 에러 처리
9. [암시적 변환 (Implicits)](part2-functional/chapter09-implicits.md) - implicit, 타입 클래스 패턴
10. [타입 시스템 심화](part2-functional/chapter10-advanced-types.md) - 제네릭, Variance, Phantom Types

### [Part 3: 고급 주제](part3-advanced/)
타입 시스템, 동시성, 메타프로그래밍 등 고급 기능

11. [타입 시스템 고급](part3-advanced/chapter11-advanced-type-system.md) - Higher-Kinded Types, Path-Dependent Types
12. [동시성과 Future](part3-advanced/chapter12-concurrency-future.md) - Future/Promise, ExecutionContext
13. [매크로와 리플렉션](part3-advanced/chapter13-macros-reflection.md) - TypeTag, 런타임 리플렉션, Macros
14. [DSL 설계](part3-advanced/chapter14-dsl-design.md) - Internal DSL, SQL/HTML/Config DSL

### [Part 4: Apache Spark](part4-spark/)
Spark 기반 실전 빅데이터 처리 프로젝트

15. [Apache Spark 기초 및 실전 프로젝트](part4-spark/chapter15-apache-spark.md)
    - Spark 기초 (DataFrame, Dataset API, Spark SQL)
    - 프로젝트 1: 로그 분석 시스템
    - 프로젝트 2: 실시간 스트리밍 처리
    - 프로젝트 3: 머신러닝 파이프라인

### [Part 5: 생태계와 도구](part5-ecosystem/)
실무 개발 환경 구축 및 생산성 도구

16. [Scala 생태계와 도구](part5-ecosystem/chapter16-ecosystem-tools.md)
    - SBT 빌드 시스템
    - ScalaTest 테스팅
    - Scala-Java 상호운용
    - 주요 라이브러리 (Cats, Akka, Play, Slick)
    - 프로덕션 배포

---

## 🚀 시작하기

### 사전 요구사항
- Java 8 이상 설치
- IntelliJ IDEA + Scala Plugin (권장)
- Git 기본 명령어 숙지

### 추천 학습 순서
1. **1-2주**: Part 1 (Scala 기초)
2. **2-3주**: Part 2 (함수형 프로그래밍)
3. **1-2주**: Part 3 (고급 주제) - 선택적
4. **3-4주**: Part 4 (Apache Spark 프로젝트)
5. **1주**: Part 5 (생태계와 도구)

**총 학습 기간**: 7-9주 (주당 10-15시간 투자 기준)

---

## 📂 프로젝트 문서

### 기획 문서
- [PRD (Product Requirements Document)](prd.md) - 프로젝트 요구사항 정의
- [Architecture Document](architecture.md) - 프로젝트 아키텍처 설계
- [Progress Tracking](progress.md) - 프로젝트 진행 상황

### 기술 스택
- **Scala**: 2.12.18
- **빌드 도구**: SBT 1.9.x
- **테스팅**: ScalaTest 3.2.x
- **Spark**: Apache Spark 3.5.x
- **JVM**: Java 11 LTS (권장)

---

## 🎯 학습 성과

이 가이드를 완료하면 다음을 할 수 있습니다:

✅ Scala로 타입 안전한 백엔드 애플리케이션 개발
✅ 함수형 프로그래밍 패러다임 실무 적용
✅ Apache Spark로 대규모 데이터 처리
✅ Scala-Java 혼합 프로젝트 개발
✅ SBT, ScalaTest 등 실무 도구 활용

---

## 📝 라이선스

이 문서는 교육 목적으로 작성되었습니다.

**작성자**: BMad Master Agent
**버전**: v1.0
**최종 업데이트**: 2025-10-03
