---
title: "공통 가이드라인: 구현"
keywords: guidelines
permalink: general_implementation.html
folder: general
sidebar: general_sidebar
---

허용 가능한 API 표면을 통해 작업을 한 뒤부터는, 서비스 클라이언트의 구현을 시작하실 수 있습니다.

## 구성

당신의 클라이언트 라이브러리를 구성할 때는, 해당 클라이언트 라이브러리를 사용하는 사용자가 Azure 서비스에 대한 연결을 (그 사용자가 사용하고 있는 다른 클라이언트 라이브러리와) 전반적으로(globally) 그리고 당신의 클라이언트 라이브러리와 특정적으로(specifically) 모두 적절하게 구성할 수 있도록 각별히 주의를 기울여야 합니다.

### 클라이언트 구성

{% include requirement/MUST id="general-config-global-config" %} 관련 글로벌 구성 설정은 기본(default)으로 사용하거나 사용자에 의해 명시적으로 요청된 경우(예: 클라이언트 생성자에 구성 객체를 전달하는 등)에 사용하세요.
<!--
use relevant global configuration settings either by default or when explicitly requested to by the user, for example by passing in a configuration object to a client constructor.
-->

{% include requirement/MUST id="general-config-for-different-clients" %} 동일 유형의 서로 다른 클라이언트들이 서로 다른 구성을 사용할 수 있도록 허용하세요.

{% include requirement/MUST id="general-config-optout" %} 
서비스 클라이언트의 소비자가 모든 글로벌 구성 설정을 한 번에 취소(opt out)할 수 있도록 허용하세요.
<!--
allow consumers of your service clients to opt out of all global configuration settings at once.
-->

{% include requirement/MUST id="general-config-global-overrides" %} 모든 글로벌 구성 설정을 클라이언트 제공 옵션으로 재정의할 수 있도록 허용하세요. 이러한 옵션의 이름들은 사용자에게 보이는 글로벌 구성 키들과 일치해야 합니다.
<!--
allow all global configuration settings to be overridden by client-provided options. The names of these options should align with any user-facing global configuration keys.
-->

{% include requirement/MUSTNOT id="general-config-behaviour-changes" %} 클라이언트가 구성된 후에 발생하는 구성 변경사항에 따라 동작을 변경하지 마세요. 클라이언트의 계층 구조는 명시적으로 변경되거나 재정의되지 않는 한 부모 클라이언트의 구성을 상속합니다. 이 요구 사항에 대한 예외는 다음과 같습니다:

1. 로그 수준 (Log level): Azure SDK 전체에 즉시 적용되어야 합니다.
2. 추적 끄기/켜기 (Tracing on/off): Azure SDK 전체에 즉시 적용되어야 합니다.
<!--
change behavior based on configuration changes that occur after the client is constructed. Hierarchies of clients inherit parent client configuration unless explicitly changed or overridden. Exceptions to this requirement are as follows:

1. Log level, which must take effect immediately across the Azure SDK.
2. Tracing on/off, which must take effect immediately across the Azure SDK.
-->

### 서비스별 환경 변수

{% include requirement/MUST id="general-config-envvars-prefix" %} Azure별 환경 변수의 접두사를 `AZURE_`로 지정하세요.

{% include requirement/MAY id="general-config-envvars-use-client-specific" %} 당신의 클라이언트 라이브러리별 환경 변수를 클라이언트 라이브러리에 매개변수로 제공되는 포털 구성 설정에 사용할 수 있습니다. 클라이언트 라이브러리별 환경 변수는 일반적으로 자격 증명 및 연결 세부 정보를 포함합니다. 예를 들어, 서비스 버스(Service Bus)는 다음과 같은 환경 변수를 지원할 수 있습니다:
<!--
YOU MAY use client library-specific environment variables for portal-configured settings which are provided as parameters to your client library. This generally includes credentials and connection details. For example, Service Bus could support the following environment variables:
-->

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
<!--
get approval from the [Architecture Board] for every new environment variable.
 -->

{% include requirement/MUST id="general-config-envvars-format" %} 특정 Azure 서비스에 특정한 환경 변수들에 대해 이 구문을 사용하세요:
<!--
use this syntax for environment variables specific to a particular Azure service:
 -->

* `AZURE_<ServiceName>_<ConfigurationKey>`

여기서 _ServiceName_은 공백이 없는 표준 단축명이고, _ConfigurationKey_는 해당 클라이언트 라이브러리에 대한 중첩되지 않은(unnested) 구성 키를 참조합니다.
<!--
where _ServiceName_ is the canonical shortname without spaces, and _ConfigurationKey_ refers to an unnested configuration key for that client library.
 -->

{% include requirement/MUSTNOT id="general-config-envvars-posix-compatible" %} 환경 변수 이름에는 밑줄을 제외하고 영숫자가 아닌 문자를 사용하지 마세요. 이는 광범위한 상호 운용성을 보장합니다.
<!--
use non-alpha-numeric characters in your environment variable names with the exception of underscore. This ensures broad interoperability.
 -->

## 매개변수 유효성 검사
<!--
Parameter validation 
 -->

서비스 클라이언트에는 서비스에 대한 요청을 수행하는 여러 방법들이 있을 것입니다. _서비스 매개변수_들은 유선을 통해 Azure 서비스로 직접 전달됩니다. _클라이언트 매개변수_는 서비스로 직접 전달되지 않고, 클라이언트 라이브러리 내에서 요청을 이행하는 데 사용됩니다. 클라이언트 매개변수의 예로는 URI를 구성하는 데 사용되는 값이나, 스토리지에 업로드해야 하는 파일이 있습니다.
<!-- 검토 필요한 번역 사항
    - "across the wire" => 정말 "유선을 통해"라는 의미인가?
 -->
<!--
The service client will have several methods that perform requests on the service.  _Service parameters_ are directly passed across the wire to an Azure service.  _Client parameters_ are not passed directly to the service, but used within the client library to fulfill the request.  Examples of client parameters include values that are used to construct a URI, or a file that needs to be uploaded to storage.
 -->

{% include requirement/MUST id="general-params-client-validation" %} 클라이언트 매개변수의 유효성을 검사하세요. 여기에는 필수 경로 매개변수에 대한 null 값 검사, 그리고 필수 경로 매개변수가 0보다 큰 `minLength`를 선언한 경우 빈 문자열 값 검사 등이 포함됩니다.
 <!--
validate client parameters. This includes checks for null values for required path parameters, and checks for empty string values if a required path parameter declares a `minLength` greater than zero.
 -->

{% include requirement/MUSTNOT id="general-params-server-validation" %} 서비스 매개변수의 유효성을 검사하지 마세요. 여기에는 널 검사, 빈 문자열, 그리고 기타 일반적인 유효성 검사 조건들이 포함됩니다. 서비스에서 모든 요청 매개변수들의 유효성을 검사하도록 합니다.
 <!--
validate service parameters. This includes null checks, empty strings, and other common validating conditions. Let the service validate any request parameters.
 -->

{% include requirement/MUST id="general-params-check-devex" %} 서비스 매개변수가 유효하지 않은 경우 개발자 경험의 유효성을 검사하여 서비스에서 적절한 오류 메시지가 생성되는지 확인하세요. 만약 서비스 측 오류 메시지로 인해 개발자 경험이 손상된 경우 서비스 팀과 협력하여 릴리스 전에 수정하세요.
 <!--
 validate the developer experience when the service parameters are invalid to ensure appropriate error messages are generated by the service. If the developer experience is compromised due to service-side error messages, work with the service team to correct prior to release.
 -->

## 네트워크 요청
 <!--
 Network requests
 -->

Each supported language has an Azure Core library that contains common mechanisms for cross cutting concerns such as configuration and doing HTTP requests.

{% include requirement/MUST id="general-requests-use-pipeline" %} use the HTTP pipeline component within Azure Core for communicating to service REST endpoints.

The HTTP pipeline consists of a HTTP transport that is wrapped by multiple policies. Each policy is a control point during which the pipeline can modify either the request and/or response. We prescribe a default set of policies to standardize how client libraries interact with Azure services.  The order in the list is the most sensible order for implementation.

{% include requirement/MUST id="general-requests-implement-policies" %} implement the following policies in the HTTP pipeline:

- Telemetry
- Unique Request ID
- Retry
- Authentication
- Response downloader
- Distributed tracing
- Logging

{% include requirement/SHOULD id="general-requests-use-azurecore-impl" %} use the policy implementations in Azure Core whenever possible.  Do not try to "write your own" policy unless it is doing something unique to your service.  If you need another option to an existing policy, engage with the [Architecture Board] to add the option.

## Authentication

When implementing authentication, don't open up the consumer to security holes like PII (personally identifiable information) leakage or credential leakage.  Credentials are generally issued with a time limit, and must be refreshed periodically to ensure that the service connection continues to function as expected.  Ensure your client library follows all current security recommendations and consider an independent security review of the client library to ensure you're not introducing potential security problems for the consumer.

{% include requirement/MUSTNOT id="general-authimpl-no-persisting" %} persist, cache, or reuse security credentials.  Security credentials should be considered short lived to cover both security concerns and credential refresh situations.  

If your service implements a non-standard credential system (that is, a credential system that is not supported by Azure Core), then you need to produce an authentication policy for the HTTP pipeline that can authenticate requests given the alternative credential types provided by the client library.

{% include requirement/MUST id="general-authimpl-provide-auth-policy" %} provide a suitable authentication policy that authenticates the HTTP request in the HTTP pipeline when using non-standard credentials.  This includes custom connection strings, if supported.

## Native code

Some languages support the development of platform-specific native code plugins.  These cause compatibility issues and require additional scrutiny.  Certain languages compile to a machine-native format (for example, C or C++), whereas most modern languages opt to compile to an intermediary format to aid in cross-platform support.

{% include requirement/SHOULD id="general-no-nativecode" %} write platform-specific / native code unless the language compiles to a machine-native format.

## Error handling

Error handling is an important aspect of implementing a client library.  It is the primary method by which problems are communicated to the consumer.  There are two methods by which errors are reported to the consumer.  Either the method throws an exception, or the method returns an error code (or value) as its return value, which the consumer must then check.  In this section we refer to "producing an error" to mean returning an error value or throwing an exception, and "an error" to be the error value or exception object.  

{% include requirement/SHOULD id="general-errors-prefer-exceptions" %} prefer the use of exceptions over returning an error value when producing an error.

{% include requirement/MUST id="general-errors-for-failed-requests" %} produce an error when any HTTP request fails with an HTTP status code that is not defined by the service/Swagger as a successful status code. These errors should also be logged as errors.

{% include requirement/MUST id="general-errors-include-request-response" %} ensure that the error produced contains the HTTP response (including status code and headers) and originating request (including URL, query parameters, and headers).  

In the case of a higher-level method that produces multiple HTTP requests, either the last exception or an aggregate exception of all failures should be produced.

{% include requirement/MUST id="general-errors-rich-info" %} ensure that if the service returns rich error information (via the response headers or body), the rich information must be available via the error produced in service-specific properties/fields.

{% include requirement/SHOULDNOT id="general-errors-no-new-types" %} create a new error type unless the developer can perform an alternate action to remediate the error.  Specialized error types should be based on existing error types present in the Azure Core package.

{% include requirement/MUSTNOT id="general-errors-use-system-types" %} create a new error type when a language-specific error type will suffice.  Use system-provided error types for validation.

{% include requirement/MUST id="general-errors-documentation" %} document the errors that are produced by each method (with the exception of commonly thrown errors that are generally not documented in the target language).

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