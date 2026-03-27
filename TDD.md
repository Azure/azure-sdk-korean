# azure-sdk-korean Technical Design Document (TDD)

## 문서 메타데이터
- **프로젝트**: `azure-sdk-korean`
- **문서 경로**: `/root/Github/azure-sdk-korean/TDD.md`
- **기준 파일**:
  - `/root/Github/azure-sdk-korean/AGENTS.md`
  - `/root/Github/azure-sdk-korean/.github/workflows/pages.yml`
  - `/root/Github/azure-sdk-korean/Gemfile`
  - `/root/Github/azure-sdk-korean/Gemfile.lock`
  - `/root/Github/azure-sdk-korean/_config.yml`
- **라이브 사이트**: `https://azure.github.io/azure-sdk-korean/`
- **저장소**: `https://github.com/Azure/azure-sdk-korean`

---

## 1) 기술 스택 개요 (Tech Stack Overview)

### 런타임/빌드 도구
- **Ruby**: `3.2.3` (AGENTS.md 기준)
- **Bundler**: `2.3.22` (AGENTS.md 기준)
- **Jekyll**: `3.9.3` (AGENTS.md 기준, GitHub Pages 호환 스택)
- **GitHub Pages gem**: `github-pages ~> 231` (`Gemfile`)

> 참고: `Gemfile.lock` 해석 시 `github-pages (231)`이 내부적으로 `jekyll (= 3.9.5)`를 고정하고 있습니다. 운영/문서 관점에서는 AGENTS 기준 버전(`3.9.3`)을 표준 표기로 유지하되, 실제 lock 해상 버전은 `3.9.5`입니다.

### 플랫폼
- **정적 사이트 생성기**: Jekyll
- **호스팅**: GitHub Pages
- **자동화**: GitHub Actions (`.github/workflows/pages.yml`)

### 핵심 설정 스니펫
```ruby
# /root/Github/azure-sdk-korean/Gemfile
gem 'github-pages', '~> 231', group: :jekyll_plugins
gem 'jekyll-relative-links'
gem 'activesupport', '>4.1.11'
gem "webrick", "~> 1.8"
```

```yaml
# /root/Github/azure-sdk-korean/_config.yml
baseurl: /azure-sdk-korean
destination: ./_site/azure-sdk-korean
plugins:
  - jekyll-github-metadata
  - jekyll-sitemap
  - jekyll-gist
  - jekyll-seo-tag
  - jekyll-relative-links
  - jemoji
```

---

## 2) 빌드 시스템 (Build System)

### Jekyll Build Process
빌드는 Jekyll의 표준 파이프라인을 따릅니다.

1. `bundle install`로 Gem dependency 설치/동기화
2. `bundle exec jekyll build` 실행
3. 출력물 생성 경로: `_site/azure-sdk-korean` (`_config.yml`의 `destination`)

### GitHub Actions Workflow
실제 배포 워크플로우: `/root/Github/azure-sdk-korean/.github/workflows/pages.yml`

주요 동작:
- `on.push.branches: [main]`
- `on.pull_request`에서 Build 검증 수행
- Ruby setup: `ruby-version: '3.2'`
- Build command:
  ```bash
  bundle exec jekyll build --baseurl "${{ steps.pages.outputs.base_path }}"
  ```
- `_site` artifact 업로드 후, `main` 브랜치일 때만 Pages deploy

### 성능 특성
- **평균 빌드 시간**: 약 `~3-4초`
  - AGENTS.md의 운영 메모: `~3초`
  - CI 환경 변동 포함 실무 표기: `~3-4초`

---

## 3) 의존성 관리 (Dependency Management)

### 관리 파일
- 선언 파일: `/root/Github/azure-sdk-korean/Gemfile`
- 잠금 파일: `/root/Github/azure-sdk-korean/Gemfile.lock`

### github-pages gem 전략
이 프로젝트는 Jekyll 및 플러그인 버전을 개별 pinning하기보다 **`github-pages` meta gem**으로 GitHub Pages 호환 세트를 일괄 관리합니다.

장점:
- GitHub Pages와의 버전 호환성 확보
- 플러그인 조합 충돌 최소화
- 업데이트 시 검증 범위 축소

### 보안/업데이트 정책
- Dependabot alert 기반 모니터링
- 2026-03 기준 상태(AGENTS.md):
  - `github-pages`: `228 -> 231`
  - `activesupport`: `6.1.7.5 -> 8.1.3`
  - Alert 수: `19 -> 17`
- 남은 17개는 build-time dependency로 분류되어 runtime 노출 없음 (Issue #196)

---

## 4) 디렉토리 구조 (Directory Structure)

### 루트 구성 (핵심)
```text
/root/Github/azure-sdk-korean/
├── .github/workflows/pages.yml
├── docs/
├── _includes/
├── _layouts/
├── _config.yml
├── Gemfile
├── Gemfile.lock
├── AGENTS.md
└── README.md
```

### 문서 콘텐츠 구조
`/root/Github/azure-sdk-korean/docs/` 하위는 언어/주제 단위로 분리됩니다.

- 주요 언어 축:
  - `python/`
  - `dotnet/`
  - `java/`
- 기타:
  - `android/`, `ios/`, `general/`, `policies/`, `contribution/`, `tables/`, `tracing/`, `redirects/`

### Jekyll 템플릿 계층
- `_layouts/`: 페이지 레이아웃
- `_includes/`: 공통 partial
- `_config.yml`: 사이트 전역 설정

---

## 5) 번역 워크플로우 (Translation Workflow)

### 표준 흐름
`Issue 생성 → Branch 작업 → 번역/수정 → Pull Request → Review → Merge`

AGENTS.md 필수 규칙 반영:
1. 작업 전 Issue 우선 생성
2. Scope creep 방지(최소 변경 원칙)
3. 변경 후 빌드 검증

### Branch 정책
- `main`: 배포 브랜치
- `feature/*`: 선택적 작업 브랜치
- 소규모 변경은 직접 `main` 반영 가능하나, 리뷰 품질을 위해 PR 권장

### Conventional Commits
형식:
```text
<type>: <subject>
```

주요 타입:
- `docs:` 문서 번역/수정
- `feat:` 기능 추가
- `fix:` 버그 수정
- `chore:` 의존성/빌드 작업

예시:
```text
docs: translate Python Guidelines Implementation
fix: correct hyperlink in header
chore: update github-pages to v231
```

---

## 6) CI/CD 파이프라인 (CI/CD Pipeline)

### Trigger & Stages
1. **Push to main**: Build + Deploy
2. **Pull Request**: Build only (검증)

### Stage 상세
- **build job**
  - Checkout
  - Ruby 3.2 setup + bundler cache
  - Configure Pages
  - Jekyll production build
  - `_site` artifact 업로드
- **deploy job**
  - 조건: `github.ref == 'refs/heads/main'`
  - GitHub Pages에 artifact 배포

### 운영 결과
- main 반영 후 자동 배포
- 사이트 URL: `https://azure.github.io/azure-sdk-korean/`

---

## 7) 보안 정책 (Security Policy)

### Dependabot alerts 처리 원칙
- Alert 발생 시 severity 분류
- **Critical/High**는 즉시 검토
- Build-time dependency 중심 이슈는 runtime 영향 분석 후 문서화된 위험 수용 가능

### 현재 리스크 상태 (AGENTS.md 기준, 2026-03-27)
- 총 17개 (`1 critical`, `2 high`, `11 moderate`, `3 low`)
- 모두 build-time dependency로 분류
- 리스크 수용 근거: **Issue #196**

### 운영 일정
- **분기별 보안 리뷰**: 다음 주기 2026년 6월

### 참조 문서
- `SECURITY.md` (Microsoft 표준 정책)
- GitHub Issue `#196` (위험 수용/분류 근거)

---

## 8) 로컬 개발 환경 (Local Development)

### 사전 요구사항
- Ruby `3.2.x` 권장 (AGENTS 기준 `3.2.3`)
- Bundler 설치

### 초기 설정
```bash
cd /root/Github/azure-sdk-korean
bundle install
```

### 자주 쓰는 명령어
```bash
# 정적 빌드
bundle exec jekyll build

# 로컬 서버 실행
bundle exec jekyll serve

# (선택) baseurl 포함 확인이 필요한 경우
bundle exec jekyll build --baseurl /azure-sdk-korean
```

기본 접속 주소:
- `http://localhost:4000`

---

## 부록 A) 실제 파일/설정 참조 매핑

- Workflow: `/root/Github/azure-sdk-korean/.github/workflows/pages.yml`
- Runtime/Plugin 설정: `/root/Github/azure-sdk-korean/_config.yml`
- Gem 선언: `/root/Github/azure-sdk-korean/Gemfile`
- Gem lock: `/root/Github/azure-sdk-korean/Gemfile.lock`
- 프로젝트 운영 기준/정책: `/root/Github/azure-sdk-korean/AGENTS.md`

---

## 부록 B) AGENTS.md 변경 이력 항목과 실제 커밋 해시 연결

AGENTS.md의 `최근 변경 이력 (2026-03-27)`에 대응하는 저장소 커밋(로컬 `git log --oneline` 기준):

- `ff3ba19` — `docs: fix Korean spacing in terminology.md`
- `abcb067` — `docs: add AGENTS.md for AI agent context`
- `e844984` — `fix: add hyperlink to Azure SDK repository in header`
- `286ac79` — `chore: update github-pages to v231`
- `b264ebf` — `chore: update gems to fix security vulnerabilities`
- `65e3f5e` — `feat: add GitHub Actions workflow and build badges`

> 해시는 시간 경과에 따라 추가 커밋과 함께 히스토리에서 상대 위치가 변동될 수 있으므로, 최신 상태는 `git log --oneline`으로 재확인합니다.
