---
title: "정책: 오픈 소스"
permalink: policies_opensource.html
folder: policies
sidebar: general_sidebar
---

{% include requirement/MUST %} 모든 라이브러리 코드가 GitHub에서 오픈 소스로 공개되어 있는지 확인합니다. 라이브러리 코드는 언어별 해당하는 Azure SDK '단일 저장소'에 위치해야 합니다.

* [Azure SDK for .NET](https://github.com/Azure/azure-sdk-for-net)
* [Azure SDK for Java](https://github.com/Azure/azure-sdk-for-java)
* [Azure SDK for JavaScript](https://github.com/Azure/azure-sdk-for-js)
* [Azure SDK for Python](https://github.com/Azure/azure-sdk-for-python)

{% include requirement/SHOULD %} GitHub에서 오픈되어 있는 상태에서 개발을 진행합니다. 디자인 선택에 대한 부분을 커뮤니티에서 피드백을 찾아보며, 커뮤니티에서는 대화에 적극적으로 참여합니다.

{% include requirement/MUST %} GitHub에서 적극적으로 임해주세요. 여러분의 클라이언트 라이브러리는 개발자 커뮤니티에서 주된 접점이 되므로, 활동을 계속 이어가는 것이 중요합니다. 이슈와 풀 리퀘스트는 제출이 이루어지고 나서 1주일 이내에 신뢰할 수 있는 코멘트가 있어야 합니다.

{% include requirement/MUST %} 건강한 오픈 소스 커뮤니티를 지향하기 위한 보다 자세한 정보에 대해서는 Microsoft Open Source Guidelines에 있는 [커뮤니티 섹션](https://docs.opensource.microsoft.com/releasing/foster-your-community.html)을 살펴 봅니다.

{% include requirement/MUST %} [Microsoft CLA](https://cla.opensource.microsoft.com/)를 사용합니다. Microsoft에서는 [cla-assistant](https://cla-assistant.io/)에 대해 상당한 기여를 하고 있습니다. 이는 모든 컨트리뷰터들이 CLA를 서명하였는지 확인하는 가장 쉬운 방법입니다.

{% include requirement/MUST %} 모든 소스 파일 (샘플 포함) 상단에 저작권(copyright) 헤더를 포함합니다. 다양한 언어에 대한 헤더 예제로 [Microsoft Open Source Guidelines](https://docs.opensource.microsoft.com/releasing/copyright-headers.html)을 살펴 봅니다.

예상되는 저작권 헤더는 다음과 같습니다:

```fundamental
Copyright (c) Microsoft Corporation.
Licensed under the MIT license.
```

{% include important.html content="현재는 Microsoft Open Source Guideline과 해당 조언에 있어 불일치가 있습니다. 해당 조언은 2019년 8월을 기준으로 합니다. 적절한 라이선스에 있어 불분명한 점이 있다면 CELA와 상의합니다." %}

## CONTRIBUTING.md

{% include requirement/MUST %} `CONTRIBUTING.md` 파일을 GitHub 저장소에 포함하여, 컨트리뷰터들이 해당 프로젝트에 컨트리뷰션을 하기 위한 과정을 설명합니다. `CONTRIBUTING.md` 예제는 [Microsoft Open Source Guidelines](https://docs.opensource.microsoft.com/releasing/overview.html)에 따라 제공됩니다:

```
# 컨트리뷰션

이 프로젝트는 컨트리뷰션과 제안을 환영합니다. 컨트리뷰션 대부분은 여러분이 컨트리뷰션을 할 권리가 있고, 실제로 하고 있으며 여러분의 컨트리뷰션을 사용한다는 권리를 우리에게 보장하는 컨트리뷰터 라이언스 약관 (CLA) 동의를 필요로 합니다. 자세한 정보는 https://cla.microsoft.com 에서 살펴 봅니다.

풀 리퀘스트를 제출하면 CLA 봇이 자동으로 여러분이 CLA를 제공하였는지 여부를 결정하모, PR에 대해 적절하게 관리되고 있는지 (예: 라벨, 코멘트) 여부를 결정합니다. 단순히, 봇이 제공하는 안내에 따르면 됩니다. 제공되는 CLA 작업은 모든 저장소에 걸쳐 한 번만 수행하면 됩니다.

이 프로젝트는 [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/)를 채택하였습니다. 보다 자세한 정보는 [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/)를 살펴 보거나 추가적인 질문 또는 코멘트가 있을 경우에는 [opencode@microsoft.com](mailto:opencode@microsoft.com)로 문의해 주세요.
```

## LICENSE

{% include requirement/MUST %} 라이선스 문구(디폴트로 [표준 MIT 라이선스](https://docs.opensource.microsoft.com/releasing/overview.html#license-files)여야 합니다)를 포함하는 `LICENSE` 파일을 포함합니다.

## CODEOWNERS

`CODEOWNERS`는 검토할 풀 리퀘스트에 대해 누가 자동으로 할당이 될지를 지정하는 GitHub 표준입니다. 이것은 풀 리퀘스트가 검토 없이 시들해지는 것을 방지하는 데 도움을 줍니다. GitHub는 풀 리퀘스트를 머지하기 전에 코드 소유자의 검토를 필요로 하도록 구성할 수도 있습니다. 다음 두 URL에서 더 자세한 내용을 확인할 수 있습니다.

- [https://blog.github.com/2017-07-06-introducing-code-owners/](https://blog.github.com/2017-07-06-introducing-code-owners/)
- [https://help.github.com/articles/about-codeowners/](https://help.github.com/articles/about-codeowners/)

{% include requirement/MUST %} 루트(root) 레벨에 있는 CODEOWNERS 파일을 편집하여 클라이언트 라이브러리 디렉터리에 대해 이 구성요소에 대한 적절한 엔지니어를 가리키도록 모든 풀 리퀘스트에 대한 리다이렉션이 업데이트되었는지를 확인합니다. 클라이언트 라이브러리가 자체 저장소 내에 존재하는 경우, CODEOWNERS 파일을 도입하고 적절하게 구성을 해야 합니다.

다음 규칙을 사용하여 GitHub 및 빌드 실패 알림 모두에 CODEOWNERS를 사용할 수 있는지 확인합니다.

* `/.github/CODEOWNERS` 파일을 사용합니다.
* `/sdk/<service name>/` (슬래시 기호를 시작과 끝 부분에 함께 사용) 통용 규칙을 따라하여 서비스 소유자를 정의합니다.
  * 이 포맷을 사용하는 경우에는, 서비스 소유자들은 자동으로 빌드 알림 실패 경고를 구독하게 될 것입니다
* GitHub가 마지막에 일치하는 표현 (규칙)을 사용하므로, 보다 일반적인 규칙을 파일 위쪽에 배치하고, 보다 구체적인 규칙은 파일 아래 부분에 배치합니다.
* 소유자에 대해 GitHub에서 (지정하는) 사람에 대한 별칭만을 사용합니다 (예: `@person`). 내부 사용자, 내부 그룹 별칭 및 이메일 주소와 연결되지 않은 GitHub 그룹 및 GitHub 사용자는 현재 지원하지 않습니다.
* 해당 폴더의 PR에 자동 라벨링을 지정하고 싶다면,  `# PRLabel: ` 다음에 적용하려는 `%Label`의 내용이 있는 경로와 함께 항목 위에 주석 줄을 추가해야 합니다. 참고: 현재 와일드카드는 지원되지 않으며, 폴더당 하나의 라벨만 작동합니다. 
* 서비스에 대한 이슈가 제시될 때 누가 알림을 받아야 하는지에 대한 정보 또한 기록할 수 있습니다. 이를 위해서는, `# ServiceLabel: ` 다음에 이슈를 적용해야 하는 `%Label` 이 오도록 하고, 아래에는 경로와 함께 이슈에 대해 지정된 사람들이 언급될 수 있도록 추가해야 합니다.
* 서비스에 대한 코드가 저장소 내에 존재하지 않는 경우에는 `#/<NotInRepo>/` 라는 특수한 주석을 경로에 사용하여 서비스에 대한 이슈를 허용하도록 할 수 있습니다.
* `%` 문자와 함께 라벨을 사용하는 경우, 공백 문자를 사용할 수 있습니다. 라벨은 `%` 문자를 시작으로 하여 구분이 이루어집니다.

```gitignore
# Catch-all for SDK changes
/sdk/  @person1 @person2

# Service teams
/sdk/azconfig/   @person3 @person4

# Example for a service that needs issues to be labeled
# ServiceLabel: %KeyVault %Service Attention
/sdk/keyvault/   @person5 @person6

# Example for a service that needs PRs to be labeled
# PRLabel: %label
/sdk/servicebus/ @person7 @person8

# Example for a service that needs both issues and PRs to be labeled
# ServiceLabel: %label
# PRLabel: %label
/sdk/eventhubs/ @person7 @person8

# Example for service that does not have the code in the repo but wants issues to be labeled
# Notice the use of the moniker /<NotInRepo>/
# ServiceLabel: %label %Service Attention
#/<NotInRepo>/ @person7 @person8

```

이 `CODEOWNERS` 파일 예제에는 파일 상단 부분에 포괄적인 소유자 목록이 있으며, 특정 서비스 팀에 대해 자세히 설명합니다. GitHub는 *마지막에* 일치하는 표현식을 사용하여 리뷰어를 할당합니다. 예를 들어 `/sdk/keyvault/`가 변경된 PR은 `@person5` 및 `@person6`이 PR에 추가됩니다. 배치와 같은 새로운 서비스가 `/sdk/batch/` 아래에 변경 사항으로 추가된 경우 `@person1` 및 `@person2`가 할당됩니다.
