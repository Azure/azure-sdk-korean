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
<!-- 검토 필요한 번역 사항
    - "across the wire" => 정말 "유선을 통해"라는 의미인가?
 -->

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

<!--
 -->
## Logging

Client libraries must support robust logging mechanisms so that the consumer can adequately diagnose issues with the method calls and quickly determine whether the issue is in the consumer code, client library code, or service.

In general, our advice to consumers of these libraries is to establish logging in their preferred manner at the `WARNING` level or above in production to capture problems with the application, and this level should be enough for customer support situations.  Informational or verbose logging can be enabled on a case-by-case basis to assist with issue resolution.

{% include requirement/MUST id="general-logging-pluggable-logger" %} support pluggable log handlers.

{% include requirement/MUST id="general-logging-console-logger" %} make it easy for a consumer to enable logging output to the console. The specific steps required to enable logging to the console must be documented. 

{% include requirement/MUST id="general-logging-levels" %} use one of the following log levels when emitting logs: `Verbose` (details), `Informational` (things happened), `Warning` (might be a problem or not), and `Error`.

{% include requirement/MUST id="general-logging-failure" %} use the `Error` logging level for failures that the application is unlikely to recover from (out of memory, etc.).

{% include requirement/MUST id="general-logging-warning" %} use the `Warning` logging level when a function fails to perform its intended task. This generally means that the function will raise an exception.  Do not include occurrences of self-healing events (for example, when a request will be automatically retried).

{% include requirement/MAY id="general-logging-slowlinks" %} log the request and response (see below) at the `Warning` when a request/response cycle (to the start of the response body) exceeds a service-defined threshold.  The threshold should be chosen to minimize false-positives and identify service issues.

{% include requirement/MUST id="general-logging-info" %} use the `Informational` logging level when a function operates normally.

{% include requirement/MUST id="general-logging-verbose" %} use the `Verbose` logging level for detailed troubleshooting scenarios. This is primarily intended for developers or system administrators to diagnose specific failures.

{% include requirement/MUST id="general-logging-no-sensitive-info" %} only log headers and query parameters that are in a service-provided "allow-list" of approved headers and query parameters.  All other headers and query parameters must have their values redacted.

{% include requirement/MUST id="general-logging-requests" %} log request line and headers as an `Informational` message. The log should include the following information:

* The HTTP method.
* The URL.
* The query parameters (redacted if not in the allow-list).
* The request headers (redacted if not in the allow-list).
* An SDK provided request ID for correlation purposes.
* The number of times this request has been attempted.

{% include requirement/MUST id="general-logging-responses" %} log response line and headers as an `Informational` message.  The format of the log should be the following:

* The SDK provided request ID (see above).
* The status code.
* Any message provided with the status code.
* The response headers (redacted if not in the allow-list).
* The time period between the first attempt of the request and the first byte of the body.

{% include requirement/MUST id="general-logging-cancellations" %} log an `Informational` message if a service call is cancelled.  The log should include:

* The SDK provided request ID (see above).
* The reason for the cancellation (if available).

{% include requirement/MUST id="general-logging-exceptions" %} log exceptions thrown as a `Warning` level message. If the log level set to `Verbose`, append stack trace information to the message.

## Distributed Tracing

Distributed tracing mechanisms allow the consumer to trace their code from frontend to backend. The distributed tracing library creates spans - units of unique work.  Each span is in a parent-child relationship.  As you go deeper into the hierarchy of code, you create more spans.  These spans can then be exported to a suitable receiver as needed.  To keep track of the spans, a _distributed tracing context_ (called a context in the remainder of this section) is passed into each successive layer.  For more information on this topic, visit the [OpenTelemetry] topic on tracing.

{% include requirement/MUST id="general-tracing-opentelemetry" %} support [OpenTelemetry] for distributed tracing.

{% include requirement/MUST id="general-tracing-accept-context" %} accept a context from calling code to establish a parent span.

{% include requirement/MUST id="general-tracing-pass-context" %} pass the context to the backend service through the appropriate headers (`traceparent`, `tracestate`, etc.) to support [Azure Monitor].  This is generally done with the HTTP pipeline.

{% include requirement/MUST id="general-tracing-new-span-per-method" %} create a new span for each method that user code calls.  New spans must be children of the context that was passed in.  If no context was passed in, a new root span must be created.

{% include requirement/MUST id="general-tracing-new-span-per-rest-call" %} create a new span (which must be a child of the per-method span) for each REST call that the client library makes.  This is generally done with the HTTP pipeline.

Some of these requirements will be handled by the HTTP pipeline.  However, as a client library writer, you must handle the incoming context appropriately.

## Dependencies

Dependencies bring in many considerations that are often easily avoided by avoiding the 
dependency. 

- **Versioning** - Many programming languages do not allow a consumer to load multiple versions of the same package. So, if we have an client library that requires v3 of package Foo and the consumer wants to use v5 of package Foo, then the consumer cannot build their application. This means that client libraries should not have dependencies by default. 
- **Size** - Consumer applications must be able to deploy as fast as possible into the cloud and move in various ways across networks. Removing additional code (like dependencies) improves deployment performance.
- **Licensing** - You must be conscious of the licensing restrictions of a dependency and often provide proper attribution and notices when using them.
- **Compatibility** - Often times you do not control a dependency and it may choose to evolve in a direction that is incompatible with your original use.
- **Security** - If a security vulnerability is discovered in a dependency, it may be difficult or time consuming to get the vulnerability corrected if Microsoft does not control the dependency's code base.

{% include requirement/MUST id="general-dependencies-azure-core" %} depend on the Azure Core library for functionality that is common across all client libraries.  This library includes APIs for HTTP connectivity, global configuration, and credential handling.

{% include requirement/MUSTNOT id="general-dependencies-approved-only" %} be dependent on any other packages within the client library distribution package. Dependencies are by-exception and need a thorough vetting through architecture review.  This does not apply to build dependencies, which are acceptable and commonly used.

{% include requirement/SHOULD id="general-dependencies-vendoring" %} consider copying or linking required code into the client library in order to avoid taking a dependency on another package that could conflict with the ecosystem. Make sure that you are not violating any licensing agreements and consider the maintenance that will be required of the duplicated code. ["A little copying is better than a little dependency"][1] (YouTube).

{% include requirement/MUSTNOT id="general-dependencies-concrete" %} depend on concrete logging, dependency injection, or configuration technologies (except as implemented in the Azure Core library).  The client library will be used in applications that might be using the logging, DI, and configuration technologies of their choice.

Language specific guidelines will maintain a list of approved dependencies.

## Service-specific common library code

There are occasions when common code needs to be shared between several client libraries.  For example, a set of cooperating client libraries may wish to share a set of exceptions or models.

{% include requirement/MUST id="general-commonlib-approval" %} gain [Architecture Board] approval prior to implementing a common library.

{% include requirement/MUST id="general-commonlib-minimize-code" %} minimize the code within a common library.  Code within the common library is available to the consumer of the client library and shared by multiple client libraries within the same namespace.

{% include requirement/MUST id="general-commonlib-namespace" %} store the common library in the same namespace as the associated client libraries.

A common library will only be approved if:

* The consumer of the non-shared library will consume the objects within the common library directly, AND
* The information will be shared between multiple client libraries.

Let's take two examples:

1. Implementing two Cognitive Services client libraries, we find a model is required that is produced by one Cognitive Services client library and consumed by another Coginitive Services client library, or the same model is produced by two client libraries.  The consumer is required to do the passing of the model in their code, or may need to compare the model produced by one client library vs. that produced by another client library.  This is a good candidate for choosing a common library.

2. Two Cognitive Services client libraries throw an `ObjectNotFound` exception to indicate that an object was not detected in an image.  The user might trap the exception, but otherwise will not operate on the exception.  There is no linkage between the `ObjectNotFound` exception in each client library.  This is not a good candidate for creation of a common library (although you may wish to place this exception in a common library if one exists for the namespace already).  Instead, produce two different exceptions - one in each client library.

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