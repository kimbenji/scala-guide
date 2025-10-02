# Scala 학습 가이드 Product Requirements Document (PRD)

## Goals and Background Context

### Goals
- Java 백엔드 개발자의 Scala 실무 적용 능력 확보 (2개월 내)
- Apache Spark 기반 빅데이터 처리 역량 배양
- 함수형 프로그래밍의 실전 활용 능력 개발

### Background Context
빅데이터 처리 기술의 확산으로 Spark, Kafka 등 Scala 기반 도구가 광범위하게 채택되고 있으나, Java 개발자에게 Scala의 함수형 패러다임은 큰 진입 장벽으로 작용합니다. 기존 학습 자료는 이론 중심이거나 지나치게 단순하여 실무 적용이 어렵습니다.

본 가이드는 Java 경험을 가진 중급 개발자를 대상으로, Java와의 비교를 통한 개념 설명과 실전 프로젝트 중심 학습을 통해 7-9주 내에 Scala를 실무에 적용할 수 있도록 설계되었습니다.

### Change Log
| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-10-02 | v1.0 | 초안 작성 | BMad Master |
| 2025-10-02 | v1.1 | NFR7-8 추가 (문서 출력 형식), Technical Assumptions에 Documentation Format and Publishing 섹션 추가 | BMad Master |
| 2025-10-02 | v1.2 | Technical Assumptions에 Documentation Workflow 섹션 추가 (Git 버전 관리 명시) | BMad Master |

## Requirements

### Functional Requirements

- **FR1**: 가이드는 Scala 2.12 문법을 기본으로 작성되며, Scala 3의 주요 변경사항 및 Breaking Changes를 명시적으로 표시해야 함. 모든 개념은 Java와의 비교 예제를 포함해야 함
- **FR2**: 각 개념마다 최소 1개 이상의 실행 가능한 코드 예제를 제공해야 함
- **FR3**: Part 1-3의 각 챕터는 독립적으로 학습 가능하되, 권장 순서를 명시해야 함
- **FR4**: Apache Spark 관련 내용(Part 4)은 실무 시나리오 기반 3개의 실습 프로젝트를 포함해야 함
  - 로그 분석 시스템
  - 실시간 스트리밍 처리
  - 머신러닝 파이프라인
  - (각 프로젝트는 공개 또는 합성 데이터셋 활용)
- **FR5**: 함수형 프로그래밍(Part 2)은 카테고리 이론 같은 고급 수학적 배경을 제외하고 실무 중심으로 다뤄야 함
- **FR6**: Java 개발자가 자주 겪는 혼동 포인트(예: val/var, 표현식 vs 문장)를 명시적으로 설명해야 함
- **FR7**: Part 1, 2, 3의 마지막에는 통합 학습 성과를 확인할 수 있는 실습 과제를 제공해야 함 (Part 4는 Chapter 15의 실전 프로젝트로 대체)
- **FR8**: Scala-Java 상호 운용성 예제(Chapter 18)를 포함해야 함

### Non-Functional Requirements

- **NFR1**: 가이드의 총 분량은 A4 기준 300-400페이지 범위 내로 작성되어야 함
- **NFR2**: 모든 코드 예제는 Scala 2.12.x 버전에서 컴파일 및 실행 가능해야 하며, Scala 3 마이그레이션 시 필요한 주요 변경사항을 간단한 주석으로 명시해야 함
- **NFR3**: 코드 예제는 GitHub 저장소에서 챕터별로 구조화되어 제공되어야 함
- **NFR4**: 모든 문서는 한글로 작성되며, 기술 용어는 영문 병기를 원칙으로 함
- **NFR5**: Spark 프로젝트는 Spark 3.5.x + Scala 2.12 조합 기준으로 작성되며, 로컬 환경(standalone mode)에서 실행 가능해야 함
- **NFR6**: 가이드 작성 및 보완 과정은 진행 상황 추적 문서(progress.md)를 통해 관리되어야 함
- **NFR7**: 문서는 Markdown 포맷으로 작성되며, Mermaid.js를 활용한 다이어그램과 코드 하이라이팅을 포함해야 함
- **NFR8**: 코드 예제는 코드 블록으로 표시하며, 긴 코드의 경우 페이지 넘김을 고려한 레이아웃 처리를 해야 함

## Technical Assumptions

### Development Environment
- **운영체제**: macOS, Linux, Windows 10+ (크로스 플랫폼 지원)
- **JVM**: Java 8 이상 (권장: Java 11 LTS)
- **빌드 도구**: SBT 1.9.x 이상
- **IDE**: IntelliJ IDEA (Community 또는 Ultimate) + Scala Plugin
  - 대안: VS Code + Metals (경량 환경 선호 시)

### Testing and Quality
- **단위 테스트**: ScalaTest 3.2.x
- **코드 스타일**: Scalafmt (자동 포매팅)
- **정적 분석**: 선택 사항 (Scalafix, WartRemover 소개 수준)

### Documentation Format and Publishing
- **문서 포맷**: Markdown (GitHub Flavored Markdown)
- **다이어그램**: Mermaid.js (플로우차트, 시퀀스 다이어그램, 클래스 다이어그램)
- **코드 하이라이팅**: 언어별 Syntax Highlighting (Scala, Java, Bash, JSON, YAML)
- **페이지 레이아웃**: A4 (210mm × 297mm)
  - 여백: 상하 25mm, 좌우 20mm
  - 본문 폰트: 11pt (권장)
  - 코드 폰트: 9-10pt 고정폭 폰트 (권장)
  - 줄 간격: 1.5 (본문), 1.2 (코드 블록)
- **코드 블록 처리**:
  - 모든 코드 예제는 펜스드 코드 블록(```) 사용
  - 긴 코드(30줄 초과)는 페이지 넘김 고려하여 논리적 단위로 분할
  - 코드 블록 상단에 파일명 또는 설명 주석 포함 (예: `// Example: HelloWorld.scala`)
- **출력 형식**: Markdown 원본 유지 (향후 PDF, HTML, ePub 변환 가능성 고려)

### Repository Structure
- **구조**: Monorepo (단일 GitHub 저장소)
- **디렉토리 구성**:
  ```
  scala-guide/
  ├── part1-basics/
  │   ├── chapter01-getting-started/
  │   ├── chapter02-variables-types/
  │   └── ...
  ├── part2-functional/
  │   └── ...
  ├── part4-spark/
  │   ├── project1-log-analysis/
  │   ├── project2-streaming/
  │   └── project3-ml-pipeline/
  └── build.sbt (루트 SBT 멀티 프로젝트 설정)
  ```

### Deployment and CI/CD
- **실행 환경**: 로컬 개발 환경만 지원 (CI/CD 파이프라인 구축 제외)
- **Spark 실행**: Local mode, Standalone mode (클러스터 배포 가이드 제외)

### Hardware Requirements
- **최소 사양** (Part 1-3 학습):
  - CPU: 2 코어 이상
  - RAM: 4GB 이상
  - 디스크: 500MB (SBT 캐시 및 의존성 포함)
- **권장 사양** (Part 4 Spark 프로젝트):
  - CPU: 4 코어 이상
  - RAM: 8GB 이상
  - 디스크: 2GB 이상

### Third-Party Dependencies
- **코어 라이브러리**: Scala Standard Library 2.12.x
- **Spark**: Apache Spark 3.5.x (spark-core, spark-sql, spark-mllib)
- **로깅**: SLF4J + Logback
- **데이터셋**: 공개 데이터셋 (Kaggle, UCI ML Repository) 또는 합성 데이터

### Assumptions and Constraints
1. 독자는 Git 기본 명령어를 숙지하고 있다고 가정
2. 독자는 CLI 환경에서 빌드 도구 실행이 가능하다고 가정
3. 모든 예제는 인터넷 연결 없이 로컬 실행 가능해야 함 (의존성 다운로드 후)
4. Scala 2.13, Scala 3 마이그레이션은 별도 부록으로 분리 (선택 사항)
5. 프로덕션 배포, 성능 튜닝, 보안은 범위 외 (입문자 대상)

### Documentation Workflow
- **버전 관리**: Git을 통한 문서 이력 추적
- **커밋 단위**: 주요 진행 단계(섹션 완성, 챕터 완료, Epic 완료 등)마다 커밋 필수
- **상세 워크플로우**: Architecture 문서에서 정의 (커밋 메시지 컨벤션, 브랜치 전략, 태그 전략 등)

## Epic List

| Epic ID | Epic Name | Description | Priority | Estimated Pages |
|---------|-----------|-------------|----------|-----------------|
| E1 | Scala 기본 문법 마스터하기 | Java 개발자를 위한 Scala 기본 문법 및 핵심 개념 학습 (Chapter 1-6) | P0 | 80-100 |
| E2 | 함수형 프로그래밍 실전 적용 | 함수형 패러다임의 실무 활용 및 Scala 컬렉션 API 숙달 (Chapter 7-10) | P0 | 70-90 |
| E3 | Scala 고급 기능 활용 | 타입 시스템, 암시적 변환, 동시성 프로그래밍 (Chapter 11-13) | P1 | 60-80 |
| E4 | Apache Spark 빅데이터 처리 | Spark 기반 실전 프로젝트 구현 (로그분석, 스트리밍, ML) (Chapter 14-15) | P0 | 80-100 |
| E5 | Scala 생태계 및 도구 활용 | 빌드 도구, 테스팅, Java 상호운용 (Chapter 16-18) | P2 | 40-50 |

**우선순위 설명:**
- **P0**: 필수 (Must Have) - 핵심 학습 목표 달성에 필수적인 내용
- **P1**: 중요 (Should Have) - 실무 활용도가 높은 고급 내용
- **P2**: 선택 (Nice to Have) - 생산성 향상 및 심화 학습

## Epic Details

### Epic 1: Scala 기본 문법 마스터하기

**목표**: Java 개발자가 Scala 기본 문법을 이해하고 간단한 프로그램을 작성할 수 있도록 함

**범위**:
- Chapter 1: Scala 시작하기 (환경 설정, Hello World, REPL)
- Chapter 2: 변수와 타입 시스템 (val/var, 기본 타입, 타입 추론)
- Chapter 3: 제어 구조와 표현식 (if, match, for/while, 표현식 vs 문장)
- Chapter 4: 함수 기초 (메서드 vs 함수, 매개변수, 반환 타입)
- Chapter 5: 객체지향 프로그래밍 (클래스, 트레이트, 케이스 클래스, 상속)
- Chapter 6: 컬렉션 기초 (List, Set, Map, 기본 연산)

**수용 기준 (Acceptance Criteria)**:
- [ ] 각 챕터는 Java 비교 예제를 최소 3개 이상 포함
- [ ] REPL 실습 가이드 제공
- [ ] Part 1 종합 실습 과제 1개 (간단한 CLI 애플리케이션)
- [ ] Scala 2.12 vs Scala 3 차이점 명시 (주요 변경사항만)

**의존성**: 없음 (첫 번째 Epic)

**예상 완료 시간**: 2-3주 (독자 학습 기준)

---

### Epic 2: 함수형 프로그래밍 실전 적용

**목표**: 함수형 프로그래밍 패러다임을 이해하고 Scala 컬렉션 API를 활용한 데이터 처리 능력 배양

**범위**:
- Chapter 7: 함수형 프로그래밍 개념 (순수 함수, 불변성, 참조 투명성)
- Chapter 8: 고차 함수와 클로저 (map, filter, reduce, fold)
- Chapter 9: 컬렉션 심화 (for-comprehension, flatMap, groupBy, 지연 평가)
- Chapter 10: 패턴 매칭과 Option (null 안전성, Either, Try)

**수용 기준 (Acceptance Criteria)**:
- [ ] 실무 시나리오 기반 예제 (로그 파싱, JSON 처리 등) 각 챕터당 2개
- [ ] Java Stream API vs Scala 컬렉션 비교표
- [ ] Part 2 종합 실습 과제 1개 (데이터 변환 파이프라인)
- [ ] 함수형 프로그래밍 안티패턴 (과도한 추상화 등) 경고

**의존성**: Epic 1 완료 (기본 문법 선행 필요)

**예상 완료 시간**: 2-3주

---

### Epic 3: Scala 고급 기능 활용

**목표**: 타입 시스템, 암시적 변환, 동시성 등 고급 기능을 통해 실무 코드 품질 향상

**범위**:
- Chapter 11: 타입 시스템 심화 (제네릭, 공변/반공변, 상한/하한)
- Chapter 12: Implicit과 타입 클래스 (implicit parameter, implicit conversion, 타입 클래스 패턴)
- Chapter 13: 동시성과 Future (Future/Promise, ExecutionContext, 병렬 컬렉션)

**수용 기준 (Acceptance Criteria)**:
- [ ] 타입 시스템 오류 트러블슈팅 가이드
- [ ] Implicit 남용 방지 가이드 (Best Practices)
- [ ] 동시성 예제: 비동기 HTTP 호출 처리
- [ ] Part 3 종합 실습 과제 1개 (비동기 데이터 처리 애플리케이션)

**의존성**: Epic 1, 2 완료

**예상 완료 시간**: 2주

---

### Epic 4: Apache Spark 빅데이터 처리

**목표**: Apache Spark를 활용한 실전 빅데이터 처리 프로젝트 구현

**범위**:
- Chapter 14: Apache Spark 기초 (RDD, DataFrame, Dataset API, Spark SQL)
- Chapter 15: Spark 실전 프로젝트
  - 프로젝트 1: 로그 분석 시스템 (배치 처리)
  - 프로젝트 2: 실시간 스트리밍 처리 (Structured Streaming)
  - 프로젝트 3: 머신러닝 파이프라인 (MLlib)

**수용 기준 (Acceptance Criteria)**:
- [ ] 3개 프로젝트 모두 로컬 환경에서 실행 가능
- [ ] 각 프로젝트는 공개 또는 합성 데이터셋 포함
- [ ] 성능 모니터링 및 디버깅 가이드
- [ ] Spark 3.5.x + Scala 2.12 호환성 검증
- [ ] RDD vs DataFrame vs Dataset 성능 비교 벤치마크

**의존성**: Epic 1, 2 완료 (Epic 3은 선택 사항)

**예상 완료 시간**: 3-4주

---

### Epic 5: Scala 생태계 및 도구 활용

**목표**: 실무 개발 환경 구축 및 생산성 도구 활용 능력 배양

**범위**:
- Chapter 16: SBT와 빌드 관리 (멀티 프로젝트, 의존성 관리, 플러그인)
- Chapter 17: 테스팅과 디버깅 (ScalaTest, 속성 기반 테스팅, 디버거 활용)
- Chapter 18: Scala-Java 상호운용 (Java 라이브러리 호출, Scala 코드를 Java에서 사용)

**수용 기준 (Acceptance Criteria)**:
- [ ] SBT 멀티 프로젝트 설정 예제
- [ ] ScalaTest 테스트 케이스 작성 가이드 (최소 10개 예제)
- [ ] Java-Scala 상호운용 시 주의사항 및 트러블슈팅
- [ ] IntelliJ IDEA 디버거 활용 가이드

**의존성**: Epic 1 완료 (독립적으로 학습 가능)

**예상 완료 시간**: 1-2주

## Checklist Results Report

### Executive Summary

**전체 완성도**: 75%
**MVP 범위 적정성**: Just Right
**Architecture 단계 준비 상태**: Nearly Ready
**가장 중요한 격차**: User Journey/Flow 미정의, 일부 NFR 누락

이 PRD는 학습 가이드북 프로젝트의 특성을 고려할 때 대부분의 핵심 요구사항을 포함하고 있습니다. 일반적인 소프트웨어 제품과 달리 "독자의 학습 경험"이 UX의 핵심이며, 기술적 복잡도보다는 교육 콘텐츠의 완성도가 중요합니다.

### Category Analysis

| Category | Status | Critical Issues | 비고 |
|----------|--------|----------------|------|
| 1. Problem Definition & Context | PASS | 없음 | 문제 정의, 목표, 대상 독자 명확 |
| 2. MVP Scope Definition | PASS | 없음 | 5개 Epic, 18개 Chapter 구조 명확, 범위 경계 잘 정의됨 |
| 3. User Experience Requirements | PARTIAL | 독자 학습 여정(Journey) 미정의 | 학습 가이드 특성상 전통적 UI/UX와 다름 |
| 4. Functional Requirements | PASS | 없음 | 8개 FR 모두 구체적이고 테스트 가능 |
| 5. Non-Functional Requirements | PARTIAL | 페이지 품질 기준 미정의, 접근성 누락 | 문서 분량은 있으나 가독성/접근성 기준 필요 |
| 6. Epic & Story Structure | PASS | Story 세부 정의 없음 (의도적) | Epic 정의 완료, Story는 Architecture 단계에서 정의 예정 |
| 7. Technical Guidance | PASS | 없음 | 개발 환경, 빌드 도구, 테스팅 전략 명확 |
| 8. Cross-Functional Requirements | PARTIAL | 예제 코드 검증 프로세스 미정의 | 저장소 구조는 있으나 코드 품질 관리 방안 필요 |
| 9. Clarity & Communication | PASS | 없음 | 한글 작성, 용어 병기 원칙 명확 |

### Top Issues by Priority

#### BLOCKERS (없음)
현재 Architecture 단계 진행을 막는 블로커는 없습니다.

#### HIGH (Architecture 단계 전 해결 권장)
1. **독자 학습 여정(Learning Journey) 미정의**
   - 문제: Chapter 간 학습 순서는 있으나, 독자가 겪을 학습 경험(예: 막힐 수 있는 지점, 학습 동기 유지 방안)이 명시되지 않음
   - 영향: Architecture 단계에서 Chapter 구조 설계 시 학습 흐름 최적화 어려움
   - 해결안: 각 Part별 학습 목표 달성 과정(Learning Path)을 시각화하거나 문서화

2. **코드 예제 검증 프로세스 미정의**
   - 문제: 300개 이상의 코드 예제가 예상되나, 컴파일/실행 검증 자동화 방안 없음
   - 영향: 코드 품질 일관성 유지 어려움, 유지보수 부담 증가
   - 해결안: SBT 테스트 통합, CI 파이프라인(선택 사항)

#### MEDIUM (품질 향상)
3. **페이지 품질 기준 미정의**
   - 문제: NFR1에서 "300-400페이지"는 있으나, 가독성(Flesch Reading Ease 등), 코드-설명 비율 등 품질 기준 없음
   - 해결안: 페이지당 예제 개수, 최대 문단 길이 등 편집 가이드라인 추가

4. **접근성 요구사항 누락**
   - 문제: 시각장애인, 색각이상자 등을 위한 접근성 고려 없음 (코드 예제의 색상 의존성, 스크린 리더 호환성)
   - 해결안: NFR 추가 또는 Technical Assumptions에 포함

#### LOW (선택 사항)
5. **Scala 3 마이그레이션 가이드 범위 모호**
   - 문제: "별도 부록"이라고만 명시, 구체적 범위 없음
   - 해결안: 부록 페이지 수, 포함 내용 명시 (현재는 YAGNI 원칙으로 스킵 가능)

### MVP Scope Assessment

**전체 평가**: 적정 (Just Right)

#### 유지해야 할 강점
- ✅ Part 1-2 (기본 문법, 함수형)을 P0로 설정한 것은 적절
- ✅ 카테고리 이론 제외, 실무 중심 접근은 2개월 학습 목표에 부합
- ✅ 3개 Spark 프로젝트는 실전 역량 배양에 필수

#### 범위 조정 검토 (선택 사항)
- **축소 고려**: Epic 3 (고급 기능)을 P1으로 설정했으나, 2개월 학습 목표에 부담일 수 있음
  - 대안: Chapter 11 (타입 시스템)만 필수로, Chapter 12-13은 선택 학습
- **확대 고려 안 함**: 현재 범위가 이미 충분히 포괄적

#### 복잡도 우려
- Epic 4 (Spark)의 3개 프로젝트가 가장 복잡할 것으로 예상
- 각 프로젝트는 독립적으로 20-30페이지 분량 권장 (현재 80-100페이지 할당은 적절)

#### 타임라인 현실성
- **7-9주 학습 목표**: 주당 40-60페이지 학습 → 현실적
- **독자 시간 투자**: 주당 10-15시간 가정 시 달성 가능
- **작성 타임라인**: 미명시 (Architecture 단계에서 정의 필요)

### Technical Readiness

#### 기술적 제약 명확성: ✅ PASS
- Scala 2.12, SBT, ScalaTest, IntelliJ IDEA 모두 명시
- Spark 3.5.x + Scala 2.12 조합 검증 필요 (Epic 4 AC에 포함됨)

#### 식별된 기술적 리스크
1. **Spark 로컬 환경 실행 제약**: 독자 하드웨어 스펙이 8GB RAM 미만일 경우 실습 어려움
   - 완화 방안: 경량화된 샘플 데이터셋 제공, Docker 이미지 제공(선택)
2. **Scala 2.12 → 2.13/3.x 마이그레이션 주석의 정확성**: Scala 3 변경사항 추적 필요
   - 완화 방안: Scala 공식 마이그레이션 가이드 참조, 주요 변경사항만 표시

#### Architect 조사 필요 영역
1. **Monorepo SBT 멀티 프로젝트 구조 설계**
   - 18개 Chapter를 어떻게 SBT 서브 프로젝트로 구성할 것인가?
   - 공통 의존성 관리 전략
2. **예제 코드 네이밍 컨벤션**
   - 파일명, 패키지명, 클래스명 규칙
3. **데이터셋 저장 및 배포 전략**
   - GitHub LFS 사용 여부, 외부 다운로드 링크 제공 방식

### Recommendations

#### Architecture 단계 전 필수 조치 (HIGH 우선순위)
1. **독자 학습 여정 정의**
   ```
   추가할 섹션 제안:
   - PRD에 "Learning Path" 섹션 추가
   - 각 Epic별 "학습 목표 달성 마일스톤" 정의
   - Java 개발자가 막힐 수 있는 지점(Stumbling Blocks) 사전 식별
   ```

2. **코드 예제 품질 관리 방안**
   ```
   Technical Assumptions에 추가:
   - 모든 코드 예제는 SBT 테스트 케이스로 검증
   - 컴파일 오류, 런타임 오류 자동 감지
   - 코드 스타일 자동 검사 (Scalafmt)
   ```

#### Architecture 단계에서 해결 가능 (MEDIUM 우선순위)
3. **페이지 품질 기준 정의** → Architecture 문서의 "Content Guidelines"에 포함
4. **접근성 요구사항** → Architecture 문서의 "Documentation Standards"에 포함

#### 다음 단계 제안
1. ✅ **PRD 승인 및 잠금** (현재 버전 v1.0 동결)
2. ⏳ **Architecture 문서 작성 시작**
   - Chapter별 상세 구조 설계
   - 코드 저장소 디렉토리 구조
   - 예제 코드 작성 가이드라인
3. ⏳ **첫 번째 Chapter 프로토타입 작성** (Chapter 1 또는 Chapter 2)
   - 실제 작성 과정에서 PRD/Architecture 검증
   - 페이지 분량, 학습 시간 추정치 보정

### Final Decision

**✅ NEARLY READY FOR ARCHITECT**

이 PRD는 Architecture 단계를 시작하기에 충분한 정보를 담고 있습니다. 학습 가이드북이라는 특수성을 고려할 때, 전통적인 소프트웨어 제품의 UX 요구사항(User Journey, Wireframe 등)을 그대로 적용하는 것은 부적절합니다.

**진행 가능 조건**:
- HIGH 우선순위 이슈 2개를 Architecture 단계 초반에 해결하면 즉시 콘텐츠 작성 시작 가능
- 또는 현재 PRD로 Architecture 문서 초안을 작성하면서 병행 해결 가능

**권장 다음 단계**:
1. 독자 학습 여정(Learning Journey) 간단히 정의 (1-2페이지)
2. Architecture 문서 작성 시작
3. Chapter 1 프로토타입 작성으로 가설 검증

## Next Steps

### Immediate Actions (1-2일 내)
1. **PRD 보완** (선택 사항)
   - [ ] 독자 학습 여정(Learning Journey) 섹션 추가
   - [ ] 코드 예제 검증 프로세스를 Technical Assumptions에 추가

2. **PRD 승인 및 버전 관리**
   - [ ] 이해관계자 검토 (본인 확인)
   - [ ] PRD v1.0 최종 승인
   - [ ] 변경 사항은 v1.1, v1.2로 추적

### Short-term Actions (1주 내)
3. **Architecture 문서 작성**
   - [ ] `*create-doc architecture` 명령으로 Architecture 문서 생성
   - [ ] Chapter별 상세 구조 설계
   - [ ] Monorepo 디렉토리 구조 정의
   - [ ] 코드 예제 작성 가이드라인 수립

4. **프로토타입 제작**
   - [ ] Chapter 1 또는 Chapter 2 샘플 작성 (5-10페이지)
   - [ ] 실제 학습 시간 측정
   - [ ] 페이지 분량 추정치 검증

### Mid-term Actions (2-4주 내)
5. **Epic 1 콘텐츠 작성 시작**
   - [ ] Chapter 1-6 작성
   - [ ] 코드 예제 GitHub 저장소 구축
   - [ ] Part 1 종합 실습 과제 설계

6. **품질 관리 체계 구축**
   - [ ] 코드 예제 자동 테스트 환경 구성
   - [ ] 문서 리뷰 프로세스 정의
   - [ ] Progress 추적 시스템 정비

### Long-term Actions (1-2개월 내)
7. **Epic 2-5 콘텐츠 작성**
   - [ ] 함수형 프로그래밍 (Part 2)
   - [ ] 고급 기능 (Part 3)
   - [ ] Spark 프로젝트 (Part 4)
   - [ ] 생태계 및 도구 (Part 5)

8. **최종 검토 및 출판 준비**
   - [ ] 전체 기술 검토 (Technical Review)
   - [ ] 독자 테스트 (베타 리더 피드백)
   - [ ] 최종 편집 및 교정
