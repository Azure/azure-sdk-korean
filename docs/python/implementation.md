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

### 지원하는 타입(types)

{% include requirement/MUST id="python-models-repr" %} 모델 유형에 대해 `__repr__`을 구현해야 합니다. 표현에는 **반드시** 유형 이름과 모든 키 속성(즉, 모델 인스턴스 식별에 도움이 되는 속성)이 포함되어야 합니다.

{% include requirement/MUST id="python-models-repr-length" %}  `__repr__`의 출력중 1024자 이후로 넘어가는 부분은 생략해야합니다.

##### 확장 가능한 열거형(Enumerations)

SDK에 정의된 모든 Enums은 대소문자를 구분하지 않는 문자열로 교체될 수 있어야 합니다. 이는 `azure-core`에 정의된 `CaseInsensitiveEnumMeta` 클래스를 사용하여 수행됩니다.

```python
from enum import Enum
from six import with_metaclass

from azure.core import CaseInsensitiveEnumMeta

class MyCustomEnum(with_metaclass(CaseInsensitiveEnumMeta, str, Enum)):
    FOO = 'foo'
    BAR = 'bar'
```

### SDK 기능 구현

#### Configuration

{% include requirement/MUST id="python-envvars-global" %} honor the following environment variables for global configuration settings:

{% include tables/environment_variables.md %}

#### Logging

{% include requirement/MUST id="python-logging-usage" %} use Pythons standard [logging module](https://docs.python.org/3/library/logging.html).

{% include requirement/MUST id="python-logging-nameed-logger" %} provide a named logger for your library.

The logger for your package **must** use the name of the module. The library may provide additional child loggers. If child loggers are provided, document them.

For example:
- Package name: `azure-someservice`
- Module name: `azure.someservice`
- Logger name: `azure.someservice`
- Child logger: `azure.someservice.achild`

These naming rules allow the consumer to enable logging for all Azure libraries, a specific client library, or a subset of a client library.

{% include requirement/MUST id="python-logging-error" %} use the `ERROR` logging level for failures where it's unlikely the application will recover (for example, out of memory).

{% include requirement/MUST id="python-logging-warn" %} use the `WARNING` logging level when a function fails to perform its intended task. The function should also raise an exception.

Don't include occurrences of self-healing events (for example, when a request will be automatically retried).

{% include requirement/MUST id="python-logging-info" %} use the `INFO` logging level when a function operates normally.

{% include requirement/MUST id="python-logging-debug" %} use the `DEBUG` logging level for detailed trouble shooting scenarios.

The `DEBUG` logging level is intended for developers or system administrators to diagnose specific failures.

{% include requirement/MUSTNOT id="python-logging-sensitive-info" %} send sensitive information in log levels other than `DEBUG`.  For example, redact or remove account keys when logging headers.

{% include requirement/MUST id="python-logging-request" %} log the request line, response line, and headers for an outgoing request as an `INFO` message.

{% include requirement/MUST id="python-logging-cancellation" %} log an `INFO` message, if a service call is canceled.

{% include requirement/MUST id="python-logging-exceptions" %} log exceptions thrown as a `WARNING` level message. If the log level set to `DEBUG`, append stack trace information to the message.

You can determine the logging level for a given logger by calling [`logging.Logger.isEnabledFor`](https://docs.python.org/3/library/logging.html#logging.Logger.isEnabledFor).

#### Distributed tracing

{% include requirement/MUST id="python-tracing-span-per-method" %} create a new trace span for each library method invocation. The easiest way to do so is by adding the distributed tracing decorator from `azure.core.tracing`.

{% include requirement/MUST id="python-tracing-span-name" %} use `<package name>/<method name>` as the name of the span.

{% include requirement/MUST id="python-tracing-span-per-call" %} create a new span for each outgoing network call. If using the HTTP pipeline, the new span is created for you.

{% include requirement/MUST id="python-tracing-propagate" %} propagate tracing context on each outgoing service request.

#### Telemetry

Client library usage telemetry is used by service teams (not consumers) to monitor what SDK language, client library version, and language/platform info a client is using to call into their service. Clients can prepend additional information indicating the name and version of the client application.

{% include requirement/MUST id="python-http-telemetry-useragent" %} send telemetry information in the [User-Agent header] using the following format:

```
[<application_id> ]azsdk-python-<package_name>/<package_version> <platform_info>
```

- `<application_id>`: optional application-specific string. May contain a slash, but must not contain a space. The string is supplied by the user of the client library, e.g. "AzCopy/10.0.4-Preview"
- `<package_name>`: client library (distribution) package name as it appears to the developer, replacing slashes with dashes and removing the Azure indicator.  For example, "azure-keyvault-secrets" would specify "azsdk-python-keyvault-secrets".
- `<package_version>`: the version of the package. Note: this is not the version of the service
- `<platform_info>`: information about the currently executing language runtime and OS, e.g. "Python/3.8.4 (Windows-10-10.0.19041-SP0)"

For example, if we re-wrote `AzCopy` in Python using the Azure Blob Storage client library, we may end up with the following user-agent strings:

- (Python) `AzCopy/10.0.4-Preview azsdk-python-storage/4.0.0 Python/3.7.3 (Ubuntu; Linux x86_64; rv:34.0)`

The `azure.core.pipeline.policies.UserAgentPolicy` will provide this functionality if added to the HttpPipeline.

{% include requirement/SHOULD id="python-azurecore-http-telemetry-dynamic" %} send additional (dynamic) telemetry information as a semi-colon separated set of key-value types in the `X-MS-AZSDK-Telemetry` header.  For example:

```http
X-MS-AZSDK-Telemetry: class=BlobClient;method=DownloadFile;blobType=Block
```

The content of the header is a semi-colon key=value list.  The following keys have specific meaning:

* `class` is the name of the type within the client library that the consumer called to trigger the network operation.
* `method` is the name of the method within the client library type that the consumer called to trigger the network operation.

Any other keys that are used should be common across all client libraries for a specific service.  **DO NOT** include personally identifiable information (even encoded) in this header.  Services need to configure log gathering to capture the `X-MS-SDK-Telemetry` header in such a way that it can be queried through normal analytics systems.

##### Considerations for clients not using the UserAgentPolicy from azure-core

{% include requirement/MUST id="python-azurecore-http-telemetry-appid" %} allow the consumer of the library to set the application ID by passing in an `application_id` parameter to the service client constructor.  This allows the consumer to obtain cross-service telemetry for their app.

{% include requirement/MUST id="python-azurecore-http-telemetry-appid-length" %} enforce that the application ID is no more than 24 characters in length.  Shorter application IDs allows service teams to include diagnostic information in the "platform information" section of the user agent, while still allowing the consumer to obtain telemetry information for their own application.

## Testing

{% include requirement/MUST id="python-testing-pytest" %} use [pytest](https://docs.pytest.org/en/latest/) as the test framework.

{% include requirement/SHOULD id="python-testing-async" %} use [pytest-asyncio](https://github.com/pytest-dev/pytest-asyncio) for testing of async code.

{% include requirement/MUST id="python-testing-live" %} make your scenario tests runnable against live services. Strongly consider using the [Python Azure-DevTools](https://github.com/Azure/azure-sdk-for-python/tree/main/tools/azure-devtools) package for scenario tests.

{% include requirement/MUST id="python-testing-record" %} provide recordings to allow running tests offline/without an Azure subscription

{% include requirement/MUST id="python-testing-parallel" %} support simultaneous test runs in the same subscription.

{% include requirement/MUST id="python-testing-independent" %} make each test case independent of other tests.

## Code Analysis and Style Tools

{% include requirement/MUST id="python-tooling-pylint" %} use [pylint](https://www.pylint.org/) for your code. Use the pylintrc file in the [root of the repository](https://github.com/Azure/azure-sdk-for-python/blob/main/pylintrc).

{% include requirement/MUST id="python-tooling-flake8" %} use [flake8-docstrings](https://gitlab.com/pycqa/flake8-docstrings) to verify doc comments.

{% include requirement/MUST id="python-tooling-black" %} use [Black](https://black.readthedocs.io/en/stable/) for formatting your code.

{% include requirement/SHOULD id="python-tooling-mypy" %} use [MyPy](https://mypy.readthedocs.io/en/latest/) to statically check the public surface area of your library.

You don't need to check non-shipping code such as tests.

## Making use of Azure Core

The `azure-core` package provides common functionality for client libraries. Documentation and usage examples can be found in the [azure/azure-sdk-for-python] repository.

### HTTP pipeline

The HTTP pipeline is an HTTP transport that is wrapped by multiple policies. Each policy is a control point that can modify either the request or response. A default set of policies is provided to standardize how client libraries interact with Azure services.

For more information on the Python implementation of the pipeline, see the [documentation](https://github.com/Azure/azure-sdk-for-python/tree/main/sdk/core/azure-core).

### Protocols

Many of the protocols mandated by the design guidelines have default implementations in `azure-core`.

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

`azure.core.polling.LROPoller` implements the `LROPoller` protocol.

#### ItemPaged

```python
T = TypeVar("T")
class ByPagePaged(Protocol, Iterable[Iterable[T]]):
    continuation_token: "str"

class ItemPaged(Protocol, Iterable[T]):
    continuation_token: "str"

    def by_page(self) -> ByPagePaged[T] ...
```

`azure.core.ItemPaged` implements the `ItemPaged` protocol.

See the [ItemPaged](#PythonPagingDesign) protocol for additional information.

#### DiagnosticsResponseHook

```python
class ResponseHook(Protocol):

    __call__(self, headers, deserialized_response): -> None ...

```

## Python language and code style

{% include requirement/MUST id="python-codestyle-pep8" %} follow the general guidelines in [PEP8](https://www.python.org/dev/peps/pep-0008/) unless explicitly overridden in this document.

{% include requirement/MUSTNOT id="python-codestyle-idiomatic" %} "borrow" coding paradigms from other languages.

For example, no matter how common Reactive programming is in the Java community, it's still unfamiliar for most Python developers.

{% include requirement/MUST id="python-codestyle-consistency" %} favor consistency with other Python components over other libraries for the same service.

It's more likely that a developer will use many different libraries using the same language than a developer will use the same service from many different languages.

### Error handling

{% include requirement/MUST id="python-errors-use-chaining" %} use exception chaining to include the original source of the error when catching and raising new exceptions.

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

{% include requirement/MUST id="python-codestyle-vars-naming" %} use snake_case for variable, function, and method names:

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

{% include requirement/MUST id="python-codestyle-type-naming" %} use Pascal case for types:

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

{% include requirement/MUST id="python-codestyle-const-naming" %} use ALL CAPS for constants:

```python
# Yes:
MAX_SIZE = 4711

# No:
max_size = 4711

# No:
MaxSize = 4711
```

{% include requirement/MUST id="python-codestyle-module-naming" %} use snake_case for module names.

### Method signatures

{% include requirement/MUSTNOT id="python-codestyle-static-methods" %} use static methods ([`staticmethod`](https://docs.python.org/3/library/functions.html#staticmethod)). Prefer module level functions instead.

Static methods are rare and usually forced by other libraries.

{% include requirement/MUSTNOT id="python-codestyle-properties" %} use simple getter and setter functions. Use properties instead.

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

{% include requirement/SHOULDNOT id="python-codestyle-long-args" %} have methods that require more than five positional parameters. Optional/flag parameters can be accepted using keyword-only arguments, or `**kwargs`.

See TODO: insert link for general guidance on positional vs. optional parameters here.

{% include requirement/MUST id="python-codestyle-optional-args" %} use keyword-only arguments for optional or less-often-used arguments for modules that only need to support Python 3.

```python
# Yes
def foo(a, b, *, c, d=None):
    # Note that I can even have required keyword-only arguments...
    ...
```

{% include requirement/MUST id="python-codestyle-kwargs" %} use keyword-only arguments for arguments that have no obvious ordering.

```python
# Yes - `source` and `dest` have logical order, `recurse` and `overwrite` do not.
def copy(source, dest, *, recurse=False, overwrite=False) ...


# No
def copy(source, dest, recurse=False, overwrite=False) ...
```

{% include requirement/MUST id="python-codestyle-positional-params" %} specify the parameter name when calling methods with more than two required positional parameters.

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

{% include requirement/MUST id="python-codestyle-optional-param-calling" %} specify the parameter name for optional parameters when calling functions.

```python
def foo(a, b=1, c=None):
    pass


# Yes:
foo(1, b=2, c=3)

# No:
foo(1, 2, 3)
```

### Public vs "private"

{% include requirement/MUST id="python-codestyle-private-api" %} use a single leading underscore to indicate that a name isn't part of the public API.  Non-public APIs aren't guaranteed to be stable.

{% include requirement/MUSTNOT id="python-codestyle-double-underscore" %} use leading double underscore prefixed method names unless name clashes in the inheritance hierarchy are likely.  Name clashes are rare.

{% include requirement/MUST id="python-codestyle-public-api" %} add public methods and types to the module's `__all__` attribute.

{% include requirement/MUST id="python-codestyle-interal-module" %} use a leading underscore for internal modules. You **may** omit a leading underscore if the module is a submodule of an internal module.

```python
# Yes:
azure.exampleservice._some_internal_module

# Yes - some_internal_module is still considered internal since it is a submodule of an internal module:
azure.exampleservice._internal.some_internal_module

# No - some_internal_module is considered public:
azure.exampleservice.some_internal_module
```

### Types (or not)

{% include requirement/MUST id="python-codestyle-structural-subtyping" %} prefer structural subtyping and protocols over explicit type checks.

{% include requirement/MUST id="python-codestyle-abstract-collections" %} derive from the abstract collections base classes `collections.abc` (or `collections` for Python 2.7) to provide custom mapping types.

{% include requirement/MUST id="python-codestyle-pep484" %} provide type hints [PEP484](https://www.python.org/dev/peps/pep-0484/) for publicly documented classes and functions.

- See the [suggested syntax for Python 2.7 and 2.7-3.x straddling code](https://www.python.org/dev/peps/pep-0484/#suggested-syntax-for-python-2-7-and-straddling-code) for guidance for Python 2.7 compatible code. Do not do this for code that is Python 3 specific (e.g. `async` clients.)

### Threading

{% include requirement/MUST id="python-codestyle-thread-affinity" %} maintain thread affinity for user-provided callbacks unless explicitly documented to not do so.

{% include requirement/MUST id="python-codestyle-document-thread-safety" %} explicitly include the fact that a method (function/class) is thread safe in its documentation.

Examples: [`asyncio.loop.call_soon_threadsafe`](https://docs.python.org/3/library/asyncio-eventloop.html#asyncio.loop.call_soon_threadsafe), [`queue`](https://docs.python.org/3/library/queue.html)

{% include requirement/SHOULD id="python-codestyle-use-executor" %} allow callers to pass in an [`Executor`](https://docs.python.org/3/library/concurrent.futures.html#concurrent.futures.Executor) instance rather than defining your own thread or process management for parallelism.

You may do your own thread management if the thread isn't exposed to the caller in any way. For example, the `LROPoller` implementation uses a background poller thread.

{% include refs.md %}
{% include_relative refs.md %}
