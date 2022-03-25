---
title: "Android 가이드라인: 구현"
keywords: guidelines android
permalink: android_implementation.html
folder: android
sidebar: general_sidebar
---

## API 구현

이 문서는 Azure SDK 클라이언트 라이브러리를 구현하기 위한 가이드라인입니다. 이 가이드라인의 일부는 코드 생성 도구에 의해 자동으로 적용된 점을 유의하시기 바랍니다.

{% include requirement/MUSTNOT id="android-implementation" %} 구현 코드(즉, 공개 API의 일부를 구성하지 않은 코드)를 공개 API로 오인하지 않도록 하십시오. 구현 코드에는 두 가지 유효한 약정이 있는데, 우선 순위는 다음과 같습니다:

1. 구현 클래스를 패키지-프라이빗으로 만들고, 이를 사용하는 클래스와 같은 패키지 내에 배치할 수 있습니다.
2. 구현 클래스를 `implementation`으로 명명된 서브패키지 내에 배치할 수 있습니다.

CheckStyle 검사는 `implementation` 패키지 내의 클래스가 공개 API를 통해 노출되지 않도록 확인합니다. 하지만 우선 API를 공개 API로 구현하지 않는 것이 좋으므로, 가능하면 패키지-프라이빗을 적용하는 것이 더 낫습니다.

### 서비스 클라이언트

#### 비동기 서비스 클라이언트

{% include requirement/MUST id="java-async-blocking" %} 비동기 클라이언트 라이브러리 내에 동기 함수 호출을 포함하십시오.

##### HTTP 파이프라인 사용하기

Azure SDK 팀은 구성이나 HTTP 요청 등의 횡단 관심사(cross cutting concerns)에 관한 일반적인 매커니즘이 포함된 [Azure Core] 라이브러리를 제공하고 있습니다.

{% include requirement/MUST id="android-requests-use-pipeline" %} REST endpoints를 서비스하기 위해 Azure Core 내의 HTTP 파이프라인 컴포넌트를 사용하십시오.

HTTP pipeline은 여러 정책에 의해 래핑된 HTTP transport로 구성되어 있다. 각 정책은 파이프라인이 요청 및/또는 응답을 수정할 수 있는 제어 지점입니다. 우리는 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방법을 표준화하는 기본 정책 세트를 규정합니다. 리스트의 순서는 구현에 있어 가장 합리적인 순서입니다.

{% include requirement/MUST id="android-requests-implement-policies" %} HTTP 파이프라인을 구성할 때 Azure Core에 의해 제공되는 아래의 정책들을 포함하십시오:

- Telemetry
- Unique Request ID
- Retry
- Authentication
- Response downloader
- Logging

{% include requirement/SHOULD id="ios-requests-use-azure-core-impl" %} 가능한 한 Azure Core의 정책 구현을 사용하십시오. 서비스의 고유한 것이 아니라면 정책을 "직접 작성하려고" 하지 마십시오. 기존 정책에 다른 옵션이 필요한 경우 [Architecture Board]와 협력하여 옵션을 추가하십시오.

#### 어노테이션

서비스 클라이언트 클래스에는 다음의 어노테이션을 포함하십시오. 예를 들어, 이 코드에서는 두 어노테이션을 사용하는 예시 클래스를 볼 수 있습니다:

```java
@ServiceClient(builder = ConfigurationAsyncClientBuilder.class, isAsync = true, service = ConfigurationService.class)
public final class ConfigurationAsyncClient {
    @ServiceMethod(returns = ReturnType.COLLECTION)
    public Mono<Response<ConfigurationSetting>> addSetting(String key, String value) {
        ...
    }
}
```

| 어노테이션 | 위치 | 설명 |
|:-----------|:---------|:------------|
| `@ServiceClient` | 서비스 클라이언트 | 클라이언트를 인스턴스화하는 빌더, API가 비동기인지 여부, 서비스 인터페이스(`@ServiceInterface` 어노테이션이 있는 인터페이스)에 대한 참조를 명시합니다. |
| `@ServiceMethod` | 서비스 메서드 | 네트워크 동작을 수행하는 모든 서비스 클라이언트 메서드에 명시합니다. |

#### 서비스 클라이언트 빌더

##### 어노테이션

`@ServiceClientBuilder` 어노테이션은 서비스 클라이언트 인스턴스화를 담당하는 클래스에 반드시 명시되어야 합니다. (즉, `@ServiceClient` 어노테이션이 적용된 클래스를 인스턴스화하는 클래스에 배치되어야 합니다.). 예시는 다음과 같습니다:

```java
@ServiceClientBuilder(serviceClients = {ConfigurationClient.class, ConfigurationAsyncClient.class})
public final class ConfigurationClientBuilder { ... }
```

위의 빌더는 `ConfigurationClient`와 `ConfigurationAsyncClient`의 인스턴스를 작성할 수 있다고 명시합니다.

### 지원 타입

#### 모델 타입

##### 어노테이션

조건에 해당하는 경우, 모델 클래스에 적용해야 하는 두 가지 어노테이션이 있습니다.

* `@Fluent` 어노테이션은 최종 사용자에게 Fluent API를 제공할 것으로 예상되는 모든 모델 클래스에 적용됩니다.
* `@Immutable` 어노테이션은 변경할 수 없는 모든 클래스에 적용됩니다.

> TODO: Include the @HeaderCollection annotation.

## SDK 기능 구현

### 구성

클라이언트 라이브러리를 구성할 때, 클라이언트 라이브러리의 사용자가 Azure 서비스에 대한 접속을 글로벌하고(사용자가 사용하고 있는 다른 클라이언트와 함께), 특히 클라이언트 라이브러리와의 접속을 적절히 설정할 수 있도록 특히 주의해야 합니다. 안드로이드 어플리케이션에 대해서, 설정을 어플리케이션 환경설정 또는 `.properties` 파일 사용 등과 같은 다양한 방법으로 적용할 수 있습니다.

> TODO: Determine a recommended way to pass configuration parameters to Android libraries

### 로깅

클라이언트 라이브러리는 Azure Core의 견고한 로깅 메커니즘을 사용하여 사용자가 메서드 호출에 관한 문제를 적절하게 진단하고 문제가 사용자의 코드, 클라이언트 라이브러리 코드 또는 서비스에 있는지 여부를 신속하게 판단할 수 있도록 해야 합니다.

요청 로깅은 `HttpPipeline`에 의해 자동으로 수행됩니다. 클라이언트 라이브러리에서 커스텀로깅를 추가할 필요가 있는 경우, 파이프라인 로깅 메커니즘과 같은 가이드라인과 메커니즘을 따르십시오. 클라이언트 라이브러리가 커스텀로깅을 실행하고자 하는 경우, 라이브러리 설계자는 로깅 메커니즘이 `HttpPipeline` 로깅 정책과 동일한 방법으로 접속 가능함을 보장해야 합니다.

{% include requirement/MUST id="android-logging-directly" %} (`HttpPipeline` 경유가 아닌) 직접 로깅하는 경우 [Azure SDK 공통 가이드라인 로깅 섹션][logging-general-guidelines]및 [다음 가이드라인](#using-the-clientlogger-interface)을 따르십시오.

#### ClientLogger 인터페이스 사용

{% include requirement/MUST id="android-logging-clientlogger" %} Azure Core에서 제공되는 `ClientLogger` API를 모든 클라이언트 라이브러리에서 유일한 로깅 API로 사용합니다. 내부적으로, `ClientLogger`는 Android Logcat 버퍼에 기록됩니다.

> TODO: Determine if we want ClientLogger to wrap SLF4J like it's Java counterpart.

{% include requirement/MUST id="android-logging-create-new" %} 모든 관련 클래스의 인스턴스별로 `ClientLogger`의 새 인스턴스를 만듭니다. 예를 들어, 아래의 코드는 `ConfigurationAsyncClient`용 `ClientLogger` 인스턴스를 생성합니다:

```java
public final class ConfigurationAsyncClient {
    private final ClientLogger logger = new ClientLogger(ConfigurationAsyncClient.class);

    // Example call to a service.
    public Response<String> setSetting(ConfigurationSetting setting) {
        Response<String> response = service.setKey(serviceEndpoint, setting.key(), setting.label(), setting, getETagValue(setting.etag()), null);
        
        logger.info("Set ConfigurationSetting - {}", response.value());
        
        return response;
    }
}
```

정적 로거 인스턴스를 생성하지 마십시오. 정적 로거 인스턴스는 수명이 길며 애플리케이션이 종료될 때까지 할당된 메모리가 해제되지 않습니다.

{% include requirement/MUST id="android-logging-log-and-throw" %} 로거 API 중 하나를 통해 클라이언트 라이브러리 코드 내에 생성된 모든 예외를 throw 하십시오 - `ClientLogger.logThrowableAsError()`, `ClientLogger.logThrowableAsWarning()`, `ClientLogger.logExceptionAsError()`, `ClientLogger.logExceptionAsWarning()`.

예를 들어:

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

### 분산 트레이스

분산 트레이스는 모바일컨텍스트에서는 거의 발생하지 않습니다. 분산 트레이스를 서포트할 필요가 있는 경우는, [Azure SDK 모바일 팀](mailto:azuresdkmobileteam@microsoft.com)에 문의해 주십시오.

### 테스팅

우리가 지원하고 싶은 핵심 사항 중 하나는 라이브러리 이용자가 서비스를 활성화하지 않고도 애플리케이션에 대해 반복 가능한 유닛 테스트를 쉽게 작성할 수 있도록 하는 것입니다. 이는 기반이 되는 서비스 구현의 예상 밖의 문제(네트워크의 상태나 서비스의 정지등)에 대해 염려하지 않고, 코드를 확실하고 빠르게 테스트할 수 있습니다. 모의 객체는 장애, 엣지 케이스, 재현하기 어려운 상황(코드가 2월 29일에는 작동하지 않는 경우)을 시뮬레이트하는데도 도움이 됩니다.

{% include requirement/MUST id="android-testing-patterns" %} 사용 가능한 모든 HTTP 클라이언트 및 서비스 버전을 사용하기 위해 적용 가능한 모든 유닛 테스트를 파라미터화하십시오. 모든 테스트의 파라미터화된 실행은 라이브 테스트의 일부로 수행되어야 합니다. PR 유효성 검사가 발생할 때마다 더 짧은 실행(Netty 및 최신 서비스 버전)을 실행할 수 있습니다.

> TODO: Document how to write good tests using JUnit on Android.

### 기타 Android 관련 고려 사항

> TODO: Revisit min API level chosen.

Android 개발자들은 자신이 실행하고 있는 런타임 환경에 대해 고려할 필요가 있습니다. Android 생태계는 매우 다양한 런타임으로 분할되어 있습니다.

{% include requirement/MUST id="android-library-sync-support" %} 최소 Android API level 15 이상을 지원하십시오(Ice Cream Sandwich). 해당 값은 프로젝트의 최상위 레벨 `build.gradle` 파일의 `minSdkVersion`에서 찾을 수 있습니다.

최소 API 수준 선택에 대해 논의할 때 다음 두 가지 사항을 고려해야 합니다:

1. Google이 지원하는 최소 API 레벨.
2. 선택하는 특정 API 수준의 도달 범위.

우리는 인기 있는 HTTP 클라이언트나 직렬화 라이브러리같이 개발자 커뮤니티에서 여전히 널리 적용되는 툴을 사용할 수 있는 안드로이드 장치가 도달할 수 있도록 구글이 지원하는 최소 API 레벨을 요구합니다. 우리는 현재 99.8% 이상을 커버하는 API level 15(2021년 5월 기준)에 정착하고 있습니다. 특정 API 수준의 도달 범위는 안드로이드 스튜디오에서 "Create New Project" 화면에서 생성할 프로젝트 유형을 선택한 후 "Help me choose" 를 클릭하면 확인할 수 있습니다.

{% include requirement/MUST id="android-library-target-sdk-version" %} 프로젝트의 최상위 `build.gradle` 파일에서 `targetSdkVersion`을 API 수준 26 이상으로 설정하십시오.

2018년 11월 현재 모든 기존 Android 앱은 API 레벨 26 이상을 대상으로 해야 합니다. 자세한 내용은 [향후 Google Play에서 앱 보안 및 성능 향상](https://android-developers.googleblog.com/2017/12/improving-app-security-and-performance.html)을 참조하십시오.

{% include requirement/MUST id="android-library-max-sdk-version" %} `maxSdkVersion`을 프로젝트의 최상위 `build.gradle` 파일에서 테스트를 실행하는 최신 API 수준으로 설정하십시오. 이는 SDK가 출시되는 시점에 Google이 지원하는 최신 API 수준이어야 합니다.

{% include requirement/MUST id="android-library-source-compat" %} Gradle 프로젝트의 source 및 target compatibility 수준을 `1.8`로 설정하십시오.

{% include requirement/MUST id="android-library-aar" %} 라이브러리를 Android AAR로 릴리스하십시오.

{% include requirement/MUST id="android-library-resource-prefix" %} 리소스를 사용하는 경우 `build.gradle`의 안드로이드 섹션에서 `azure_<service>`의 `resourcePrefix`를 정의하십시오.

{% include requirement/SHOULD id="android-library-shrink-code" %} AAR에는 개발자가 라이브러리를 사용할 때 애플리케이션을 올바르게 최소화하는 것을 보조할 수 있도록 Proguard 구성이 포함되어 있어야 합니다.

{% include requirement/MUST id="android-library-proguard" %} 라이브러리에 Proguard 구성을 포함하는 경우 `consumerProguardFiles`를 사용하십시오.

{% include refs.md %}
{% include_relative refs.md %}
