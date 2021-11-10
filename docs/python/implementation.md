---
title: "Python 가이드라인: 구현"
keywords: guidelines python
permalink: python_implementation.html
folder: python
sidebar: general_sidebar
---

## API 구현

### 서비스 클라이언트

#### Http 파이프라인

클라이언트 라이브러리는 일반적으로 하나 이상의 HTTP 요청을 래핑하므로 표준 네트워크 기능을 지원하는 것이 중요합니다. 널리 이해되지는 않지만, 비동기 프로그래밍 기술은 탄력적인 웹 서비스를 개발하는 데 필수적입니다. 많은 개발자들은 기술 사용법을 학습할때 동기식 방법을 더 선호합니다. HTTP pipeline은 HTTP-기반 Azure 서비스에 대한 연결을 제공하는 데 도움을 주는 `azure-core` 라이브러리의 구성 요소입니다.

{% include requirement/MUST id="python-network-http-pipeline" %} [HTTP pipeline]을 사용하여 서비스 REST 엔드포인트에 요청을 보내십시오.

{% include requirement/SHOULD id="python-network-use-policies" %} 다음의 정책을 HTTP 파이프라인에 포함하십시오:

- 고유 요청 ID (`azure.core.pipeline.policies.RequestIdPolicy`)
- 헤더 (`azure.core.pipeline.policies.HeadersPolicy`)
- 원격 분석 (`azure.core.pipeline.policies.UserAgentPolicy`)
- 프록시 (`azure.core.pipeline.policies.ProxyPolicy`)
- 콘텐츠 디코딩 (`azure.core.pipeline.policies.ContentDecodePolicy`)
- 재시도 (`azure.core.pipeline.policies.RetryPolicy` 와 `azure.core.pipeline.policies.AsyncRetryPolicy`)
- 자격 증명 (e.g. `BearerTokenCredentialPolicy`, `AzureKeyCredentialPolicy`, 등)
- 분산 추적 (`azure.core.pipeline.policies.DistributedTracingPolicy`)
- 로깅 (`azure.core.pipeline.policies.NetworkTraceLoggingPolicy`)

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

일부 서비스에는 사용자 지정 정책을 구현해야 합니다. 예를 들어 사용자 지정 정책이 재시도, 서명 요청 혹은 기타 특수 인증 기술에 보조 끝점으로의 대체를 구현할 수 있습니다.

{% include requirement/SHOULD id="python-pipeline-core-policies" %} 가능하면 `azure-core`에서 정책 구현을 사용하십시오.

{% include requirement/MUST id="python-custom-policy-review" %} 제안된 정책을 Azure SDK [Architecture Board]에서 리뷰하십시오. 사용자의 요구를 충족하기 위해 수정/매개 변수화할 수 있는 기존 정책이 이미 존재할 수도 있습니다.

{% include requirement/MUST id="python-custom-policy-base-class" %} [HTTPPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.HTTPPolicy)/[AsyncHTTPPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.AsyncHTTPPolicy) (네트워크 호출을 해야하는 경우) 또는 [SansIOHTTPPolicy](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/azure.core.pipeline.policies.html#azure.core.pipeline.policies.SansIOHTTPPolicy) (그렇지 않은 경우)에서 파생.

{% include requirement/MUST id="python-custom-policy-thread-safe" %} 사용자 지정 정책을 위한 스레드 보안을 보장하십시오. 이것의 실질적인 결과는 정책 인스턴스 자체가 아니라 컨텍스트에서 요청당 또는 연결 부기 데이터를 유지해야합니다.

{% include requirement/MUST id="python-pipeline-document-policies" %} 패키지에 있는 사용자 지정 정책을 문서화하십시오. 설명서에 사용자가 정책을 어떻게 사용해야 하는지 명확히 작성해야합니다.

{% include requirement/MUST id="python-pipeline-policy-namespace" %} 네임스페이스에 `azure.<package name>.pipeline.policies`와 같이 사용할 수 있도록 정책을 추가하십시오.

#### 서비스 메서드

##### 매개 변수 유효성 검사

{% include requirement/MUSTNOT id="python-client-parameter-validation" %} [built-in types](https://docs.python.org/3/library/stdtypes.html) (예를 들어 `str` 등) 이외의 매개 변수 값 유형의 유효성을 확인하려면 `isinstance`를 사용하십시오. 다른 유형들은 [structural type checking]을 사용하십시오.

### 지원 타입

{% include requirement/MUST id="python-models-repr" %} 모델 타입에 대해 `__repr__`을 구현하십시오. 표시에는 **꼭** 유형의 이름과 모든 키 속성(즉, 모델 인스턴스를 식별하는데 도움이 되는 속성)이 포함되어야 합니다.

{% include requirement/MUST id="python-models-repr-length" %} `__repr__`의 출력내용이 1024자를 넘길 경우 출력을 잘라내십시오.

##### 확장 가능 열거형

SDK에 정의된 모든 열거형은 대소문자를 구분하지 않는 문자열과 바꿀 수 있어야합니다. 이는 `azure-core`에서 정의된 `CaseInsensitiveEnumMeta` 클래스를 사용하여 이루어집니다.

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

{% include requirement/MUST id="python-envvars-global" %} 전역 구성 설정에 대해 다음 환경 변수를 준수하여 주십시오 :

{% include tables/environment_variables.md %}

#### 로깅

{% include requirement/MUST id="python-logging-usage" %} Python 표준인 [logging module](https://docs.python.org/3/library/logging.html)을 사용하십시오.

{% include requirement/MUST id="python-logging-nameed-logger" %} 라이브러리에 대해 이름이 지정된 로거를 사용하십시오

패키지의 로거는 **꼭** 모듈 이름을 사용해야합니다. 라이브러리는 추가적으로 하위 로거를 제공할 수 있습니다. 하위 로거가 제공된 경우 그것을 명시하십시오.

예를 들어:
- Package 이름: `azure-someservice`
- Module 이름: `azure.someservice`
- Logger 이름: `azure.someservice`
- Child 로거: `azure.someservice.achild`

이 규칙을 통해 소비자는 모든 Azure 라이브러리, 특정 클라이언트 라이브러리, 혹은 클라이언트의 하위 집합에 대한 로깅을 사용할 수 있습니다.

{% include requirement/MUST id="python-logging-error" %} 어플리케이션이 복구될 가능성이 낮을 경우(예: 메모리부족) `ERROR` 로깅 수준을 사용해야 합니다.

{% include requirement/MUST id="python-logging-warn" %} 함수가 의도된 작업을 수행하지 못한 경우 `WARNING` 로깅 수준을 사용하십시오. 함수는 예외를 발생시켜야합니다.

자동 복구 이벤트 발생을 포함하지 마십시오(예: 요청이 자동으로 다시 시작되는 경우).

{% include requirement/MUST id="python-logging-info" %} 함수가 정상적으로 작동될 시 `INFO` 로깅 수준을 사용하십시오.

{% include requirement/MUST id="python-logging-debug" %} 자세한 문제 해결 시나리오는 `DEBUG` 로깅 수준을 사용하십시오.

`DEBUG` 로깅 수준을 개발자 또는 시스템 관리자가 특정 오류를 진단하기 위한 것입니다.

{% include requirement/MUSTNOT id="python-logging-sensitive-info" %} `DEBUG` 이외의 로그 수준에서 중요한 정보를 보내지 말아야 합니다. 예시: 헤더를 로깅할 때 계정키를 수정 혹은 삭제합니다.

{% include requirement/MUST id="python-logging-request" %} 발신 요청에 대한 요청 라인, 응답 라인 및 헤더를 'INFO' 메시지로 기록해야 합니다.

{% include requirement/MUST id="python-logging-cancellation" %} 서비스 호출이 취소되었을 경우 `INFO` 메세지를 기록해야 합니다.

{% include requirement/MUST id="python-logging-exceptions" %} `WARNING` 수준 메시지로 Throw된 예외를 기록해야 합니다. 만약 `DEBUG`로 로그 레벨이 설정되어 있다면, 스택 추적 정보를 메시지에 추가합니다.

[`logging.Logger.isEnabledFor`](https://docs.python.org/3/library/logging.html#logging.Logger.isEnabledFor)를 호출하여 주어진 로거에 대한 로깅 레벨을 결정할 수 있습니다.

#### 분산 추적

{% include requirement/MUST id="python-tracing-span-per-method" %} 각 라이브러리 메서드 호출에 대해서 새 추적 범위를 만들어야 합니다. 가장 쉬운 방법은 `azure.core.tracing`의 분산 트레이싱 데코레이터를 추가하는 것입니다.

{% include requirement/MUST id="python-tracing-span-name" %} 스팬의 이름으로 `<package name>/<method name>`를 사용해야합니다.

{% include requirement/MUST id="python-tracing-span-per-call" %} 나가는 네트워크 호출마다 새 스팬을 만들어야 합니다. HTTP 파이프라인을 사용한다면 새로운 스팬이 생성됩니다.

{% include requirement/MUST id="python-tracing-propagate" %} 나가는 서비스 요청마다 추적 컨텍스트를 전파해야 합니다.

#### 원격 분석

클라이언트 라이브러리 사용 원격 측정 기능은 소비자가 아닌 서비스 팀에서 클라이언트가 서비스에 호출하는 데 사용되는 SDK 언어, 클라이언트 라이브러리 버전 및 언어/플랫폼 정보를 모니터링하는 데 사용됩니다. 클라이언트는 클라이언트 응용 프로그램의 이름과 버전을 나타내는 정보를 추가할 수 있습니다.

{% include requirement/MUST id="python-http-telemetry-useragent" %} 다음 방식을 사용하여 [User-Agent header]로 원격 측정 정보 전송하십시오:

```
[<application_id> ]azsdk-python-<package_name>/<package_version> <platform_info>
```

- `<application_id>`: 선택적 응용 프로그램별 문자열입니다. 슬래시를 포함할 수 있지만 공백을 포함할 수 없습니다. 이 문자열은 클라이언트 라이브러리의 사용자에 의해 제공됩니다(예: "AzCopy/10.0.4-Preview").
- `<package_name>`: 슬래시를 대시로 바꾸고 Azure 표시기를 제거하여 개발자에게 표시되는 클라이언트 라이브러리(배포) 패키지 이름입니다. 예를 들어 "azure-keyvault-secrets"는 "azsdk-python-keyvault-secrets"를 지정합니다.
- `<package_version>`: 패키지의 버전입니다. 메모: 이것은 서비스의 버전이 아닙니다.
- `<platform_info>`: 현재 실행 중인 언어 런타임 및 OS에 대한 정보(예: "Python/3.8.4 (Windows-10.0.19041-SP0)"

예를 들어 Azure Blob Storage 클라이언트 라이브러리를 사용하여 파이썬에서 'AzCopy'를 다시 작성하면 다음과 같은 사용자-에이전트 문자열이 생성될 수 있습니다:

- (Python) `AzCopy/10.0.4-Preview azsdk-python-storage/4.0.0 Python/3.7.3 (Ubuntu; Linux x86_64; rv:34.0)`

`azure.core.pipeline.policies.UserAgentPolicy`는 HttpPipeline에 추가되는 경우 기능을 제공합니다.

{% include requirement/SHOULD id="python-azurecore-http-telemetry-dynamic" %} 추가 (동적) 원격측정 정보를 'X-MS-AZSDK-원격측정' 헤더에서 키-값 유형의 세미콜론으로 구분된 집합으로 전송합니다. 예를 들어:

```http
X-MS-AZSDK-Telemetry: class=BlobClient;method=DownloadFile;blobType=Block
```

헤더의 내용은 세미콜론 키=값 목록입니다. 다음의 키는 특정한 의미를 가지고 있습니다:

* `class`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 내 유형의 이름입니다.
* `method`는 소비자가 네트워크 작업을 트리거하기 위해 호출한 클라이언트 라이브러리 유형 내의 메서드 이름입니다.

사용되는 다른 모든 키는 특정 서비스에 대한 모든 클라이언트 라이브러리에서 공통적이어야 합니다. 이 헤더에 개인 식별 정보(인코딩된 경우도 포함)를 **포함하지 않아야**합니다. 서비스는 일반 분석 시스템을 통해 쿼리할 수 있는 방식으로 'X-MS-SDK-Telemetry' 헤더를 캡처할 수 있게 로그 수집을 구성해야 합니다.

##### azure-core에서 UserAgentPolicy를 사용하지 않는 클라이언트에 대한 고려 사항

{% include requirement/MUST id="python-azurecore-http-telemetry-appid" %} 라이브러리 소비자가 'application_id' 매개변수를 서비스 클라이언트 생성자에게 전달하여 애플리케이션 ID를 설정하도록 허용해야 합니다. 이를 통해 소비자는 앱에 대한 교차 서비스 원격 분석을 얻을 수 있습니다.

{% include requirement/MUST id="python-azurecore-http-telemetry-appid-length" %} 응용 프로그램 ID의 길이는 24자를 넘지 않도록 해야 합니다. 짧은 애플리케이션 ID는 서비스 팀이 사용자 에이전트의 "플랫폼 정보" 섹션에 진단 정보를 포함시킬 수 있게하며, 소비자는 여전히 자신의 애플리케이션에 대한 원격 측정 정보를 얻을 수 있습니다.

## Testing

{% include requirement/MUST id="python-testing-pytest" %} 테스트 프레임워크로 [pytest](https://docs.pytest.org/en/latest/)를 사용하십시오.

{% include requirement/SHOULD id="python-testing-async" %} 비동기 코드 테스트로 [pytest-asyncio](https://github.com/pytest-dev/pytest-asyncio)를 사용하십시오.

{% include requirement/MUST id="python-testing-live" %} 실시간 서비스에 대하여 시나리오 테스트를 실행할 수 있도록 하십시오. 시나리오 테스트에 [Python Azure-DevTools](https://github.com/Azure/azure-sdk-for-python/tree/main/tools/azure-devtools)을 사용하는 것을 강력히 권장합니다.

{% include requirement/MUST id="python-testing-record" %} Azure 구독 없이 오프라인에서 테스트를 실행할 수 있는 레코딩을 제공하십시오.

{% include requirement/MUST id="python-testing-parallel" %} 동일한 구독에서 동시 테스트 실행을 지원하십시오.

{% include requirement/MUST id="python-testing-independent" %} 테스트 사례를 다른 테스트와 독립적으로 만드십시오.

## 코드 분석과 스타일 도구

{% include requirement/MUST id="python-tooling-pylint" %} [pylint](https://www.pylint.org/)를 사용하십시오. [root of the repository](https://github.com/Azure/azure-sdk-for-python/blob/main/pylintrc)에서 pylintrc을 사용하십시오.

{% include requirement/MUST id="python-tooling-flake8" %} [flake8-docstrings](https://gitlab.com/pycqa/flake8-docstrings)를 사용하여 문서 주석을 확인하십시오.

{% include requirement/MUST id="python-tooling-black" %} 코드 서식을 지정하려면 [Black](https://black.readthedocs.io/en/stable/)을 사용해주십시오.

{% include requirement/SHOULD id="python-tooling-mypy" %} [MyPy](https://mypy.readthedocs.io/en/latest/)를 사용하여 라이브러리의 공개 코드 영역을 정적으로 확인하십시오.

테스트 등의 non-shipping 코드에서는 확인할 필요는 없습니다.

## Azure Core 활용

`azure-core`패키지는 클라이언트 라이브러리에 대한 일반적인 기능을 제공합니다. 설명서 및 예제는 [azure/azure-sdk-for-python] 저장소에서 확인하십시오.

### HTTP 파이프라인

HTTP 파이프라인은 여러 정책들에 의해 구성된 HTTP 전송입니다. 각 정책은 요청 또는 응답을 수정할 수 있는 제어점입니다. 클라이언트 라이브러리가 Azure 서비스와 상호 작용하는 방식을 표준화하기 위한 기본 정책 집합이 제공됩니다.

파이프라인의 python 구현에 대해 더 자세히 알고 싶다면 [documentation](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/core/azure-core)을 참고하십시오.

### 프로토콜

디자인 가이드라인에서 요구되는 많은 프로토콜은 `azure-core`에서 기본 구현되어있습니다.

#### LROPoller

```python
T = TypeVar("T")
class LROPoller(Protocol):

    def result(self, timeout=None) -> T:
        """ Retrieve the final result of the long running operation.

        :param timeout: How long to wait for operation to complete (in seconds). If not specified, there is no timeout.
        :raises TimeoutException: If the operation has not completed before it timed out.
        """
        ...

    def wait(self, timeout=None) -> None:
        """ Wait for the operation to complete.

        :param timeout: How long to wait for operation to complete (in seconds). If not specified, there is no timeout.
        """

    def done(self) -> boolean:
        """ Check if long running operation has completed.
        """

    def add_done_callback(self, func) -> None:
        """ Register callback to be invoked when operation completes.

        :param func: Callable that will be called with the eventual result ('T') of the operation.
        """
        ...
```

`azure.core.polling.LROPoller`는 `LROPoller` 프로토콜을 구현합니다.

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

프로토콜에 대한 더 많은 정보를 알고 싶다면 [ItemPaged](#PythonPagingDesign)를 참고하십시오.

#### DiagnosticsResponseHook

```python
class ResponseHook(Protocol):

    __call__(self, headers, deserialized_response): -> None ...

```

## 파이썬 언어와 코드스타일

{% include requirement/MUST id="python-codestyle-pep8" %} 본 문서에서 재정의되지 않는 한 [PEP8](https://www.python.org/dev/peps/pep-0008/)의 일반 지침을 따르십시오.

{% include requirement/MUSTNOT id="python-codestyle-idiomatic" %} 다른 언어의 코딩 패러다임을 빌려서는 안됩니다.

예를 들어 Java 커뮤니티에서 Reactive 프로그래밍이 얼마나 일반적인지에 상관없이 대부분의 Python 개발자에게는 여전히 익숙하지 않습니다.

{% include requirement/MUST id="python-codestyle-consistency" %} 동일한 서비스를 위해서는 다른 라이브러리보다 다른 파이썬 구성 요소와의 일관성을 선호해야 합니다.

개발자가 여러 언어의 동일한 서비스를 사용하는 것보다 개발자가 동일한 언어의 많은 다른 라이브러리를 사용할 가능성이 더 높습니다.

### Error handling

{% include requirement/MUST id="python-errors-use-chaining" %} 새 예외를 수집 및 생성할 때 오류의 원래 소스를 포함하려면 예외 체인을 사용해야 합니다.

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

### Naming conventions

{% include requirement/MUST id="python-codestyle-vars-naming" %} 변수, 함수 및 메서드 이름에 snake_case를 사용해야 합니다:

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

{% include requirement/MUST id="python-codestyle-type-naming" %} 다음 유형에 대해 파스칼 케이스를 사용해야 합니다:

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

{% include requirement/MUST id="python-codestyle-const-naming" %} 상수에 모두 대문자를 사용해야 합니다:

```python
# Yes:
MAX_SIZE = 4711

# No:
max_size = 4711

# No:
MaxSize = 4711
```

{% include requirement/MUST id="python-codestyle-module-naming" %} 모듈 이름에 snake_case를 사용해야 합니다.

### Method signatures

{% include requirement/MUSTNOT id="python-codestyle-static-methods" %} 정적 메서드 ([`staticmethod`](https://docs.python.org/3/library/functions.html#staticmethod))를 사용해서는 안됩니다. 대신 모듈 레벨 함수를 사용하는 것을 추천합니다.

정적 메서드는 드물며 일반적으로 다른 라이브러리에 의해 강제됩니다.

{% include requirement/MUSTNOT id="python-codestyle-properties" %} 게터 및 세터 함수를 사용하면 안 됩니다. 대신 속성을 사용할 수 있습니다.

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

{% include requirement/SHOULDNOT id="python-codestyle-long-args" %} 5개 이상의 위치 매개 변수를 필요로 하는 메서드는 사용할 수 없습니다. 키워드 전용 인수 또는 `**kwargs`를 사용하여 선택적/플래그 매개 변수를 사용할 수 있습니다.

위치 vs 선택적 매개변수에 대한 지침은 TODO: 링크를 참조하십시오.

{% include requirement/MUST id="python-codestyle-optional-args" %} Python 3만 지원해야 하는 모듈의 경우 선택적 또는 덜 자주 사용되는 인수에는 키워드 전용 인수를 사용해야 합니다.

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

{% include requirement/MUST id="python-codestyle-positional-params" %} 필요한 위치 매개 변수가 세 개 이상인 메서드를 호출할 때 매개 변수 이름을 지정해야 합니다.

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

{% include requirement/MUST id="python-codestyle-optional-param-calling" %} 함수를 호출할 때 선택적 파라미터의 파라미터 이름을 지정해야 합니다.

```python
def foo(a, b=1, c=None):
    pass


# Yes:
foo(1, b=2, c=3)

# No:
foo(1, 2, 3)
```

### Public vs "private"

{% include requirement/MUST id="python-codestyle-private-api" %} 이름이 퍼블릭 API의 일부가 아님을 나타내려면 선행 밑줄을 하나 사용해야 합니다. 퍼블릭이 아닌 API는 안정성이 보장되지 않습니다.

{% include requirement/MUSTNOT id="python-codestyle-double-underscore" %} 계층 구조에서 이름 충돌이 발생할 가능성이 있는 경우가 아니라면 선행 이중 밑줄 접두사 메서드 이름을 사용하면 안됩니다. 이름 충돌 발생은 드뭅니다.

{% include requirement/MUST id="python-codestyle-public-api" %} 모듈의 `__all__` 속성에 공용 메서드와 형식을 추가해야 합니다.

{% include requirement/MUST id="python-codestyle-interal-module" %} 내부 모듈에는 선행 밑줄을 사용해야 합니다. 모듈이 내부 모듈의 하위 모듈인 경우 선행 밑줄을 **생략**할 수 있습니다.

```python
# Yes:
azure.exampleservice._some_internal_module

# Yes - some_internal_module is still considered internal since it is a submodule of an internal module:
azure.exampleservice._internal.some_internal_module

# No - some_internal_module is considered public:
azure.exampleservice.some_internal_module
```

### Types (or not)

{% include requirement/MUST id="python-codestyle-structural-subtyping" %} 명시적 유형 검사보다 구조 하위 유형 지정 및 프로토콜을 선호해야 합니다.

{% include requirement/MUST id="python-codestyle-abstract-collections" %} 사용자 정의 매핑 유형을 제공하려면 추상 컬렉션 기본 클래스 'collections.abc'(또는 Python 2.7의 'collections')에서 파생되어야 합니다.

{% include requirement/MUST id="python-codestyle-pep484" %} 공개적으로 문서화된 클래스 및 함수에 대한 유형 힌트 [PEP484](https://www.python.org/dev/peps/pep-0484/) 를 제공해야 합니다.

- Python 2.7 호환 코드에 대한 지침은 [suggested syntax for Python 2.7 and 2.7-3.x straddling code](https://www.python.org/dev/peps/pep-0484/#suggested-syntax-for-python-2-7-and-straddling-code)에서 확인할 수 있습니다. 파이썬 3 전용(예: 'async' 클라이언트)에서는 사용해서는 안됩니다.

### Threading

{% include requirement/MUST id="python-codestyle-thread-affinity" %} 사용자가 제공한 콜백에 대해 명시적으로 문서화되지 않은 경우 스레드 선호도를 유지해야 합니다.

{% include requirement/MUST id="python-codestyle-document-thread-safety" %} 설명서에 메서드(함수/클래스)가 스레드 안전하다는 사실을 포함해야 합니다.

예시: [`asyncio.loop.call_soon_threadsafe`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_soon_threadsafe), [`queue`](https://docs.python.org/3/library/queue.html)

{% include requirement/SHOULD id="python-codestyle-use-executor" %} 호출자가 병렬 처리를 위해 자신만의 스레드 또는 프로세스 관리를 정의하지 않고 [`Executor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Executor) 를 통과하도록 허용해야 합니다.

스레드가 호출자에게 노출되지 않은 경우 직접 스레드를 관리할 수 있습니다. 예를 들어 `LROPoller` 구현에서는 백그라운드 poller 스레드를 사용합니다.

{% include refs.md %}
{% include_relative refs.md %}
