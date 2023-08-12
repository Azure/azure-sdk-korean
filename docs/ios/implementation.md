---
title: "iOS 가이드라인: 구현"
keywords: guidelines ios
permalink: ios_implementation.html
folder: ios
sidebar: general_sidebar
---

## API 구현

이 섹션에서는 Azure SDK 클라이언트 라이브러리를 구현하기 위한 지침을 설명합니다. 이러한 지침 중 일부는 코드 생성 도구에 의해 자동으로 적용됩니다.

{% include requirement/MUSTNOT id="ios-implementation" %} 구현 코드(즉, 공개 API의 일부를 구성하지 않은 코드)를 공개 API로 오인하지 않도록 하십시오. 구현 코드는 내부 또는 파일 전용으로 만들 수 있으며 소비 코드와 동일한 소스 파일 내에 배치할 수 있습니다.

### 서비스 클라이언트

> TODO: Add introductory sentence.

#### 서비스 메소드

> TODO: Add introductory sentence.

##### HTTP 파이프라인 사용

Azure SDK 팀은 구성이나 HTTP 요청 등의 횡단 관심사(cross cutting concerns)에 관한 일반적인 매커니즘이 포함된 `AzureCore` 라이브러리를 제공하고 있습니다.

{% include requirement/MUST id="ios-requests-use-pipeline" %} endpoints를 서비스하기 위해 `AzureCore` 내의 HTTP 파이프라인 컴포넌트를 사용하십시오.

HTTP pipeline은 여러 정책에 의해 래핑된 HTTP transport로 구성되어 있습니다. 각 정책은 파이프라인이 요청 및/또는 응답을 수정할 수 있는 제어 지점입니다. 우리는 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방법을 표준화하는 기본 정책 세트를 규정합니다. 리스트의 순서는 구현에 있어 가장 합리적인 순서입니다.

{% include requirement/MUST id="ios-requests-implement-policies" %} HTTP 파이프라인을 구성할 때 `AzureCore` 에 의해 제공되는 아래의 정책들을 포함하십시오:

- Telemetry
- Unique Request ID
- Retry
- Authentication
- Response downloader
- Logging

{% include requirement/SHOULD id="ios-requests-use-azurecore-impl" %} 가능한 한 `AzureCore` 의 정책 구현을 사용하십시오. 서비스의 고유한 것이 아니라면 정책을 "직접 작성하려고" 하지 마십시오. 기존 정책에 다른 옵션이 필요한 경우 [Architecture Board] 와 협력하여 옵션을 추가하십시오.

### 지원 타입

> TODO: Add introductory sentence.

#### 모델 타입

> TODO

## SDK 기능 구현

> TODO: Add introductory sentence.

### 구성

클라이언트 라이브러리를 구성할 때 클라이언트 라이브러리의 소비자가 전역적으로(소비자가 사용하는 다른 클라이언트 라이브러리와 함께) 특히 클라이언트 라이브러리와 함께 Azure 서비스에 대한 연결을 적절하게 구성할 수 있도록 특별한 주의를 기울여야 합니다.

#### 클라이언트 구성

{% include requirement/MUST id="ios-config-global-config" %} 기본적으로 또는 사용자가 명시적으로 요청한 경우(예: 구성 개체를 클라이언트 생성자에 전달) 관련 전역 구성 설정을 사용하세요.

{% include requirement/MUST id="ios-config-for-different-clients" %} 동일한 유형의 다른 클라이언트는 다른 구성을 사용하도록 하십시오.

{% include requirement/MUST id="ios-config-optout" %} 서비스 클라이언트의 소비자는 모든 글로벌 구성 설정을 한 번에 선택할 수 있도록 하십시오.

{% include requirement/MUST id="ios-config-global-overrides" %} 모든 글로벌 구성 설정을 클라이언트 제공 옵션에서 재정의할 수 있도록 하십시오. 이러한 옵션의 이름은 어떤 user-facing 글로벌 구성 키와도 일치해야 합니다.

{% include requirement/MUSTNOT id="general-config-behaviour-changes" %} 클라이언트를 구성한 후 발생하는 구성 변경 사항에 따라 동작을 변경하지 마십시오. 클라이언트의 계층은 명시적으로 변경되거나 재정의되지 않는 한 부모 클라이언트 구성을 상속합니다. 이 요구 사항에 대한 예외는 다음과 같습니다:

1. 로그 레벨은 Azure SDK에서 즉시 적용해야 합니다.
2. Telemetry on/off는 Azure SDK에서 즉시 적용해야 합니다.

> TODO: Update these guidelines to specify exactly how to do these things in Swift

#### 서비스별 구성

iOS 애플리케이션의 경우 `Info.plist`를 사용하여 구성을 적용하고 다른 언어로 작성된 서비스 내에서 환경 변수를 직접 반영하십시오.

{% include requirement/MUST id="ios-config-plist-key-prefix" %} Azure-specific plist 키인 `AZURE_`를 prefix로 사용하십시오.

{% include requirement/MAY id="ios-config-plist-key-use-client-specific" %} 클라이언트 라이브러리에 매개 변수로 제공되는 포털 구성 설정에는 `Info.plist`에 설정된 클라이언트 라이브러리별 목록 키를 사용할 수 있습니다. 여기에는 일반적으로 자격 증명 및 연결 세부 정보가 포함됩니다. 예를 들어 서비스 버스는 다음 환경 변수를 지원할 수 있습니다:

* `AZURE_SERVICEBUS_CONNECTION_STRING`
* `AZURE_SERVICEBUS_NAMESPACE`
* `AZURE_SERVICEBUS_ISSUER`
* `AZURE_SERVICEBUS_ACCESS_KEY`

스토리지는 다음과 같이 지원합니다:

* `AZURE_STORAGE_ACCOUNT`
* `AZURE_STORAGE_ACCESS_KEY`
* `AZURE_STORAGE_DNS_SUFFIX`
* `AZURE_STORAGE_CONNECTION_STRING`

{% include requirement/MUST id="ios-config-plist-key-get-approval" %} [Architecture Board]에서 모든 새로운 plist 키에 대한 승인을 얻어야 합니다. 환경 변수가 다른 언어 내에서 동일한 클라이언트 라이브러리에 대해 승인된 경우, 동일한 이름의 키가 iOS에 대해 승인됩니다.

{% include requirement/MUST id="ios-config-plist-key-format" %} 특정 Azure 서비스에 고유한 'Info.plist' 키에 대해 다음 구문을 사용하십시오:

* `AZURE_<ServiceName>_<ConfigurationKey>`

여기서 _ServiceName_ 은 공백이 없는 표준 단축 이름이며 _ConfigurationKey_ 는 해당 클라이언트 라이브러리에 포함되지 않은 않은 구성 키를 나타냅니다.

{% include requirement/MUSTNOT id="ios-config-plist-key-posix-compatible" %} 환경 변수 이름에는 밑줄을 제외하고 영숫자가 아닌 문자를 사용하지 마십시오. 이를 통해 광범위한 상호 운용성을 보장합니다.

### 로깅

클라이언트 라이브러리는 강력한 로깅 메커니즘을 지원하여 소비자가 메서드 호출의 문제를 적절하게 진단하고 문제가 소비자 코드, 클라이언트 라이브러리 코드 또는 서비스에 있는지 여부를 신속하게 확인할 수 있어야 합니다.

{% include requirement/MUST id="ios-logging-pluggable-logger" %} 착탈식(pluggable) 로그 핸들러를 지원하도록 하십시오. 이는 소비자가 클라이언트 옵션에 `logger` 매개변수를 지정할 수 있도록 제공되어야 합니다. `logger` 매개변수는 `AzureCoreLogger` 프로토콜의 인스턴스를 가리킵니다.

{% include requirement/MUST id="ios-logging-console-logger" %} 소비자가 콘솔에 대한 로그 출력을 쉽게 활성화할 수 있도록 하십시오. 이 작업은 `logger` 클라이언트 옵션을 `AzureCoreOSLogger`의 새 인스턴스로 설정함으로써 수행됩니다.

{% include requirement/MUST id="ios-logging-levels" %} 로그를 내보낼 때는 `Verbose`(상세 정보), `Informational`(일이 발생함), `Warning`(문제일 수도 있고 그렇지 않을 수도 있음), `Error`(오류) 중 하나를 사용하십시오. 이것은 열거형 `AzureCoreLogLevels`에 의해 제공됩니다.

{% include requirement/MUST id="ios-logging-failure" %} 응용 프로그램이 복구할 가능성이 없는 오류(메모리 부족 등)에 대해 `Error` 수준의 로깅을 사용하십시오.

{% include requirement/MUST id="ios-logging-warning" %} 함수가 의도한 작업을 수행하지 못할 경우 `Warning` 로깅 수준을 사용하십시오. 이는 일반적으로 함수가 예외를 발생시킨다는 것을 의미합니다. Self-healing events(예: 요청이 자동으로 재시도되는 경우)의 발생은 포함하지 마십시오.

{% include requirement/MUST id="ios-logging-info" %} 기능이 정상적으로 작동할 때 `Informational` 로깅 수준을 사용하십시오.

{% include requirement/MUST id="ios-logging-verbose" %} 자세한 문제 해결 시나리오에 대해 'Verbose' 로깅 수준을 사용하십시오. 이는 주로 개발자나 시스템 관리자가 특정 장애를 진단하기 위한 것입니다.

클라이언트 라이브러리는 다음과 같이 로그를 기록할 수 있습니다:

{% highlight swift %}
logger.writeLog(for: "MyClient", withLevel: AzureCoreLogLevels.Verbose, message: "A message")
{% endhighlight %}

{% include requirement/MUSTNOT id="ios-logging-no-sensitive-info" %} 중요한 정보를 `Verbose`가 아닌 로그 수준으로 보내지 마십시오. 예를 들어, 헤더를 기록할 때는 계정 키를 제거하십시오. 클라이언트 옵션에 지정된 `loggingAllowedHeaders` 배열과 'loggingAllowedQueryParams' 배열을 사용하여 헤더와 쿼리 매개 변수를 수정해야 합니다.

{% include requirement/MUST id="ios-logging-default-allowedlist" %} 클라이언트 옵션에서 `loggingAllowedHeaders` 및 `loggingAllowedQueryParams` 속성의 합리적인 기본값을 설정하십시오.

{% include requirement/MUST id="ios-logging-requests" %} 로그 요청 라인, 응답 라인 및 헤더를 `Informational` 메시지로 기록하십시오.

{% include requirement/MUST id="ios-logging-cancellations" %} 서비스 호출이 취소된 경우 `Informational` 메시지를 기록하십시오.

{% include requirement/MUST id="ios-logging-exceptions" %} 로그 예외가 `Warning` 수준의 메시지로 던져집니다. 로그 수준이 `Verbose`로 설정되어 있으면 메시지에 스택 추적 정보를 추가합니다.

> TODO: Logging (see general guidelines)
> * Provide abstracted logger in AzureCore
> * Use os_logger unless over-ridden
> * allowHeaders & allowQueryParams

> TBD:
> * Hook in to HockeyApp

### 분산 트레이스

분산 트레이스는 모바일컨텍스트에서는 거의 발생하지 않습니다. 분산 트레이스를 서포트할 필요가 있는 경우는, [Azure SDK 모바일 팀](mailto:azuresdkmobileteam@microsoft.com)에 문의해 주십시오.

### 테스팅

> TODO: Document how to write good tests with the existing XCTest framework.
> TODO: Say something about mocking of the requests and how to design for it.

{% include refs.md %}
{% include_relative refs.md %}
