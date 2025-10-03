# Part 5: 생태계와 도구

> 실무 개발 환경 구축 및 생산성 도구 활용

## 📚 학습 목표

- SBT 빌드 시스템 심화 학습
- ScalaTest를 활용한 효과적인 테스팅
- Scala-Java 상호운용성 이해
- 주요 라이브러리 및 프레임워크 활용
- 프로덕션 배포 베스트 프랙티스

## 📖 챕터 목록

### [Chapter 16: Scala 생태계와 도구](chapter16-ecosystem-tools.md)

#### SBT 빌드 시스템
- build.sbt 구조 및 설정
- 멀티 프로젝트 구성
- SBT 플러그인 (scalafmt, scoverage, assembly)
- 주요 명령어 및 워크플로우

#### ScalaTest 테스팅
- 테스트 스타일 (FlatSpec, BDD)
- Matchers (should, contain, have)
- 비동기 테스팅 (ScalaFutures)
- 속성 기반 테스팅 (ScalaCheck)

#### Scala-Java 상호운용
- Java 컬렉션 ↔ Scala 컬렉션 변환
- @BeanProperty (Java 친화적 클래스)
- 주의사항 (기본 파라미터, Option vs null)
- 실전 예제

#### 주요 라이브러리
- **Cats**: 함수형 프로그래밍 라이브러리
- **Akka**: 액터 모델 기반 동시성
- **Play Framework**: 웹 애플리케이션 프레임워크
- **Slick**: 데이터베이스 액세스 라이브러리

#### 코드 품질 도구
- Scalafmt (코드 포매팅)
- Scoverage (코드 커버리지)
- WartRemover (정적 분석)

#### 프로덕션 배포
- Fat JAR 생성 (sbt-assembly)
- Docker 이미지 빌드 (sbt-native-packager)
- 배포 전략 및 모니터링

## ⏱️ 예상 학습 시간

**총 1-2주** (주당 10-15시간 투자 기준)

- SBT & 테스팅: 1주
- 라이브러리 & 배포: 1주 (선택)

## 🎯 학습 성과

Part 5를 완료하면 다음을 할 수 있습니다:

✅ SBT로 멀티 모듈 프로젝트 구성
✅ ScalaTest로 효과적인 테스트 작성
✅ Scala-Java 혼합 프로젝트 개발
✅ 주요 Scala 라이브러리 활용
✅ 프로덕션 환경 배포

## 🔧 실습 환경

**필수 도구:**
- SBT 1.9.x 이상
- ScalaTest 3.2.x
- IntelliJ IDEA (또는 VS Code + Metals)

**선택 도구:**
- Docker (배포용)
- Git (버전 관리)

---

**[← Part 4: Apache Spark](../part4-spark/README.md)** | **[메인 목차](../README.md)**

---

## 🎉 축하합니다!

모든 Part를 완료하셨다면, 이제 Scala를 실무에 적용할 준비가 되었습니다!

**다음 단계:**
1. 실전 프로젝트에 Scala 적용
2. Scala 커뮤니티 참여 (Scala Days, 로컬 밋업)
3. 오픈소스 프로젝트 기여
4. Scala 3 학습 및 마이그레이션

**추천 자료:**
- [Scala 공식 문서](https://docs.scala-lang.org)
- [Scala Exercises](https://www.scala-exercises.org)
- [Coursera: Functional Programming in Scala](https://www.coursera.org/specializations/scala)
- [Twitter Scala School](https://twitter.github.io/scala_school/)
