---
title: "정책: 지원"
permalink: policies_support.html
folder: policies
sidebar: general_sidebar
---

## **Azure SDK 수명 주기 및 지원 정책**

Azure SDK 수명 주기 및 지원은 [최신 수명 주기 정책](https://docs.microsoft.com/ko-kr/lifecycle/policies/modern)이 적용되며, 아래 정보와 다르더라도 해당 문서가 우선시 됩니다.

### **패키지 수명주기**

다음은 일반적인 패키지 수명 주기의 단계입니다 (주 버전의 경우).

1. **Beta** – 조기 액세스 및 피드백 용도로 사용할 수 있으며 프로덕션 환경에서 사용하지 않는 것이 권장되는 신규 SDK입니다.
   베타 버전 지원은 GitHub issues로 제한되며 응답 시간이 보장되지 않습니다. 베타 릴리스는 일반적으로 1년 미만 동안 유지되며, 그 후 사용 중지되거나 GA에 릴리스됩니다.

2. **Active** - SDK는 일반적으로 사용 가능하며 충분히 지원되며 버그 및 보안 수정뿐만 아니라 새로운 기능 업데이트를 받게 됩니다.
   주 버전은 출시일로부터 최소 12개월 동안 활성 상태를 유지합니다. 주 릴리스에 대한 호환 가능한 업데이트는 부 버전 또는 패치 버전을 통해 제공됩니다.
   고객은 오류 수정 및 업데이트를 받을 수 있는 최신 버전을 사용하시는 것이 권장됩니다.

3. **Maintenance** - 일반적으로 유지보수 모드는 다음 주 버전이 Active로 전환되는 동시에 발표되며,
   그 이후에는 최소 12개월 동안 가장 중요한 버그 수정 및 보안 수정만 처리합니다.

4. **Community** - SDK는 별도의 고객 계약에 달리 명시되지 않는 한 더 이상 Microsoft로부터 업데이트를 받지 않습니다.
   패키지는 커뮤니티가 관리할 수 있는 공용 패키지 관리자와 GitHub repo를 통해 계속 사용할 수 있습니다.

[해당 페이지](https://azure.github.io/azure-sdk/releases/latest/index.html) 에서 패키지의 수명 주기 단계를 확인할 수 있습니다.

### **Azure 클라우드**

기본적으로 Azure 라이브러리는 글로벌 Azure 클라우드에 연결하도록 구성됩니다.
Azure Stack, Azure China 또는 Government 클라우드와 같은 추가 클라우드 플랫폼을 사용할 수 있습니다.
패키지 수명 주기는 대상 플랫폼에 따라 다를 수 있습니다. 자세한 내용은 대상 플랫폼 설명서를 참조하십시오.

### **Azure SDK 종속성**

Azure SDK 라이브러리는 Azure Service REST API, 프로그래밍 언어 런타임, OS 및 타사 라이브러리에 따라 달라집니다.

> Azure SDK 라이브러리는 수명이 다한 플랫폼 및 기타 종속성에서는 작동을 보장하지 않습니다. Azure SDK 라이브러리의 주 버전을 늘리지 않고 이러한 종속성에 대한 지원을 중단할 수 있습니다. 기술 지원을 받으려면 지원되는 플랫폼 및 기타 종속성으로 마이그레이션하는 것을 권장드립니다.

다음은 참조용으로 지원되는 Azure SDK 플랫폼 및 런타임 목록입니다:

**운영체제:** Windows 10, macOS-10.15 , Linux (Ubuntu 18.04에서 테스트됨)
모바일 개발의 경우 [IOS 지원 플랫폼](https://azure.github.io/azure-sdk-korean/ios_design.html#ios-library-support), [Android 지원 플랫폼](https://azure.github.io/azure-sdk-korean/android_design.html)을 확인해주세요.

**런타임:**

- .NET Standard 2.0를 지원하는 모든 플랫폼. .NET Framework 4.6.1 및 .NET Core 2.1, .NET 5.0에서 테스트됨
- Java: Java 8 , Java 11
- Node.js: [Node.js의 LTS 버전](https://nodejs.org/ko/about/releases/)에는 활성 상태뿐만 아니라 유지보수 상태에 있는 버전도 포함됩니다.
- Python 3.5+, 2.7
- Go 런타임– 2개의 최신 주요 Go 릴리스를 지원합니다. 자세한 내용은 https://golang.org/doc/devel/release.html 참조하세요.
- C의 경우 [여기](https://azure.github.io/azure-sdk-korean/clang_design.html)에서 지원되는 플랫폼 및 컴파일러 목록을 참조하십시오.

**테스트 구성:**

다음은 다양한 운영 체제 및 런타임을 다루는 테스트 구성입니다. 지원을 중단하는 발신 버전이나 아직 공식적으로 지원하지 않는 수신 버전을 볼 수 있습니다. 공식적으로 지원되는 세트에 대한 자세한 내용은 이전 섹션을 참조하십시오.

- [.NET 테스트 구성](https://github.com/Azure/azure-sdk-for-java/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [Java 테스트 구성](https://github.com/Azure/azure-sdk-for-java/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [JavaScript 테스트 구성](https://github.com/Azure/azure-sdk-for-js/blob/main/eng/pipelines/templates/stages/platform-matrix.json)
- [Python 테스트 구성](https://github.com/Azure/azure-sdk-for-python/blob/main/eng/pipelines/templates/stages/platform-matrix.json)

### **지원**:

지원 계획이 있는 고객은 [여기](https://azure.microsoft.com/ko-kr/support/create-ticket/)에서 Azure 지원 티켓을 시작할 수 있습니다.
[Azure SDK GitHub repositories](https://github.com/Azure/azure-sdk/blob/main/README.md)에서 GitHub issues룰 시작하여 버그 및 기능 요청을 추적할 수 있습니다. GitHub issues는 비용이 들지 않지만, 처리하는 데 시간이 더 오래 걸릴 수 있습니다.
