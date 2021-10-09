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

{% include requirement/MUST id="azurecore-http-telemetry-useragent" %} 다음 형식을 사용하여 [User-Agent 헤더]로 원격 측정 정보를 전송하십시오:

```
[<application_id> ]azsdk-<sdk_language>-<package_name>/<package_version> <platform_info>
```

- `<application_id>`: 선택적 애플리케이션별 문자열. 슬래시를 포함할 수 있지만, 공백을 포함해서는 안 됩니다. 문자열은 클라이언트 라이브러리의 사용자가 제공합니다. 예시: "AzCopy/10.0.4-Preview"
- `<sdk_language>`: SDK 언어 이름 (모두 소문자): "net", "python", "java", 또는 "js"
- `<package_name>`: 슬래시를 대시로 바꾸고 Azure 표시기를 제거하여 개발자에게 표시되는 클라이언트 라이브러리 패키지 이름. 예시: "Security.KeyVault" (.NET), "security.keyvault" (Java), "keyvault" (JavaScript & Python)
- `<package_version>`: 패키지의 버전. 참고: 서비스 버전이 아닙니다.
- `<platform_info>`: 현재 실행 중인 언어 런타임 및 OS에 대한 정보. 예시: "(NODE-VERSION v4.5.0; Windows_NT 10.0.14393)"

예를 들어, Azure Blob Storage 클라이언트 라이브러리를 사용하여 각 언어로 `AzCopy`를 다시 작성하면, 다음과 같은 user-agent 문자열이 발생할 수 있습니다.

- (.NET) `AzCopy/10.0.4-Preview azsdk-net-Storage.Blobs/11.0.0 (.NET Standard 2.0; Windows_NT 10.0.14393)`
- (JavaScript) `AzCopy/10.0.4-Preview azsdk-js-storage-blob/11.0.0 (Node 10.16.0; Ubuntu; Linux x86_64; rv:34.0)`
- (Java) `AzCopy/10.0.4-Preview azsdk-java-storage.blobs/11.0.0 (Java/1.8.0_45; Macintosh; Intel Mac OS X 10_10; rv:33.0)`
- (Python) `AzCopy/10.0.4-Preview azsdk-python-storage/4.0.0 Python/3.7.3 (Ubuntu; Linux x86_64; rv:34.0)`

{% include requirement/MUST id="azurecore-http-telemetry-appid" %} 라이브러리 소비자가 애플리케이션 ID를 설정할 수 있도록 허용하십시오. 이를 통해 소비자는 앱에 대한 서비스 간 원격 분석을 얻을 수 있습니다. 응용 프로그램 ID는 관련 `ClientOptions` 개체에서 설정할 수 있어야 합니다.

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

> **TODO(추가 예정)** Add Unique Request ID Policy Requirements

### 재시도 정책

클라이언트 애플리케이션이 서비스에 네트워크 요청을 보내려고 할 때 오류가 발생하는 데에는 여러 이유가 있습니다. 몇 가지 예로는 시간 초과, 네트워크 인프라 오류, 제한/사용 중으로 인한 서비스 거부, 스케일 다운으로 인한 서비스 인스턴스 종료, 다른 버전으로 교체하기 위해 다운되는 서비스 인스턴스, 처리되지 않은 예외로 인한 서비스 충돌 등이 있습니다. 기본 제공 재시도 메커니즘(소비자가 재정의할 수 있는 기본 구성)을 제공함으로써 SDK와 소비자 애플리케이션은 이러한 장애에 탄력적으로 대처할 수 있습니다. 일부 서비스는 각 시도에 대해 실제 비용을 청구하므로, 소비자가 복원력보다 비용 절감을 선호하는 경우 재시도를 완전히 비활성화할 수 있어야 합니다.

더 많은 정보는 다음을 참고하십시오: [일시적인 오류 처리]

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

### 인증 정책 

Azure 전역의 서비스에서는 클라이언트를 인증하기 위해 다양한 인증 스키마를 사용합니다. 개념적으로, 서비스 클라이언트 요청을 인증하는 두 가지 엔터티인 자격 증명과 인증 정책이 있습니다. 자격 증명은 요청을 인증하는 데 필요한 기밀 인증 데이터를 제공합니다. 인증 정책은 서비스에 대한 요청을 인증하기 위해 자격 증명이 제공한 데이터를 사용합니다. 클라이언트 수명 기간 동안 자격 증명 데이터는 필요에 따라 업데이트가 가능해야 하며, 인증 정책은 항상 최신 자격 증명 데이터를 사용해야 합니다.

{% include requirement/MUST id="azurecore-http-auth-bearer-token" %} (토큰 자격 증명 및 범위를 허용하는) Bearer 권한 부여 정책을 구현하십시오.

### 응답 다운로더(Response downloader) 정책 

서비스가 반환하는 모든 것을 소비자 코드가 사용할 수 있는 모델로 변경하기 위해, 대부분의(전부는 아님) 작업에 응답 다운로더가 필요합니다. 응답 페이로드를 역직렬화하지 않는 메서드의 예로, Blob Storage 클라이언트 라이브러리 내에서 원시 Blob을 다운로드하는 메서드가 있습니다. 이 경우, 원시 데이터 바이트가 필요합니다. 대부분의 작업에서 본문은 역직렬화 전에 전체적으로 다운로드되어야 합니다. 이 파이프라인 정책은 다음 요구 사항을 구현해야 합니다:

{% include requirement/MUST id="azurecore-http-response-body" %} 전체 응답 본문을 다운로드하고, 완전히 다운로드 된 응답 본문을 응답 페이로드를 역직렬화하는 메서드에 대한 작업 메서드에 전달하십시오. 본문을 읽는 동안 네트워크 연결이 실패하면, 재시도 정책이 자동으로 작업을 재시도해야 합니다.

### 분산 추적 정책

분산 추적 메커니즘을 통해 소비자는 그들의 코드를 프론트엔드부터 백엔드까지 추적할 수 있습니다. 분산 추적 라이브러리는 고유한 작업 단위인 범위를 생성합니다. 각각의 범위는 부모-자식 관계에 있습니다. 코드 계층 구조에 더 깊이 들어갈수록 더 많은 범위가 생성됩니다. 이러한 범위는 필요에 따라 적합한 수신자로 내보내질 수 있습니다. 범위를 추적하기 위해, _분산 추적 컨텍스트(이하 컨텍스트)_는 각 연속 계층으로 전달됩니다. 자세한 정보는, [OpenTelemetry]의 추적 항목을 참조하십시오.

분산 추적 정책은 다음 작업을 담당합니다:

* REST 호출당 범위 생성.
* 백엔드 서비스에 컨텍스트 전달.

> 클라이언트 라이브러리 구현 문서에도 분산 추적 관련 주제가 있습니다. 

{% include requirement/MUST id="azurecore-http-tracing-opentelemetry" %} 분산 추적을 위해 [OpenTelemetry]를 지원하십시오. 

{% include requirement/MUST id="azurecore-http-tracing-accept-context" %} 부모 범위를 설정하기 위해 코드 호출에서 컨텍스트를 수락하십시오.

{% include requirement/MUST id="azurecore-http-tracing-pass-context" %} [Azure Monitor]를 지원하기 위해 적절한 헤더(`traceparent`, `tracestate` 등)를 통해 백엔드 서비스에 컨텍스트를 전달하십시오.

{% include requirement/MUST id="azurecore-http-tracing-create-span" %} 클라이언트 라이브러리가 수행하는 각 REST 호출에 대해 새 범위(메서드당 범위의 자식 범위어야 함)를 만드십시오.

### 로깅 정책

Azure Core 내의 많은 로깅 요구 사항은 클라이언트 라이브러리 내의 로깅에 대한 동일한 요구 사항을 반영합니다.

{% include requirement/MUST id="azurecore-http-logging-handlers" %} 클라이언트 라이브러리가 로그 핸들러 및 로그 설정을 지정할 수 있도록 하용하십시오.

{% include requirement/MUST id="azurecore-http-logging-levels" %} 로그를 내보낼 때는 다음 로그 레벨 중 하나를 사용하십시오: `Verbose`(상세 정보), `Informational`(발생한 상황), `Warning`(문제일 수 있는 상황), `Error`.

{% include requirement/MUST id="azurecore-http-logging-error" %} `Error` 레벨은 응용 프로그램이 복구될 가능성이 거의 없는 오류(메모리 부족 등)에 사용하십시오.

{% include requirement/MUST id="azurecore-http-logging-warning" %} `Warning` 레벨은 함수가 의도한 작업을 수행하지 못한 경우 사용하십시오. 이는 일반적으로 함수가 예외를 발생시킨다는 것을 의미합니다. 스스로 복구하는 이벤트 발생(예를 들어, 요청이 자동으로 재시도되는 경우)은 포함하지 마십시오.

{% include requirement/MUST id="azurecore-http-logging-info" %} `Informational` 레벨은 함수가 정상적으로 작동할 때 사용하십시오.

{% include requirement/MUST id="azurecore-http-logging-verbose" %} `Verbose` 레벨은 자세한 문제 해결 시나리오를 위해 사용하십시오. 이는 주로 개발자 혹은 시스템 관리자가 특정 오류를 진단하기 위한 것입니다.

{% include requirement/MUSTNOT id="azurecore-http-logging-sensitive-info" %} 승인된 헤더와 쿼리 파라미터의 서비스 제공 "허용 목록"에 있는 헤더 및 쿼리 파라미터만 로그로 기록하십시오. 다른 모든 헤더와 쿼리 파라미터는 해당 값을 수정해야 합니다.

{% include requirement/MUST id="azurecore-http-logging-request-line" %} 요청 행, 응답 행, 헤더는 `Informational` 로그를 남기십시오.

{% include requirement/MUST id="azurecore-http-logging-cancellation" %} 서비스 요청이 취소된 경우 `Informational` 로그를 기록하십시오.

{% include requirement/MUST id="azurecore-http-logging-exceptions" %} 예외는 `Warning` 레벨 메시지로 기록하십시오. 로그 레벨이 `Verbose`로 설정된 경우, 스택 추적 정보를 메시지에 포함하십시오.

{% include requirement/MUST id="azurecore-http-logging-configuration" %} 문제를 완화할 수 있는 관련 구성 설정이 있는 경우, HTTP 파이프라인이 오류를 발생시킬 때 관련 구성 정보를 로그에 포함하십시오. 모든 오류가 구성 변경으로 해결될 수 있는 것은 아닙니다.

### 프록시 정책 

Azure SDK를 통합하는 앱은 일반 기업에서 운영되어야 합니다. 제어 및 캐싱 목적으로 HTTP 프록시를 구현하는 것이 일반적인 관행입니다. 프록시는 일반적으로 머신 수준에서 구성되며, 따라서 환경의 일부입니다. 그러나 프록시를 조정해야 하는 이유가 있습니다(예를 들어, 테스트는 URL 프로덕션 환경 대신 테스트 환경에 URL을 다시 작성하기 위해 프록시를 사용할 수 있음). Azure SDK와 모든 클라이언트 라이브러리는 해당 환경에서 작동해야 합니다.

프록시 구성에는 여러 가지 일반적인 방법이 있습니다. 그러나 그들은 네 그룹으로 나뉩니다:

1. 인라인(Inline), 인증 없음(필터링만 적용)
2. 인라인(Inline), 인증 있음
3. 대역 외(Out-of-band), 인증 없음 
4. 대역 외(Out-of-band), 인증 있음 

인라인/비인증 프록시의 경우 아무것도 수행할 필요가 없습니다. Azure SDK는 프록시 구성 없이 작동합니다. 인라인/인증 프록시의 경우, 연결에 `407 Proxy Authentication Required` 상태 코드가 수신될 수 있습니다. 여기에는 스키마, 영역 및 잠재적인 다른 정보(예시: Digest 인증을 위한 `nonce`)가 포함됩니다. 클라이언트 라이브러리는 스키마에 적합하게 인코딩된 인증 정보를 제공하는 `Proxy-Authorization` 헤더를 사용하여 요청을 다시 제출해야 합니다. 가장 일반적인 스키마는 Basic, Digest, 그리고 NTLM입니다. 

대역 외/인증 없음 프록시의 경우, 클라이언트는 전체 요청 URL을 서비스 대신 프록시로 보냅니다. 예를 들어, 클라이언트가 `https://foo.blob.storage.azure.net/path/to/blob` 에 통신하는 경우, 클라이언트는 `HTTPS_PROXY`에 연결하여 `GET https://foo.blob.storage.azure.net/path/to/blob HTTP/1.1` 요청을 보냅니다. 대역 외/인증 프록시의 경우, 클라이언트는 대역 외/인증 없음 프록시 버전처럼 전체 요청 URL을 보내지만, (인라인/인증 프록시와 마찬가지로) `407 Proxy Authentication Required` 상태 코드를 다시 보낼 수 있습니다.

WebSocket은 일반적으로 HTTP 프록시를 통해 터널링될 수 있으며, 이 경우 프록시 인증은 CONNECT 호출 중에 발생합니다. 이는 비 HTTP 트래픽을 Azure 서비스로 터널링하기 위해 자주 사용되는 메커니즘입니다. 한편, 다른 유형의 프록시가 있습니다. 가장 주목할 만한 것은 비 HTTP 트래픽(AMQP 또는 MQTT 등)에 사용되는 SOCKS 프록시입니다. SOCKS에 대한 지원을 추천하지도, 반대하지도 않습니다. 클라이언트 라이브러리 내에서 SOCKS 프록시를 지원하는 것은 명시적으로 요구 사항이 아닙니다.

대부분의 프록시 구성은 모든 Azure 서비스 클라이언트 라이브러리에 공통적인 HTTP 파이프라인을 채택하여 수행됩니다.

{% include requirement/MUST id="azurecore-http-proxy-global-config" %} 플랫폼 또는 런타임 기반으로 구성된 공통 글로벌 구성 지시문을 통해 프록시 구성을 지원하십시오.

- Linux 및 macOS는 일반적으로 `HTTPS_PROXY`(및 관련된) 환경 변수를 사용하여 프록시를 구성합니다.
- Windows 환경은 일반적으로 WinHTTP 프록시 구성을 사용하여 프록시를 구성합니다.
- 다른 옵션(예를 들어, 구성 파일에서 로드)은 플랫폼 및 런타임 기반으로 존재할 수 있습니다.

{% include requirement/MUST id="azurecore-http-proxy-azure-sdk-config" %} 프록시 기능 비활성화를 포함한 프록시 구성에 대한 Azure SDK 전체 구성 지시문을 지원하십시오.

{% include requirement/MUST id="azurecore-http-proxy-client-config" %} 프록시 기능 비활성화를 포함한 프록시 구성에 대한 클라이언트 라이브러리별 구성 지시문을 지원하십시오.

{% include requirement/MUST id="azurecore-http-proxy-logging" %} `407 Proxy Authentication Required` 요청 및 응답 로그를 남기십시오.

{% include requirement/MUST id="azurecore-http-proxy-always-log" %} 프록시 인증이 필요하지 않더라도, 프록시를 통해 요청이 서비스로 전송되고 있는 경우 로깅에 표시하십시오.

{% include requirement/MUST id="azurecore-http-proxy-support" %} Basic 및 Digest 인증 스키마를 지원하십시오.

{% include requirement/SHOULD id="azurecore-http-proxy-ntlm-support" %} NTLM 인증 체계를 지원해야 합니다.

현재 SOCKS를 지원해야 하는 요구 사항은 없습니다. 프록시와의 호환성을 보장하기 위해 서비스에서 WebSocket 연결 옵션(예를 들어, WebSocket을 통한 AMQP 또는 MQTT)을 채택하는 것이 좋습니다.

## 전역 구성

Azure SDK는 다양한 소스로 구성할 수 있으며, 그 중 일부는 반드시 언어에 따라 다릅니다. 이것은 일반적으로 Azure Core 라이브러리에서 코드화됩니다. 구성 소스는 다음과 같습니다:

1. 시스템 설정
2. 환경 변수
3. 전역 구성 저장소(코드)
4. 런타임 매개 변수

{% include requirement/MUST id="azurecore-config-ordering" %} 기본적으로 위의 순서대로 구성을 적용하여, 목록의 후속 항목이 이전 항목의 설정을 재정의하도록 하십시오.

{% include requirement/MAY id="azurecore-config-opt-in" %} 사용자가 위의 순서를 따르지 않는 구성 시스템을 지원할 수 있습니다.

{% include requirement/MUST id="azurecore-config-consistent-naming" %} 환경 변수와 구성 키 사이의 이름을 일관되게 지정하십시오.

{% include requirement/MUST id="azurecore-config-log-config-settings" %} 환경 또는 전역 구성 저장소의 어딘가에서 구성 설정이 발견되면 로그로 남기십시오.

{% include requirement/MAY id="azurecore-config-ignore-irrelevant-config" %} 클라이언트 라이브러리와 무관한 구성 설정은 무시할 수 있습니다.

### 시스템 설정

{% include requirement/SHOULD id="azurecore-config-respect-the-system" %} 프록시에 대한 시스템 설정을 준수해야 합니다.

### 환경 변수

환경 변수는 IT 관리자가 클라우드에서 코드를 실행할 때 기본 설정을 구성할 수 있는 잘 알려진 방법입니다.

{% include requirement/MUST id="azurecore-config-envvars-system-list" %} 아래 표에 나열된 환경 변수에서 관련 구성 설정을 로드하십시오:

{% include tables/environment_variables.md %}

{% include requirement/MUST id="azurecore-config-envvars-azure-prefix" %} Azure 특정 환경 변수에 접두사를 `AZURE_`로 지정하십시오.

{% include requirement/MUST id="azurecore-config-envvars-no-proxy-cidr" %} `NO_PROXY`에 대해 [CIDR notation]를 지원하십시오.

### 전역 구성

전역 구성은 어떤 방식으로든 적용 가능한 모든 클라이언트 생성자에 적용되는 구성 설정을 나타냅니다.

{% include requirement/MUST id="azurecore-config-shared-pipeline-policies" %} 다음을 포함한 공유 파이프라인 정책의 전역 구성을 지원하십시오:

* 로깅: 로그 레벨, 로거 구현 교체(swapping out)
* HTTP: 프록시 설정, 최대 재시도 횟수, 시간 초과, 전송 구현 교체(swapping out)
* 원격 분석: 활성화/비활성화
* 추적: 활성화/비활성화

{% include requirement/MUST id="azurecore-config-override-global-config" %} 시스템 또는 환경 변수에서 상속된 모든 구성 설정을 설정하거나 재정의하기 위한 구성 키를 제공하십시오.

{% include requirement/MUST id="azurecore-config-opt-out" %} 시스템 설정 및 환경 변수를 구성으로 가져오는 것을 거부하는 방법을 제공하십시오.

## 인증 및 자격 증명

MSI(Managed Security Identity) 또는 Azure Identity를 통해 얻는 OAuth 토큰 인증은 서비스 요청을 인증하는 기본 메커니즘이며, Azure Core 라이브러리에서 지원하는 유일한 인증 자격 증명입니다.

{% include requirement/MUST id="azurecore-auth-token-credential" %} 서비스에 대한 요청을 비차단 원자성 방식으로 인증하는 데 필요한 OAuth 호환 토큰을 가져올 수 있는 토큰 자격 증명 유형을 제공하십시오.

{% include refs.md %}

[User-Agent 헤더]: https://tools.ietf.org/html/rfc7231#section-5.5.3
[일시적인 오류 처리]: https://docs.microsoft.com/azure/architecture/best-practices/transient-faults
[OpenTelemetry]: https://opentelemetry.io/
[Azure Monitor]: https://azure.microsoft.com/services/monitor/
[CIDR notation]: https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing
