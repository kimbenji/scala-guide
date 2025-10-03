# Scala 학습 가이드

> Java 백엔드 개발자를 위한 Scala 2.12 기반 실무 가이드북

[![GitHub Pages](https://img.shields.io/badge/docs-GitHub%20Pages-blue)](https://username.github.io/Write-Scala-Book/)
[![Scala](https://img.shields.io/badge/Scala-2.12-red)](https://www.scala-lang.org/)
[![Spark](https://img.shields.io/badge/Spark-3.5.x-orange)](https://spark.apache.org/)

## 📚 프로젝트 소개

이 프로젝트는 **경력 3-5년의 Java 백엔드 개발자**가 **2개월 내에 Scala를 실무에 적용**할 수 있도록 설계된 종합 학습 가이드입니다.

### 핵심 특징

- ✅ **Java 비교 학습**: 모든 개념을 Java와 비교하여 설명
- ✅ **실전 프로젝트**: Apache Spark 기반 3개 프로젝트 (로그 분석, 스트리밍, ML)
- ✅ **48개 실습 과제**: 각 챕터별 실습으로 학습 강화
- ✅ **Scala 3 대비**: 모든 챕터에 Scala 3 마이그레이션 가이드 포함
- ✅ **300-350페이지**: 체계적이고 완성도 높은 콘텐츠

### 학습 목표

1. Scala 2.12 기본 문법 및 함수형 프로그래밍 습득
2. Apache Spark를 활용한 빅데이터 처리 능력 개발
3. 실무 프로젝트에 즉시 적용 가능한 실전 역량 배양

---

## 📖 문서 읽는 방법

### 🌐 온라인으로 읽기 (권장)

**GitHub Pages로 웹사이트 형태로 읽기:**

1. **메인 페이지 접속**: [https://username.github.io/Write-Scala-Book/](https://username.github.io/Write-Scala-Book/)
2. **목차에서 원하는 Part 선택**
3. **챕터별로 순차 학습**

> 💡 **Tip**: 브라우저 북마크에 추가하여 언제든지 접근하세요!

### 📂 로컬에서 읽기

**방법 1: GitHub에서 직접 읽기**

```bash
# 저장소 클론
git clone https://github.com/username/Write-Scala-Book.git
cd Write-Scala-Book

# VS Code 또는 에디터로 열기
code docs/
```

**방법 2: Jekyll 로컬 서버 실행**

```bash
# Jekyll 설치 (Ruby 필요)
gem install bundler jekyll

# docs 디렉토리로 이동
cd docs

# 의존성 설치
bundle install

# 로컬 서버 실행
bundle exec jekyll serve

# 브라우저에서 http://localhost:4000 접속
```

### 📱 오프라인으로 읽기

**Markdown 뷰어 앱 사용:**

- **macOS**: Marked 2, MacDown, Typora
- **Windows**: Typora, MarkText
- **Linux**: ReText, Ghostwriter
- **멀티플랫폼**: Obsidian, Logseq

```bash
# docs 폴더를 Markdown 뷰어로 열기
open docs/README.md  # macOS
```

---

## 🗂️ 프로젝트 구조

```
Write-Scala-Book/
├── README.md                    # 📍 현재 파일 (프로젝트 소개)
│
├── docs/                        # 📚 학습 가이드 문서
│   ├── README.md               # 메인 목차 (시작점)
│   ├── NAVIGATION.md           # 챕터별 네비게이션 가이드
│   ├── _config.yml             # Jekyll 설정 (GitHub Pages용)
│   │
│   ├── prd.md                  # 프로젝트 요구사항 정의
│   ├── architecture.md         # 프로젝트 아키텍처
│   ├── progress.md             # 작성 진행 상황
│   │
│   ├── part1-basics/           # Part 1: Scala 기초 (6 챕터)
│   │   ├── README.md
│   │   ├── chapter01-getting-started.md
│   │   ├── chapter02-variables-types.md
│   │   ├── chapter03-control-structures.md
│   │   ├── chapter04-functions.md
│   │   ├── chapter05-oop.md
│   │   └── chapter06-collections.md
│   │
│   ├── part2-functional/       # Part 2: 함수형 프로그래밍 (4 챕터)
│   │   ├── README.md
│   │   ├── chapter07-advanced-functional.md
│   │   ├── chapter08-option-try-either.md
│   │   ├── chapter09-implicits.md
│   │   └── chapter10-advanced-types.md
│   │
│   ├── part3-advanced/         # Part 3: 고급 주제 (4 챕터)
│   │   ├── README.md
│   │   ├── chapter11-advanced-type-system.md
│   │   ├── chapter12-concurrency-future.md
│   │   ├── chapter13-macros-reflection.md
│   │   └── chapter14-dsl-design.md
│   │
│   ├── part4-spark/            # Part 4: Apache Spark (1 챕터 + 3 프로젝트)
│   │   ├── README.md
│   │   └── chapter15-apache-spark.md
│   │
│   └── part5-ecosystem/        # Part 5: 생태계와 도구 (1 챕터)
│       ├── README.md
│       └── chapter16-ecosystem-tools.md
│
└── code/                        # 💻 실습 코드 (향후 추가 예정)
    └── part1-basics/
```

---

## 🚀 학습 가이드

### 📋 학습 순서

**전체 학습 기간: 7-9주** (주당 10-15시간 투자 기준)

| Week | Part | 내용 | 시간 |
|------|------|------|------|
| 1-2주 | [Part 1](docs/part1-basics/) | Scala 기초 | 20-30시간 |
| 3-4주 | [Part 2](docs/part2-functional/) | 함수형 프로그래밍 | 20-30시간 |
| 5주 | [Part 3](docs/part3-advanced/) | 고급 주제 (선택) | 10-15시간 |
| 6-8주 | [Part 4](docs/part4-spark/) | Apache Spark 프로젝트 | 30-40시간 |
| 9주 | [Part 5](docs/part5-ecosystem/) | 생태계와 도구 | 10-15시간 |

### 📍 시작하는 방법

**Step 1: 문서 메인 페이지 열기**
```bash
# 온라인
https://username.github.io/Write-Scala-Book/

# 또는 로컬
open docs/README.md
```

**Step 2: Part 1부터 순차적으로 학습**
- [Part 1: Scala 기초](docs/part1-basics/README.md)부터 시작
- 각 챕터의 실습 과제 반드시 완료
- 이해 안 되는 부분은 이전 챕터 복습

**Step 3: 실전 프로젝트 실습**
- Part 4의 Spark 프로젝트에 충분한 시간 투자
- 3개 프로젝트 모두 직접 구현

**Step 4: 실무 적용**
- Part 5에서 실무 도구 습득
- 실제 프로젝트에 Scala 적용

### 🎯 학습 팁

1. **순차 학습**: Part 1부터 순서대로 (Part 3은 선택적)
2. **실습 중심**: 코드를 직접 작성하며 학습
3. **Java 비교**: Java와 비교하며 차이점 이해
4. **커뮤니티**: 막히는 부분은 Scala 커뮤니티에 질문

---

## 🛠️ 개발 환경 설정

### 사전 요구사항

- **Java**: JDK 11 이상 (권장: Java 11 LTS)
- **SBT**: 1.9.x 이상
- **IDE**: IntelliJ IDEA + Scala Plugin (권장)
- **Git**: 버전 관리

### 설치 가이드

**macOS:**
```bash
# Homebrew로 설치
brew install openjdk@11
brew install sbt
```

**Linux:**
```bash
# Ubuntu/Debian
sudo apt-get install openjdk-11-jdk
echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | sudo tee /etc/apt/sources.list.d/sbt.list
sudo apt-get update
sudo apt-get install sbt
```

**Windows:**
- [AdoptOpenJDK](https://adoptopenjdk.net/) 다운로드
- [SBT](https://www.scala-sbt.org/download.html) 다운로드

**IDE 설정:**
1. [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) 설치
2. Scala Plugin 설치 (Preferences > Plugins > Scala)
3. Import SBT Project

---

## 📊 학습 콘텐츠

### Part별 개요

#### [Part 1: Scala 기초](docs/part1-basics/) (6 챕터)
Scala 개발 환경, 기본 문법, 타입 시스템, 제어 구조, 함수, OOP, 컬렉션

#### [Part 2: 함수형 프로그래밍](docs/part2-functional/) (4 챕터)
순수 함수, 함수 합성, Option/Try/Either, Implicit, 고급 타입 시스템

#### [Part 3: 고급 주제](docs/part3-advanced/) (4 챕터)
Higher-Kinded Types, Future/Promise, 매크로, 리플렉션, DSL 설계

#### [Part 4: Apache Spark](docs/part4-spark/) (3 프로젝트)
- 프로젝트 1: 로그 분석 시스템
- 프로젝트 2: 실시간 스트리밍 처리
- 프로젝트 3: 머신러닝 파이프라인

#### [Part 5: 생태계와 도구](docs/part5-ecosystem/) (1 챕터)
SBT, ScalaTest, Scala-Java 상호운용, 주요 라이브러리, 프로덕션 배포

---

## 🎓 학습 성과

이 가이드를 완료하면 다음을 할 수 있습니다:

✅ Scala로 타입 안전한 백엔드 애플리케이션 개발
✅ 함수형 프로그래밍 패러다임 실무 적용
✅ Apache Spark로 대규모 데이터 처리
✅ Scala-Java 혼합 프로젝트 개발
✅ SBT, ScalaTest 등 실무 도구 활용

---

## 📚 추천 학습 자료

### 공식 문서
- [Scala 공식 문서](https://docs.scala-lang.org)
- [Apache Spark 문서](https://spark.apache.org/docs/latest/)
- [SBT 문서](https://www.scala-sbt.org/documentation.html)

### 온라인 강의
- [Coursera: Functional Programming in Scala](https://www.coursera.org/specializations/scala)
- [Scala Exercises](https://www.scala-exercises.org)
- [Twitter Scala School](https://twitter.github.io/scala_school/)

### 커뮤니티
- [Scala Korea](https://www.facebook.com/groups/ScalaKorea/)
- [Scala Users Forum](https://users.scala-lang.org/)
- [Stack Overflow - Scala](https://stackoverflow.com/questions/tagged/scala)

---

## 🤝 기여 및 피드백

### 오타 및 개선사항 제보

이슈를 등록하거나 Pull Request를 보내주세요:

```bash
# 1. Fork this repository
# 2. Create your feature branch
git checkout -b feature/improvement

# 3. Commit your changes
git commit -m "Add some improvement"

# 4. Push to the branch
git push origin feature/improvement

# 5. Create Pull Request
```

### 피드백

- **이슈 등록**: [GitHub Issues](https://github.com/username/Write-Scala-Book/issues)
- **이메일**: your-email@example.com

---

## 📝 라이선스

이 문서는 교육 목적으로 작성되었습니다.

---

## 👥 크레딧

**작성자**: BMad Master Agent
**버전**: v1.0
**최종 업데이트**: 2025-10-03

**기술 스택**:
- Scala 2.12.18
- Apache Spark 3.5.x
- SBT 1.9.x
- ScalaTest 3.2.x

---

## 📞 문의

프로젝트에 대한 질문이나 제안사항이 있으시면 언제든지 연락주세요!

- **GitHub Issues**: [이슈 등록](https://github.com/username/Write-Scala-Book/issues)
- **Discussions**: [토론 참여](https://github.com/username/Write-Scala-Book/discussions)

---

**🎉 Happy Scala Learning!** 🚀
