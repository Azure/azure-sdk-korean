---
title: "정책: 저장소 구조"
permalink: policies_repostructure.html
folder: policies
sidebar: general_sidebar
---

팀뿐만 아니라 커뮤니티에서도 저장소를 보다 일관되고 쉽게 접근할 수 있도록 하려면 일관된 구조를 갖춰야 합니다. 이러한 구조는 저장소 루트에 많은 내용을 배치하지 않도록 해야 하며 저장소를 방문하는 사람들이 스크롤하지 않고도 루트 README.md을 빠르게 볼 수 있도록 해야 합니다. 디렉터리 구조는 다음과 같습니다:

- `common` - 만들어 진 것을 제공하지 않지만 SDK 라이브러리에서 공유 및 사용하는 소스 코드 또는 프로젝트를 포함합니다. 일반적인 테스트 프로젝트나 공유 테스트 또는 소스 코드와 같은 것입니다.
- `doc` - 저장소의 모든 항목에 대한 설명서(일반적으로 Markdown 파일)를 포함합니다. 또한 `doc` 아래에 있는 모든 폴더의 용도를 설명하는 README.md을 포함해야 합니다.  ([예제](https://github.com/Azure/azure-sdk-for-python/blob/main/doc/README.md))
  - `doc\dev` - 저장소에 기여하는 개발자에게 필요한 설명서 모음을 포함합니다.
- `eng` - 엔지니어링 시스템이 다른 관련 작업을 구축, 테스트 또는 수행하는 데 필요한 것들을 포함하는 데 사용됩니다. 일반적으로 구성 파일, 빌드 정의, 스크립트 및 기타 도구(일반적으로 이진에서는 확인되지 않음)가 포함됩니다.
- `sdk` - SDK 라이브러리 소스 코드를 포함할 기본 디렉터리입니다. 레이아웃에 대한 자세한 내용은 아래를 참조하십시오.


이를 위해 모든 Azure-sdk 언어 저장소가 루트에 README.md 및 sdk 폴더를 배치하고, 이 폴더에는 서비스와 연결된 각 패키지의 폴더가 포함됩니다. 개발자들이 우리가 azure sdk를 하나 가지고 있고 많지 않다는 것을 깨닫기를 원하기 때문에 여러 개의 sdk가 아닌 하나의 sdk가 될 것입니다.

### SDK 디렉터리 레이아웃

모든 azure 언어 저장소는 루트에 README.md 및 sdk 폴더를 배치합니다. 이 폴더에는 각 서비스에 대한 폴더가 포함되어 서비스와 연결된 각 패키지의 폴더가 포함됩니다. 개발자들이 우리가 azure sdk를 하나 가지고 있고 많지 않다는 것을 깨닫기를 원하기 때문에 여러 개의 sdk가 아닌 하나의 sdk가 될 것입니다.

```
sdk\<service name>\<package name>\README.md
sdk\<service name>\<package name>\*src*
sdk\<service name>\<package name>\*tests*
sdk\<service name>\<package name>\*samples*
```

- **`<서비스 이름>`** - Azure 서비스의 줄임말이어야 합니다. 관리 및 데이터 플레인 패키지를 비롯한 특정 서비스에 대한 모든 다양한 패키지와 멀티 api(예: 또는 다른 프로파일 버전(예: AzureStack))를 그룹화하는 데 사용됩니다. 릴리스 정보 또는 버전과 같은 모든 공유 산유물은 이 루트 저장소에 저장되어야 합니다. 이러한 이름은 서로 다른 모든 언어 저장소에서 일치해야 하며 기본적으로 [azure-rest-api-specs](https://github.com/azure/azure-rest-api-specs) 저장소의 규격 폴더에 있어야 합니다.
- **`<패키지 이름>`** - 배송 패키지의 이름 또는 지정된 서비스에 대한 지정된 배송 창작물을  구분하는 약어여야 합니다. 중요한 것은 이 폴더에 하나의 배송 패키지에 대한 소스가 있다는 것입니다.
    - 이름이 같지만 scope/org/groupid/SxS-version 등이 서로 다른 패키지가 여러 개 있는 경우 서로 구분되는 이름과 함께 서비스 이름 디렉터리 아래의 고유한 폴더에 각각 넣습니다.
    - 파일 경로 문제가 긴 경우 패키지 이름의 약어를 사용하여 문제를 방지할 수 있습니다. 예를 들어 Java 도구는 package/namespace의 모든 부분을 별도의 폴더여야 하므로 경로가 너무 길 수 있으므로 긴 파일 경로가 창에 미치는 영향을 완화하기 위해 패키지 이름을 축약해야 할 수 있습니다.
    - 공유 라이브러리/소스 코드 또는 도구와 같이 패키지를 제공하지 않는 다른 자산이 있는 경우 서비스 이름 디렉토리 아래의 폴더로 이동할 수도 있습니다.
- 모든 패키지 디렉터리는 다음을 포함해야 합니다:
    - 패키지 루트 폴더에 패키지에 대한 설명서가 들어 있는 README.md입니다.
    - 패키지에 포함된 라이브러리의 소스 코드가 특정 언어 및 도구에 적합한 형식으로 들어 있는 폴더입니다.
    -	패키지에 포함된 라이브러리의 테스트 코드가 특정 언어 및 도구에 적합한 형식으로 들어 있는 폴더입니다.
    -	특정 언어 및 도구에 적합한 형식으로 패키지에 포함된 라이브러리의 샘플 코드가 들어 있는 폴더입니다.


#### 애플리케이션 프레임워크에 대한 특별한 고려 사항

Azure-sdk 언어 저장소에는 Azure 서비스로 널리 사용되는 애플리케이션 프레임워크 간의 통합을 제공하는 모듈/라이브러리/패키지가 포함될 수 있습니다. 예를 들어 스프링 부트, 스프링 데이터 또는 ASP.NET가 있습니다. 일반적으로 특정 서비스와의 통합을 제공하는 모듈은 해당 서비스의 다른 모듈과 함께 배치해야 합니다. 매우 제한된 상황에서 애플리케이션 프레임워크는 여러 통합에서 사용되는 공유 논리를 포함하거나 모든 통합이 단일 모듈에 포함될 수 있습니다. 이 경우 모듈은 응용 프로그램 프레임워크의 이름을 딴 디렉토리에 배치될 수 있습니다. (e.g. ```sdk/spring/```).

### 예제:

#### .NET Repo
```
sdk\keyvault\Microsoft.Azure.Management.KeyVault\README.md
sdk\keyvault\Microsoft.Azure.Management.KeyVault\src
sdk\keyvault\Microsoft.Azure.Management.KeyVault\tests
sdk\keyvault\Microsoft.Azure.Management.KeyVault\samples
sdk\keyvault\Microsoft.Azure.KeyVault\README.md
sdk\keyvault\Microsoft.Azure.KeyVault\src
sdk\keyvault\Microsoft.Azure.KeyVault\tests
sdk\keyvault\Microsoft.Azure.KeyVault\samples
```

#### Python Repo
```
sdk\keyvault\azure-mgmt-keyvault\README.md
sdk\keyvault\azure-mgmt-keyvault\azure\mgmt\keyvault\
sdk\keyvault\azure-mgmt-keyvault\tests
sdk\keyvault\azure-mgmt-keyvault\samples
sdk\keyvault\azure-keyvault\README.md
sdk\keyvault\azure-keyvault\azure\keyvault\
sdk\keyvault\azure-keyvault\tests
sdk\keyvault\azure-keyvault\samples
```

#### Java Repo
```
sdk\keyvault\azure-mgmt-keyvault\README.md
sdk\keyvault\azure-mgmt-keyvault\src\main
sdk\keyvault\azure-mgmt-keyvault\src\test
sdk\keyvault\azure-mgmt-keyvault\src\samples
sdk\keyvault\azure-keyvault\README.md
sdk\keyvault\azure-keyvault\src\main
sdk\keyvault\azure-keyvault\src\test
sdk\keyvault\azure-keyvault\src\samples
```

#### JS Repo
```
sdk\keyvault\arm-keyvault\README.md
sdk\keyvault\arm-keyvault\src
sdk\keyvault\arm-keyvault\test
sdk\keyvault\arm-keyvault\samples
sdk\keyvault\keyvault\README.md
sdk\keyvault\keyvault\src
sdk\keyvault\keyvault\test
sdk\keyvault\keyvault\samples
```
