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

## 인증
Azure 서비스는 클라이언트가 서비스에 액세스할 수 있도록 다양한 인증 체계를 사용합니다. 개념적으로 이 프로세스에는 자격 증명과 인증 정책이라는 두 가지 엔터티가 있습니다. 자격 증명은 기밀 인증 데이터를 제공합니다. 인증 정책은 자격 증명에서 제공한 데이터를 사용하여 서비스에 대한 요청을 인증합니다.

기본적으로 모든 Azure 서비스는 Azure Active Directory OAuth 토큰 인증을 지원해야 하며 모든 클라이언트는 이러한 방식으로 인증 요청을 지원해야 합니다.

{% include requirement/MUST id="general-auth-provide-token-client-constructor" %} Azure Core에서 TokenCredential 추상화 인스턴스를 허용하는 서비스 클라이언트 생성자 또는 팩토리를 제공하십시오.

{% include requirement/MUSTNOT id="auth-client-no-token-persistence" %} 토큰 자격 증명에서 반환된 토큰을 유지, 캐시 또는 재사용하지 마십시오. 일반적으로 자격 증명의 유효 기간이 짧고, 토큰 자격 증명이 이러한 자격 증명을 갱신하는 역할을 담당하기 때문에 이는 __매우 중요합니다__.

{% include requirement/MUST id="general-auth-use-core" %} 사용 가능한 경우 Azure Core 라이브러리의 인증 정책 구현을 사용하십시오.

{% include requirement/MUST id="general-auth-reserve-when-not-suported" %} 서비스가 아직 Azure Active Directory 인증을 지원하지 않는 경우가 드물지만, 이러한 경우 TokenCredential 인증에 필요한 API 표면을 유지하십시오.

Azure Active Directory OAuth 외에도 서비스는 사용자 지정 인증 체계를 제공할 수 있습니다. 이 경우 다음 지침이 적용됩니다.

{% include requirement/MUST id="general-auth-support" %} 서비스가 지원하는 모든 인증 체계를 지원하십시오.

{% include requirement/MUST id="general-auth-provide-credential-types" %} 클라이언트가 사용자 지정 체계를 사용하여 요청을 인증할 수 있도록 하는 공개 사용자 지정 자격 증명 유형을 정의하십시오.

{% include requirement/SHOULDNOT id="general-auth-credential-type-base" %} Azure Core에서 TokenCredential 추상화를 확장하거나 구현하는 사용자 지정 자격 증명 형식을 정의하면 안 됩니다. 이는 특히 이러한 추상화를 확장하거나 구현하면 다른 서비스 클라이언트의 형식 안전성이 훼손되어 사용자가 잘못된 서비스에 대한 사용자 정의 자격 증명으로 해당 클라이언트를 인스턴스화할 수 있는 타입 세이프(type safe) 언어에서 더욱 그렇습니다.

{% include requirement/MUST id="general-auth-credential-type-placement" %} 사용자 지정 자격 증명 형식은 Azure Core나 Azure Identity가 아닌, 클라이언트와 동일한 네임 스페이스 및 패키지 또는 서비스 그룹 네임스페이스와 공유 패키지 에서 지정하십시오.

{% include requirement/MUST id="general-auth-credential-type-prefix" %} 서비스 이름 또는 서비스 그룹 이름에 사용자 지정 자격 증명 유형 이름을 추가하여 의도한 범위 및 사용에 대한 명확한 컨텍스트를 제공하십시오.

{% include requirement/MUST id="general-auth-credential-type-suffix" %} 사용자 지정 자격 증명 유형 이름 끝에 자격 증명(Credential)을 추가하십시오. 이것은 복수가 아닌 단수여야 합니다.

{% include requirement/MUST id="general-auth-provide-credential-constructor" %} 사용자 정의 인증 프로토콜에 필요한 모든 데이터를 가져오는 사용자 정의 자격 증명 유형에 대한 생성자 또는 팩토리를 정의하십시오.

{% include requirement/MUST id="general-auth-provide-update-method" %} 변경 가능한 모든 자격 증명 데이터를 허용하고, 원자적인 스레드 안전 방식으로 자격 증명을 갱신하는 업데이트 메서드를 정의하십시오.

{% include requirement/MUSTNOT id="general-auth-credential-set-properties" %} 사용자가 인증 데이터를 비원자적 방식으로 직접 업데이트할 수 있도록 공개 설정 가능한 속성 또는 필드를 정의하지 마십시오.

{% include requirement/SHOULDNOT id="general-auth-credential-get-properties" %} 사용자가 인증 데이터에 직접 접근할 수 있도록 허용하는 공용 속성 또는 필드를 정의하면 안 됩니다. 이는 최종 사용자가 필요로 하지 않는 경우가 대부분이며, 스레드로부터 안전한 방식으로 사용하기 어렵습니다. 인증 데이터를 노출해야 하는 경우, 요청을 인증하는 데 필요한 모든 데이터는 반환된 데이터가 일관된 상태임을 보장하는 단일 API에서 반환되어야 합니다.

{% include requirement/MUST id="general-auth-provide-client-constructor" %} 지원되는 모든 자격 증명 유형을 허용하는 서비스 클라이언트 생성자 또는 팩토리를 제공하십시오.

클라이언트 라이브러리는 서비스가 포털 또는 기타 도구를 통해 사용자에게 연결 문자열(connection string)을 제공하는 __경우에만__ 연결 문자열을 통한 자격 증명 데이터 제공을 지원할 수 있습니다. 연결 문자열은 일반적으로 포털에서 복사/붙여넣기를 통해 애플리케이션에 쉽게 통합되므로 시작하기에 좋습니다. 그러나 연결 문자열은 실행 중인 프로세스 내에서 자격 증명을 교체할 수 없기 때문에, 덜 인증된 형식으로 간주됩니다.

{% include requirement/MUSTNOT id="general-auth-connection-strings" %} 도구 내에서 (복사/붙여넣기 작업 용으로) 이러한 연결 문자열을 사용할 수 있는 경우가 아니라면, 연결 문자열로 서비스 클라이언트를 구성하는 것을 지원하지 마십시오.

## 응답 형식

서비스에 대한 요청은 두 가지 기본 그룹, 즉 단일 논리적(single logical) 요청을 만드는 메서드 또는 요청의 결정론적 시퀀스(deterministic sequence)로 나뉩니다. *단일 논리적 요청*의 예는 작업 내에서 재시도할 수 있는 요청입니다. *결정론적 요청 시퀀스*의 예로는 페이징 작업이 있습니다.

*논리적 엔티티*는 응답의 프로토콜 중립적 표현입니다. HTTP의 경우, 논리적 엔터티는 헤더, 본문 및 상태 표시줄의 데이터를 결합할 수 있습니다. 일반적인 예로는 본문에서 역직렬화된 내용 외에도 ETag 헤더를 논리적 엔티티의 속성으로 노출하는 것입니다.

{% include requirement/MUST id="general-response-logical-entity" %} 주어진 요청에 대한 논리적 엔터티를 반환하도록 최적화하십시오. 논리적 엔터티는 99% 이상의 경우에 필요한 정보를 나타내야 합니다. 

{% include requirement/MUST id="general-response-complete" %} 소비자가 상태 표시줄, 헤더 및 본문을 포함하여 전체 응답에 접근할 수 있도록 허용하십시오. 클라이언트 라이브러리는 이를 수행하기 위해 언어별 지침을 따라야 합니다.

{% include requirement/MUST id="general-response-streaming" %} 클라이언트 라이브러리에 의해 노출되는 주어진 요청에 대한 원시 및 스트림 응답에 액세스하는 방법의 예시를 제공하십시오. 모든 메서드가 스트림된 응답을 노출할 것으로 기대하지는 않습니다.

{% include requirement/MUST id="general-response-enumeration" %} 필요에 따라 새 페이지를 자동으로 가져오는 페이징 작업에 대한 모든 논리적 엔터티를 열거하는 언어 관용적 방법을 제공하십시오.

예를 들어, 파이썬은 다음과 같습니다:

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

여러 요청을 단일 호출로 결합하는 메서드의 경우:

{% include requirement/MUST id="general-response-return-headers" %} 메서드 반환 값이 어떤 특정 HTTP 요청에 해당하는지 명확하지 않은 경우 헤더 및 기타 요청별 메타데이터를 반환하십시오.

{% include requirement/MUST id="general-response-failure-cases" %} 실패 시 애플리케이션이 적절한 시정 조치를 취할 수 있도록 충분한 정보를 제공하십시오.

{% include requirement/SHOULDNOT id="general-response-no-reserved-words" %} 논리 엔터티 내에 반환된 모델 내에서 공통 예약 단어를 속성 이름으로 사용하면 안 됩니다. 예를 들어:

- `object`
- `value`

이러한 사용법은 혼동을 일으킬 수 있으며, 언어별로 반드시 변경해야 하므로 일관성 문제가 발생할 수 있습니다.

## 조건부 요청

[HTTP 조건부 요청](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests)은 일반적으로 HTTP 헤더를 사용하여 수행됩니다. 기본 사용법은 알려진 값에 `ETag`와 일치시키는 헤더를 제공합니다. `ETag`는 단일 버전의 리소스를 나타내는 불투명한 식별자입니다. 예를 들어 다음 헤더를 추가하면 "`ETag`로 지정된 레코드 버전이 동일하지 않은 경우"로 변환됩니다.

{% highlight text %}
If-Not-Match: "etag-value"
{% endhighlight %}

헤더를 사용하여 다음을 테스트할 수 있습니다:

* 조건 없음 (추가 헤더 없음)
* 버전 이후 수정(안)한 경우 (`If-Match` 및 `If-Not-Match`)
* 날짜 이후 수정(안)한 경우 (`If-Modified-Since` 및 `If-Unmodified-Since`)
* 존재하는(하지 않는) 경우 (`ETag=*` 값이 있는 `If-Match` 및 `If-Not-Match`)

모든 서비스가 이러한 의미 체계를 모두 지원하는 것은 아니며, 어느 것도 지원하지 않을 수 있습니다. 개발자는 `ETag` 및 조건부 요청에 대한 이해 수준이 다양하므로 API 표면에서 이 개념을 추상화하는 것이 가장 좋습니다. 우리가 고려해야 할 조건부 요청에는 두 가지 유형이 있습니다.

**안전한(Safe) 조건부 요청** (예시: GET)

이는 일반적으로 "캐시 업데이트" 시나리오에서 대역폭을 절약하는 데 사용됩니다. (예시: 나에게 캐시된 값이 있습니다. 서비스에 있는 데이터가 내 복사본보다 최신인 경우에만 데이터를 보내주십시오.) 이 경우 값이 수정되지 않았음을 나타내는 200 또는 304 상태 코드가 반환되며, 호출자에게 캐시된 값이 최신 상태임을 알 수 있습니다.

**안전하지 않은(Unsafe) 조건부 요청** (예시 POST, PUT, DELETE)

이는 일반적으로 낙관적 동시성(optimistic concurrency) 시나리오에서 업데이트 손실을 방지하는 데 사용됩니다. (예시: 나는 현재 보유하고 있는 캐시된 값을 수정했습니다. 하지만 동일한 복사본이 없으면 서비스 버전을 업데이트하지 마십시오.) 이 경우 성공 혹은 값이 수정되었음을 나타내는 412 오류 상태 코드를 반환하여, 호출자에게 업데이트가 성공하려면 업데이트를 다시 시도해야 함을 나타냅니다.

이 두 경우는 클라이언트 라이브러리에서 다르게 처리됩니다. 그러나 호출 형식은 두 경우 모두 동일합니다. 메서드의 서명은 다음과 같아야 합니다:

{% highlight text %}
client.<method>(<item>, requestOptions)
{% endhighlight %}

`requestOptions` 필드는 HTTP 요청에 대한 전제 조건을 제공합니다. `Etag` 값은 가능한 경우 메서드에 전달된 항목에서 검색되고, 불가능한 경우 메서드 인수에서 검색됩니다. 메서드의 형식은 선택한 언어의 관용적 사용 패턴을 기반으로 수정됩니다. `ETag` 값을 모르는 경우 작업은 조건부일 수 없습니다.
라이브러리 개발자가 사전 조건 헤더의 고급 사용을 지원할 필요가 없는 경우, 조건을 설정하기 위해 true로 설정된 부울(boolean) 매개변수를 추가할 수 있습니다.  예를 들어, 조건 연산자 대신 다음 부울 이름 중 하나를 사용합니다.

* `onlyIfChanged`
* `onlyIfUnchanged`
* `onlyIfMissing`
* `onlyIfPresent`

모든 경우에 조건식은 "옵트인(opt-in)"이며 기본값은 작업을 무조건 수행하는 것입니다.

조건부 연산의 반환 값은 신중하게 고려해야 합니다. 안전한 연산자(예: GET)의 경우 본문에 참조할 값이 없으므로, 값에 접근하면(또는 `204 No Content` 응답에 사용된 것과 동일한 규칙을 따름) 던져지는 응답을 반환합니다. 안전하지 않은 연산자(예: PUT, DELETE 또는 POST)의 경우 `Precondition Failed` 또는 `Conflict` 결과가 수신되면 특정 오류를 발생시킵니다. 이렇게 하면 결과가 충돌되는 경우 소비자가 다른 작업을 수행할 수 있습니다.

{% include requirement/SHOULD %} 서비스에 대한 조건부 검사를 허용하는 서비스 메소드에 대한 (열거형 유형의) `conditions` 매개변수를 수락해야 합니다.

{% include requirement/SHOULD %} `ETag`를 사용하여 조건부 검사를 활성화하려면 필요에 따라 서비스 메서드에 대한 추가 부울 또는 열거형 매개변수를 수락해야 합니다.

{% include requirement/SHOULD %} 조건부 연산이 지원되는 경우 `ETag` 필드를 객체 모델의 일부로 포함해야 합니다.

{% include requirement/SHOULDNOT %} 서비스로부터 `304 Not Modified` 응답이 수신되었을 때, 이러한 오류가 언어에 관용적이지 않은 한 오류를 발생시켜서는 안 됩니다.

{% include requirement/SHOULD %} 조건부 확인으로 인해 서비스로부터 `412 Precondition Failed` 응답 또는 `409 Conflict` 응답이 수신되면 고유한 오류를 발생시켜야 합니다.

## 페이지 매김(Pagination)

Azure 클라이언트 라이브러리는 항목별 반복기(iterator)를 구현하는 높은 수준의 추상화를 위해 낮은 수준의 페이지 매김 API를 사용하지 않습니다. 고수준 API는 대부분의 사용 사례에 대해 개발자가 사용하기 쉽지만, 보다 세분화된 제어(예시: 할당량 초과/제한)가 필요할 때 그리고 문제가 발생하여 디버깅이 필요할 때 더 혼란스러울 수 있습니다. 이 문서의 다른 지침은 예를 들어 강력한 로깅, 추적 및 파이프라인 사용자 지정 옵션을 제공하여 이러한 제한을 완화합니다.

{% include requirement/MUST id="general-pagination-use-async-iterators" %} 페이지 내의 항목에 대해 언어 표준 반복자를 사용하여 페이지를 매긴 컬렉션을 노출하십시오. 비동기 반복기를 노출하는 데 사용되는 API는 언어에 따라 다르지만, 에코시스템의 기존 일반적인 관행과 일치해야 합니다.

{% include requirement/MUST id="general-pagination-use-iterators" %} 비동기 반복기가 해당 언어의 기본 기능이 아닌 경우, 반복기 또는 커서 패턴을 사용하여 페이지가 매겨진 컬렉션을 노출하십시오.

{% include requirement/MUST id="general-pagination-expose-lists-equally" %} 페이지가 매겨지지 않은 목록 끝점을 페이지가 매겨진 목록 끝점과 동일하게 노출하십시오. 사용자는 그 차이를 인식할 필요가 없습니다.

{% include requirement/MUST id="general-pagination-use-distinct-types" %} 목록 끝점의 엔터티와 가져오기(get) 끝점에서 반환된 엔터티가 서로 다른 유형인 경우 고유한 유형을 사용하고, 그렇지 않으면 이러한 상황에서 동일한 유형을 사용해야 합니다.

{% include important.html content="서비스는 목록에 존재하는 특정 엔티티 형식과 해당 개별 항목에 대한 GET 요청의 결과 간의 차이가 발생하지 않도록 해야 하는데, 이는 클라이언트 라이브러리의 노출 영역을 더욱 단순하게 만들기 때문입니다." %}

{% include requirement/MUSTNOT id="general-pagination-expose-individual-items" %} 각 항목을 가져오려면 해당 GET 요청이 서비스에 필요한 경우 각 개별 항목에 대해 반복기를 노출하지 마십시오. 항목당 하나의 GET는 너무 비싸기 때문에 사용자를 대신하여 수행할 작업이 아닙니다.

{% include requirement/MUSTNOT id="general-pagination-no-arrays" %} 페이지가 매겨진 컬렉션을 배열로 가져오기 위해 API를 노출하지 마십시오. 이는 여러 페이지를 반환할 수 있는 서비스에 위험한 기능입니다.

{% include requirement/MUST id="general-pagination-expose-paging-apis" %} 컬렉션을 반복할 때 페이징 API를 노출하십시오. 페이징 API는 (이전 실행에서 반환할) 연속 토큰과 최대 항목 수를 수락해야 하며, 반복기가 잠재적으로 다른 시스템에서 계속될 수 있도록 응답의 일부로 연속 토큰을 반환해야 합니다.

## 장기 실행 작업

장기 실행 작업은 작업을 시작하기 위한 초기 요청과 작업이 완료되었는지 또는 실패했는지 여부를 확인하기 위한 폴링에 이은 작업으로 구성됩니다. Azure의 장기 실행 작업은 [장기 실행 작업에 대한 REST API 지침][rest-lro]을 따르는 경향이 있지만 예외가 있습니다. 

{% include requirement/MUST id="general-lro-expose-poller" %} 폴링 및 작업 상태를 캡슐화하는 일부 객체로 장기 실행 작업을 나타내십시오. *poller*라고 하는 이 객체는 다음에 대한 API를 제공해야 합니다:

1. 현재 작업 상태 쿼리 (서비스를 참조할 수 있는 비동기식 또는 동기화해서는 안 되는 비동기식)
2. 작업이 완료되었을 때 비동기 알림 요청
3. 서비스에서 취소를 지원하는 경우 작업 취소
4. 폴링이 중지되도록 작업에 무관심 등록
5. 수동으로 폴링 작업 트리거 (자동 폴링을 비활성화해야 함)
6. 진행 상황 보고 (서비스에서 지원하는 경우)

{% include requirement/MUST id="general-lro-polling-config" %} 다음 폴링 구성 옵션을 지원하십시오:

* `pollInterval`

폴링 구성은 서비스에서 관련 재시도 후 헤더가 없는 경우에만 사용할 수 있으며, 그렇지 않은 경우 무시해야 합니다.

{% include requirement/MUST id="general-lro-prefix" %} `begin` 또는 `start`를 사용하여 폴러를 반환하는 접두사 메서드 이름을 지정하십시오. 언어별 지침에 따라 사용할 동사가 지정됩니다.

{% include requirement/MUST id="general-lro-continuation" %} 다른 폴러의 직렬화된 상태로 폴러를 인스턴스화하는 방법을 제공하십시오. 예를 들어 상태를 작업을 시작한 동일한 메소드에 매개변수로 전달하거나 해당 상태로 폴러를 직접 인스턴스화하여 중단된 위치에서 시작합니다.

{% include requirement/MUSTNOT id="general-lro-cancellation" %} 취소 토큰을 통해 취소를 요청한 경우 장기 실행 작업을 취소하지 마십시오. 취소 토큰이 폴링 작업을 취소하고 있으므로 서비스에 영향을 미치지 않아야 합니다.

{% include requirement/MUST id="general-lro-logging" %} `Info` 수준에서 폴링 상태를 기록하십시오(다음 재시도 시간 포함). 

{% include requirement/MUST id="general-lro-progress-reporting" %} 서비스가 폴링 작업의 일부로 진행 상황을 보고하는 경우, 진행률 보고 메커니즘을 소비자에게 노출하십시오. 언어 의존적 지침은 이 경우에 진행 상황 보고를 노출하는 방법에 대한 추가 지침을 제시합니다.

## HTTP가 아닌 프로토콜 지원

대부분의 Azure 서비스는 HTTPS를 통해 RESTful API를 노출합니다. 그러나 일부 서비스는 [AMQP](https://www.amqp.org/), [MQTT](http://mqtt.org/), 또는 [WebRTC](https://webrtc.org/)와 같은 다른 프로토콜을 사용합니다. 이러한 경우 프로토콜 작동은 두 단계로 나눌 수 있습니다:

* 연결별(per-connection, 연결 시작하고 종료될 때를 둘러쌈)
* 작업별(per-operation, 열린 연결을 통해 작업이 전송될 때를 둘러쌈)

HTTP 요청/응답에 추가된 정책(인증, 고유한 요청 ID, 원격 측정, 분산 추적 및 로깅)은 여전히 연결별 및 작업별로 모두 유효합니다. 그러나 이러한 정책을 구현하는 방법은 프로토콜에 따라 다릅니다.

{% include requirement/MUST id="general-proto-policies" %} 연결 및 작업별로 가능한 한 많은 정책을 구현하십시오.

예를 들어 WebSockets를 통한 MQTT는 WebSockets 연결을 시작하는 동안 헤더를 추가할 수 있는 기능을 제공하므로, 인증, 원격 측정 및 분산 추적 정책을 추가하기에 좋습니다. 그러나 MQTT에는 (HTTP 헤더와 동일한) 메타데이터가 없으므로 작업별 정책이 불가능합니다.  AMQP는 계약에 따라 작업별 메타데이터를 가지고 있습니다. AMQP를 사용하여 작업별로 고유 요청 ID 및 분산 추적 헤더를 제공할 수 있습니다.

{% include requirement/MUST id="general-proto-adparch" %} HTTP가 아닌 프로토콜에 대한 정책 결정에 대해서는 [Architecture Board]에 문의하십시오. 모든 정책의 구현이 기대됩니다. 프로토콜이 정책을 지원할 수 없는 경우 [Architecture Board]에서 예외를 얻습니다.

{% include requirement/MUST id="general-proto-config" %} Azure Core 라이브러리에 설정된 전역 구성을 사용하여 HTTP가 아닌 프로토콜에 대한 정책을 구성하십시오.  소비자는 클라이언트 라이브러리에서 어떤 프로토콜을 사용하는지 반드시 알 필요는 없습니다. 이들은 클라이언트 라이브러리가 전체 Azure SDK에 대해 설정한 전역 구성을 준수할 것으로 예상합니다.

{% include refs.md %}
