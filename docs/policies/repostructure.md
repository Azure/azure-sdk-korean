---
title: "정책: 저장소 구조"
permalink: policies_repostructure.html
folder: policies
sidebar: general_sidebar
---

 Repo들을 더욱 일관성 있게 유지하고 우리 뿐만 아니라 커뮤니티가 더 쉽게 접근할 수 있도록 돕기 위해 우리는 일관성 있는 구조를 갖추어야 합니다. 해당 구조는 repo의 root에 많은 것을 포함시키는 것을 피하여 단정한 모습을 갖추어야 하고, repo를 방문하는 많은 이들이 스크롤 내릴 필요 없이 한눈에 root README.md를 볼 수 있도록 해야 합니다. 디렉토리 구조는 이와 같아야 합니다:

- `common` - artifact를 포함하지 않으나 sdk 라이브러리에 의해 사용되고 공유되는 소스 코드나 프로젝트를 포함합니다. 이 예시는 test projects, shared test 또는 source code와 같습니다.
- `doc` - repo에 존재하는 어떠한 것들을 위해, 주로 markdown 파일로 작성된 문서를 포함합니다. `doc` 하위 모든 폴더의 용도가 명시된 README.md 파일을 포함해야 합니다. ([예시](https://github.com/Azure/azure-sdk-for-python/blob/main/doc/README.md))
  - `doc\dev` - 저장소에 기여하는 개발자들을 위한 문서 집합을 포함합니다.
- `eng` - 공학 체계에서 빌드, 테스트, 기타 연관된 기능을 수행해야 할 때 필요한 것들을 포함합니다. 이는 주로 설정 파일, 빌드 정의, 스크립트 또는 기타 도구 (대게 바이너리로 확인되지 않는 것)들을 포함합니다. 
- `sdk` - sdk 라이브러리 소스 코드를 포함하는 주요 디렉토리입니다. 레이아웃의 자세한 설명은 아래를 참조하시오. 


 모든 azure-sdk 언어 repo가 README.md와 root에서 `sdk` 폴더를 포함하고, 그 것이 또한 각 서비스를 위한 폴더와 해당 서비스와 연관된 각 패키지들을 위한 폴더를 포함시키는 것을 이루어내기 위함입니다. 이것은 `sdk`로 칭해지며, 복수형에 반대되는 단수형입니다. 그 이유는 우리는 개발자들이 우리는 여러개가 아닌 오로지 하나의 azure sdk를 가졌음을 인지시키기 위함입니다.

### SDK directory layout

Every azure-sdk language repo will put a README.md and a `sdk` folder in the root that will contain a folder for each service which will then contain a folder for each package associated with the service. It will be `sdk`, singular as opposed to plural, because we want developers to realize we only have one azure sdk and not many.

```
sdk\<service name>\<package name>\README.md
sdk\<service name>\<package name>\*src*
sdk\<service name>\<package name>\*tests*
sdk\<service name>\<package name>\*samples*
```

- **`<service name>`** - Should be the short name for the azure service. It will be used to group all the various packages for a given service, including the management and data-plane packages, as well as any multi-api (e.g. or different profile versions (e.g AzureStack). Any shared artifacts like release notes or versions should go into this root repo. These names should match across all the different language repos and by default should be what is in the specification folder in the [azure-rest-api-specs](https://github.com/azure/azure-rest-api-specs) repo.
- **`<package name>`** - Should be the name of the shipping package, or an abbreviation that distinguishes the given shipping artifact for the given service. The key is that there is source for only one shipping package in this folder.
    - If there are multiple packages with the same name but different scope/org/groupid/SxS-version/etc then put them each in their own folder under the service name directory with names that match whatever distinguishes them from each other.
    - If there are long file path issues then you can use an abbreviation of the package name to help avoid issues. For example Java tools require every part of the package/namespace to be a separate folder which can make the paths extra long and thus may need to abbreviate the package name to help alleviate the impact of long file paths on windows.
    - If there are other assets that aren't shipping packages, such as shared libraries/source code or tools, they can also go in a folder under the service name directory.
- Every package directory must contain the following:
    - A README.md in the package root folder that contains documentation for the package
    - A folder which contains the source code for the library contained in the package in whatever format is appropriate for the specific language and tools.
    - A folder which contains the test code for the library contained in the package in whatever format is appropriate for the specific language and tools.
    - A folder which contains sample code for the library contained in the package in whatever format is appropriate for the specific language and tools.

#### Special considerations for application frameworks

The azure-sdk language repositories will sometimes contain modules/libraries/packages which provide integration between popular application frameworks as Azure services. For example Spring Boot, Spring Data, or ASP.NET. In general the modules that provide integration with a specific service should be co-located with the other modules for that service. In very limited circumstances an application framework may contains shared logic used across multiple integrations, or all integrations are in a single module. In those cases the module may be placed in a directory named after the application framework (e.g. ```sdk/spring/```).

### Examples:

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