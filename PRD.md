# azure-sdk-korean PRD (Product Requirements Document)

## 문서 정보
- 문서명: Azure SDK Korean Documentation 프로젝트 PRD
- 버전: v1.0
- 작성일: 2026-03-27
- 기준 문서: `AGENTS.md`, `README.md`, `CONTRIBUTING.md`

---

## 프로젝트 비전 (Project Vision)

`azure-sdk-korean`은 Azure SDK 공식 가이드라인의 핵심 지식을 한국어로 신뢰 가능하게 제공하는 **커뮤니티 기반 번역·큐레이션 허브**를 지향한다. 

장기적으로는 한국어 개발자가 영문 원문 접근 장벽 때문에 설계 원칙(Design Guidelines), 구현 패턴(Implementation Patterns), 기여 방식(Contribution Process)을 놓치지 않도록, **최신성(Freshness)과 정확성(Accuracy)을 갖춘 표준 참고 레퍼런스**가 되는 것을 목표로 한다.

---

## 목표 및 미션 (Goals & Mission)

### 미션 (Mission)
한국어 Azure 개발자가 언어 장벽 없이 Azure SDK 생태계의 설계 철학과 실무 지침을 빠르게 습득하고, 실제 제품/라이브러리 개발에 적용하도록 돕는다.

### 2026~2027 핵심 목표 (Goals)
1. **핵심 언어권 문서 완성도 강화**: Python, .NET, Java 가이드 문서의 번역 완성도 및 유지보수 체계 확립
2. **업스트림 동기화 체계화**: Azure SDK 원문 변경사항을 주기적으로 반영하는 운영 루프 구축
3. **기여 경험 개선**: 신규 기여자가 1주 내 첫 PR을 제출할 수 있는 온보딩 경로 제공
4. **운영 안정성 확보**: GitHub Pages + Jekyll 기반 문서 배포 품질 및 보안 관리의 상시 운영

---

## 타겟 사용자 (Target Users)

### 1) 한국어 Azure 개발자 (Primary)
- Azure SDK를 실무에서 사용하는 애플리케이션/플랫폼 개발자
- 영문 원문 독해는 가능하지만, 빠른 이해를 위해 한국어 정리본이 필요한 사용자

### 2) SDK 기여자 및 기술 문서 작성자 (Secondary)
- Azure SDK 관련 오픈소스 기여를 준비하는 개발자
- 번역/검수/용어 표준화에 참여하는 커뮤니티 멤버

### 3) 기술 리드/아키텍트 (Tertiary)
- 팀 내 SDK 사용 가이드 표준을 수립하려는 리드 엔지니어
- 다수 언어 스택(Python, .NET, Java)을 비교/검토하는 의사결정자

---

## 핵심 가치 제안 (Value Proposition)

1. **신뢰 가능한 한국어 레퍼런스**: 원문 링크 기반의 추적 가능한 번역
2. **실무 중심 구조화**: 언어별(예: Python/.NET/Java)로 탐색 가능한 가이드 체계
3. **커뮤니티 검증 루프**: 이슈/PR 중심의 공개 검토 프로세스
4. **운영 안정성**: GitHub Actions + GitHub Pages 자동 배포로 변경 반영 속도 확보
5. **기여 친화성**: `CONTRIBUTING.md` 기반의 명확한 기여 경로와 행동 규범(Code of Conduct)

---

## 현재 상태 (Current State) - 2026년 3월 재활성화

프로젝트는 2023년 10월 이후 활동이 둔화되었으나, **2026년 3월 재활성화**를 통해 기반 운영을 복구했다.

### 완료 성과 (2026-03 기준)
- **Infrastructure 복구**: `.github/workflows/pages.yml` 기반 자동 배포 파이프라인 확보
- **운영 가시성**: README 빌드 배지 추가 및 라이브 사이트 상태 명확화
- **Security 개선**:
  - `github-pages` 228 → 231
  - `activesupport` 6.1.7.5 → 8.1.3
  - Dependabot 알림 19건 → 17건
  - 잔여 알림은 빌드 타임 의존성 중심이며 위험 수용 근거를 이슈 #196에 문서화
- **Issue Triage 진전**: 12건 이슈 정리/처리(닫힘 및 중복/완료 정리 포함), 운영 백로그 재정돈
- **콘텐츠 기반 확보**:
  - Python Guidelines (`design.md`, `implementation.md`) 대규모 번역 자산 확보
  - .NET Guidelines `introduction.md` 확보
  - Java Guidelines `introduction.md`, `spring.md` 확보

### 현재 과제
- 2021~2022년 stale issue 검토 및 상태 재분류
- Quick wins 용어 수정(#148, #147, #142) 반영
- 업스트림 변경으로 outdated 된 문서 섹션 동기화(#171, #170, #169)
- Governance 문서 보강(#189, #155)

> 운영/기술 컨텍스트의 상세 기준은 `AGENTS.md`를 우선 참조한다.

---

## 우선순위 및 로드맵 (Priorities & Roadmap)

### Phase 1: Infrastructure ✅ (완료)
**목표**: 문서 사이트 운영을 다시 가능한 상태로 안정화

완료 범위:
- GitHub Actions 기반 배포 자동화
- 빌드 상태 가시화(README badge)
- 주요 보안 의존성 업데이트 및 위험도 문서화
- 백로그 1차 정리(이슈 triage)

완료 기준(Definition of Done):
- main 반영 시 사이트 자동 배포
- 빌드 실패율 최소화
- 보안 이슈의 위험 수용 근거 명시

### Phase 2: Content Translation (Python, .NET, Java)
**목표**: 핵심 언어 문서의 품질·최신성·일관성 확보

우선순위:
1. Python 문서 검증/마감 (#158 연계)
2. .NET/Java 주요 섹션의 업스트림 동기화
3. 용어 표준화(`docs/general/terminology.md`)와 리뷰 프로세스 정비

핵심 실행 항목:
- 번역 미완/검증 필요 문서에 대한 상태표(Backlog Board) 운영
- 문서별 "원문 링크 + 마지막 동기화 일자" 메타데이터 표준화
- PR 리뷰 체크리스트에 번역 품질 항목 추가

완료 기준(Definition of Done):
- Python/.NET/Java 우선 섹션 100% 번역 또는 최신성 검증 완료
- 주요 용어 불일치 이슈 감소
- outdated 문서 이슈의 순차적 종료

### Phase 3: Community Building
**목표**: 지속 가능한 기여 생태계 구축

핵심 실행 항목:
- 신규 기여자 온보딩 문서 강화 (`CONTRIBUTING.md` 연동)
- "좋은 첫 이슈(good first issue)" 큐레이션
- 분기별 번역/검수 스프린트 운영

완료 기준(Definition of Done):
- 월간 활성 기여자 증가
- 신규 기여자 첫 기여 리드타임 단축
- 리뷰 대기 시간 안정화

---

## 성공 지표 (Success Metrics)

### 제품/콘텐츠 지표
1. **핵심 문서 최신성 유지율**: Python/.NET/Java 핵심 페이지의 최근 90일 내 동기화 비율 ≥ 85%
2. **우선 백로그 처리율**: stale issue + quick wins + outdated docs의 분기별 처리율 ≥ 70%
3. **번역 품질 결함률**: 리뷰 후 재수정이 필요한 중대 용어/의미 오류 비율 분기별 20% 이상 감소

### 커뮤니티/운영 지표
4. **활성 기여자 수**: 월 3명 이상(코드/문서 PR 기준)
5. **첫 기여 리드타임**: 신규 기여자의 첫 PR 생성까지 평균 7일 이내
6. **PR 리뷰 SLA**: 첫 리뷰 코멘트 평균 72시간 이내
7. **배포 안정성**: 문서 배포 워크플로우 성공률 월 98% 이상

### 품질/신뢰 지표
8. **링크/구조 유효성**: 정기 점검 시 치명적 링크 오류 0건 유지
9. **보안 검토 준수율**: Dependabot 알림 분기 검토 100% 수행

---

## 비기능 요구사항 (Non-functional Requirements)

### 1) 번역 품질 (Translation Quality)
- 의미 충실도(Faithfulness): 원문의 기술적 의미를 왜곡 없이 전달
- 용어 일관성(Consistency): `docs/general/terminology.md` 기준 준수
- 가독성(Readability): 한국어 개발자가 빠르게 이해 가능한 문장 구조 유지
- 검수 가능성(Reviewability): 원문 링크와 변경 근거를 PR에 명시

### 2) 최신성 유지 (Freshness & Maintainability)
- 업스트림 변경 탐지 후 우선순위 기반 반영
- 문서별 마지막 동기화 일자 관리
- 오래된 이슈(stale) 정기 점검 루틴 운영

### 3) 운영 안정성 (Operational Reliability)
- GitHub Pages 배포 자동화 유지
- 빌드 실패 시 원인 추적 가능한 로그 확보
- 문서 변경 시 최소 1회 빌드 검증

### 4) 협업/거버넌스 (Collaboration & Governance)
- `CONTRIBUTING.md`의 기여 절차와 Code of Conduct 준수
- 이슈 기반 작업 추적 및 리뷰 이력 공개
- 커밋/PR 규칙은 `AGENTS.md`의 운영 가이드와 정합성 유지

---

## 참조 문서 (References)
- 프로젝트 운영 컨텍스트: `AGENTS.md`
- 프로젝트 개요 및 링크 허브: `README.md`
- 기여 프로세스 및 협업 원칙: `CONTRIBUTING.md`
- 보안 정책: `SECURITY.md`

본 PRD는 상기 문서를 기준으로 2026년 3월 재활성화 시점의 목표/우선순위를 제품 관점으로 재구성한 문서다.
