---
title: "공통 가이드라인: 구현"
keywords: guidelines
permalink: general_implementation.html
folder: general
sidebar: general_sidebar
---

(For English, please visit https://azure.github.io/azure-sdk/general_implementation.html)

허용 가능한 API 표면을 통해 작업을 한 뒤부터는, 서비스 클라이언트의 구현을 시작하실 수 있습니다.

## 구성

당신의 클라이언트 라이브러리를 구성할 때는, 해당 클라이언트 라이브러리를 사용하는 소비자가 Azure 서비스에 대한 연결을 (그 소비자가 사용하고 있는 다른 클라이언트 라이브러리와) 전반적으로(globally) 그리고 당신의 클라이언트 라이브러리와 특정적으로(specifically) 모두 적절하게 구성할 수 있도록 각별히 주의를 기울여야 합니다.

### 클라이언트 구성

{% include requirement/MUST id="general-config-global-config" %} 관련 글로벌 구성 설정은 기본(default)으로 사용하거나 사용자에 의해 명시적으로 요청된 경우(예: 클라이언트 생성자에 구성 객체를 전달하는 등)에 사용하세요.

{% include requirement/MUST id="general-config-for-different-clients" %} 동일 유형의 서로 다른 클라이언트들이 서로 다른 구성을 사용할 수 있도록 허용하세요.

{% include requirement/MUST id="general-config-optout" %} 
서비스 클라이언트의 소비자가 모든 글로벌 구성 설정을 한 번에 취소(opt out)할 수 있도록 허용하세요.

{% include requirement/MUST id="general-config-global-overrides" %} 모든 글로벌 구성 설정을 클라이언트 제공 옵션으로 재정의할 수 있도록 허용하세요. 이러한 옵션의 이름들은 사용자에게 보이는 글로벌 구성 키들과 일치해야 합니다.

{% include requirement/MUSTNOT id="general-config-behaviour-changes" %} 클라이언트가 구성된 후에 발생하는 구성 변경사항에 따라 동작을 변경하지 마세요. 클라이언트의 계층 구조는 명시적으로 변경되거나 재정의되지 않는 한 부모 클라이언트의 구성을 상속합니다. 이 요구 사항에 대한 예외는 다음과 같습니다:

1. 로그 수준 (Log level): Azure SDK 전체에 즉시 적용되어야 합니다.
2. 추적 끄기/켜기 (Tracing on/off): Azure SDK 전체에 즉시 적용되어야 합니다.


### 서비스별 환경 변수

{% include requirement/MUST id="general-config-envvars-prefix" %} Azure별 환경 변수의 접두사를 `AZURE_`로 지정하세요.

{% include requirement/MAY id="general-config-envvars-use-client-specific" %} 당신의 클라이언트 라이브러리별 환경 변수를 클라이언트 라이브러리에 매개변수로 제공되는 포털 구성 설정에 사용할 수 있습니다. 클라이언트 라이브러리별 환경 변수는 일반적으로 자격 증명 및 연결 세부 정보를 포함합니다. 예를 들어, 서비스 버스(Service Bus)는 다음과 같은 환경 변수를 지원할 수 있습니다:

* `AZURE_SERVICEBUS_CONNECTION_STRING`
* `AZURE_SERVICEBUS_NAMESPACE`
* `AZURE_SERVICEBUS_ISSUER`
* `AZURE_SERVICEBUS_ACCESS_KEY`

스토리지(Storage)는 다음을 지원할 수 있습니다:

* `AZURE_STORAGE_ACCOUNT`
* `AZURE_STORAGE_ACCESS_KEY`
* `AZURE_STORAGE_DNS_SUFFIX`
* `AZURE_STORAGE_CONNECTION_STRING`

{% include requirement/MUST id="general-config-envvars-get-approval" %} 모든 새로운 환경 변수에 대해 [Architecture Board] 로부터 승인을 받으세요.

{% include requirement/MUST id="general-config-envvars-format" %} 특정 Azure 서비스에 특정한 환경 변수들에 대해 이 구문을 사용하세요:

* `AZURE_<ServiceName>_<ConfigurationKey>`

여기서 _ServiceName_은 공백이 없는 표준 단축명이고, _ConfigurationKey_는 해당 클라이언트 라이브러리에 대한 중첩되지 않은(unnested) 구성 키를 참조합니다.

{% include requirement/MUSTNOT id="general-config-envvars-posix-compatible" %} 환경 변수 이름에는 밑줄을 제외하고 영숫자가 아닌 문자를 사용하지 마세요. 이는 광범위한 상호 운용성을 보장합니다.


## 매개변수 유효성 검사

서비스 클라이언트에는 서비스에 대한 요청을 수행하는 여러 방법들이 있을 것입니다. _서비스 매개변수_들은 유선을 통해 Azure 서비스로 직접 전달됩니다. _클라이언트 매개변수_는 서비스로 직접 전달되지 않고, 클라이언트 라이브러리 내에서 요청을 이행하는 데 사용됩니다. 클라이언트 매개변수의 예로는 URI를 구성하는 데 사용되는 값이나, 스토리지에 업로드해야 하는 파일이 있습니다.

{% include requirement/MUST id="general-params-client-validation" %} 클라이언트 매개변수의 유효성을 검사하세요. 여기에는 필수 경로 매개변수에 대한 null 값 검사, 그리고 필수 경로 매개변수가 0보다 큰 `minLength`를 선언한 경우 빈 문자열 값 검사 등이 포함됩니다.

{% include requirement/MUSTNOT id="general-params-server-validation" %} 서비스 매개변수의 유효성을 검사하지 마세요. 여기에는 널 검사, 빈 문자열, 그리고 기타 일반적인 유효성 검사 조건들이 포함됩니다. 서비스에서 모든 요청 매개변수들의 유효성을 검사하도록 합니다.

{% include requirement/MUST id="general-params-check-devex" %} 서비스 매개변수가 유효하지 않은 경우 개발자 경험의 유효성을 검사하여 서비스에서 적절한 오류 메시지가 생성되는지 확인하세요. 만약 서비스 측 오류 메시지로 인해 개발자 경험이 손상된 경우 서비스 팀과 협력하여 릴리스 전에 수정하세요.


## 네트워크 요청

지원되는 각 언어에는 구성 및 HTTP 요청 수행과 같은 횡단 관심사(cross cutting concerns)를 해결하기 위한 공통 메커니즘이 포함된 Azure Core 라이브러리가 있습니다.

{% include requirement/MUST id="general-requests-use-pipeline" %} 서비스 REST 엔드포인트와 통신하려면 Azure Core 내의 HTTP 파이프라인 컴포넌트를 사용하세요.

HTTP 파이프라인은 여러 정책에 의해 감싸지는 HTTP 전송으로 구성됩니다. 각 정책은 파이프라인이 요청 그리고/또는 응답을 수정할 수 있는 제어 지점입니다. 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방식을 표준화하기 위해 기본 정책 집합을 규정합니다. 목록의 순서는 구현을 위한 가장 합리적인 순서입니다.

{% include requirement/MUST id="general-requests-implement-policies" %} HTTP 파이프라인에서 다음 정책을 구현하세요:

- Telemetry
- Unique Request ID
- Retry
- Authentication
- Response downloader
- Distributed tracing
- Logging

{% include requirement/SHOULD id="general-requests-use-azurecore-impl" %} 가능하면 Azure Core의 정책 구현을 사용해야 합니다. 당신의 서비스에 고유한 작업을 수행하는 경우가 아니라면 "직접" 정책을 작성하려고 시도하지 마세요. 기존 정책에 다른 옵션이 필요하다면, [Architecture Board]에 참여하여 옵션을 추가하세요.


## 인증

인증을 구현할 때는 PII(개인 식별 정보) 유출이나 자격증명 유출과 같은 보안 허점에 소비자를 노출시키지 마세요. 자격 증명은 일반적으로 시간 제한이 있는 상태로 발급되며, 서비스 연결이 예상대로 계속 작동하고 있는지 확인하기 위해 주기적으로 갱신(refresh)되어야 합니다. 클라이언트 라이브러리가 현재의 모든 보안 권장 사항을 따르고 있는지 확인하고 소비자에게 잠재적인 보안 문제가 발생하지 않도록 클라이언트 라이브러리에 대한 독립적인 보안 검토를 고려하세요.

{% include requirement/MUSTNOT id="general-authimpl-no-persisting" %} 보안 자격증명을 지속, 캐시 또는 재사용하지 마세요. 보안 자격증명은 보안 문제와 자격증명 갱신 상황을 모두 처리하기 위한 일시적인 것(short lived)으로 간주해야 합니다.   

만약 당신의 서비스에서 비표준 자격 증명 시스템(즉, Azure Core에서 지원하지 않는 자격증명 시스템)을 구현하는 경우라면, 당신은 클라이언트 라이브러리에서 제공하는 대체 자격증명 타입을 사용하여 요청을 인증할 수 있는 HTTP 파이프라인에 대한 인증 정책을 생성해야 합니다.

{% include requirement/MUST id="general-authimpl-provide-auth-policy" %} 비표준 자격증명을 사용하는 경우 HTTP 파이프라인에서 HTTP 요청을 인증하는 적절한 인증 정책을 제공하세요. 여기에는 지원되는 경우 사용자 지정 연결 문자열(custom connection strings)이 포함됩니다.


## 네이티브 코드

일부 언어는 플랫폼별 네이티브 코드 플러그인 개발을 지원합니다. 이러한 경우 호환성 문제가 발생할 수 있으며 추가적인 검토가 필요합니다. 특정 언어는 머신 네이티브 형식(예: C 또는 C++)으로 컴파일되지만, 대부분의 현대 언어들은 크로스 플랫폼 지원을 위해 중간 형식으로 컴파일하는 방식을 선택합니다.

{% include requirement/SHOULD id="general-no-nativecode" %} 언어가 머신 네이티브 형식으로 컴파일되지 않는 한 플랫폼별 / 네이티브 코드를 작성해야 합니다.


## 오류 처리

오류 처리는 클라이언트 라이브러리의 구현에 있어 중요한 측면입니다. 이는 문제가 소비자에게 전달되는 주요 방법입니다. 오류가 소비자에게 보고되는 방법에는 두 가지가 있습니다. 메서드가 예외를 던지거나, 메서드가 그 반환값으로 오류 코드(또는 값)를 반환하면, 소비자는 이를 확인해야 합니다. 이 섹션에서는 "오류를 발생시킨다"는 것은 오류 값을 반환하거나 예외를 던지는 것을 의미하며, "오류"는 오류 값 또는 예외 객체입니다.

{% include requirement/SHOULD id="general-errors-prefer-exceptions" %} 오류를 생성할 때는 오류 값을 반환하는 것보다 예외를 사용하는 것을 선호해야 합니다.

{% include requirement/MUST id="general-errors-for-failed-requests" %} 서비스/Swagger에서 성공 상태 코드로 정의되지 않은 HTTP 상태 코드로 HTTP 요청이 실패할 경우 오류를 생성하세요. 이러한 오류들도 오류로 기록되어야 합니다.

{% include requirement/MUST id="general-errors-include-request-response" %} 발생한 오류에 HTTP 응답 (상태 코드 및 헤더 포함)과 발신 요청 (URL, 쿼리 매개변수, 헤더 포함)이 포함되어 있는지 확인하세요.

여러 HTTP 요청을 생성하는 상위 수준의 메서드의 경우, 마지막 예외 또는 모든 실패에 대한 집계 예외가 생성되어야 합니다.
 
{% include requirement/MUST id="general-errors-rich-info" %} 서비스가 (응답 헤더 또는 바디를 통해) 풍부한 오류 정보를 반환하는 경우, 서비스별 프로퍼티/필드에서 생성된 오류를 통해 풍부한 정보를 사용할 수 있어야 합니다.

{% include requirement/SHOULDNOT id="general-errors-no-new-types" %} 개발자가 오류를 해결하기 위한 대체 작업을 수행할 수 있는 경우가 아니라면 새로운 오류 타입을 만들어서는 안 됩니다. 특수한 오류 타입은 Azure Core 패키지에 있는 기존 오류 타입을 기반으로 해야 합니다.

{% include requirement/MUSTNOT id="general-errors-use-system-types" %} 언어별 오류 타입으로 충분할 경우 새 오류 타입을 만들지 마세요. 유효성 검사를 위해 시스템에서 제공하는 오류 타입을 사용하세요.

{% include requirement/MUST id="general-errors-documentation" %} 각 메서드에서 생성되는 오류를 문서화하세요 (일반적으로 타깃 언어로 문서화되지 않은 흔히 발생하는 오류는 제외).


## 로깅

클라이언트 라이브러리는 소비자가 메서드 호출의 이슈를 적절히 진단하고 이슈가 소비자 코드, 클라이언트 라이브러리 코드 또는 서비스에 있는지 신속하게 판단할 수 있도록 강력한 로깅 메커니즘을 지원해야 합니다.

일반적으로, 이러한 라이브러리를 사용하는 소비자에게는 애플리케이션의 문제를 포착하기 위해 프로덕션 환경에서 '경고(WARNING)' 수준 이상으로 원하는 방식으로 로깅을 설정할 것을 권장하며, 이 수준은 고객 지원 상황에 충분해야 합니다. 정보가 담기거나 자세한 로깅은 사례별로 이슈 해결을 도울 수 있습니다.

{% include requirement/MUST id="general-logging-pluggable-logger" %} 연결 가능한(pluggable) 로그 핸들러를 지원하세요.

{% include requirement/MUST id="general-logging-console-logger" %} 소비자가 콘솔에 로깅 출력을 쉽게 할 수 있도록 하세요. 콘솔에 로깅하는 데 필요한 구체적인 단계는 문서화되어야 합니다.

{% include requirement/MUST id="general-logging-levels" %} 로그를 내보낼 때는 다음 로그 수준 중 하나를 사용하세요: `Verbose` (세부 사항), `Informational` (발생한 사항), `Warning`(문제이거나 아닐 수도 있음), `Error`.

{% include requirement/MUST id="general-logging-failure" %} (메모리 부족 등) 애플리케이션이 복구될 가능성이 없는 실패에 대해서는 `Error` 로깅 수준을 사용하세요.

{% include requirement/MUST id="general-logging-warning" %} 어떤 기능이 의도한 작업을 수행하지 못할 경우 `Warning` 로깅 수준을 사용하세요. 이는 일반적으로 함수가 예외를 발생시킬 것을 의미합니다. 자기 회복(self-healing) 이벤트의 발생(예: 요청이 자동으로 재시도되는 경우)은 포함하지 마세요.

{% include requirement/MAY id="general-logging-slowlinks" %} (응답 본문 시작까지의) 요청/응답 주기가 서비스 정의 임계값을 초과하는 경우 당신은 `Warning`에서 요청 및 응답(아래 참조)을 로깅할 수 있습니다. 임계값은 긍정오류(false-positive)을 최소화하고 서비스 문제를 식별할 수 있도록 선택되어야 합니다.

{% include requirement/MUST id="general-logging-info" %} 기능이 정상적으로 작동할 경우에는 `Informational` 로깅 수준을 사용하세요.

{% include requirement/MUST id="general-logging-verbose" %} 자세한 문제 해결 시나리오가 필요한 경우 `Verbose` 로깅 수준을 사용하세요. 이는 주로 개발자 또는 시스템 관리자가 특정 실패를 진단하기 위한 것입니다.

{% include requirement/MUST id="general-logging-no-sensitive-info" %} 승인된 헤더 및 쿼리 매개변수 중 서비스에서 제공하는 "허용 목록(allow-list)"에 있는 헤더 및 쿼리 매개변수만 로깅하세요. 다른 모든 헤더 및 쿼리 매개 변수에 대해서는 해당 값들이 편집(redacted)되어야 합니다.

{% include requirement/MUST id="general-logging-requests" %} 요청 라인과 헤더를 `Informational` 메시지로 기록하세요. 로그에는 다음 정보가 포함되어야 합니다:

* HTTP 메서드.
* URL.
* 쿼리 매개변수 (허용 목록에 없을 경우 삭제).
* 요청 헤더 (허용 목록에 없을 경우 삭제).
* 상관관계 목적으로 SDK에서 제공한 요청 ID.
* 이 요청이 시도된 횟수.

{% include requirement/MUST id="general-logging-responses" %} 응답 라인과 헤더를 `Informational` 메시지로 기록하세요. 로그 형식은 다음과 같아야 합니다:

* SDK에서 제공한 요청 ID (위 참조).
* 상태 코드.
* 상태 코드와 함께 제공된 모든 메시지.
* 응답 헤더 (허용 목록에 없을 경우 삭제).
* 요청의 첫 번째 시도와 본문의 첫 번째 바이트 사이의 주기.

{% include requirement/MUST id="general-logging-cancellations" %} 서비스 호출이 취소된 경우 `Informational` 메세지를 기록하세요. 로그에는 다음이 포함되어야 합니다:

* SDK에서 제공한 요청 ID (위 참조).
* 취소 사유 (가능한 경우).

{% include requirement/MUST id="general-logging-exceptions" %} 발생한 예외를 `Warning` 수준 메세지로 기록하세요. 만약 로그 수준이 `Verbose`로 설정되어있는 경우, 메세지에 스택 추적 정보(stack trace information)를 추가합니다.


## 분산 추적

분산 추적 메커니즘은 소비자가 프론트엔드에서 백엔드까지 코드를 추적할 수 있도록 합니다. 분산 추적 라이브러리는 고유한 작업 단위인 스팬(span)을 생성합니다.  각 스팬은 부모-자식 관계에 있습니다. 코드의 계층 구조가 깊어질수록 더 많은 스팬이 생성됩니다. 이후에 이러한 스팬은 필요에 따라 적절한 수신기로 내보내질 수 있습니다. 스팬을 추적하기 위해, _분산 추적 컨텍스트_ (이하 컨텍스트)가 각각의 연속된 레이어에 전달됩니다. 이 항목에 대한 자세한 내용은, 추적에 대한 [OpenTelemetry] 항목을 참조하세요.

{% include requirement/MUST id="general-tracing-opentelemetry" %} 분산 추적을 위해 [OpenTelemetry]를 지원하세요.

{% include requirement/MUST id="general-tracing-accept-context" %} 부모 스팬을 설정하기 위해 호출 코드로부터 컨텍스트를 받으세요.

{% include requirement/MUST id="general-tracing-pass-context" %} [Azure Monitor]를 지원하기 위해 적절한 헤더 (`traceparent`, `tracestate` 등)를 통해 백엔드 서비스에 컨텍스트를 전달하세요. 이는 일반적으로 HTTP 파이프라인에서 수행됩니다.

{% include requirement/MUST id="general-tracing-new-span-per-method" %} 사용자 코드가 호출한 각 메서드에 대해 새 스팬을 생성하세요. 새 스팬은 전달된 컨텍스트의 자식 스팬이어야 합니다. 전달된 컨텍스트가 없었다면, 새로운 루트 스팬이 만들어져야 합니다.

{% include requirement/MUST id="general-tracing-new-span-per-rest-call" %} 클라이언트 라이브러리에서 일으키는 각 REST 호출에 대해 새 스팬 (메서드별 스팬의 자식 스팬이어야 함)을 생성하세요. 이는 일반적으로 HTTP 파이프라인에서 수행됩니다.

클라이언트 라이브러리가 수행하는 각 REST 호출에 대해 새 범위(메소드별 범위의 자식이어야 함)를 생성합니다. 이 작업은 일반적으로 HTTP 파이프라인을 사용하여 수행됩니다.

Some of these requirements will be handled by the HTTP pipeline.  However, as a client library writer, you must handle the incoming context appropriately.

이러한 요구사항들 중 일부는 HTTP 파이프라인에서 처리될 것입니다. 그러나, 클라이언트 라이브러리 작성자로서, 당신은 들어오는 컨텍스트를 적절하게 처리해야 합니다.


## 의존성

의존성은 의존성을 피함으로써 쉽게 피할 수 있는 많은 고려 사항들을 불러일으킵니다.

- **버전 관리** - 많은 프로그래밍 언어에서는 소비자가 동일한 패키지의 여러 버전들을 로드하는 것을 허용하지 않습니다. 예를 들어, 클라이언트 라이브러리는 버전 3의 Foo 패키지를 필요로 하고 소비자는 버전 5의 Foo 패키지 사용을 원하는 경우, 소비자는 애플리케이션을 빌드할 수 없습니다. 이는 클라이언트 라이브러리는 기본적으로 의존성을 가지지 않아야 한다는 것을 의미합니다.
- **크기** - 소비자 애플리케이션은 가능한 한 빠르게 클라우드에 배포하고 네트워크를 통해 다양한 방식으로 이동할 수 있어야 합니다. 부가적인 코드(예: 의존성)의 제거는 배포 성능을 향상시킵니다.
- **라이선스** - 의존성의 라이선스 제한을 인지하고 있어야 하며 사용할 때 적절한 저작자표시(attribution)와 고지사항(notices)를 제공해야 합니다.
- **호환성** - 종종 의존성을 제어하지 않아 원래의 사용과 호환되지 않는 방향으로 비약할 수 있습니다.
- **보안** - 의존성에서 보안 취약점이 발견되었다면, Microsoft가 의존성의 코드 베이스를 제어하지 않는 경우 취약점을 수정하는 데 어려움이 있거나 시간이 오래 걸릴 수 있습니다.

{% include requirement/MUST id="general-dependencies-azure-core" %} 모든 클라이언트 라이브러리에서 공통되는 기능은 Azure Core 라이브러리에 의존하세요. 이 라이브러리에는 HTTP 연결, 글로벌 구성, 자격증명 처리를 위한 API들이 포함되어 있습니다.

{% include requirement/MUSTNOT id="general-dependencies-approved-only" %} 클라이언트 라이브러리 배포 패키지 내의 다른 패키지에 의존하지 마세요. 의존성은 예외적인 경우이며 아키텍처 검토를 통해 철저한 심사가 필요합니다. 이는 허용 가능하고 일반적으로 사용되는, 빌드 의존성에는 적용되지 않습니다.

{% include requirement/SHOULD id="general-dependencies-vendoring" %} 생태계(ecosystem)와 충돌할 수 있는 다른 패키지에 의존성을 갖지 않으려면 클라이언트 라이브러리에 필요한 코드를 복사하거나 연결(link)하는 것을 고려해야 합니다. 라이선스 계약을 위반하지 않았는지 확인하고 복제된 코드에 필요할 유지보수를 고려하세요. ["A little copying is better than a little dependency"][1] (YouTube).

{% include requirement/MUSTNOT id="general-dependencies-concrete" %} 구체적인 로깅, 의존성 주입, 또는 구성 기술에 의존하지 마세요 (Azure Core 라이브러리에서 구현된 경우 제외). 클라이언트 라이브러리는 애플리케이션에서 자체적으로 선택한 로깅, 의존성 주입(DI), 구성 기술을 사용할 애플리케이션에서 사용될 것입니다.

언어별 가이드라인은 승인된 의존성의 목록을 유지관리할 것입니다.


## 서비스별 공통 라이브러리 코드

여러 클라이언트 라이브러리 간에 공통 코드를 공유해야 하는 경우가 있습니다. 예를 들어, 서로 협력하는 클라이언트 라이브러리들의 집합이 예외 또는 모델의 한 집합을 공유하고자 할 수 있습니다.

{% include requirement/MUST id="general-commonlib-approval" %} 공통 라이브러리를 구현하기 전에 [Architecture Board]의 승인을 받으세요.

{% include requirement/MUST id="general-commonlib-minimize-code" %} 공통 라이브러리 내의 코드를 최소화하세요. 공통 라이브러리 내의 코드는 클라이언트 라이브러리의 소비자가 사용할 수 있으며 동일한 네임스페이스 내의 여러 클라이언트 라이브러리에서 공유할 수 있습니다.

{% include requirement/MUST id="general-commonlib-namespace" %} 공통 라이브러리를 연결된 클라이언트 라이브러리들과 동일한 네임스페이스에 저장하세요.

공통 라이브러리는 다음과 같은 경우에만 승인됩니다:

* 공유되지 않은 라이브러리의 소비자가 공통 라이브러리 내의 객체를 직접 사용할 경우, 그리고
* 정보가 여러 클라이언트 라이브러리들 간에 공유되는 경우.

두 가지 예를 들어 보겠습니다:

1. 두 개의 Coginitive Services 클라이언트 라이브러리를 구현하며, 한 Coginitive Services 클라이언트 라이브러리에서 생성되고 다른 Coginitive Services 클라이언트 라이브러리에서 소비되는 모델이 필요하거나, 두 클라이언트 라이브러리에서 동일한 모델이 생성되도록 하고 싶습니다. 소비자는 코드에서 모델을 전달해야 하거나, 한 클라이언트 라이브러리에서 생성된 모델과 다른 클라이언트 라이브러리에서 생성된 모델을 비교해야 할 수 있습니다. 이러한 경우는 공통 라이브러리를 선택하기에 좋은 사례입니다.

2. 두 개의 Cognitive Services 클라이언트 라이브러리는 이미지에서 객체를 감지하지 못했음을 나타내는 `ObjectNotFound` 예외를 던집니다. 사용자는 예외를 trap하겠지만, 그렇지 않으면 예외에 대해 작동하지 않습니다. 각 클라이언트 라이브러리에서 ObjectNotFound 예외 사이에는 연결 고리가 없습니다. (해당 네임스페이스에 이미 예외가 있는 경우 이 예외를 공통 라이브러리에 배치하고 싶을 수 있겠지만) 이러한 경우는 공통 라이브러리를 만드는 데 좋은 사례가 아닙니다. 대신, 두 개의 서로 다른 예외를 생성하세요 - 각 클라이언트 라이브러리에 하나씩.
<!--
'trap'에 대한 부가 설명을 넣는 게 좋을까
ex. (software-triggered interrupt)
--> 


## Testing

Software testing provides developers a safety net. Investing in tests upfront saves time overall due to increased certainty over the development process that changes are not resulting in divergence from stated requirements and specifications. The intention of these testing guidelines is to focus on the complexities around testing APIs that are backed by live services when in their normal operating mode. We want to enable open source development of our client libraries, with certainty that regardless of the developer making code changes there always remains conformance to the initial design goals of the code. Additionally, our goal is to ensure that developers building atop the Azure client libraries can meaningfully test their own code, without incurring additional complexity or expense through unnecessary interactions with a live Azure service.

{% include requirement/MUST id="general-testing-1" %} write tests that ensure all APIs fulfil their contract and algorithms work as specified. Focus particular attention on client functionality, and places where payloads are serialized and deserialized.

{% include requirement/MUST id="general-testing-2" %} ensure that client libraries have appropriate unit test coverage, [focusing on quality tests][2], using code coverage reporting tools to identify areas where more tests would be beneficial. Each client library should define its minimum level of code coverage, and ensure that this is maintained as the code base evolves.

{% include requirement/MUST id="general-testing-3" %} use unique, descriptive test case names so test failures in CI (especially external PRs) are readily understandable.

{% include requirement/MUST id="general-testing-4" %} ensure that users can run all tests without needing access to Microsoft-internal resources. If internal-only tests are necessary, these should be a separate test suite triggered via a separate command, so that they are not executed by users who will then encounter test failures that they cannot resolve.

{% include requirement/MUSTNOT id="general-testing-5" %} rely on pre-existing test resources or infrastructure and **DO NOT** leave test resources around after tests have completed. Anything needed for a test should be initialized and cleaned up as part of the test execution (whether by running an ARM template prior to starting tests, or by setting up and tearing down resources in the tests themselves).

### Recorded tests

{% include requirement/MUST id="general-testing-6" %} ensure that all tests work without the need for any network connectivity or access to Azure services.

{% include requirement/MUST id="general-testing-7" %} write tests that use a mock service implementation, with a set of recorded tests per service version supported by the client library. This ensures that the service client continues to properly consume service responses as APIs and implementations evolve. Recorded tests must be run using the language-appropriate trigger to enable the specific service version support in the client library.

{% include requirement/MUST id="general-testing-8" %} recreate recorded tests for latest service version when notified by the service team of any changes to the endpoint APIs for that service version. In the absence of this notification, recordings should not be updated needlessly. When the service team requires recorded tests to be recreated, or when a recorded test begins to fail unexpectedly, notify the architecture board before recreating the tests.

{% include requirement/MUST id="general-testing-9" %} enable all network-mocked tests to also connect to live Azure service. The test assertions should remain unchanged regardless of whether the service call is mocked or not.

{% include requirement/MUSTNOT id="general-testing-10" %} include sensitive information in recorded tests.

### Testability

As outlined above, writing tests that we can run constantly is critical for confidence in our client library offering, but equally critical is enabling users of the Azure client libraries to write tests for their applications and libraries. End users want to be certain that their code is performing appropriately, and in cases where this code interacts with the Azure client libraries, end users do not want complex or costly Azure interactions to prevent their ability to test their software.

{% include requirement/MUST id="general-testing-mocking" %} support mocking of service client methods through standard mocking frameworks or other means.

{% include requirement/MUST id="general-testing-11" %} support the ability to instantiate and set all properties on model objects, such that users may return these from their code.

{% include requirement/MUST id="general-testing-12" %} support user tests to operate in a network-mocked manner without the need for network access.

{% include requirement/MUST id="general-testing-13" %} provide clear documentation on how users should instantiate the client library such that it can be mocked.

{% include refs.md %}

[OpenTelemetry]: https://opentelemetry.io
[Azure Monitor]: https://azure.microsoft.com/services/monitor/
[1]: https://www.youtube.com/watch?v=PAAkCSZUG1c&t=9m28s
[2]: https://martinfowler.com/bliki/TestCoverage.html