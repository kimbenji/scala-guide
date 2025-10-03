# Part 3: 고급 주제

> 타입 시스템, 동시성, 메타프로그래밍 등 고급 기능 활용

## 📚 학습 목표

- Higher-Kinded Types와 타입 레벨 프로그래밍 이해
- Future/Promise를 활용한 비동기 프로그래밍 습득
- 매크로와 리플렉션으로 메타프로그래밍 구현
- DSL 설계 패턴 및 실무 활용법 학습

## 📖 챕터 목록

### [Chapter 11: 타입 시스템 고급](chapter11-advanced-type-system.md)
- Higher-Kinded Types (Functor, Monad)
- Path-Dependent Types
- Self Types (Cake Pattern)
- Type Lambdas (Kind Projector)
- 실전 예제: Free Monad, Tagless Final

### [Chapter 12: 동시성과 Future](chapter12-concurrency-future.md)
- Future와 Promise 기본
- ExecutionContext 관리
- Future 조합 (sequence, traverse, zip)
- 비동기 에러 처리
- 실전 예제: 병렬 API 호출

### [Chapter 13: 매크로와 리플렉션](chapter13-macros-reflection.md)
- TypeTag와 ClassTag
- 런타임 리플렉션
- Scala 2 Macros
- 컴파일 타임 검증
- Scala 3 Inline Macros

### [Chapter 14: DSL 설계](chapter14-dsl-design.md)
- Internal DSL vs External DSL
- 설계 기법 (메서드 체이닝, 중위 표기법)
- 실전 예제: SQL DSL, HTML 빌더
- 설계 원칙 (가독성, 타입 안전성)

## ⏱️ 예상 학습 시간

**총 2주** (주당 10-15시간 투자 기준)

- Chapter 11-12: 1주
- Chapter 13-14: 1주

## 🎯 학습 성과

Part 3을 완료하면 다음을 할 수 있습니다:

✅ 고급 타입 시스템으로 복잡한 로직 구현
✅ 비동기/병렬 프로그래밍으로 성능 최적화
✅ 매크로로 코드 자동 생성
✅ DSL 설계로 도메인 로직 표현

---

**[← Part 2: 함수형 프로그래밍](../part2-functional/README.md)** | **[메인 목차](../README.md)** | **[다음: Part 4 Apache Spark →](../part4-spark/README.md)**
