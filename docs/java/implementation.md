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

## SDK 기능 구현

### 로깅

클라이언트 라이브러리는 Azure Core의 강력한 로깅 메커니즘을 사용해야 합니다. 이는 소비자가 메서드 호출의 문제를 적절하게 진단하고, 그 문제가 개발자 코드, 클라이언트 라이브러리 코드, 서비스 중 어디에서 발생했는지 확인할 수 있도록 하기 위함입니다.


{% include requirement/MUST id="java-logging-clientlogger" %} 모든 클라이언트 라이브러리에서는 Azure Core가 제공하는 `ClientLogger` API를 유일한 로깅 API로 사용하십시오. 내부적으로, `ClientLogger`는 [SLF4J]를 감싸고 있으므로, SLF4J를 통해 제공되는 모든 외부 구성이 유효합니다. 최종 사용자에게 SLF4J 구성을 노출하기를 권장합니다. 자세한 내용은 [SLF4J user manual]을 참조하십시오.

{% include requirement/MUST id="java-logging-new-clientlogger" %} 모든 관련 클래스에 `ClientLogger` 인스턴스를 생성하십시오. 단, 성능이 매우 중요하거나, 인스턴스의 수명이 짧아 고유한 로거를 사용하기에 비용이 과하거나, 클래스 인스턴스화가 허용되지 않는 static-only 클래스는 예외입니다. 이러한 예외의 경우, 공유 (또는 static) 로거 인스턴스가 허용됩니다. 예를 들어, 아래 코드는 `ConfigurationAsyncClient`에 대한 `ClientLogger` 인스턴스를 생성합니다:

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

Static 로거는 JVM 인스턴스에서 실행되는 모든 클라이언트 라이브러리 인스턴스 간에 공유된다는 점을 참고하십시오. Static 로거는 인스턴스 수명이 짧은 경우에만 신중히 사용해야 합니다.

{% include requirement/MUST id="java-logging-levels" %} 로그를 내보낼 때는 다음 로그 레벨 중 하나를 사용하십시오: `Verbose`(상세 정보), `Informational`(발생한 상황), `Warning`(문제일 수 있는 상황), `Error`.

{% include requirement/MUST id="java-logging-errors" %} `Error` 레벨은 응용 프로그램이 복구될 가능성이 거의 없는 오류(메모리 부족 등)에 사용하십시오.

{% include requirement/MUST id="java-logging-warn" %} `Warning` 레벨은 함수가 의도한 작업을 수행하지 못한 경우 사용하십시오. 이는 일반적으로 함수가 예외를 발생시킨다는 것을 의미합니다. 스스로 복구하는 이벤트 발생(예를 들어, 요청이 자동으로 재시도되는 경우)은 포함하지 마십시오.


{% include requirement/MAY id="java-logging-slowlinks" %} 요청/응답 주기(응답 본문 시작까지)가 서비스 정의 임계계 값을 초과할 때, `Warning` 수준에서 요청 및 응답(아래 참조)을 기록할 수 있습니다. 임계 값은 거짓-긍정(false-positives)을 최소화하고, 서비스 문제를 식별하기 위해 선택되어야 합니다.

{% include requirement/MUST id="java-logging-info" %} `Informational` 레벨은 함수가 정상적으로 작동할 때 사용하십시오.

{% include requirement/MUST id="java-logging-verbose" %} `Verbose` 레벨은 자세한 문제 해결 시나리오를 위해 사용하십시오. 이는 주로 개발자 혹은 시스템 관리자가 특정 오류를 진단하기 위한 것입니다.

{% include requirement/MUST id="java-logging-no-sensitive-info" %} 승인된 헤더와 쿼리 파라미터의 서비스 제공 "허용 목록"에 있는 헤더 및 쿼리 파라미터만 로그로 기록하십시오. 다른 모든 헤더와 쿼리 파라미터는 해당 값을 수정해야 합니다.

{% include requirement/MUST id="java-logging-requests" %} `Informational` 메시지로 요청 행과 헤더를 로그에 남기십시오. 로그는 다음의 정보를 포함해야 합니다:

* HTTP 메서드
* URL
* 쿼리 파라미터(허용 목록에 없는 경우 수정)
* 요청 헤더(허용 목록에 없는 경우 수정)
* SDK가 상관 광계 목적을 위해 제공하는 요청 ID
* 요청이 시도된 횟수

이는 기본적으로 azure-core 내에서 수행하지만, 사용자가 `httpLogOptions` 구성 빌더를 통해 설정할 수 있습니다.

{% include requirement/MUST id="java-logging-responses" %} `Informational` 메시지로 응답 행과 헤더를 로그에 남기십시오. 로그 형식은 다음과 같아야 합니다:

* SDK에서 제공한 요청 ID(위 참조)
* 상태 코드
* 상태 코드와 함께 제공된 메시지
* 응답 헤더(허용 목록에 없는 경우 수정)
* 요청의 첫 번째 시도와 본문의 첫 번째 바이트 사이의 기간

{% include requirement/MUST id="java-logging-cancellations" %} 서비스 요청이 취소된 경우 `Informational` 로그를 기록하십시오. 로그에는 다음이 포함되어야 합니다:

* SDK에서 제공한 요청 ID(위 참조)
* 취소 사유(가능한 경우)

{% include requirement/MUST id="java-logging-exceptions" %} 예외는 `Warning` 레벨 메시지로 기록하십시오. 로그 레벨이 `Verbose`로 설정된 경우, 스택 추적 정보를 메시지에 포함하십시오.

{% include requirement/MUST id="java-logging-log-and-throw" %} 클라이언트 라이브러리 코드 내에서 발생한 모든 예외는 다음 로거 API 중 하나를 통해 발생시킵시오 - `ClientLogger.logThrowableAsError()`, `ClientLogger.logThrowableAsWarning()`, `ClientLogger.logExceptionAsError()` 혹은 `ClientLogger.logExceptionAsWarning()`.

아래 예시가 있습니다:

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

### 분산 추적

분산 추적 메커니즘을 통해 소비자는 그들의 코드를 프론트엔드부터 백엔드까지 추적할 수 있습니다. 분산 추적 라이브러리는 고유한 작업 단위인 범위를 생성합니다. 각각의 범위는 부모-자식 관계에 있습니다. 코드 계층 구조에 더 깊이 들어갈수록 더 많은 범위가 생성됩니다. 이러한 범위는 필요에 따라 적합한 수신자로 내보내질 수 있습니다. 범위를 추적하기 위해, _분산 추적 컨텍스트_(이하 컨텍스트)는 각 연속 계층으로 전달됩니다. 자세한 정보는 [OpenTelemetry]의 추적 항목을 참조하십시오.

Azure Core 라이브러리는 런타임 시점에 파이프라인 정책을 추가하기 위한 Service Provider Interface (SPI)를 제공합니다. 파이프라인 정책은 소비자 배포에서 추적을 활성화하는 데 사용됩니다. 분산 추적을 사용하려면 모든 클라이언트 라이브러리에서 플러그형 파이프라인 정책을 지원해야 합니다. 추가 메타데이터는 서비스별 메서드에 따라 지정하여 소비자에게 더 풍부한 추적 경험을 제공할 수 있습니다.

{% include requirement/MUST id="java-tracing-pluggable" %} HTTP 파이프라인 인스턴스화의 일부로써 플러그형 파이프라인 정책을 지원하십시오. 이를 통해 개발자는 추적 플러그인을 포함하고, 해당 파이프라인 정책을 사용 중인 모든 클라이언트 라이브러리에 자동으로 주입할 수 있습니다.

아래의 예시 코드를 검토해보십시오. 이 코드에서는 서비스 클라이언트 빌더가 해당 정책에 `HttpPipeline`을 생성하고 있습니다. 동시에, 빌더는 `HttpPolicyProviders.addBeforeRetryPolicies(policies)` 및 `HttpPolicyProviders.addAfterRetryPolicies(policies)` 행을 사용하여 플러그인이 ‘before retry’와 ‘after retry’ 정책을 추가할 수 있도록 허용합니다:

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

{%include requirement/MUST id="java-tracing-tracerproxy" %} 추적 범위와 함께 제공해야 하는 추가 메타데이터를 설정하려면 Azure Core `TracerProxy` API를 사용하십시오. 특히 `setAttribute(String key, String value, Context context)` 메서드를 사용하여 추적 컨텍스트에서 새 키/값 쌍을 설정합니다.

{%include requirement/MUST id="java-tracing-tracing-context" %} 서비스 메서드 인수로 받은 `Context` 객체 전부를 모든 경우에 생성된 서비스 인터페이스 메서드로 전달하십시오.

## 테스팅

{% include requirement/MUST id="java-testing-params" %} 사용 가능한 모든 HTTP 클라이언트와 서비스 버전을 사용하려면, 적용 가능한 모든 단위 테스트를 매개변수화하십시오. 모든 테스트의 매개변수화된 실행은 라이브 테스트의 일부로 이루어져야 합니다. Netty와 최신 서비스 버전으로 구성된 더 짧은 실행은 PR 유효성 검사가 발생할 때마다 실행할 수 있습니다.

> **TODO(추가 예정)** refer to cases where this is done in Java today

{% include refs.md %}
{% include_relative refs.md %}
