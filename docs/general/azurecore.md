---
title: "공통 가이드라인: Azure Core"
keywords: guidelines
permalink: general_azurecore.html
folder: general
sidebar: general_sidebar
---

Azure Core 라이브러리는 다른 클라이언트 라이브러리에 교차 서비스를 제공합니다. 이러한 서비스는 다음과 같습니다:

* HTTP 파이프라인
* 전역 구성 
* 자격 증명 

다음 절에서는 Azure Core 라이브러리에 대한 요구 사항을 정의합니다. Azure Core 라이브러리를 이미 갖고 있는 언어로 클라이언트 라이브러리를 구현하는 경우, 이 섹션을 읽을 필요가 없습니다. 주로 Azure Core 라이브러리에서 일하는 개발자를 대상으로 합니다.

## HTTP 파이프라인

HTTP 파이프라인은 여러 정책으로 구성된 HTTP 전송으로 구성됩니다. 각 정책은 파이프라인이 요청 그리고/또는 응답을 수정할 수 있는 제어 지점입니다. 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방식을 표준화하기 위해 기본 정책 집합을 사용합니다.

- 원격 분석
- 고유한 요청 ID
- 재시도
- 인증
- 응답 다운로더 
- 분산 추적
- 로깅
- 프록시

일반적으로 클라이언트 라이브러리는 이러한 정책만 구성하면 됩니다. 그러나 (새로운 언어를 위한) 새로운 Azure Core 라이브러리를 생성하는 경우, 각 정책에 대한 요구 사항을 이해해야 합니다.

### 원격 분석 정책 

클라이언트 라이브러리 사용 원격 분석은 (소비자가 아닌) 서비스 팀에서 클라이언트가 서비스에 호출하는 데 사용하는 SDK 언어, 클라이언트 라이브러리 및 언어/플랫폼 정보를 모니터링하는 데 사용됩니다. 클라이언트는 클라이언트 애플리케이션의 이름과 버전을 나타내는 추가 정보를 앞에 추가할 수 있습니다.

{% include requirement/MUST id="azurecore-http-telemetry-useragent" %} 다음 형식을 사용하여 [User-Agent header]로 원격 측정 정보를 전송하십시오:

```
[<application_id> ]azsdk-<sdk_language>-<package_name>/<package_version> <platform_info>
```

- `<application_id>`: 선택적 애플리케이션별 문자열입니다. 슬래시를 포함할 수 있지만, 공백을 포함해서는 안 됩니다. 문자열은 클라이언트 라이브러리의 사용자가 제공합니다. 예시: "AzCopy/10.0.4-Preview"
- `<sdk_language>`: SDK 언어 이름 (모두 소문자): "net", "python", "java", 또는 "js"
- `<package_name>`: 슬래시를 대시로 바꾸고 Azure 표시기를 제거하여 개발자에게 표시되는 클라이언트 라이브러리 패키지 이름. 예를 들어, "Security.KeyVault" (.NET), "security.keyvault" (Java), "keyvault" (JavaScript & Python)
- `<package_version>`: 패키지의 버전. 참고: 서비스 버전이 아닙니다.
- `<platform_info>`: 현재 실행 중인 언어 런타임 및 OS에 대한 정보, 예시: "(NODE-VERSION v4.5.0; Windows_NT 10.0.14393)"

예를 들어, Azure Blob Storage 클라이언트 라이브러리를 사용하여 각 언어로 'AzCopy'를 다시 작성하면, 다음과 같은 사용자-에이전트 문자열이 발생할 수 있습니다.

- (.NET) `AzCopy/10.0.4-Preview azsdk-net-Storage.Blobs/11.0.0 (.NET Standard 2.0; Windows_NT 10.0.14393)`
- (JavaScript) `AzCopy/10.0.4-Preview azsdk-js-storage-blob/11.0.0 (Node 10.16.0; Ubuntu; Linux x86_64; rv:34.0)`
- (Java) `AzCopy/10.0.4-Preview azsdk-java-storage.blobs/11.0.0 (Java/1.8.0_45; Macintosh; Intel Mac OS X 10_10; rv:33.0)`
- (Python) `AzCopy/10.0.4-Preview azsdk-python-storage/4.0.0 Python/3.7.3 (Ubuntu; Linux x86_64; rv:34.0)`

라이브러리 소비자가 애플리케이션 ID를 설정할 수 있도록 허용하십시오. 이를 통해 소비자는 앱에 대한 서비스 간 원격 분석을 얻을 수 있습니다. 응용 프로그램 ID는 관련 `ClientOptions` 개체에서 설정할 수 있어야 합니다.

{% include requirement/MUST id="azurecore-http-telemetry-appid-length" %} 애플리케이션 ID가 길이는 24자를 넘지 않도록 강제하십시오. 애플리케이션 ID가 짧을수록 서비스 팀은 사용자 에이전트의 "플랫폼 정보" 섹션에 진단 정보를 포함시킬 수 있으며, 소비자는 여전히 자신의 애플리케이션에 대한 원격 분석 정보를 얻을 수 있습니다.

{% include requirement/SHOULD id="azurecore-http-telemetry-x-ms-useragent" %} 플랫폼이 `User-Agent` 헤더 변경을 지원하지 않을 경우, 일반적으로 `User-Agent` 헤더로 전송하는 원격 분석 정보를 `X-MS-UserAgent` 헤더에 보내야 합니다. 서비스는 일반적인 분석 시스템을 통해 쿼리할 수 있는 방식으로 `X-MS-UserAgent` 헤더를 캡처하도록 로그 수집을 구성해야 합니다.

{% include requirement/SHOULD id="azurecore-http-telemetry-dynamic" %} 추가적인 (동적) 원격 분석 정보는 `X-MS-AZSDK-Telemetry` 헤더에서 키-값 유형의 세미콜론으로 구분된 집합으로 전송해야 합니다. 예를 들어:

```http
X-MS-AZSDK-Telemetry: class=BlobClient;method=DownloadFile;blobType=Block
```

헤더의 내용은 세미콜론 키=값 목록입니다. 다음 키는 특정한 의미를 갖습니다:

* `class`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 내 타입의 이름입니다.
* `method`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 내 메서드의 이름입니다.

사용되는 다른 모든 키는 특정 서비스의 모든 클라이언트 라이브러리에서 공통적이어야 합니다.  이 헤더에 (인코딩된 경우에도) 개인 식별 정보를 **포함하지 마십시오**. 서비스는 일반적인 분석 시스템을 통해 쿼리할 수 있는 방식으로 `X-MS-SDK-Telemetry` 헤더를 캡처하도록 로그 수집을 구성해야 합니다.

### 고유한 요청 ID 정책

> **TODO** Add Unique Request ID Policy Requirements

### 재시도 정책

클라이언트 애플리케이션이 서비스에 네트워크 요청을 보내려고 할 때 오류가 발생하는 데에는 여러 이유가 있습니다. 몇 가지 예로는 시간 초과, 네트워크 인프라 오류, 제한/사용 중으로 인한 서비스 거부, 스케일 다운으로 인한 서비스 인스턴스 종료, 다른 버전으로 교체하기 위해 다운되는 서비스 인스턴스, 처리되지 않은 예외로 인한 서비스 충돌 등이 있습니다. 기본 제공 재시도 메커니즘(소비자가 재정의할 수 있는 기본 구성)을 제공함으로써 SDK와 소비자 애플리케이션은 이러한 장애에 탄력적으로 대처할 수 있습니다. 일부 서비스는 각 시도에 대해 실제 비용을 청구하므로 소비자가 복원력보다 비용 절감을 선호하는 경우 재시도를 완전히 비활성화할 수 있어야 합니다.

더 많은 정보는 다음을 참고하십시오: [Transient fault handling]

HTTP 파이프라인은 이 기능을 제공합니다.

{% include requirement/MUST id="azurecore-http-retry-options" %} 다음 구성 설정을 제공하십시오:

- 재시도 정책 유형 (지수 또는 고정)
- 최대 재시도 횟수
- 재시도 간 지연(시간 간격/기간; 고정 정책의 경우 이 양만큼 지연, 지수 정책의 경우 지연이 이 양만큼 기하급수적으로 증가함)
- 최대 재시도 지연(시간 간격/기간)
- 재시도를 유발하는 HTTP 상태 코드 목록(기본값은 "재시도 가능한 모든 서비스 오류")

{% include requirement/MAY id="azurecore-http-retry-mechanisms" %} 서비스에서 지원하는 경우 추가 재시도 메커니즘을 제공할 수 있습니다. 예를 들어 Azure Storage Blob 서비스는 보조 데이터 센터에 대한 읽기 작업 재시도를 지원하거나, 복원력을 위해 시도당 제한 시간 사용을 권장합니다.

{% include requirement/MUST id="azurecore-http-retry-reset-data-stream" %} 요청을 재시도하기 전에 모든 요청 데이터 스트림을 재설정(또는 위치 0으로 다시 탐색)하십시오.

{% include requirement/MUST id="azurecore-http-retry-honor-cancellation" %} 재시도가 실행되기 전에 요청을 종료할 수 있는 호출자에게 전달된 취소 메커니즘을 준수하십시오.

{% include requirement/MUST id="azurecore-http-retry-update-queryparams" %} 서비스가 개별 요청 시도를 처리해야 하는 데 걸리는 시간을 알려주는 서비스로 전송되는 쿼리 매개변수 또는 요청 헤더를 업데이트하십시오.

{% include requirement/MUST id="azurecore-http-retry-hardware-failure" %} 하드웨어 네트워크 오류의 경우 자체 수정될 수 있으므로 다시 시도하십시오.

{% include requirement/MUST id="azurecore-http-retry-service-not-found" %} "서비스를 찾을 수 없음" 오류가 발생한 경우 다시 시도하십시오. 서비스가 다시 온라인 상태가 되거나 로드 밸런서가 자체적으로 재구성 중일 수 있습니다.

{% include requirement/MUST id="azurecore-http-retry-throttling" %} 서비스가 요청을 제한하고 있음을 나타내는 응답을 성공적으로 수행할 경우(예시, "x-ms-delay-until" 헤더 또는 유사한 메타데이터) 재시도하십시오.

{% include requirement/MUSTNOT id="azurecore-http-retry-after" %} 서비스가 400 수준 응답 코드로 응답하는 경우, Retry After 헤더가 반환되지 않는 한 재시도하지 마십시오.

{% include requirement/MUSTNOT id="azurecore-http-retry-requestid" %} 요청을 재시도할 때 클라이언트 측에서 생성한 요청 ID를 변경하지 마십시오. 요청 ID는 논리적 작업을 나타내며, 이 작업의 모든 물리적 재시도 동안 동일해야 합니다.  서버 로그를 볼 때, 동일한 클라이언트 요청 ID를 가진 여러 항목이 각 재시도를 표시합니다.

{% include requirement/SHOULD id="azurecore-http-retry-defaults" %} 지수(지터 포함) 백오프와 함께 0.8초 지연으로 3회 재시도에서 시작하는 기본 정책을 구현해야 합니다.

### Authentication policy

Services across Azure use a variety of different authentication schemes to authenticate clients. Conceptually there are two entities responsible for authenticating service client requests, a credential and an authentication policy.  Credentials provide confidential authentication data needed to authenticate requests.  Authentication policies use the data provided by a credential to authenticate requests to the service. It is essential that credential data can be updated as needed across the lifetime of a client, and authentication policies must always use the most current credential data.

{% include requirement/MUST id="azurecore-http-auth-bearer-token" %} implement Bearer authorization policy (which accepts a token credential and
scope).

### Response downloader policy

The response downloader is required for most (but not all) operations to change whatever is returned by the service into a model that the consumer code can use.  An example of a method that does not deserialize the response payload is a method that downloads a raw blob within the Blob Storage client library.  In this case, the raw data bytes are required.  For most operations, the body must be downloaded in totality before deserialization. This pipeline policy must implement the following requirements:

{% include requirement/MUST id="azurecore-http-response-body" %} download the entire response body and pass the complete downloaded body up to the operation method for methods that deserialize the response payload.  If a network connection fails while reading the body, the retry policy must automatically retry the operation.

### Distributed tracing policy

Distributed tracing allows the consumer to trace their code from frontend to backend.  The distributed tracing library creates spans (units of unique work) to facilitate tracing.  Each span is in a parent-child relationship.  As you go deeper into the hierarchy of code, you create more spans.  These spans can then be exported to a suitable receiver as needed.  To keep track of the spans, a _distributed tracing context_ (called a context within the rest of this section) is passed into each successive layer.  For more information on this topic, visit the [OpenTelemetry]topic on tracing.

The Distributed Tracing policy is responsible for:

* Creating a SPAN per REST call.
* Passing the context to the backend service.

> There is also a distributed tracing topic for implementing a client library.

{% include requirement/MUST id="azurecore-http-tracing-opentelemetry" %} support [OpenTelemetry] for distributed tracing.

{% include requirement/MUST id="azurecore-http-tracing-accept-context" %} accept a context from calling code to establish a parent span.

{% include requirement/MUST id="azurecore-http-tracing-pass-context" %} pass the context to the backend service through the appropriate headers (`traceparent`, `tracestate`, etc.) to support [Azure Monitor].

{% include requirement/MUST id="azurecore-http-tracing-create-span" %} create a new span (which must be a child of the per-method span) for each REST call that the client library makes.

### Logging policy

Many logging requirements within Azure Core mirror the same requirements for logging within the client library.

{% include requirement/MUST id="azurecore-http-logging-handlers" %} allow the client library to set the log handler and log settings.

{% include requirement/MUST id="azurecore-http-logging-levels" %} use one of the following log levels when emitting logs: `Verbose` (details), `Informational` (things happened), `Warning` (might be a problem or not), and `Error`.

{% include requirement/MUST id="azurecore-http-logging-error" %} use the `Error` logging level for failures that the application is unlikely to recover from (out of memory, etc.).

{% include requirement/MUST id="azurecore-http-logging-warning" %} use the `Warning` logging level when a function fails to perform its intended task. This generally means that the function will raise an exception. Don't include occurrences of self-healing events (for example, when a request will be automatically retried).

{% include requirement/MUST id="azurecore-http-logging-info" %} use the `Informational` logging level when a function operates normally.

{% include requirement/MUST id="azurecore-http-logging-verbose" %} use the `Verbose` logging level for detailed troubleshooting scenarios. This is primarily intended for developers or system administrators to diagnose specific failures.

{% include requirement/MUSTNOT id="azurecore-http-logging-sensitive-info" %} send sensitive information in log levels other than `Verbose`. For example, remove account keys when logging headers.

{% include requirement/MUST id="azurecore-http-logging-request-line" %} log request line, response line, and headers as an `Informational` message.

{% include requirement/MUST id="azurecore-http-logging-cancellation" %} log an `Informational` message if a service call is cancelled.

{% include requirement/MUST id="azurecore-http-logging-exceptions" %} log exceptions thrown as a `Warning` level message. If the log level set to `Verbose`, append stack trace information to the message.

{% include requirement/MUST id="azurecore-http-logging-configuration" %} include information about relevant configuration when the HTTP pipeline throws an error if there is a relevant configuration setting that would alleviate the problem.  Not all errors can be fixed with a configuration change.

### Proxy policy

Apps that integrate the Azure SDK need to operate in common enterprises.  It is a common practice to implement HTTP proxies for control and caching purposes.  Proxies are generally configured at the machine level and, as such, are part of the environment.  However, there are reasons to adjust proxies (for example, testing may use a proxy to rewrite URLs to a test environment instead of a production environment).  The Azure SDK and all client libraries should operate in those environments.

There are a number of common methods for proxy configuration.  However, they fall into four groups:

1. Inline, no authentication (filtering only)
2. Inline, with authentication
3. Out-of-band, no authentication
4. Out of band, with authentication

For inline/no-auth proxy, nothing needs to be done.  The Azure SDK will work without any proxy configuration.  For inline/auth proxy, the connection may receive a `407 Proxy Authentication Required` status code.  This will include a scheme, realm, and potentially other information (such as a `nonce` for digest authentication).  The client library must resubmit the request with a `Proxy-Authorization` header that provides authentication information suitably encoded for the scheme.  The most common schemes are Basic, Digest, and NTLM.

For an out-of-band/no-auth proxy, the client will send the entire request URL to the proxy instead of the service.  For example, if the client is communicating to `https://foo.blob.storage.azure.net/path/to/blob`, it will connect to the `HTTPS_PROXY` and send a `GET https://foo.blob.storage.azure.net/path/to/blob HTTP/1.1`.   For an out-of-band/auth proxy, the client will send the entire request URL just as in the out-of-band/no-auth proxy version, but it may send back a `407 Proxy Authentication Required` status code (as with the inline/auth proxy).

WebSockets can normally be tunneled through an HTTP proxy, in which case the proxy authentication happens during the CONNECT call.  This is the preferred mechanism for tunneling non-HTTP traffic to the Azure service.  However, there are other types of proxies.  The most notable is the SOCKS proxy used for non-HTTP traffic (such as AMQP or MQTT).  We make no recommendation (for or against) support of SOCKS.  It is explicitly not a requirement to support SOCKS proxy within the client library.

Most proxy configuration will be done by adopting the HTTP pipeline that is common to all Azure service client libraries.

{% include requirement/MUST id="azurecore-http-proxy-global-config" %} support proxy configuration via common global configuration directives configured on a platform or runtime basis.

- Linux and macOS generally use the `HTTPS_PROXY` (and associated) environment variables to configure proxies.
- Windows environments generally use the WinHTTP proxy configuration to configure proxies.
- Other options (such as loading from configuration files) may exist on a platform and runtime basis.

{% include requirement/MUST id="azurecore-http-proxy-azure-sdk-config" %} support Azure SDK-wide configuration directives for proxy configuration, including disabling the proxy functionality.

{% include requirement/MUST id="azurecore-http-proxy-client-config" %} support client library-specific configuration directives for proxy configuration, including disabling the proxy functionality.

{% include requirement/MUST id="azurecore-http-proxy-logging" %} log `407 Proxy Authentication Required` requests and responses.

{% include requirement/MUST id="azurecore-http-proxy-always-log" %} indicate in logging if the request is being sent to the service via a proxy, even if proxy authentication is not required.

{% include requirement/MUST id="azurecore-http-proxy-support" %} support Basic and Digest authentication schemes.

{% include requirement/SHOULD id="azurecore-http-proxy-ntlm-support" %} support the NTLM authentication scheme.

There is no requirement to support SOCKS at this time.  We recommend services adopt a WebSocket connectivity option (for example, AMQP or MQTT over WebSockets) to ensure compatibility with proxies.

## Global configuration

The Azure SDK can be configured by a variety of sources, some of which are necessarily language-dependent.  This will generally be codified in the Azure Core library.  The configuration sources include:

1. System settings
2. Environment variables
3. Global configuration store (code)
4. Runtime parameters

{% include requirement/MUST id="azurecore-config-ordering" %} apply configuration in the order above by default, such that subsequent items in the list override settings from previous items in the list.

{% include requirement/MAY id="azurecore-config-opt-in" %} support configuration systems that users opt in to that do not follow the above ordering.

{% include requirement/MUST id="azurecore-config-consistent-naming" %} be consistent with naming between environment variables and configuration keys.

{% include requirement/MUST id="azurecore-config-log-config-settings" %} log when a configuration setting is found somewhere in the environment or global configuration store.

{% include requirement/MAY id="azurecore-config-ignore-irrelevant-config" %} ignore configuration settings that are irrelevant for your client library.

### System settings

{% include requirement/SHOULD id="azurecore-config-respect-the-system" %} respect system settings for proxies.

### Environment variables

Environment variables are a well-known method for IT administrators to configure basic settings when running code in the cloud.

{% include requirement/MUST id="azurecore-config-envvars-system-list" %}  load relevant configuration settings from the environment variables listed in the table below:

{% include tables/environment_variables.md %}

{% include requirement/MUST id="azurecore-config-envvars-azure-prefix" %} prefix Azure-specific environment variables with `AZURE_`.

{% include requirement/MUST id="azurecore-config-envvars-no-proxy-cidr" %} support [CIDR notation] for `NO_PROXY`.

### Global configuration

Global configuration refers to configuration settings that are applied to all applicable client constructors in some manner.

{% include requirement/MUST id="azurecore-config-shared-pipeline-policies" %} support global configuration of shared pipeline policies including:

* Logging: Log level, swapping out logger implementation
* HTTP: Proxy settings, max retries, timeout, swapping out transport implementation
* Telemetry: enabled/disabled
* Tracing: enabled/disabled

{% include requirement/MUST id="azurecore-config-override-global-config" %} provide configuration keys for setting or overriding every configuration setting inherited from the system or environment variables.

{% include requirement/MUST id="azurecore-config-opt-out" %} provide a method of opting out from importing system settings and environment variables into the configuration.

## Authentication and credentials

OAuth token authentication, obtained via Managed Security Identities (MSI) or Azure Identity is the preferred mechanism for authenticating service requests, and the only authentication credentials supported by the Azure Core library.

{% include requirement/MUST id="azurecore-auth-token-credential" %} provide a token credential type that can fetch an OAuth-compatible token needed to authenticate a request to the service in a non-blocking atomic manner.

{% include refs.md %}

[User-Agent header]: https://tools.ietf.org/html/rfc7231#section-5.5.3
[Transient fault handling]: https://docs.microsoft.com/azure/architecture/best-practices/transient-faults
[OpenTelemetry]: https://opentelemetry.io/
[Azure Monitor]: https://azure.microsoft.com/services/monitor/
[CIDR notation]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
