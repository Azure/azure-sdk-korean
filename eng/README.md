## 패키지 정보

모든 언어의 패키지 정보는 현재 [여기](https://github.com/Azure/azure-sdk/blob/main/_data/releases/latest)에 있는 csv 파일에 저장되어 있습니다.
각 언어별로 하나의 csv 파일을 관리하며, 매일 [자동화 파이프라인](https://github.com/Azure/azure-sdk/blob/main/eng/pipelines/version-updater.yml)이 실행되어 새로운 릴리스를 확인한 후, 해당 패키지의 csv에 버전 정보와 문서 링크를 업데이트합니다. 버전 정보는 주어진 패키지 생태계의 특정 패키지 관리자로부터 업데이트될 것이며, 우리의 모노 레포(mono repos)에서의 릴리스 태그를 통해서도 업데이트됩니다. 모노 레포에서 출시되는 항목에 대한 저장소와 문서 정보 등의 추가적인 정보도 업데이트할 수 있습니다.

## CSV 필드

- `Package` - 패키지 관리자에 표시되는 패키지의 전체 이름입니다.
- `GroupId` - Java 전용으로 패키지의 GroupId를 포함합니다. Java의 경우 Package 필드에는 ArtifactId만 포함됩니다.
- `VersionGA` - 패키지의 최신 GA 버전을 포함합니다. 별도의 버전이 없는 경우 해당 필드는 비어 있습니다.
- `VersionPreview` - 패키지의 최신 미리보기/베타/사전 릴리즈 버전을 포함합니다. 미리보기가 없거나 미리보기가 최신 GA보다 오래된 경우 해당 필드는 비어 있습니다.
- `DisplayName` - 패키지 인덱스 및 기타 문맥에서 패키지의 보다 더 나은 이름으로 표시하기 위해 사용되는 패키지의 친숙한 이름입니다.
- `ServiceName` - 패키지와 관련된 서비스의 이름으로, 문서에서 특정 서비스와 관련된 패키지 그룹을 구분하기 위해 사용됩니다.
- `RepoPath` - 패키지의 github 레포로의 링크를 생성하기 위한 정보를 포함합니다. 우리의 표준 서비스는 모노 레포에서 배포되기 때문에 이것은 단순히 서비스 디렉토리의 이름이어야 합니다 (예: `/sdk/<service directory>/<package>`). 패키지가 다른 곳에서 오는 경우 전체 링크가 될 수 있습니다. 소스 링크가 없다면 `NA` 값이 있어야 합니다.
- `MSDocs` - 패키지의 Microsoft Docs 사이트로의 링크를 생성하기 위한 정보를 포함합니다. 우리의 표준 서비스는 우리의 모노 레포에서 배포되므로 문서가 발행되면 CSV 파일의 다른 데이터에서 링크가 구성되기 때문에 해당 필드는 비어 있어야 합니다. 문서가 표준이 아닌 위치에 있는 경우 해당 필드에 전체 링크를 포함할 수 있습니다. 문서 링크가 알려지지 않았거나 존재하지 않는 경우 `NA` 값이 있어야 합니다.
- `GHDocs` - Github IO 참조 문서로의 링크를 생성하기 위한 정보를 포함합니다. 우리의 표준 서비스는 우리의 모노 레포에서 배포되므로 문서가 발행되면 CSV 파일의 다른 데이터에서 링크가 구성되기 때문에 해당 필드는 비어 있어야 합니다. 문서가 표준이 아닌 위치에 있는 경우 해당 필드에 전체 링크를 포함할 수 있습니다. 문서 링크가 알려지지 않았거나 존재하지 않는 경우 `NA` 값이 있어야 합니다.
- `Type` - 해당 필드는 패키지의 분류 유형을 나타냅니다. 분류가 알려지지 않은 경우 해당 필드는 비어 있습니다. 현재 분류는 다음과 같습니다:
  - `client` - 데이터 플레인 라이브러리를 나타내는 데 사용됩니다.
  - `mgmt` - 관리 플레인 라이브러리를 나타내는 데 사용됩니다.
  - `spring` - Java를 위한 스프링 라이브러리를 나타내는 특별한 분류입니다.
- `New` - 해당 필드는 리포지토리에서 제시된 지침을 따르는 새로운 라이브러리에 대해 true로 설정됩니다.
- `PlannedVersions` - 해당 필드는 `[version1],[date1]|[version2],[date2]|[version3],[date3]` 형식으로 예상 날짜와 함께 결합된 버전 세트를 나열합니다. 버전은 `X.Y.Z[bN|-beta.N]` 형식이고 날짜는 `MM/dd/yyyy` 형식입니다. 이러한 날짜는 로드맵 페이지에 표시를 위해 사용됩니다.
- `FirstGADate` - 해당 필드는 새 패키지가 처음 GA를 배포하였을 때의 날짜를 확인하기 위해 사용됩니다.
- `Support` - 해당 필드는 패키지의 지원 수준을 확인하는 데 사용됩니다. [지원 지침](https://azure.github.io/azure-sdk/policies_support.html#package-lifecycle)을 참조하십시오. `active, maintenance, community` 중 하나를 포함해야 합니다. 값이 비어 있으면 일반적으로 알려지지 않았거나 beta 지원 수준을 의미합니다.
- `Hide` - 해당 필드는 패키지 인덱스, 문서, 자동 업데이트 등에서 해당 패키지를 숨길지 결정하는데 사용됩니다. 값은 숨기려면 true, 숨기지 않으려면 비워둡니다. 패키지 관리자에 아직 등록된 오래된 패키지를 선별하며, 이를 알리지 않거나 다른 곳에 표시하고 싶지 않을 때 유용합니다.
- `Replace` - 해당 필드는 관련된 이전(대체될) 또는 새로운(대체할) 패키지의 이름을 저장하는 데 사용됩니다. 값은 패키지의 정확한 이름이어야 합니다(Java의 경우 `groupdid\artifactid` 형식이어야 합니다). 여러 개의 패키지가 있다면 쉼표(`,`)로 구분해야 합니다.
- `Notes` - 주어진 패키지에 대한 특정 메모나 주석을 추가할 수 있는 개방된 필드입니다.

## 링크 템플릿

우리는 jekyll 사이트에서 사용하는 다양한 md 파일에 각 언어별 [링크 템플릿](https://github.com/Azure/azure-sdk/tree/main/_includes/releases/variables)을 사용하여 모든 링크를 csv 파일에 저장하지 않고 표준 링크를 생성합니다.
우리는 자동화 과정에서도 이러한 템플릿을 분석하여 업데이트 시 링크가 유효한지 확인합니다. 예를 들어, 현재 사용하는 [링크 템플릿](https://raw.githubusercontent.com/Azure/azure-sdk/main/_includes/releases/variables/java.md)은 다음과 같습니다:

```
{% assign package_label = "maven" %}
{% assign package_trim = "azure-" %}
{% assign pre_suffix = "" %}
{% assign package_url_template = "https://search.maven.org/artifact/item.GroupId/item.Package/item.Version/jar/" %}
{% assign msdocs_url_template =  "https://docs.microsoft.com/java/api/overview/azure/item.TrimmedPackage-readme" %}
{% assign ghdocs_url_template = "https://azuresdkdocs.blob.core.windows.net/$web/java/item.Package/item.Version/index.html" %}
{% assign source_url_template = "https://github.com/Azure/azure-sdk-for-java/tree/item.Package_item.Version/sdk/item.RepoPath/item.Package/" %}
```

템플릿 내의 `item.<property>` 형식의 값들은 CSV 파일 내의 필드 값으로 대체됩니다

## CSV 파일 편집

CSV 파일을 편집해야 한다면 Excel을 사용하지 마세요. Excel은 CSV의 사용자(즉, Jekyll 및 자동화 도구)에게 필요한 따옴표를 모두 제거합니다. 대신 작은 수정이 필요하다면 일반 텍스트 편집기를 사용해주세요. 큰 수정이 필요한 경우, VS Code용 [Edit csv](https://marketplace.visualstudio.com/items?itemName=janisdd.vscode-edit-csv) 확장 프로그램 사용을 추천합니다. 해당 확장 프로그램을 사용할 때 일관성을 유지하기 위해 아래의 설정 옵션을 지정해주세요.

```
"csv-edit.quoteAllFields": true,
"csv-edit.quoteEmptyOrNullFields": "true",
"csv-edit.readOption_hasHeader": "true",
"csv-edit.writeOption_hasHeader": "true"
```
