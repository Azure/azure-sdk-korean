---
title: "정책: 지원"
permalink: policies_support.html
folder: policies
sidebar: general_sidebar
---

## **Azure SDK의 생명 주기 및 지원 정책**

Azure SDK의 생명 주기와 지원은 아래 정보와 충돌하는 경우 우세한 최신[Microsoft Modern Lifecycle Policy](https://docs.microsoft.com/en-US/lifecycle/policies/modern)의 적용을 받습니다.

### **패키지 생명 주기**

다음은 일반적인 패키지 생명 주기 단계입니다(주요 버전용).

1. **베타** – 조기 액세스 및 피드백을 위해 사용할 수 있는 새로운 SDK로 실가동에서는 사용하지 않는 것이 좋습니다. 베타 버전 지원은 GitHub 문제로 한정되며 응답 시간은 보장되지 않습니다. 베타 릴리스는 일반적으로 1년 미만이며 그 후 폐기되거나 GA에 출시됩니다.

2. **활성** - SDK는 일반적으로 이용 가능하며 완전히 지원되며 새로운 기능 업데이트와 버그 및 보안 수정이 제공됩니다. 메이저 버전은 출시일로부터 최소 12개월 동안 활성화됩니다. 메이저 릴리스에 호환되는 업데이트는 마이너버전 또는 패치버전을 통해 제공됩니다. 최신 버전은 수정 및 업데이트를 받을 수 있으므로 최신 버전을 사용하는 것이 좋습니다.

3. **유지 관리** - 통상 유지 관리 모드는 다음 메이저 버전이 액티브로 전환되는 동시에 발표되며, 그 후 릴리즈에서는 최소 12개월 동안 가장 중요한 버그 수정 및 보안 수정 사항만 해결합니다.

4. **커뮤니티** - SDK는 고객 계약에 별도로 명시되지 않는 한 Microsoft로부터 업데이트를 받지 않습니다. 패키지는 공용 패키지 관리자와 커뮤니티에서 유지 관리할 수 있는 GitHub repo를 통해 계속 사용할 수 있습니다.

[이 페이지에서](https://azure.github.io/azure-sdk/releases/latest/index.html) 패키지의 생명 주기 단계를 확인할 수 있습니다. 

### **Azure 클라우드**

기본적으로 Azure 라이브러리는 전역 Azure 클라우드에 연결하도록 구성됩니다. Azure 스택, Azure China 및 정부 클라우드와 같은 추가 클라우드 대상 플랫폼을 사용할 수 있습니다. 패키지 생명 주기는 대상 플랫폼에 따라 다를 수 있습니다. 자세한 내용은 대상 플랫폼 메뉴얼을 참조해 주세요.

### **Azure SDK 종속성**

Azure SDK 라이브러리는 Azure Service REST API, 프로그래밍 언어 런타임, OS 및 타사 라이브러리에 의존합니다.

> Azure SDK 라이브러리는 플랫폼 및 수명이 다한 다른 의존 관계에서 동작하는 것을 보증하지 않습니다. Azure SDK 라이브러리의 메이저버전을 늘리지 않고 이러한 의존관계에 대한 지원을 폐기할 수 있습니다. 기술 지원을 받을 수 있도록 지원 플랫폼 및 기타 의존관계로 이행할 것을 강력히 권장합니다.

다음은 지원되는 Azure SDK 플랫폼 및 런타임 목록입니다:

**운영 체제:** Windows 10, macOS-10.15 , Linux (Ubuntu 18.04에서 테스트)
모바일 개발의 경우 [IOS 지원 플랫폼](https://azure.github.io/azure-sdk/ios_design.html#ios-library-support)과 [Android 지원 플랫폼](https://azure.github.io/azure-sdk/android_design.html)을 확인하십시오.

**런타임:**

- .NET 표준 2.0을 지원하는 모든 플랫폼. .NET 프레임워크 4.6.1 및 .NET 코어 2.1, .NET 5.0에서 테스트
- Java: Java 8 , Java 11
- Node.js: [Node.js의 LTS 버전](https://nodejs.org/about/releases/) 에는 활성 상태의 LTS 버전뿐만 아니라 유지 관리 상태의 LTS 버전도 포함됩니다.
- Python 3.5+, 2.7
- Go runtime– 최신 메이저 Go 릴리즈 2개를 지원하고 있습니다. 자세한 내용은 https://golang.org/doc/devel/release.html 를 참조해 주세요.
- C의 경우 [여기](https://azure.github.io/azure-sdk/clang_design.html)에서 지원되는 플랫폼 및 컴파일러 목록을 참조하십시오. 

**테스트 구성:**

다음은 다양한 운영 체제 및 런타임에 적용되는 테스트 구성입니다. 지원을 중단하는 일부 발신 버전과 아직 공식적으로 지원되지 않는 수신 버전이 표시될 수 있습니다. 공식적으로 지원되는 세트에 대해서는 이전 섹션의 세부 정보를 참조하십시오.

- [.NET 테스트 구성](https://github.com/Azure/azure-sdk-for-java/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [Java 테스트 구성](https://github.com/Azure/azure-sdk-for-java/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [JavaScript 테스트 구성](https://github.com/Azure/azure-sdk-for-js/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [Python 테스트 구성](https://github.com/Azure/azure-sdk-for-python/blob/main/eng/pipelines/templates/stages/platform-matrix.json)

### **지원:**

지원 계획이 있는 고객은 [여기](https://azure.microsoft.com/en-us/support/create-ticket/)에서 Azure 지원 티켓을 열 수 있습니다. [Azure SDK GitHub 저장소](https://github.com/Azure/azure-sdk/blob/main/README.md)에서 GitHub 문제를 열어 버그 및 기능 요청을 추적할 수 있습니다. GitHub 문제는 무료이지만 처리 시간이 오래 걸릴 수 있습니다.
