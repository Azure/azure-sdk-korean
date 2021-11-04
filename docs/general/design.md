---
title: "공통 가이드라인: API 디자인"
keywords: guidelines
permalink: general_design.html
folder: general
sidebar: general_sidebar
---

클라이언트 라이브러리의 API 표면은 소비자가 서비스와 갖는 주요 상호 작용이기 때문에 가장 많이 고려해야 합니다.

## 네임스페이스

일부 언어에는 관련 유형을 그룹화하기 위한 네임스페이스 개념이 있습니다. 클라우드 인프라 내에서 서비스를 그룹화하는 것은 검색 가능성을 지원하고 참조 문서에 구조를 제공하기 때문에 일반적입니다.

{% include requirement/SHOULD id="general-namespaces-support" %} 언어 생태계에서 네임스페이스 사용이 일반적인 경우 네임스페이스를 지원해야 합니다.

{% include requirement/MUST id="general-namespaces-naming" %} `<AZURE>.<group>.<service>` 형식의 루트 네임스페이스를 사용하십시오. 일반적으로 사용되는 모든 소비자 대면 API는 이 네임스페이스 내에 있어야 합니다. 네임스페이스는 세 부분으로 구성됩니다:

- `<AZURE>`는 모든 Azure 서비스에 대한 공통 접두사를 나타냅니다. 이는 언어 내 공통 형식에 따라 `Azure` 또는 `com.azure`이거나 이와 유사할 수 있습니다.
- `<group>`은 서비스의 그룹입니다. 아래의 리스트를 보십시오.
- `<service>`는  줄여 쓴 서비스 이름입니다.

{% include requirement/MUST id="general-namespaces-shortened-name" %} 소비자가 패키지를 사용 중인 서비스에 연결할 수 있도록 단축된 서비스 이름을 선택하십시오. 기본적으로, 줄여서 쓴 서비스 이름을 사용합니다. 네임스페이스는 제품 브랜딩이 변경될 때 변경되지 **않습니다**. 따라서 변경될 수 있는 마케팅 이름의 사용을 피하세요.

줄여 쓴 서비스 이름은 공백이 없는 서비스 이름입니다. 커뮤니티에서 줄여 쓴 이름이 잘 알려진 경우, 더 줄여 쓸 수 있습니다. 예를 들어, "Azure Media Analytics"는 줄여 쓴 서비스 이름이 `MediaAnalytics`인 한편, "Azure Service Bus"는 `ServiceBus`가 됩니다.

{% include requirement/MUST id="general-namespaces-approved-list" %} (대상 언어가 네임스페이스를 지원하는 경우) 다음 목록을 서비스 그룹으로 사용하십시오:

{% include tables/data_namespaces.md %}

클라이언트 라이브러리가 그룹 목록에 맞지 않는 경우, [Architecture Board]에 문의하여 네임스페이스 요구 사항에 대해 논의하십시오.

{% include requirement/MUST id="general-namespaces-mgmt" %} 관리(Azure 리소스 관리자) API를 `management` 그룹에 배치하십시오. 네임스페이스에 대해 `<AZURE>.management.<group>.<service>` 그룹을 사용하십시오. 더 많은 서비스가 컨트롤 플레인 API를 필요로 하기 때문에, 다른 네임스페이스는 제어 플레인에만 명시적으로 사용할 수 있습니다. 데이터 플레인 사용은 예외입니다. 컨트롤 플레인 SDK에 사용할 수 있는 추가 네임스페이스는 다음과 같습니다:

{% include tables/mgmt_namespaces.md %}

많은 `management` API는 Azure 계정 관리를 처리하기 때문에 데이터 플레인이 없습니다. 관리 라이브러리를 `<AZURE>.management` 네임스페이스에 배치하십시오. 예를 들어, 'azure.management.management.costanalysis' 대신 'azure.management.costanalysis'를 사용하십시오.

{% include requirement/MUSTNOT id="general-namespaces-similar-names" %} 서로 다른 작업을 수행하는 클라이언트에 대해 유사한 이름을 선택하지 마십시오.

{% include requirement/MUST id="general-namespaces-registration" %} 선택한 네임스페이스를 [Architecture Board]에 등록하십시오. 네임스페이스를 요청하려면 이슈를 생성하십시오. 현재 등록된 네임스페이스 목록은 [등록된 네임스페이스 목록](registered_namespaces.html)을 참조하세요.

### 네임스페이스 예시

다음은 이러한 지침을 충족하는 네임스페이스의 몇 가지 예입니다:

- `Azure.Data.Cosmos`
- `Azure.Identity.ActiveDirectory`
- `Azure.IoT.DeviceProvisioning`
- `Azure.Storage.Blobs`
- `Azure.Messaging.NotificationHubs` (Notification Hubs를 위한 클라이언트 라이브러리)
- `Azure.Management.Messaging.NotificationHubs` (Notification Hubs를 위한 관리 라이브러리)

다음은 지침에 맞지 않는 몇 가지 네임스페이스입니다.

- `Microsoft.Azure.CosmosDB` (`Azure` 네임스페이스에 속하지 않으며, 그룹핑을 사용하지 않음)
- `Azure.MixedReality.Kinect` (그룹이 승인된 목록에 없음)
- `Azure.IoT.IoTHub.DeviceProvisioning` (그룹에 너무 많은 단계가 있음)

## 클라이언트 인터페이스

API 표면은 소비자가 서비스에 연결하기 위해 인스턴스화하는 하나 이상의 _service clients_ 및 지원 유형의 집합으로 구성됩니다.

{% include requirement/MUST id="general-client-naming" %} `client` 접미사를 사용하여 서비스 클라이언트 유형의 이름을 지정하십시오.

선택적 데이터, 즉 구어체로는 "옵션 백(options bag)"으로 알려진 것 안에 제공되는 데이터를 작업에서 추가해야 하는 경우가 있습니다.  라이브러리는 일관된 이름을 지정하기 위해 노력해야 합니다.

{% include requirement/SHOULD id="general-client-options-naming" %} `client_options` 접미사를 사용하여 서비스 클라이언트 옵션 백의 유형 이름을 지정해야 합니다.

{% include requirement/SHOULD id="general-client-options-param-naming" %} `option` 접미사를 사용하여 작업 옵션 백 유형의 이름을 지정해야 합니다. 예를 들어, 작업이 `get_secret`인 경우 옵션 백의 유형은 `get_secret_options`이라고 명명됩니다. 

{% include requirement/MUST id="general-client-in-namespace" %} 소비자가 상호 작용할 가능성이 가장 높은 서비스 클라이언트 유형을 클라이언트 라이브러리의 루트 네임스페이스에 배치하십시오(네임스페이스가 대상 언어에서 지원된다고 가정). 특수 서비스 클라이언트는 하위 네임스페이스에 배치할 수 있습니다.

{% include requirement/MUST id="general-client-minimal-constructor" %} 소비자가 서비스에 연결하고 인증하는 데 필요한 최소한의 정보로 서비스 클라이언트를 구성할 수 있도록 허용하십시오.

{% include requirement/MUST id="general-client-standardize-verbs" %} 서비스에 대한 클라이언트 라이브러리 세트 내에서 동사 접두사를 표준화하십시오. 서비스는 아웃바운드 자료(예시: 문서, 블로그 및 대중 연설) 내에서 교차 언어 간 방식으로 특정 작업에 대해 말할 수 있어야 합니다. 동일한 작업이 다른 언어의 다른 동사에 의해 참조되는 경우에는 이 작업을 수행할 수 없습니다.

다음은 표준 동사 접두사입니다. 이러한 작업 중 하나에 대해 대체 동사가 있어야 하는 타당한(명확한) 이유가 있어야 합니다. 예를 들어, .NET은 언어에 관용적이기 때문에 `List<noun>s` 대신 `Get<noun>s`를 사용합니다.

{% include tables/standard_verbs.md %}

{% include requirement/MUST id="general-client-feature-support" %} 클라이언트 라이브러리가 나타내는 Azure 서비스에서 제공하는 기능을 100% 지원하십시오. 기능상의 격차는 개발자들 사이에 혼란과 좌절을 야기합니다.

## 서비스 API 버전

클라이언트 라이브러리의 목적은 Azure 서비스와 통신하는 것입니다. Azure 서비스는 여러 API 버전을 지원합니다. 서비스의 기능을 이해하려면, 클라이언트 라이브러리가 여러 서비스 API 버전을 지원할 수 있어야 합니다.

{% include requirement/MUST id="general-service-apiversion-1" %} 클라이언트 라이브러리의 GA 버전을 출시할 때는 일반적으로 사용 가능한 서비스 API 버전만 대상으로 하십시오.

{% include requirement/MUST id="general-service-apiversion-2" %} 클라이언트 라이브러리의 GA 버전에서는 기본적으로 일반적으로 사용 가능한 최신 서비스 API 버전을 대상으로 하십시오.

{% include requirement/MUST id="general-service-apiversion-5" %} 기본적으로 사용되는 서비스 API 버전을 문서화하십시오.

{% include requirement/MUST id="general-service-apiversion-3" %} 클라이언트 라이브러리의 공개(public) 베타 버전을 출시할 때는 기본적으로 최신 공개 미리 보기 API 버전을 대상으로 하십시오.

{% include requirement/MUST id="general-service-apiversion-4" %} `ServiceVersion` 열거(enumerated) 값에 클라이언트 라이브러리에서 지원하는 모든 서비스 API 버전을 포함하십시오. 

{% include requirement/MUST id="general-service-apiversion-6" %} `ServiceVersion` 열거 값의 값이 서비스 Swagger 정의의 버전 문자열과 "일치"하는지 확인하십시오.

이 요구 사항의 목적을 위해 의미론적(semantic) 변경이 허용됩니다. 예를 들어, 많은 버전 문자열은 점과 대시를 허용하는 SemVer를 기반으로 합니다. 그러나 이러한 문자는 식별자에서는 허용되지 않습니다. 개발자는 서비스의 버전이 `ServiceVersion` 열거 값의 각 값으로 설정될 때, 어떤 서비스 API 버전이 사용될지 **반드시** 명확하게 이해할 수 있어야 합니다.

## 모델 형식

클라이언트 라이브러리는 Azure 서비스와 주고받는 엔터티를 모델 형식으로 나타냅니다. 특정 형식은 서비스 왕복(round-trips)에 사용됩니다. 그것들은 (추가 또는 업데이트 작업으로) 서비스에 보내지고 (가져오기 작업으로) 서비스에서 검색될 수 있습니다. 그들은 형식에 따라 이름 지어져야 합니다. 예를 들어, 앱 구성(App Configuration)의 `ConfigurationSetting`, 또는 이벤트 그리드(Event Grid)의 `Event`입니다.

모델 형식 내에 데이터는 일반적으로 두 부분으로 나눌 수 있습니다 - 서비스에 대한 최상의 시나리오 중 하나를 지원하는 데 사용되는 데이터와 덜 중요한 데이터입니다. `Foo` 형식이 주어지면, 덜 중요한 세부 정보를 `FooDetails`라는 유형으로 수집하고, `details`속성으로 `Foo`에 첨부할 수 있습니다.

예시:

{% highlight csharp %}
class ConfigurationSettingDetails {
    DateTimeOffset lastModifiedDate;
    DateTimeOffset receivedDate;
    ETag eTag;
}

class ConfigurationSetting {
    String key;
    String value;
    ConfigurationSettingDetails details;
}
{% endhighlight %}

작업에 대한 선택적 매개변수 및 설정은 `<operation>Options`라는 옵션 백에 수집해야 합니다. 예를 들어 `GetConfigurationSetting` 메서드는 선택적 매개 변수를 지정하기 위해 `GetConfigurationSettingOptions` 클래스를 사용할 수 있습니다.

결과는 반환 값이 모델에 대한 완전한 데이터 세트인 모델 형식(예: `ConfigurationSetting`)을 사용해야 합니다. 그러나 부분 스키마가 반환되는 경우, 다음 형식을 사용합니다:

* `<model>Item`: 열거(enumeration)가 모델에 대한 부분 스키마를 반환하는 경우, 열거의 각 항목에 대한 `<model>Item`를 사용합니다. 예를 들어, `GetBlobs()`은 Blob의 이름과 메타데이터를 포함하지만 Blob의 내용은 포함하지 않는 `BlobItem`의 열거를 반환합니다.
* `<operation>Result`: 작업의 결과를 위해 `<operation>Result`를 사용합니다. `<operation>`은 특정 서비스 작업에 연결됩니다. 여러 작업에 동일한 결과를 사용할 수 있는 경우, 적절한 명사-동사구를 대신 사용하십시오. 예를 들어, `UploadBlob`의 결과에는 `UploadBlobResult`를 사용하되, Blob 컨테이너를 변경하는 다양한 메서드의 결과에는 `ContainerChangeResult`를 사용합니다.

다음 표에는 생성할 수 있는 다양한 모델이 나열되어 있습니다:

| 형식 | 예시 | 사용 |
| `<model>` | `Secret` | 리소스의 전체 데이터. |
| `<model>Details` | `SecretDetails` | 리소스에 대한 덜 중요한 세부정보. `<model>.details`에 첨부됨.|
| `<model>Item` | `SecretItem` | 열거에 대해 반환된 데이터의 부분 집합. |
| `<operation>Options` | `AddSecretOptions` | 단일 작업에 대한 선택적 매개변수. |
| `<operation>Result` | `AddSecretResult` | 단일 작업에 대한 부분 또는 다른 데이터 집합. |
| `<model><verb>Result` | `SecretChangeResult` | 모델에 대한 여러 작업에 대한 부분적 또는 다른 데이터 집합.  |

## 네트워크 요청

클라이언트 라이브러리는 일반적으로 하나 이상의 HTTP 요청을 래핑하므로, 표준 네트워크 기능을 지원하는 것이 중요합니다. 비동기 프로그래밍 기법은 널리 이해되지는 않지만, 이러한 기법은 확장 가능한 웹 서비스 개발에 필수적이며 특정 환경(모바일 또는 노드 환경)에서 필요합니다. 많은 개발자들은 기술 사용법을 배울 때 쉬운 의미론(semantics) 때문에 동기식 메서드 호출을 선호합니다. 또한 소비자는 네트워크 스택에서 호출 취소, 자동 재시도, 로깅과 같은 특정 기능을 기대하게 되었습니다.

{% include requirement/MUST id="general-network-support-sync-and-async" %}  동기 및 비동기 메서드 호출을 모두 지원하십시오. 언어 또는 기본 런타임이 둘 중 하나를 지원하지 않는 경우는 예외입니다.

{% include requirement/MUST id="general-network-identify-sync-methods" %} 소비자가 어떤 메서드가 비동기식이고, 어떤 메서드가 동기식인지 식별할 수 있는지 확인하십시오.

애플리케이션이 네트워크 요청을 할 때, (라우터와 같은) 네트워크 인프라와 호출된 서비스가 응답하는 데 오랜 시간이 걸릴 수 있으며, 실제로는 아예 응답하지 않을 수 있습니다. 잘 작성된 애플리케이션은 네트워크 인프라나 서비스에 대한 제어를 절대 포기해서는 안 됩니다. 이것이 왜 중요한지에 대한 몇 가지 예는 다음과 같습니다:

- 오케스트레이터가 (스케일 인, 재구성 또는 새 버전으로 업그레이드하기 위해) 서비스를 종료해야 하는 경우, 일반적으로 오케스트레이터는 Posix SIGINT를 전송하여 실행 중인 서비스 인스턴스에 알립니다. 서비스가 이 신호를 수신하면, 현재 진행 중인 모든 네트워크 작업에 적용되는 취소 메커니즘을 설정하여 최대한 빠르고 정상적으로(gracefully) 종료해야 합니다.
- 소비자의 웹 서버가 요청을 수신하면, 사용자에게 응답을 제공해야 하기 전에 허용되는 시간을 나타내는 시간 제한을 설정할 수 있습니다.
- 소비자의 GUI 애플리케이션이 당사의 SDK를 통해 Azure 서비스에 요청할 때, GUI에서 취소 버튼을 제공하여 최종 사용자가 더 이상 작업이 완료되기를 기다리고 있지 않음을 나타낼 수 있습니다.

소비자가 취소 작업을 수행하는 가장 좋은 방법은 취소 객체를 트리를 형성하는 것으로 생각하는 것입니다. 예를 들어:

- 부모 항목을 취소하면 자식 항목이 자동으로 취소됩니다.
- 자식 항목은 부모보다 일찍 타임아웃할 수 있지만 총 시간을 연장할 수는 없습니다.
- 취소는 시간 초과 또는 수동/명시적 취소로 인해 발생할 수 있습니다.

다음은 애플리케이션이 취소 트리를 사용하는 방법의 예입니다:

- 애플리케이션이 시작되면, 전체 애플리케이션을 나타내는 취소 객체를 만들어야 합니다. 이 객체는 SIGINT 알림을 받는 애플리케이션에 대한 응답으로 명시적으로 종료됩니다.
- 웹 서버가 수신 요청을 받으면, 애플리케이션 노드의 하위 항목인 새 취소 객체를 만듭니다. 새 취소 객체는 웹 서버가 요청에 대해 작업할 수 있는 최대 시간을 지정합니다.
- 수신 요청에 대한 작업의 일부로 웹 서버는 (데이터베이스와 같은) 다른 서비스에 여러 요청을 해야 할 수 있습니다. 이러한 요청을 직렬 또는 병렬로 수행할 수 있는 경우, 이전에 생성된 취소 객체를 공유할 수 있습니다. 그러나 웹 서버가 하나 이상의 요청에 소요되는 시간을 제한하려는 경우, (원하는 시간 초과 값으로) 새 취소 객체를 만들고, 이 객체를 들어오는 노드의 자식 객체로 만들 수 있습니다. 이렇게 하면 전체 요청 시간이 초과되거나 이 작업의 최대 시간이 초과될 때(둘 중 먼저 발생하는 경우) 개별 요청 시간이 초과됩니다.
- 여러 요청이 병렬로 이루어진 경우, 일반적으로 소비자는 그 중 하나가 실패하면 모든 요청을 취소하고자 합니다. 이것은 지원되는 시나리오여야 합니다.

{% include requirement/MUST id="general-network-accept-cancellation" %} 모든 비동기식 호출에서 (시간 초과를 구현하는) 플랫폼 고유(platform-native) 취소 토큰을 수락하십시오.

{% include requirement/SHOULD id="general-network-check-cancellation" %} 취소 토큰은 I/O 통화(예시: 네트워크 요청 및 파일 로드)에서만 확인해야 합니다. 클라이언트 라이브러리 내에서 I/O 호출 사이에 취소 토큰을 확인하지 마십시오(예: I/O 호출 간에 데이터를 처리할 때). 취소 토큰은 I/O 통화(예: 네트워크 요청 및 파일 로드)에서만 확인해야 합니다. 클라이언트 라이브러리 내에서 I/O 호출 사이에 취소 토큰을 확인하지 마십시오(예시: I/O 호출 간에 데이터를 처리할 때).

{% include requirement/MUSTNOT id="general-network-no-leakage" %} 기본 프로토콜 전송 구현 세부 정보를 소비자에게 누설하지 마십시오. 프로토콜 전송 구현의 모든 유형은 적절하게 추상화되어야 합니다.

## Authentication
Azure services use a variety of different authentication schemes to allow clients to access the service.  Conceptually, there are two entities responsible in this process: a credential and an authentication policy.  Credentials provide confidential authentication data.  Authentication policies use the data provided by a credential to authenticate requests to the service.

Primarily, all Azure services should support Azure Active Directory OAuth token authentication, and all clients must support authenticating requests in this manner.

{% include requirement/MUST id="general-auth-provide-token-client-constructor" %} provide a service client constructor or factory that accepts an instance of the TokenCredential abstraction from Azure Core.

{% include requirement/MUSTNOT id="auth-client-no-token-persistence" %} persist, cache, or reuse tokens returned from the token credential. This is __CRITICAL__ as credentials generally have a short validity period and the token credential is responsible for refreshing these.

{% include requirement/MUST id="general-auth-use-core" %} use authentication policy implementations from the Azure Core library where available.

{% include requirement/MUST id="general-auth-reserve-when-not-suported" %} reserve the API surface needed for TokenCredential authentication, in the rare case that a service does not yet support Azure Active Directory authentication.

In addition to Azure Active Directory OAuth, services may provide custom authentication schemes. In this case the following guidelines apply.

{% include requirement/MUST id="general-auth-support" %} support all authentication schemes that the service supports.

{% include requirement/MUST id="general-auth-provide-credential-types" %} define a public custom credential type which enables clients to authenticate requests using the custom scheme.

{% include requirement/SHOULDNOT id="general-auth-credential-type-base" %} define custom credential types extending or implementing the TokenCredential abstraction from Azure Core. This is especially true in type safe languages where extending or implementing this abstraction would break the type safety of other service clients, allowing users to instantiate them with the custom credential of the wrong service.

{% include requirement/MUST id="general-auth-credential-type-placement" %} define custom credential types in the same namespace and package as the client, or in a service group namespace and shared package, not in Azure Core or Azure Identity.

{% include requirement/MUST id="general-auth-credential-type-prefix" %} prepend custom credential type names with the service name or service group name to provide clear context to its intended scope and usage.

{% include requirement/MUST id="general-auth-credential-type-suffix" %} append Credential to the end of the custom credential type name. Note this must be singular not plural.

{% include requirement/MUST id="general-auth-provide-credential-constructor" %} define a constructor or factory for the custom credential type which takes in ALL data needed for the custom authentication protocol.

{% include requirement/MUST id="general-auth-provide-update-method" %} define an update method which accepts all mutable credential data, and updates the credential in an atomic, thread safe manner.

{% include requirement/MUSTNOT id="general-auth-credential-set-properties" %} define public settable properties or fields which allow users to update the authentication data directly in a non-atomic manner.

{% include requirement/SHOULDNOT id="general-auth-credential-get-properties" %} define public properties or fields which allow users to access the authentication data directly. They are most often not needed by end users, and are difficult to use in a thread safe manner. In the case that exposing the authentication data is necessary, all the data needed to authenticate requests should be returned from a single API which guarantees the data returned is in a consistent state.

{% include requirement/MUST id="general-auth-provide-client-constructor" %} provide service client constructors or factories that accept all supported credential types.

Client libraries may support providing credential data via a connection string __ONLY IF__ the service provides a connection string to users via the portal or other tooling.   Connection strings are generally good for getting started as they are easily integrated into an application by copy/paste from the portal.  However, connection strings are considered a lesser form of authentication because the credentials cannot be rotated within a running process.

{% include requirement/MUSTNOT id="general-auth-connection-strings" %} support constructing a service client with a connection string unless such connection string is available within tooling (for copy/paste operations).

## Response formats

Requests to the service fall into two basic groups - methods that make a single logical request, or a deterministic sequence of requests.  An example of a *single logical request* is a request that may be retried inside the operation.  An example of a *deterministic sequence of requests* is a paged operation.

The *logical entity* is a protocol neutral representation of a response. For HTTP, the logical entity may combine data from headers, body and the status line. A common example is exposing an ETag header as a property on the logical entity in addition to any deserialized content from the body.

{% include requirement/MUST id="general-response-logical-entity" %} optimize for returning the logical entity for a given request. The logical entity must represent the information needed in the 99%+ case.

{% include requirement/MUST id="general-response-complete" %} allow a consumer to access the complete response, including the status line, headers and body. The client library must follow the language specific guidance for accomplishing this.

{% include requirement/MUST id="general-response-streaming" %} provide examples on how to access the raw and streamed response for a given request, where exposed by the client library.  We do not expect all methods to expose a streamed response.

{% include requirement/MUST id="general-response-enumeration" %} provide a language idiomatic way to enumerate all logical entities for a paged operation, automatically fetching new pages as needed.

For example, in Python:

```python
# Yes:
for instance in client.list_instances():
    print(instance)

# No - don't force the caller of the library to do paging:
next_page = None
while not done:
    list_instance_result = client.list_instances(page=next_page):
    for instance ln list_instance_result.response():
        print(instance)
    next_page = list_instance_result.next_page
    done = next_page is None
```

For methods that combine multiple requests into a single call:

{% include requirement/MUST id="general-response-return-headers" %} return headers and other per-request metadata unless it is obvious as to which specific HTTP request the methods return value corresponds to.

{% include requirement/MUST id="general-response-failure-cases" %} provide enough information in failure cases for an application to take appropriate corrective action.

{% include requirement/SHOULDNOT id="general-response-no-reserved-words" %} use common reserved words as a property name within the models returned within the logical entity.  For example:

- `object`
- `value`

Such usage can cause confusion and will inevitably have to be changed on a per-language basis, which can cause consistency problems.

## Conditional requests

[Conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests) are normally performed using HTTP headers.  The primary usage provides headers that match the `ETag` to some known value.  The `ETag` is an opaque identifier that represents a single version of a resource. For example, adding the following header will translate to "if the record's version, specified by the `ETag`, is not the same".

{% highlight text %}
If-Not-Match: "etag-value"
{% endhighlight %}

With headers, tests are possible for the following:

* Unconditionally (no additional headers)
* If (not) modified since a version (`If-Match` and `If-Not-Match`)
* If (not) modified since a date (`If-Modified-Since` and `If-Unmodified-Since`)
* If (not) present (`If-Match` and `If-Not-Match` with a `ETag=*` value)

Not all services support all of these semantics, and may not support any of them.  Developers have varying levels of understanding of the `ETag` and conditional requests, so it is best to abstract this concept from the API surface.  There are two types of conditional requests we need to be concerned with:

**Safe conditional requests** (e.g. GET)

These are typically used to save bandwidth in an "update cache" scenario, i.e. I have a cached value, only send me the data if what the service has is newer than my copy. These return either a 200 or a 304 status code, indicating the value was not modified, which tells the caller that their cached value is up to date.

**Unsafe conditional requests** (e.g. POST, PUT, or DELETE)

These are typically used to prevent losing updates in an optimistic concurrency scenario, i.e. I've modified the cached value I'm holding, but don't update the service version unless it has the same copy I've got. These return either a success or a 412 error status code, indicating the value was modified, to indicate to the caller that they'll need to retry their update if they want it to succeed.

These two cases are handled differently in client libraries.  However, the form of the call is the same in both cases.  The signature of the method should be:

{% highlight text %}
client.<method>(<item>, requestOptions)
{% endhighlight %}

The `requestOptions` field provides preconditions to the HTTP request.  The `Etag` value will be retrieved from the item that is passed into the method where possible, and method arguments where not possible. The form of the method will be modified based on idiomatic usage patterns in the language of choice.  In cases where the `ETag` value is not known, the operation cannot be conditional.
If the library developer doens't need to support advanced usage of precondition headers, they can add a boolean parameter that is set to true to establish the condition.  For example, use one of the following boolean names instead of the conditions operator:

* `onlyIfChanged`
* `onlyIfUnchanged`
* `onlyIfMissing`
* `onlyIfPresent`

In all cases, the conditional expression is "opt-in", and the default is to perform the operation unconditionally.

The return value from a conditional operation must be carefully considered.  For safe operators (e.g. GET), return a response that will throw if the value is accessed (or follow the same convention used fro a `204 No Content` response), since there is no value in the body to reference.  For unsafe operators (e.g. PUT, DELETE, or POST), throw a specific error when a `Precondition Failed` or `Conflict` result is received.  This allows the consumer to do something different in the case of conflicting results.

{% include requirement/SHOULD %} accept a `conditions` parameter (which takes an enumerated type) on service methods that allow a conditional check on the service.

{% include requirement/SHOULD %} accept an additional boolean or enum parameter on service methods as necessary to enable conditional checks using `ETag`.

{% include requirement/SHOULD %} include the `ETag` field as part of the object model when conditional operations are supported.

{% include requirement/SHOULDNOT %} throw an error when a `304 Not Modified` response is received from the service, unless such errors are idiomatic to the language.

{% include requirement/SHOULD %} throw a distinct error when a `412 Precondition Failed` response or a `409 Conflict` response is received from the service due to a conditional check.

## Pagination

Azure client libraries eschew low-level pagination APIs in favor of high-level abstractions that implement per-item iterators. High-level APIs are easy for developers to use for the majority of use cases but can be more confusing when finer-grained control is required (for example, over-quota/throttling) and debugging when things go wrong. Other guidelines in this document work to mitigate this limitation, for example by providing robust logging, tracing, and pipeline customization options.

{% include requirement/MUST id="general-pagination-use-async-iterators" %} expose paginated collections using language-canonical iterators over items within pages. The APIs used to expose the async iterators are language-dependent but should align with any existing common practices in your ecosystem.

{% include requirement/MUST id="general-pagination-use-iterators" %} expose paginated collections using an iterator or cursor pattern if async iterators aren't a built-in feature of your language.

{% include requirement/MUST id="general-pagination-expose-lists-equally" %} expose non-paginated list endpoints identically to paginated list endpoints. Users shouldn't need to appreciate the difference.

{% include requirement/MUST id="general-pagination-use-distinct-types" %} use distinct types for entities in a list endpoint and an entity returned from a get endpoint if these are different types, and otherwise you must use the same types in these situations.

{% include important.html content="Services should refrain from having a difference between the type of a particular entity as it exists in a list versus the result of a GET request for that individual item as it makes the client library's surface area simpler." %}

{% include requirement/MUSTNOT id="general-pagination-expose-individual-items" %} expose an iterator over each individual item if getting each item requires a corresponding GET request to the service. One GET per item is often too expensive and so not an action we want to take on behalf of users.

{% include requirement/MUSTNOT id="general-pagination-no-arrays" %} expose an API to get a paginated collection into an array. This is a dangerous capability for services which may return many pages.

{% include requirement/MUST id="general-pagination-expose-paging-apis" %} expose paging APIs when iterating over a collection. Paging APIs must accept a continuation token (from a prior run) and a maximum number of items to return, and must return a continuation token as part of the response so that the iterator may continue, potentially on a different machine.

## Long running operations

Long-running operations are operations which consist of an initial request to start the operation followed by polling to determine when the operation has completed or failed. Long-running operations in Azure tend to follow the [REST API guidelines for Long-running Operations][rest-lro], but there are exceptions.

{% include requirement/MUST id="general-lro-expose-poller" %} represent long-running operations with some object that encapsulates the polling and the operation status. This object, called a *poller*, must provide APIs for:

1. querying the current operation state (either asynchronously, which may consult the service, or synchronously which must not)
2. requesting an asynchronous notification when the operation has completed
3. cancelling the operation if cancellation is supported by the service
4. registering disinterest in the operation so polling stops
5. triggering a poll operation manually (automatic polling must be disabled)
6. progress reporting (if supported by the service)

{% include requirement/MUST id="general-lro-polling-config" %} support the following polling configuration options:

* `pollInterval`

Polling configuration may be used only in the absence of relevant retry-after headers from service, and otherwise should be ignored.

{% include requirement/MUST id="general-lro-prefix" %} prefix method names which return a poller with either `begin` or `start`.  Language-specific guidelines will dictate which verb to use.

{% include requirement/MUST id="general-lro-continuation" %} provide a way to instantiate a poller with the serialized state of another poller to begin where it left off, for example by passing the state as a parameter to the same method which started the operation, or by directly instantiating a poller with that state.

{% include requirement/MUSTNOT id="general-lro-cancellation" %} cancel the long-running operation when cancellation is requested via a cancellation token. The cancellation token is cancelling the polling operation and should not have any effect on the service.

{% include requirement/MUST id="general-lro-logging" %} log polling status at the `Info` level (including time to next retry)

{% include requirement/MUST id="general-lro-progress-reporting" %} expose a progress reporting mechanism to the consumer if the service reports progress as part of the polling operation.  Language-dependent guidelines will present additional guidance on how to expose progress reporting in this case.

## Support for non-HTTP protocols

Most Azure services expose a RESTful API over HTTPS.  However, a few services use other protocols, such as [AMQP](https://www.amqp.org/), [MQTT](http://mqtt.org/), or [WebRTC](https://webrtc.org/). In these cases, the operation of the protocol can be split into two phases:

* Per-connection (surrounding when the connection is initiated and terminated)
* Per-operation (surrounding when an operation is sent through the open connection)

The policies that are added to a HTTP request/response (authentication, unique request ID, telemetry, distributed tracing, and logging) are still valid on both a per-connection and per-operation basis.  However, the methods by which these policies are implemented are protocol dependent.

{% include requirement/MUST id="general-proto-policies" %} implement as many of the policies as possible on a per-connection and per-operation basis.

For example, MQTT over WebSockets provides the ability to add headers during the initiation of the WebSockets connection, so this is a good place to add authentication, telemetry, and distributed tracing policies.  However, MQTT has no metadata (the equivalent of HTTP headers), so per-operation policies are not possible.  AMQP, by contract, does have per-operation metadata.  Unique request ID, and distributed tracing headers can be provided on a per-operation basis with AMQP.

{% include requirement/MUST id="general-proto-adparch" %} consult the [Architecture Board] on policy decisions for non-HTTP protocols.  Implementation of all policies is expected.  If the protocol cannot support a policy, obtain an exception from the [Architecture Board].

{% include requirement/MUST id="general-proto-config" %} use the global configuration established in the Azure Core library to configure policies for non-HTTP protocols.  Consumers don't necessarily know what protocol is used by the client library.  They will expect the client library to honor global configuration that they have established for the entire Azure SDK.

{% include refs.md %}
