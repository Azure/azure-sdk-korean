---
title: "Java 가이드라인: 구현"
keywords: guidelines java
permalink: java_implementation.html
folder: java
sidebar: general_sidebar
---

## API 구현

이 문서는 Azure SDK 클라이언트 라이브러리를 구현하기 위한 가이드라인입니다. 이 가이드라인의 일부는 코드 생성 도구에 의해 자동으로 적용된 점을 유의하시기 바랍니다. 

{% include requirement/MUSTNOT id="java-namespaces-implementation" %} 구현 코드(즉, 공개 API의 일부를 구성하지 않은 코드)를 공개 API로 오인하지 않도록 하십시오. 구현 코드에는 두 가지 유효한 약정이 있는데, 우선 순위는 다음과 같습니다:

1. 구현 클래스를 패키지-프라이빗으로 만들고, 이를 사용하는 클래스와 같은 패키지 내에 배치할 수 있습니다.
2. 구현 클래스를 `implementation`으로 명명된 서브패키지 내에 배치할 수 있습니다.

CheckStyle 검사는 `implementation` 패키지 내의 클래스가 공개 API를 통해 노출되지 않도록 확인합니다. 하지만 우선 API를 공개 API로 구현하지 않는 것이 좋으므로, 가능하면 패키지-프라이빗을 적용하는 것이 더 낫습니다.

### 서비스 클라이언트

#### 비동기 서비스 클라이언트

{% include requirement/MUSTNOT id="java-async-blocking" %} 비동기 클라이언트 라이브러리 내에 동기 함수 호출을 포함하지 마십시오. 비동기 API 안의 동기 함수 호출을 감지하기 위해 [BlockHound]를 사용하십시오.

#### 주석

서비스 클라이언트 클래스에는 다음의 주석을 포함하십시오. 예를 들어, 이 코드에서는 두 주석을 사용하는 예시 클래스를 볼 수 있습니다:

```java
@ServiceClient(builder = ConfigurationAsyncClientBuilder.class, isAsync = true, service = ConfigurationService.class)
public final class ConfigurationAsyncClient {

    @ServiceMethod(returns = ReturnType.COLLECTION)
    public Mono<Response<ConfigurationSetting>> addSetting(String key, String value) {
        ...
    }
}
```

| 주석 | 위치  | 설명 |
|:-----------|:---------|:------------|
| `@ServiceClient` | 서비스 클라이언트 |서비스 클라이언트를 인스턴스화하는 빌더, API가 비동기인지 여부, 서비스 인터페이스(`@ServiceInterface` 주석이 있는 인터페이스)에 대한 참조를 명시합니다.|
| `@ServiceMethod` | 서비스 메서드 |클라이언트 클래스인지 여부에 관계 없이, 네트워크 작업을 수행하는 모든 메서드에 명시합니다.|

#### 서비스 클라이언트 빌더

##### 주석

`@ServiceClientBuilder` 주석은 서비스 클라이언트 인스턴스화를 담당하는 클래스에 반드시 명시되어야 합니다. (즉, `@ServiceClient` 주석이 적용된 클래스를 인스턴스화하는 클래스에 배치되어야 합니다.) 예시는 다음과 같습니다:

```java
@ServiceClientBuilder(serviceClients = {ConfigurationClient.class, ConfigurationAsyncClient.class})
public final class ConfigurationClientBuilder { ... }
```

위의 빌더는 `ConfigurationClient`와 `ConfigurationAsyncClient`의 인스턴스를 작성할 수 있다고 명시합니다.

### 지원 타입

#### 모델 타입

##### 주석 

조건에 해당하는 경우, 모델 클래스에 적용해야 하는 두 가지 주석이 있습니다:

* `@Fluent` 주석은 최종 사용자에게 Fluent API를 제공할 것으로 예상되는 모든 모델 클래스에 적용됩니다.
* `@Immutable` 주석은 변경할 수 없는 모든 클래스에 적용됩니다.

## SDK Feature Implementation

### Logging

Client libraries must make use of the robust logging mechanisms in azure core, so that the consumers can adequately diagnose issues with method calls and quickly determine whether the issue is in the consumer code, client library code, or service.

{% include requirement/MUST id="java-logging-clientlogger" %} use the `ClientLogger` API provided within Azure Core as the sole logging API throughout all client libraries. Internally, `ClientLogger` wraps [SLF4J], so all external configuration that is offered through SLF4J is valid.  We encourage you to expose the SLF4J configuration to end users. For more information, see the [SLF4J user manual].

{% include requirement/MUST id="java-logging-new-clientlogger" %} create a new instance of a `ClientLogger` per instance of all relevant classes, except in situations where performance is critical, the instances are short-lived (and therefore the cost of unique loggers is excessive), or in static-only classes (where there is no instantiation of the class allowed). In these cases, it is acceptable to have a shared (or static) logger instance. For example, the code below will create a `ClientLogger` instance for the `ConfigurationAsyncClient`:

```java
public final class ConfigurationAsyncClient {
    private final ClientLogger logger = new ClientLogger(ConfigurationAsyncClient.class);

    // example async call to a service that uses the Project Reactor APIs to log request, success, and error
    // information out to the service logger instance
    public Mono<Response<ConfigurationSetting>> setSetting(ConfigurationSetting setting) {
        return service.setKey(serviceEndpoint, setting.key(), setting.label(), setting, getETagValue(setting.etag()), null)
            .doOnRequest(ignoredValue -> logger.info("Setting ConfigurationSetting - {}", setting))
            .doOnSuccess(response -> logger.info("Set ConfigurationSetting - {}", response.value()))
            .doOnError(error -> logger.warning("Failed to set ConfigurationSetting - {}", setting, error));
    }
}
```

Note that static loggers are shared among all client library instances running in a JVM instance. Static loggers should be used carefully and in short-lived cases only.

{% include requirement/MUST id="java-logging-levels" %} use one of the following log levels when emitting logs: `Verbose` (details), `Informational` (things happened), `Warning` (might be a problem or not), and `Error`.

{% include requirement/MUST id="java-logging-errors" %} use the `Error` logging level for failures that the application is unlikely to recover from (out of memory, etc.).

{% include requirement/MUST id="java-logging-warn" %} use the `Warning` logging level when a function fails to perform its intended task. This generally means that the function will raise an exception.  Do not include occurrences of self-healing events (for example, when a request will be automatically retried).

{% include requirement/MAY id="java-logging-slowlinks" %} log the request and response (see below) at the `Warning` logging level when a request/response cycle (to the start of the response body) exceeds a service-defined threshold.  The threshold should be chosen to minimize false-positives and identify service issues.

{% include requirement/MUST id="java-logging-info" %} use the `Informational` logging level when a function operates normally.

{% include requirement/MUST id="java-logging-verbose" %} use the `Verbose` logging level for detailed troubleshooting scenarios. This is primarily intended for developers or system administrators to diagnose specific failures.

{% include requirement/MUST id="java-logging-no-sensitive-info" %} only log headers and query parameters that are in a service-provided "allow-list" of approved headers and query parameters.  All other headers and query parameters must have their values redacted.

{% include requirement/MUST id="java-logging-requests" %} log request line and headers as an `Informational` message. The log should include the following information:

* The HTTP method.
* The URL.
* The query parameters (redacted if not in the allow-list).
* The request headers (redacted if not in the allow-list).
* An SDK provided request ID for correlation purposes.
* The number of times this request has been attempted.

This happens within azure-core by default, but users can configure this through the builder `httpLogOptions` configuration setting.

{% include requirement/MUST id="java-logging-responses" %} log response line and headers as an `Informational` message.  The format of the log should be the following:

* The SDK provided request ID (see above).
* The status code.
* Any message provided with the status code.
* The response headers (redacted if not in the allow-list).
* The time period between the first attempt of the request and the first byte of the body.

{% include requirement/MUST id="java-logging-cancellations" %} log an `Informational` message if a service call is cancelled.  The log should include:

* The SDK provided request ID (see above).
* The reason for the cancellation (if available).

{% include requirement/MUST id="java-logging-exceptions" %} log exceptions thrown as a `Warning` level message. If the log level set to `Verbose`, append stack trace information to the message.

{% include requirement/MUST id="java-logging-log-and-throw" %} throw all exceptions created within the client library code through one of the logger APIs - `ClientLogger.logThrowableAsError()`, `ClientLogger.logThrowableAsWarning()`, `ClientLogger.logExceptionAsError()` or `ClientLogger.logExceptionAsWarning()`.

For example:

```java
// NO!!!!
if (priority != null && priority < 0) {
    throw new IllegalArgumentException("'priority' cannot be a negative value. Please specify a zero or positive long value.");
}

// Good

// Log any Throwable as error and throw the exception
if (!file.exists()) {
    throw logger.logThrowableAsError(new IOException("File does not exist " + file.getName()));
}

// Log any Throwable as warning and throw the exception
if (!file.exists()) {
    throw logger.logThrowableAsWarning(new IOException("File does not exist " + file.getName()));
}

// Log a RuntimeException as error and throw the exception
if (priority != null && priority < 0) {
    throw logger.logExceptionAsError(new IllegalArgumentException("'priority' cannot be a negative value. Please specify a zero or positive long value."));
}

// Log a RuntimeException as warning and throw the exception
if (numberOfAttempts < retryPolicy.getMaxRetryCount()) {
    throw logger.logExceptionAsWarning(new RetryableException("A transient error occurred. Another attempt will be made after " + retryPolicy.getDelay()));
}
```

### Distributed tracing

Distributed tracing mechanisms allow the consumer to trace their code from frontend to backend.  Distributed tracing libraries creates spans - units of unique work.  Each span is in a parent-child relationship.  As you go deeper into the hierarchy of code, you create more spans.  These spans can then be exported to a suitable receiver as needed.  To keep track of the spans, a _distributed tracing context_ (called a context in the remainder of this section) is passed into each successive layer.  For more information on this topic, visit the [OpenTelemetry] topic on tracing.

The Azure core library provides a service provider interface (SPI) for adding pipeline policies at runtime. The pipeline policy is used to enable tracing on consumer deployments. Pluggable pipeline policies must be supported in all client libraries to enable distributed tracing. Additional metadata can be specified on a per-service-method basis to provide a richer tracing experience for consumers.  

{% include requirement/MUST id="java-tracing-pluggable" %} support pluggable pipeline policies as part of the HTTP pipeline instantiation. This enables developers to include a tracing plugin and have its pipeline policy automatically injected into all client libraries that they are using.

Review the code sample below, in which a service client builder creates an `HttpPipeline` from its set of policies.  At the same time, the builder allows plugins to add 'before retry' and 'after retry' policies with the lines `HttpPolicyProviders.addBeforeRetryPolicies(policies)` and `HttpPolicyProviders.addAfterRetryPolicies(policies)`:

```java
public ConfigurationAsyncClient build() {
    ...

    // Closest to API goes first, closest to wire goes last.
    final List<HttpPipelinePolicy> policies = new ArrayList<>();
    policies.add(new UserAgentPolicy(AzureConfiguration.NAME, AzureConfiguration.VERSION, buildConfiguration));
    policies.add(new RequestIdPolicy());
    policies.add(new AddHeadersPolicy(headers));
    policies.add(new AddDatePolicy());
    policies.add(new ConfigurationCredentialsPolicy(buildCredentials));
    HttpPolicyProviders.addBeforeRetryPolicies(policies);
    policies.add(retryPolicy);
    policies.addAll(this.policies);
    HttpPolicyProviders.addAfterRetryPolicies(policies);
    policies.add(new HttpLoggingPolicy(httpLogDetailLevel));

    ...
}
```

{%include requirement/MUST id="java-tracing-tracerproxy" %} use the Azure core `TracerProxy` API to set additional metadata that should be supplied along with the tracing span. In particular, use the `setAttribute(String key, String value, Context context)` method to set a new key/value pair on the tracing context.

{%include requirement/MUST id="java-tracing-tracing-context" %} pass all `Context` objects received as service method arguments through to the generated service interface methods in all cases.

## Testing

{% include requirement/MUST id="java-testing-params" %} parameterize all applicable unit tests to make use of all available HTTP clients and service versions. Parameterized runs of all tests must occur as part of live tests. Shorter runs, consisting of just Netty and the latest service version, can be run whenever PR validation occurs.

> TODO refer to cases where this is done in Java today

{% include refs.md %}
{% include_relative refs.md %}
