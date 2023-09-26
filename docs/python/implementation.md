---
title: "Python Guidelines: Implementation"
keywords: guidelines python
permalink: python_implementation.html
folder: python
sidebar: general_sidebar
---

## API 구현

### 서비스 클라이언트

#### Http 파이프라인

클라이언트 라이브러리는 일반적으로 하나 이상의 HTTP 요청을 래핑하기 때문에 표준 네트워크 기능을 지원하는 것이 중요합니다. 널리 알려져 있지는 않지만 탄력적인 웹 서비스를 개발하는 데는 비동기 프로그래밍 기술이 필수적입니다. 많은 개발자들은 기술 사용 방법을 배울 때 쉬운 의미론을 위해 동기식 메서드 호출을 선호합니다. HTTP 파이프라인은 HTTP 기반의 Azure 서비스에 대한 연결을 지원하는 `azure-core` 라이브러리의 구성 요소입니다.

{% include requirement/MUST id="python-network-http-pipeline" %} [HTTP 파이프라인]을 사용하여 서비스 REST endpoints에 요청을 보냅니다.

{% include requirement/SHOULD id="python-network-use-policies" %} HTTP 파이프라인에 다음 정책들을 포함시킵니다:

- Unique Request ID (`azure.core.pipeline.policies.RequestIdPolicy`)
- Headers (`azure.core.pipeline.policies.HeadersPolicy`)
- Telemetry (`azure.core.pipeline.policies.UserAgentPolicy`)
- Proxy (`azure.core.pipeline.policies.ProxyPolicy`)
- Content decoding (`azure.core.pipeline.policies.ContentDecodePolicy`)
- Retry (`azure.core.pipeline.policies.RetryPolicy` and `azure.core.pipeline.policies.AsyncRetryPolicy`)
- Credentials (e.g. `BearerTokenCredentialPolicy`, `AzureKeyCredentialPolicy`, etc)
- Distributed tracing (`azure.core.pipeline.policies.DistributedTracingPolicy`)
- Logging (`azure.core.pipeline.policies.NetworkTraceLoggingPolicy`)

```python

from azure.core.pipeline import Pipeline

from azure.core.pipeline.policies import (
    BearerTokenCredentialPolicy,
    ContentDecodePolicy,
    DistributedTracingPolicy,
    HeadersPolicy,
    HttpLoggingPolicy,
    NetworkTraceLoggingPolicy,
    UserAgentPolicy,
)

class ExampleClient(object):

    ...

    def _create_pipeline(self, credential, base_url=None, **kwargs):
        transport = kwargs.get('transport') or RequestsTransport(**kwargs)

        try:
            policies = kwargs['policies']
        except KeyError:
            scope = base_url.strip("/") + "/.default"
            if hasattr(credential, "get_token"):
                credential_policy = BearerTokenCredentialPolicy(credential, scope)
            else:
                raise ValueError(
                    "Please provide an instance from azure-identity or a class that implement the 'get_token protocol"
                )
            policies = [
                HeadersPolicy(**kwargs),
                UserAgentPolicy(**kwargs),
                ContentDecodePolicy(**kwargs),
                RetryPolicy(**kwargs),
                credential_policy,
                HttpLoggingPolicy(**kwargs),
                DistributedTracingPolicy(**kwargs),
                NetworkTraceLoggingPolicy(**kwargs)
            ]

        return Pipeline(transport, policies)

```

##### 사용자 지정 정책

일부 서비스는 사용자 지정 정책을 구현하도록 요구할 수 있습니다. 예를 들어, 사용자 지정 정책은 재시도, 요청 서명 또는 다른 특수 인증 기법 중에 보조 엔드포인트로 폴백기능을 구현할 수 있습니다.

{% include requirement/SHOULD id="python-pipeline-core-policies" %} 가능하면 `azure-core`의 구현 정책을 준용해야합니다.

{% include requirement/MUST id="python-custom-policy-review" %} Azure SDK [Architecture Board]를 참고하여 제안된 정책을 검토합니다. 당신의 요구를 만족할 수 있는 수정/매개 변수화할 수 있는 기존 정책이 이미 정의되어 있을 수 있습니다.

{% include requirement/MUST id="python-custom-policy-base-class" %} [HTTPPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.HTTPPolicy)/ [AsyncHTTPPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.AsyncHTTPPolicy) (네트워크 호출이 필요한 경우) 또는 [SansIOHTTPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.SansIOHTTPPolicy) (네트워크 호출이 필요하지 않는 경우)에서 파생되어야 합니다.

{% include requirement/MUST id="python-custom-policy-thread-safe" %} 사용자 지정 정책에 대한 스레드 안전성을 보장해야 합니다. 이로 인한 실질적인 결과는 요청별 또는 연결 부기 데이터를 정책 인스턴스 자체가 아닌 컨텍스트에 유지해야 한다는 것입니다.

{% include requirement/MUST id="python-pipeline-document-policies" %} 패키지의 사용자 지정 정책을 문서화해야 합니다. 해당 문서에는 라이브러리의 사용자가 정책을 준용하는 방법이 명확하게 기재되어야 합니다.

{% include requirement/MUST id="python-pipeline-policy-namespace" %} 정책을 `azure.<package name>.pipeline.policies` 네임스페이스에 추가해야 합니다.

#### 서비스 메소드

##### 매개 변수 유효성 검사

{% include requirement/MUSTNOT id="python-client-parameter-validation" %} `isinstance`를 사용하여 [built-in types](https://docs.python.org/3/library/stdtypes.html) (예: 'str' 등) 이외의 매개 변수 값 유형의 유효성을 검사하면 안됩니다. 다른 유형의 경우 [structural type checking]를 사용해야 합니다.

### 지원하는 타입

{% include requirement/MUST id="python-models-repr" %} 모델 타입에 대해 `__repr__`을 구현해야 합니다. 표현에는 **반드시** 타입 이름과 모든 키 속성(즉, 모델 인스턴스 식별에 도움이 되는 속성)이 포함되어야 합니다.

{% include requirement/MUST id="python-models-repr-length" %}  `__repr__`의 출력중 1024자 이후로 넘어가는 부분은 생략해야합니다.

##### 확장 가능한 열거형(Enumerations)

SDK에 정의된 모든 Enum은 대소문자를 구분하지 않는 문자열로 교체될 수 있어야 합니다. 이는 `azure-core`에 정의된 `CaseInsensitiveEnumMeta` 클래스를 사용하여 수행됩니다.

```python
from enum import Enum
from six import with_metaclass

from azure.core import CaseInsensitiveEnumMeta

class MyCustomEnum(with_metaclass(CaseInsensitiveEnumMeta, str, Enum)):
    FOO = 'foo'
    BAR = 'bar'
```

### SDK 기능 구현

#### 구성

{% include requirement/MUST id="python-envvars-global" %} 글로벌 구성 설정에 대해 다음 환경 변수를 준수해야 합니다:

{% include tables/environment_variables.md %}

#### 로깅

{% include requirement/MUST id="python-logging-usage" %} Python 표준 [logging module](https://docs.python.org/3/library/logging.html) 을 준용해야 합니다.

{% include requirement/MUST id="python-logging-nameed-logger" %} 라이브러리에 이름이 지정된 로거를 제공해야 합니다.

패키지의 로거는 **반드시** 모듈 이름을 사용해야 합니다. 라이브러리에서 하위 로거를 추가로 제공할 수 있습니다. 하위 로거가 제공된 경우 문서화하십시오.

예시:
- 패키지 이름: `azure-someservice`
- 모듈 이름: `azure.someservice`
- 로거 이름: `azure.someservice`
- 하위 로거: `azure.someservice.achild`

이러한 명명 규칙을 통해 사용자는 모든 Azure 라이브러리, 특정 클라이언트 라이브러리 또는 클라이언트 라이브러리의 하위 집합에 대한 로깅을 사용할 수 있습니다.

{% include requirement/MUST id="python-logging-error" %} 응용 프로그램이 복구될 가능성이 낮은 오류(예: 메모리 부족)에 대해서는 `ERROR` 로깅 수준을 사용해야 합니다.

{% include requirement/MUST id="python-logging-warn" %} 함수가 의도한 작업을 수행하지 못할 경우 `WARNING` 로깅 수준을 사용해야 합니다. 또한 함수는 예외를 발생시켜야 합니다.

자가 복구 이벤트가 발생하는 경우(예를 들어, 자동으로 요청이 재시도되는 경우)는 포함하지 않습니다.

{% include requirement/MUST id="python-logging-info" %} 함수가 정상적으로 작동할 때는 `INFO` 로깅 수준을 사용해야 합니다.

{% include requirement/MUST id="python-logging-debug" %} 상세한 트러블 슈팅 시나리오에는 `DEBUG` 로깅 레벨을 사용해야 합니다.

`DEBUG` 로깅 수준은 개발자나 시스템 관리자가 특정 장애를 진단할 수 있도록 하기 위한 것입니다.

{% include requirement/MUSTNOT id="python-logging-sensitive-info" %} 민감한 정보를 `DEBUG` 이외의 로그 수준으로 보내지 마십시오. 예를 들어, 헤더 로그를 기록할 때 계정 키를 수정하거나 제거하십시오.

{% include requirement/MUST id="python-logging-request" %} 송신 요청에 대한 요청 라인, 응답 라인 및 헤더를 `INFO` 메시지로 기록해야 합니다.

{% include requirement/MUST id="python-logging-cancellation" %} 서비스 호출이 취소된 경우 `INFO` 메시지를 기록해야 합니다.

{% include requirement/MUST id="python-logging-exceptions" %} 로그 예외는 `WARNING` 레벨 메시지로 호출됩니다. 로그 레벨이 `DEBUG`로 설정되어 있으면 메시지에 스택 트레이스 정보를 추가해야 합니다.

[`logging.Logger.isEnabledFor`](https://docs.python.org/3/library/logging.html#logging.Logger.isEnabledFor).를 호출하면 해당 로거에 대한 로깅 수준을 확인할 수 있습니다.

#### 분산 추적

{% include requirement/MUST id="python-tracing-span-per-method" %} 라이브러리 메서드 호출마다 새 추적 범위(span)를 만들어야 합니다. 가장 쉬운 방법은 `azure.core.tracing`에서 분산 추적 데코레이터를 추가하는 것입니다.

{% include requirement/MUST id="python-tracing-span-name" %} 해당 span의 이름으로 `<package name>/<method name>`을 사용해야 합니다.

{% include requirement/MUST id="python-tracing-span-per-call" %} 각 발신 네트워크 호출에 대해 새로운 span을 만들어야 합니다. HTTP 파이프라인을 사용하는 경우, 새 span이 사용자에게 생성됩니다.

{% include requirement/MUST id="python-tracing-propagate" %} 각 발신 서비스 요청에 대해 추적 컨텍스트를 전파해야 합니다.

#### 원격 측정

클라이언트 라이브러리 사용 원격 측정은 (소비자가 아닌) 서비스 팀에서 클라이언트가 자신의 서비스를 호출하기 위해 사용하는 SDK 언어, 클라이언트 라이브러리 버전 및 언어/플랫폼 정보를 모니터링하는 데 사용됩니다. 클라이언트는 클라이언트 응용 프로그램의 이름과 버전을 나타내는 추가 정보를 첨부할 수 있습니다.

{% include requirement/MUST id="python-http-telemetry-useragent" %} 다음 형식을 사용하여 [User-Agent header]에 원격 측정 정보를 전송해야 합니다:

```
[<application_id> ]azsdk-python-<package_name>/<package_version> <platform_info>
```

- `<application_id>`: 선택적 응용 프로그램별 문자열입니다. 슬래시를 포함할 수 있지만 공백을 포함할 수는 없습니다. "AzCopy/10.0.4-Preview" 같은 문자열은 클라이언트 라이브러리의 사용자에 의해 제공됩니다.
- `<package_name>`: 슬래시를 대시로 바꾸고 Azure 지시자를 제거하는 클라이언트 라이브러리(배포) 패키지 이름입니다. 예를 들어 "azure-keyvault-secrets"는 "azsdk-python-keyvault-secrets"를 지정합니다.
- `<package_version>`: 패키지 버전입니다. 참고: 서비스 버전이 아닙니다.
- `<platform_info>`: 현재 실행 중인 언어 런타임 및 OS에 대한 정보입니다(예시: "Python/3.8.4 (Windows-10-10.0.19041-SP0)").

예를 들어, Azure Blob Storage 클라이언트 라이브러리를 사용하여 Python으로 `AzCopy`를 다시 작성한 경우 다음과 같은 사용자-에이전트(user-agent) 문자열로 끝날 수 있습니다:

- (Python) `AzCopy/10.0.4-Preview azsdk-python-storage/4.0.0 Python/3.7.3 (Ubuntu; Linux x86_64; rv:34.0)`

`azure.core.pipeline.policies.UserAgentPolicy`가 HttpPipeline에 추가되면 이 기능을 제공합니다.

{% include requirement/SHOULD id="python-azurecore-http-telemetry-dynamic" %} `X-MS-AZSDK-Telemetry` 헤더에서 세미콜론으로 구분된 키-값 유형의 추가적인 (동적) 텔레메트리 정보를 전송해야 합니다. 예시:

```http
X-MS-AZSDK-Telemetry: class=BlobClient;method=DownloadFile;blobType=Block
```

헤더의 내용은 세미콜론 키=값 리스트입니다. 다음의 키는 구체적인 의미를 갖습니다:

* `class`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 내의 타입 이름입니다.
* `method`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 타입 내 메서드의 이름입니다.

사용되는 다른 모든 키는 특정 서비스에 대한 모든 클라이언트 라이브러리에서 공통되어야 합니다. 이 헤더에는 개인 식별 정보(인코딩된 정보더라도)를 포함하지 **마십시오**. 서비스들은 정상적인 분석 시스템을 통해 조회할 수 있도록 `X-MS-SDK-Telemetry` 헤더를 캡처하도록 로그 수집을 구성해야 합니다.

##### azure-core에서 UserAgentPolicy를 사용하지 않는 클라이언트에 대한 고려 사항

{% include requirement/MUST id="python-azurecore-http-telemetry-appid" %} 라이브러리의 소비자가 `application_id` 파라미터를 서비스 클라이언트 구성자에게 전달하여 애플리케이션 ID를 설정할 수 있도록 해야 합니다. 이를 통해 소비자는 자신의 앱에 대한 교차 서비스 원격 측정을 얻을 수 있습니다.

{% include requirement/MUST id="python-azurecore-http-telemetry-appid-length" %} 응용프로그램 ID의 길이가 24자 이하가 되도록 강제해야 합니다. 짧은 응용프로그램 ID를 사용하면 소비자가 자신의 응용프로그램에 대한 원격 측정 정보를 얻는 것을 허용하면서 서비스 팀이 사용자 에이전트의 "플랫폼 정보" 섹션에 진단 정보를 포함할 수 있습니다.

## 테스팅

{% include requirement/MUST id="python-testing-pytest" %} [pytest](https://docs.pytest.org/en/latest/)를 테스트 프레임워크로 사용해야 합니다.

{% include requirement/SHOULD id="python-testing-async" %} 비동기 코드를 테스트하려면 [pytest-asyncio](https://github.com/pytest-dev/pytest-asyncio)를 사용해야 합니다.

{% include requirement/MUST id="python-testing-live" %} 라이브 서비스들에 대해 시나리오 테스트를 실행할 수 있도록 해야 합니다. 시나리오 테스트를 위해서는 [Python Azure-DevTools](https://github.com/Azure/azure-sdk-for-python/tree/main/tools/azure-devtools) 패키지를 사용하는 것을 강력히 고려합니다.

{% include requirement/MUST id="python-testing-record" %} 오프라인/Azure 구독 없이 테스트를 실행할 수 있도록 기록을 제공해야 합니다.

{% include requirement/MUST id="python-testing-parallel" %} 동일한 구독에서 동시 테스트 실행을 지원해야 합니다.

{% include requirement/MUST id="python-testing-independent" %} 각 테스트 케이스를 다른 테스트와는 독립적으로 만들어야 합니다.

## 코드 분석 및 스타일 도구

{% include requirement/MUST id="python-tooling-pylint" %} 코드 정적 분석을 위해 [pylint](https://www.pylint.org/)를 사용해야 합니다. [repository의 root](https://github.com/Azure/azure-sdk-for-python/blob/main/pylintrc) 에서 pylintrc 파일을 사용합니다.

{% include requirement/MUST id="python-tooling-flake8" %} 문서 comment를 확인하려면 [flake8-docstrings](https://gitlab.com/pycqa/flake8-docstrings)를 사용해야 합니다.

{% include requirement/MUST id="python-tooling-black" %} 코드를 포매팅하려면 [Black](https://black.readthedocs.io/en/stable/)을 사용해야 합니다.

{% include requirement/SHOULD id="python-tooling-mypy" %} [MyPy](https://mypy.readthedocs.io/en/latest/)를 사용하여 라이브러리의 공용 노출 영역을 정적으로 확인해야 합니다.

테스트와 같은 비포함 코드는 코드 분석을 하지 않으셔도 됩니다.

## Azure Core 사용하기

`azure-core` 패키지는 클라이언트 라이브러리에 공통 기능을 제공합니다. 문서 및 사용 예시는 [azure/azure-sdk-for-python] 저장소에서 확인할 수 있습니다.

### HTTP 파이프라인

HTTP 파이프라인은 여러 정책으로 래핑되는 HTTP 전송책입니다. 각 정책은 요청 또는 응답을 수정할 수 있는 컨트롤 포인트입니다. 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방법을 표준화하기 위해 기본 정책 집합이 제공됩니다.

Python을 활용한 파이프라인 구현에 대한 자세한 내용은 [문서](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/core/azure-core)를 참조하십시오.

### 프로토콜

디자인 가이드라인에 의해 의무화된 프로토콜들 중 많은 부분이 `azure-core`에 기본적으로 구현이 되어 있습니다.

#### LROPoller

```python
T = TypeVar("T")
class LROPoller(Protocol):

    def result(self, timeout=None) -> T:
        """ Retrieve the final result of the long running operation.

        :param timeout: How long to wait for operation to complete (in seconds). If not specified, there is no timeout.
        :raises TimeoutException: If the operation has not completed before it timed out.
        ---
        장시간 실행 작업의 최종 결과를 가져옵니다.

        :param timeout: 작업이 완료될 때까지 기다리는 시간(단위 : 초). 지정하지 않으면 타임아웃이 발생하지 않습니다.
        :raises TimeoutException:타임아웃 전에 작업이 완료되지 않은 경우 발생.
        """
        ...

    def wait(self, timeout=None) -> None:
        """ Wait for the operation to complete.

        :param timeout: How long to wait for operation to complete (in seconds). If not specified, there is no timeout.
        ---
        작업이 완료될 때까지 기다립니다.

        :param timeout: 작업이 완료될 때까지 기다리는 시간(단위 : 초). 지정하지 않으면 타임아웃이 발생하지 않습니다.
        """

    def done(self) -> boolean:
        """ Check if long running operation has completed.
        ---
        long running 작업이 완료되었는지 확인합니다.
        """

    def add_done_callback(self, func) -> None:
        """ Register callback to be invoked when operation completes.

        :param func: Callable that will be called with the eventual result ('T') of the operation.
        ---
        작업이 완료되면 호출할 콜백을 등록합니다.

        :param func: 연산의 결과('T')와 함께 호출될 호출 가능.
        """
        ...
```

`azure.core.polling.LROPoller`'는 `LROPoller` 프로토콜을 구현합니다.

#### ItemPaged

```python
T = TypeVar("T")
class ByPagePaged(Protocol, Iterable[Iterable[T]]):
    continuation_token: "str"

class ItemPaged(Protocol, Iterable[T]):
    continuation_token: "str"

    def by_page(self) -> ByPagePaged[T] ...
```

`azure.core.ItemPaged`는 `ItemPaged` 프로토콜을 구현합니다.

자세한 내용은 [ItemPaged](#PythonPagingDesign) 프로토콜을 참조하십시오.

#### DiagnosticsResponseHook

```python
class ResponseHook(Protocol):

    __call__(self, headers, deserialized_response): -> None ...

```

## 파이썬 언어와 코드 스타일

{% include requirement/MUST id="python-codestyle-pep8" %} 본 문서에서 명시적으로 재정의하지 않는 한 [PEP8](https://www.python.org/dev/peps/pep-0008/)의 일반 지침을 따라야 합니다.

{% include requirement/MUSTNOT id="python-codestyle-idiomatic" %} 코딩 패러다임을 다른 언어에서 "변환"해서는 안 됩니다.

예를 들어, 자바 커뮤니티에서 Reactive 프로그래밍이 아무리 일반적이라고 해도, 대부분의 Python 개발자들에게는 아직 생소한 패러다임입니다.

{% include requirement/MUST id="python-codestyle-consistency" %} 동일한 서비스에 대해서는 다른 라이브러리보다 다른 Python 구성 요소와의 일관성을 선호해야 합니다.

개발자는 여러 언어의 동일한 서비스를 사용하는 것보다 동일한 언어를 사용하는 여러 다른 라이브러리를 사용할 가능성이 더 높습니다.

### 오류 처리

{% include requirement/MUST id="python-errors-use-chaining" %} 새로운 예외를 catch 하고 raise 할 때는 예외 체인을 사용하여 오류의 원본을 포함시켜야 합니다.

```python
# Yes:
try:
    # do something
    something()
except:
    # __context__ will be set correctly
    raise MyOwnErrorWithNoContext()

# No:
success = True
try:
    # do something
    something()
except:
    success = False
if not success:
    # __context__ is lost...
    raise MyOwnErrorWithNoContext()
```

### 명명 규칙

{% include requirement/MUST id="python-codestyle-vars-naming" %} 변수, 함수 및 메서드 이름에는 snake_case를 사용해야 합니다:

```python
# Yes:
service_client = ServiceClient()

service_client.list_things()

def do_something():
    ...

# No:
serviceClient = ServiceClient()

service_client.listThings()

def DoSomething():
    ...
```

{% include requirement/MUST id="python-codestyle-type-naming" %} Type의 경우 Pascal 케이스를 사용해야 합니다:

```python
# Yes:
class ThisIsCorrect(object):
    pass

# No:
class this_is_not_correct(object):
    pass

# No:
class camelCasedTypeName(object):
    pass
```

{% include requirement/MUST id="python-codestyle-const-naming" %} 상수의 경우 모두 대문자를 사용해야 합니다:

```python
# Yes:
MAX_SIZE = 4711

# No:
max_size = 4711

# No:
MaxSize = 4711
```

{% include requirement/MUST id="python-codestyle-module-naming" %} 모듈 이름에는 snake_case를 사용해야 합니다.

### 메서드 시그니처

{% include requirement/MUSTNOT id="python-codestyle-static-methods" %} 정적 메서드 ([`staticmethod`](https://docs.python.org/3/library/functions.html#staticmethod)) 을 사용해서는 안 됩니다. 모듈 수준의 기능을 선호하십시오.

정적 메서드는 드물고 일반적으로 다른 라이브러리에 의해 강제됩니다.

{% include requirement/MUSTNOT id="python-codestyle-properties" %} 단순 getter 및 setter 함수를 사용해서는 안됩니다. 대신, 속성을 사용하세요.

```python
# Yes
class GoodThing(object):

    @property
    def something(self):
        """ Example of a good read-only property."""
        return self._something

# No
class BadThing(object):

    def get_something(self):
        """ Example of a bad 'getter' style method."""
        return self._something
```

{% include requirement/SHOULDNOT id="python-codestyle-long-args" %} 5개 이상의 위치 매개 변수를 필요로 하는 메서드를 사용하면 안 됩니다. 키워드 전용 인수 또는 `**kwargs`를 사용하여 옵션/플래그 매개 변수를 허용할 수 있습니다.

이 위치 파라미터와 옵션 파라미터에 대한 일반적인 지침은 TODO: 삽입 링크를 참조하십시오.

{% include requirement/MUST id="python-codestyle-optional-args" %} Python 3만 지원하면 되는 모듈의 선택적이거나 자주 사용되지 않는 인수에는 키워드 전용 인수(keyword-only arguments)를 사용해야 합니다.

```python
# Yes
def foo(a, b, *, c, d=None):
    # Note that I can even have required keyword-only arguments...
    ...
```

{% include requirement/MUST id="python-codestyle-kwargs" %} 명확한 순서가 없는 인수에는 키워드 전용 인수를 사용해야 합니다.

```python
# Yes - `source` and `dest` have logical order, `recurse` and `overwrite` do not.
def copy(source, dest, *, recurse=False, overwrite=False) ...


# No
def copy(source, dest, recurse=False, overwrite=False) ...
```

{% include requirement/MUST id="python-codestyle-positional-params" %} 필요한 위치 매개변수가 두 개 이상인 메서드를 호출할 때 매개변수 이름을 지정해야 합니다.

```python
def foo(a, b, c):
    pass


def bar(d, e):
    pass


# Yes:
foo(a=1, b=2, c=3)
bar(1, 2)
bar(e=3, d=4)

# No:
foo(1, 2, 3)
```

{% include requirement/MUST id="python-codestyle-optional-param-calling" %} 함수를 호출할 때 옵션 파라미터의 파라미터 이름을 지정해야 합니다.

```python
def foo(a, b=1, c=None):
    pass


# Yes:
foo(1, b=2, c=3)

# No:
foo(1, 2, 3)
```

### Public vs "private"

{% include requirement/MUST id="python-codestyle-private-api" %} 이름이 공용 API의 일부가 아님을 나타내려면 선행 밑줄 하나를 사용해야 합니다. 공용이 아닌 API는 안정성이 보장되지 않습니다.

{% include requirement/MUSTNOT id="python-codestyle-double-underscore" %} 상속 계층에서 이름이 충돌할 가능성이 없는 한 앞에 오는 이중 밑줄 접두사 메서드 이름을 사용해서는 안 됩니다. 이름이 충돌하는 경우는 거의 없습니다.

{% include requirement/MUST id="python-codestyle-public-api" %} 모듈의 `__all__` 속성에 공개 메서드 및 유형을 추가해야 합니다.

{% include requirement/MUST id="python-codestyle-interal-module" %} 내부 모듈의 경우 선행 밑줄을 사용해야 합니다. 모듈이 내부 모듈의 하위 모듈인 경우 선행 밑줄을 **생략할 수도** 있습니다.

```python
# Yes:
azure.exampleservice._some_internal_module

# Yes - some_internal_module은 내부 모듈의 하위 모듈이므로 여전히 내부로 간주됩니다:
azure.exampleservice._internal.some_internal_module

# No - some_internal_module은 공개모듈로 간주됩니다:
azure.exampleservice.some_internal_module
```

### 타입 (or not)

{% include requirement/MUST id="python-codestyle-structural-subtyping" %} 명시적 타입 검사보다 구조적 하위 타입 및 프로토콜을 선호해야 합니다.

{% include requirement/MUST id="python-codestyle-abstract-collections" %} 사용자 지정 매핑 타입을 제공하려면 추상 컬렉션 기본 클래스 `collections.abc`(또는 Python 2.7의 `collections`)에서 파생해야 합니다.

{% include requirement/MUST id="python-codestyle-pep484" %} 공개 문서화된 클래스 및 기능에 대한 타입 힌트 [PEP484](https://www.python.org/dev/peps/pep-0484/) 를 제공해야 합니다.

- Python 2.7 호환 코드에 대한 지침은 [suggested syntax for Python 2.7 and 2.7-3.x straddling code](https://www.python.org/dev/peps/pep-0484/#suggested-syntax-for-python-2-7-and-straddling-code)를 참조하십시오. Python 3의 특정 코드(예를 들어: `async` 클라이언트)에서는 해당 코드 지침을 참조하지 마십시오.

### Threading

{% include requirement/MUST id="python-codestyle-thread-affinity" %} 문서에서 개별적으로 제한하지 않는 한, 사용자가 제공한 콜백에 대해 thread affinity를 유지해야 합니다.

{% include requirement/MUST id="python-codestyle-document-thread-safety" %} 메서드(함수/클래스)가 thread-safe하다는 사실을 문서에 명시적으로 포함해야 합니다.

예시: [`asyncio.loop.call_soon_threadsafe`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_soon_threadsafe), [`queue`](https://docs.python.org/3/library/queue.html)

{% include requirement/SHOULD id="python-codestyle-use-executor" %} 병렬 처리를 위해 사용자 자신의 스레드나 프로세스 관리를 정의하는 것이 아니라 [`Executor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Executor) 인스턴스를 호출자가 통과할 수 있도록 허용해야 합니다.

만약 스레드가 호출자에게 어떤 방식으로든 노출되지 않는다면 자신의 스레드 관리를 할 수 있습니다. 예를 들어, `LROPoller` 구현에서는 백그라운드 폴러 스레드를 사용합니다.

{% include refs.md %}
{% include_relative refs.md %}
