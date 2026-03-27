# Azure SDK 기여 가이드

본 저장소는 [Microsoft Open Source Code of Conduct(Microsoft 오픈 소스 규정)](https://opensource.microsoft.com/codeofconduct/)를 준수합니다.
자세한 내용은 [준수 사항 FAQ(영문)](https://opensource.microsoft.com/codeofconduct/faq/)를 참조하세요.
또는 opencode@microsoft.com에 궁금한 사항을 문의하거나 의견을 제시하실 수 있습니다.

Azure SDK 프로젝트는 기여와 제안을 환영합니다. 대부분의 기여는 당신이 기여한 내용에 대한 사용 권리를 가지고 있으며, 그 권리를 우리에게 부여할 수 있다는 것을 선언하는 기여자 라이선스 계약(CLA)에 동의해야 합니다. 자세한 내용은 https://cla.microsoft.com를 방문하세요.

풀 리퀘스트를 제출하면, CLA-bot이 자동으로 CLA를 제공해야 하는지 여부를 판단하고 풀 리퀘스트를 적절하게 꾸며줍니다(예: 레이블, 코멘트). 봇이 제공하는 지시사항을 따르기만 하면 됩니다. CLA를 사용하는 우리의 모든 레포지토리에 대해 이 작업을 한 번만 수행하면 됩니다.

## Azure SDK 블로그 기여

[Azure SDK Blog](https://aka.ms/azsdk/blog)는 새로운 Azure SDK와 관련된 기여를 환영합니다. 블로그 포스트에 기여에 관심이 있다면 [azsdkblog@microsoft.com](mailto:azsdkblog@microsoft.com)으로 연락해 주세요.

## Azure SDK GitHub 페이지 사이트 기여

다음은 일반적인 기여 과정입니다:

1. 이 레포지토리를 포크합니다.
1. 새로운 브랜치를 생성합니다.
1. 해당 브랜치에 변경 사항을 커밋합니다.
1. 포크/브랜치에서 azure-sdk/main으로 PR을 생성합니다.

### 번역 작업 워크플로우 (Translation Workflow)

이 저장소의 번역/문서 작업은 반드시 아래 순서를 따릅니다.

- **이슈 우선 생성 (MANDATORY)**: 작업 시작 전에 GitHub Issue를 먼저 등록합니다.
- **기본 단계**: `Issue → Branch → Translate → PR → Review → Merge`

#### 권장 진행 절차

1. **Issue 생성**: 번역 대상 문서, 범위, 완료 기준(Definition of Done)을 명확히 작성합니다.
2. **Branch 생성**: 이슈 단위로 브랜치를 분리합니다. (예: `docs/issue-147-terminology-fix`)
3. **Translate**: 원문 최신 상태를 확인한 뒤 번역합니다.
4. **PR 생성**: 변경 이유(왜), 변경 범위(무엇), 검증 결과(어떻게)를 포함합니다.
5. **Review 반영**: 리뷰 코멘트를 반영하고 대화(Conversation)를 모두 해결(Resolve)합니다.
6. **Merge**: 승인 및 빌드 검증 완료 후 병합합니다.

#### 용어 일관성

- 전문 용어는 `docs/general/terminology.md`를 기준으로 통일합니다.
- 기존 번역 스타일(문장 톤, 표기법, 링크 스타일)을 유지합니다.
- 동일 문서 내 동일 개념은 같은 번역어를 사용합니다.

#### 번역 품질 체크리스트

- [ ] 원문 의미를 누락/과장 없이 전달했는가?
- [ ] 고유 명사, 제품명, API 명칭을 정확히 유지했는가?
- [ ] 용어집(`docs/general/terminology.md`)과 충돌이 없는가?
- [ ] Markdown 링크/코드 블록/표 형식이 깨지지 않았는가?
- [ ] 오탈자, 띄어쓰기, 문장 호흡을 점검했는가?
- [ ] `bundle exec jekyll build`로 로컬 빌드를 확인했는가?

### 커밋 메시지 컨벤션 (Commit Message Convention)

커밋 메시지는 **Conventional Commits** 형식을 사용합니다.

```text
<type>: <subject>

[optional body]

[optional footer]
```

#### Type

- `docs:` 문서 번역/수정
- `feat:` 기능 추가
- `fix:` 버그 수정
- `chore:` 빌드/의존성/운영 작업
- `style:` 포맷/스타일 변경 (동작 변화 없음)
- `refactor:` 동작 변화 없는 구조 개선

#### Footer 규칙

- `Refs #N`: 이슈를 참조하지만 자동 종료하지 않음
- `Closes #N`: 병합 시 이슈 자동 종료

#### 실제 예시 (AGENTS.md 기준)

- `docs: add AGENTS.md for AI agent context` (`abcb067`)  
  `Refs #189`
- `docs: fix Korean spacing in terminology.md` (`ff3ba19`)  
  `Refs #147`
- `fix: add hyperlink to Azure SDK repository in header` (`e844984`)  
  `Closes #149`
- `chore: update github-pages to v231` (`286ac79`)  
  `Refs #196`
- `chore: update gems to fix security vulnerabilities` (`b264ebf`)  
  `Refs #196`

### 초보자 가이드 (Beginner's Guide)

처음 기여하는 분은 범위를 작게 시작하면 빠르게 익숙해질 수 있습니다.

#### 무엇부터 시작하면 좋나요?

1. **Good First Issues 찾기**: Issues 탭에서 `good first issue`, `documentation`, `translation` 라벨을 우선 확인합니다.
2. **작은 단위 작업 선택**: 문단 1~2개 번역, 링크 수정, 용어 정리처럼 검증이 쉬운 작업부터 시작합니다.
3. **이슈 먼저 등록/할당**: 중복 작업을 방지하고 리뷰어가 맥락을 이해하기 쉽도록 합니다.

#### 초보자 Quick Wins 예시

- `docs/general/terminology.md` 용어 통일
- 문서 내 오탈자/띄어쓰기 수정
- 깨진 링크 및 앵커(anchor) 정리
- 문장 가독성 개선(의미 유지 범위 내)

#### 커뮤니티 도움 받기

- 작업 전 이슈에서 접근 방법을 간단히 공유합니다.
- PR 본문에 애매했던 표현/선택 근거를 남겨 리뷰를 요청합니다.
- 리뷰 피드백은 학습 과정으로 받아들이고, 반영 후 변경 요약을 댓글로 남깁니다.

### 코드 리뷰 프로세스 (Code Review Process)

#### 번역 리뷰 기준

- 정확성(Accuracy): 원문 의미가 정확히 보존되었는가?
- 일관성(Consistency): 용어/스타일/톤이 기존 문서와 일치하는가?
- 가독성(Readability): 자연스러운 한국어 문장으로 읽히는가?
- 형식 안정성(Formatting): Markdown 구조, 링크, 코드 블록이 유효한가?

#### 필수 검증

- PR 전 또는 리뷰 반영 후 `bundle exec jekyll build` 성공이 필수입니다.
- 빌드 실패 상태에서는 승인/병합을 진행하지 않습니다.

#### 승인 및 병합

1. 리뷰 코멘트 해결 및 CI 통과 확인
2. 최소 1명 이상의 승인(Approve) 확보 권장
3. 필요 시 최종 셀프 체크(체크리스트 재확인) 후 병합

## Codespaces

코드스페이스는 컨테이너를 개발 환경으로 사용할 수 있게 해주는 새로운 기술입니다. 이 레포지토리에서는 GitHub 코드스페이스와 VS Code 코드스페이스 둘 다에서 지원되는 코드스페이스 컨테이너를 제공합니다.

### GitHub 기반 Codespaces

1. Azure SDK GitHub 레포지토리에서 "Code -> Open with Codespaces" 버튼을 클릭합니다.
1. 터미널을 엽니다.
1. 아래의 명령어를 실행하고 생성된 링크를 `Ctrl+Click` 합니다. 새 창이 열리면서 Azure SDK 웹사이트가 표시됩니다.

    ```bash
    bundle install
    bundle exec jekyll serve
    ```
Note: If you encounter a `Not Found` error while accessing the website, try adding `/azure-sdk-korean/` to the end of the URL.

### VS Code 기반 Codespaces

1. [VS Code Remote Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)을 설치합니다.
2. VS Code에서 Azure SDK 레포지토리를 열면 Dev Container에서 프로젝트를 열라는 프롬프트가 표시됩니다. 프롬프트가 표시되지 않으면 CTRL+P를 누르고 "Remote-Containers: Open Folder in Container..."를 선택합니다.
3. 터미널을 엽니다.
4. `Ctrl+Shift+T`를 누르거나 아래 명령어를 실행하고 생성된 링크를 `Ctrl+Click`합니다. 새 창이 열리면서 Azure SDK 웹사이트가 표시됩니다.

    ```bash
    bundle install
    bundle exec jekyll serve
    ```
참고: 웹사이트에 접근하면서 Not Found 에러를 만나게 된다면, URL 끝에 /azure-sdk-korean/를 추가해보세요.

## 완전한 로컬 설정

이 사이트는 Jekyll과 GitHub 페이지를 사용합니다. 설치 지침은 여기에서 찾을 수 있습니다: https://jekyllrb.com/docs/installation

GitHub 지침은 https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll 에서 찾을 수 있습니다.

로컬에서 사이트를 구성하는 방법은 다음과 같습니다:

1. [Ruby+DevKit 2.6 설치](https://rubyinstaller.org/downloads/) - 2.7 버전은 이 사이트와 호환되지 않으므로 설치하지 마세요.

    완전한 설치 지침은 여기에서 확인할 수 있습니다: https://jekyllrb.com/docs/installation

1. 컴퓨터를 재시작합니다

    Ruby를 설치한 후에는 컴퓨터를 재시작해야 할 수 있습니다.

1. Jekyll 설치합니다

    ```bash
    gem install jekyll bundler
    ```

1. 의존성 설치

    azure-sdk 프로젝트의 루트에서 아래 명령을 실행합니다.

    ```bash
    bundle install
    ```

1. 터미널을 열고 아래의 명령어를 실행하여 사이트를 시작하세요:

    ```bash
    bundle exec jekyll serve --incremental
    ```

1. http://127.0.0.1:4000/azure-sdk-korean 주소로 브라우저를 열어 사이트를 실행하세요.
