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

## Codespaces

코드스페이스는 컨테이너를 개발 환경으로 사용할 수 있게 해주는 새로운 기술입니다. 이 레포지토리에서는 GitHub 코드스페이스와 VS Code 코드스페이스 둘 다에서 지원되는 코드스페이스 컨테이너를 제공합니다.

### GitHub 기반 Codespaces

1. Azure SDK GitHub 레포지토리에서 "Code -> Open with Codespaces" 버튼을 클릭합니다.
1. 터미널을 엽니다.
1. 아래의 명령어를 실행하고 생성된 링크를 `Ctrl+Click` 합니다. 새 창이 열리면서 Azure SDK 웹사이트가 표시됩니다.

    ```bash
    bundle exec jekyll serve
    ```

### VS Code 기반 Codespaces

1. [VS Code Remote Extension Pack](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)을 설치합니다.
2. VS Code에서 Azure SDK 레포지토리를 열면 Dev Container에서 프로젝트를 열라는 프롬프트가 표시됩니다. 프롬프트가 표시되지 않으면 CTRL+P를 누르고 "Remote-Containers: Open Folder in Container..."를 선택합니다.
3. 터미널을 엽니다.
4. `Ctrl+Shift+T`를 누르거나 아래 명령어를 실행하고 생성된 링크를 `Ctrl+Click`합니다. 새 창이 열리면서 Azure SDK 웹사이트가 표시됩니다.

    ```bash
    bundle exec jekyll serve
    ```

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
