---
title: "파이썬 안내서 : API 디자인"
keywords: guidelines python
permalink: python_design.html
folder: python
sidebar: general_sidebar
---

## 개요

### 디자인 원칙

Azure SDK는 무엇보다 Azure 서비스를 사용하는 개발자의 생산성을 높이도록 설계되어야 합니다. SDK의 완전성, 확장성, 성능 등 다른 요소들도 중요하지만 부차적입니다. 아래 원칙들을 고수함으로써 생산성을 높입니다.

#### 직관성 (Idiomatic)

* SDK는 대상 언어에 대한 디자인 가이드라인 및 규칙을 따라야 합니다. 해당 언어의 개발자가 자연스럽게 받아들일 수 있어야 합니다.
* 우리는 생태계의 장점과 단점을 함께 포괄합니다.
* 우리는 모든 개발자들을 위한 생태계를 개선시키기 위해 협력합니다.

#### 일관성 (Consistent)

* 클라이언트 라이브러리는 언어 내에서 일관성이 있어야 하며, 서비스와 대상 언어 간의 일관성이 있어야 합니다. 충돌이 발생하면, 우선 언어 내의 일관성을 가장 높은 우선순위를 가지고, 모든 대상의 언어 간의 일관성을 가장 낮은 우선순위로 가집니다.
* 로깅, HTTP 통신, 예외 처리 같은 일률적인 서비스는 일관성이 있어야 하며, 개발자는 클라이언트 라이브러리들 간에 이동할 때 위와 같은 일률적인 서비스를 다시 학습할 필요가 없어야 합니다.
* 클라이언트 라이브러리와 서비스 간 용어의 일관성은 진단 가능성을 높이는 데 도움이 됩니다.
* 서비스와 클라이언트 라이브러리 간의 모든 차이점은 변덕이 아닌 관용적 사용에 근거한 타당한(구체적인) 이유가 있어야 합니다.
* 각 대상 언어에 대한 Azure SDK는 하나의 팀이 개발한 하나의 제품처럼 느껴져야 합니다.
* 대상 언어 간에 기능 동등성이 있어야 합니다. 이것은 서비스와의 기능 동등성보다 더 중요합니다.

#### 접근성 (Approachable)

* 우리는 지원되는 기술들의 전문가로 우리의 고객들과 개발자들은 전문성을 가질 필요가 없습니다.
* 개발자들은 Azure 서비스를 성공적으로 사용하기 쉽게 해주는 좋은 문서(튜토리얼, 방법 문서, 샘플들 및 API 문서)를 찾아야 합니다.
* 모범 사례를 구현하는 예측 가능한 기본값들을 사용함으로써 쉽게 시작할 수 있어야 합니다. 점진적인 개념의 공개에 대해 생각해보세요.
* SDK는 목표하는 언어와 생태계 안에서 가장 평범한 방법을 통해 쉽게 얻을 수 있어야 합니다.
* 개발자들은 새로운 서비스 개념을 배울 때 압도당할 수 있습니다. 핵심 사용 사례들은 쉽게 발견할 수 있어야 합니다.

#### 진단가능성 (Diagnosable)

* 개발자는 무슨 일이 일어나고 있는지 이해할 수 있어야 합니다.
* 언제, 어떤 상황에서 네트워크 호출이 이루어졌는지 발견할 수 있어야 합니다.
* 기본값은 검색할 수 있으며 의도가 명확합니다.
* 로깅, 추적 및 예외 처리는 기본적이며 신중해야 합니다.
* 오류 메시지는 간결해야 하고 서비스와 연관되어 있어야 하며 실행 가능하고 사람이 읽을 수 있어야 합니다. 이상적으로는 오류 메시지를 통해 소비자가 취할 수 있는 유용한 조치를 하도록 유도해야 합니다.
* 대상 언어에 대해 선호하는 디버거와 통합이 쉬워야 합니다.

#### 호환성 (Dependable)

* 하위 호환성이 보장되지 않은 새로운 기능이나 개선은 사용자 경험에 좋을 때보다 그 반대인 경우가 더 많습니다.
* 일부러 호환성을 깨뜨리는 일은 타당한 이유와 함께 반드시 꼼꼼한 검토를 거쳐야만 합니다.
* 향후 호환성을 점검할 일이 빈번하지 않도록 의존관계를 점검해야 합니다.

### 일반적 가이드라인

클라이언트 라이브러리의 API에 정말 많은 노력을 기울여야합니다. 서비스의 첫인상이자 주로 쓰는 상호작용 방법이기 때문입니다.

{% include requirement/MUST id="python-feature-support" %} Azure 서비스가 제공하는 100%의 기능을 모두 클라이언트 라이브러리에서 지원해야합니다. 기능이 빠져있다면 개발자로서 아주 난감할 것이기 때문입니다.

### HTTP 기반이 아닌 서비스

이 안내서는 HTTP 기반 요청/응답 구조를 염두에 두고 작성했지만, 아닌 서비스들에도 많은 부분을 적용하실 수 있습니다. 예로 들면 패키징과 네이밍, 도구와 프로젝트 구조를 비롯한 서비스들이 있습니다.

HTTP/REST 기반이 아닌 서비스들에 대해 더 자세한 문의사항이 있으시다면 [아키텍처 위원회](https://azure.github.io/azure-sdk-korean/policies_reviewprocess.html)에 문의해주시길 바랍니다.

### 지원 중인 파이썬 버전

{% include requirement/MUST id="python-general-version-support" %} 파이썬 3.7 이상을 지원하고 있습니다.

## Azure SDK API 디자인

API 표면은 사용자가 서비스에 연결하기 위해 인스턴스화할 하나 이상의 서비스 클라이언트와, 이를 지원하는 몇 가지 타입들로 구성됩니다.

### 서비스 클라이언트

서비스 클라이언트는 라이브러리를 사용할 때 처음으로 다루는 부분입니다. 하나 이상의 메소드를 개방하여 하여금 서비스와 상호작용할 수 있게 만들어 줍니다.

{% include requirement/MUST id="python-client-namespace" %} 상호작용이 빈번할 확률이 높은 서비스 클라이언트는 패키지의 최상위 네임스페이스에 개방시켜주세요. 특화된 특정 서비스 클라이언트는 하위 네임스페이스에 둘 수 있습니다. 

{% include requirement/MUST id="python-client-naming" %} 서비스 클라이언트 타입을 가진 객체의 이름은 **Client** 로 끝나야합니다.

{% include requirement/MUST id="python-client-sync-async-separate-clients" %} 동기와 비동기 클라이언트를 별도로 제공해주세요. [비동기 지원](#비동기-지원) 절에서 더 자세한 내용을 보실 수 있습니다.

```python
# Yes
class CosmosClient(object) ...

# No
class CosmosProxy(object) ...

# Yes
class CosmosUrl(object) ...
```

{% include requirement/MUST id="python-client-immutable" %} 서비스 클라이언트는 불변해야합니다. [클라이언트 불변성](#클라이언트-불변성) 절에서 더 자세한 내용을 보실 수 있습니다.

#### 생성자와 팩토리 메서드

클라이언트 인스턴스를 구성 시, 서비스를 연결하고 상호 작용하는 데 필요한 최소한의 정보만이 요구됩니다. 그 외의 모든 추가 정보는 선택적이어야 하며 선택적인 키워드 전용 인수로 전달되어야 합니다.

##### 클라이언트 구성

{% include requirement/MUST id="python-client-constructor-form" %} 위치 바인딩 위치 바인딩 매개변수(예: 서비스 인스턴스의 이름이나 해당 서비스 인스턴스를 가리키는 URL), 위치 `자격증명` 매개변수, 키워드 전용 `전송` 매개변수, 그리고 개별 HTTP 파이프라인 정책에 설정을 전달하기 위한 키워드 전용 인수를 받는 생성자를 제공하세요. `자격증명` 매개변수에 대한 자세한 내용은 [인증](#인증) 섹션을 참조할 수 있습니다.

{% include requirement/MUSTNOT id="python-client-options-naming" %} "옵션 묶음(option bag)" 객체를 사용하여 선택적 매개변수를 그룹화하는 대신 개별 키워드 전용 인자로 전달하세요.

{% include requirement/MUST id="python-client-constructor-policy-arguments" %} 선택적인 기본 요청 옵션을 키워드 인자로 받아들이고, 이를 파이프라인 정책에 전달하세요. 추가적인 정보는 [공통 서비스 작업 매개변수](#공통-서비스-작업-매개변수)를 참조할 수 있습니다.

```python
# Change default number of retries to 18 and overall timeout to 2s.
client = ExampleClient('https://contoso.com/xmpl',
                       DefaultAzureCredential(),
                       max_retries=18,
                       timeout=2)
```

{% include requirement/MUST id="python-client-constructor-transport-argument" %} 사용자가 특정 전송 인스턴스를 지정할 수 있도록 키워드 전용 `전송` 인수를 전달하도록 허용해야 합니다. 기본값은 동기 클라이언트의 경우 [`RequestsTransport`](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.1.1/azure.core.pipeline.transport.html?highlight=transport#azure.core.pipeline.transport.RequestsTransport), 비동기 클라이언트의 경우 [`AioHttpTransport`](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.1.1/azure.core.pipeline.transport.html?highlight=transport#azure.core.pipeline.transport.AioHttpTransport)이어야 합니다.

{% include requirement/MUST id="python-client-connection-string" %} 클라이언트가 연결 문자열을 지원하는 경우 연결 문자열에서 클라이언트를 생성하기 위해 별도의 팩토리 클래스 메서드 `from_connection_string`를 사용하세요. from_connection_string 팩토리 메서드는 연결 문자열에 제공된 정보를 제외하고 생성자와 동일한 인수 세트를 가져야 합니다. 생성자 (`__init__` 메서드)는 연결 문자열을 **반드시 받아들이지 않아야** 하며, 이는 `from_connection_string`를 사용하는 것이 클라이언트 인스턴스를 생성하는 유일한 지원 방법이라는 것을 의미합니다. 

해당 메서드는 연결 문자열을 파싱하고 값과 `자격증명`을 제외한 기타 키워드 전용 인수를 생성자에 **전달해야** 합니다. Azure 포털이 귀하의 서비스에 대한 연결 문자열을 제공하는 경우에만 `from_connection_string` 팩토리 메서드를 제공하세요.

```python
class ExampleClientWithConnectionString(object):

    @classmethod
    def _parse_connection_string(cls, connection_string): ...

    @classmethod
    def from_connection_string(cls, connection_string, **kwargs):
        endpoint, credential = cls._parse_connection_string(connection_string)
        return cls(endpoint, credential, **kwargs)
```

```python
{% include_relative _includes/example_client.py %}
```

{% include requirement/MAY id="python-client-constructor-from-url" %} URL에서 클라이언트를 생성하기 위해 별도의 팩토리 클래스 메서드 `from_<리소스 타입>_url` (예: `from_blob_url`)를 사용하세요 (서비스가 리소스에 대한 URL 전달에 의존하는 경우 - 예: Azure Blob Storage). `from_url` 팩토리 메서드는 생성자와 동일한 선택적 키워드 인수 세트를 가져야 합니다."

##### 서비스 버전 지정

{% include requirement/MUST id="python-client-constructor-api-version-argument-1" %} 문자열 타입의 선택적 api_version 키워드 전용 인수를 사용합니다. 만약 지정되었다면, 해당 API 버전은 서비스와 상호작용할 때 반드시 사용되어야 합니다. 인자가 제공되지 않으면, 기본값은 클라이언트 라이브러리가 이해하는 가장 최신의 비 미리보기 API 버전(서비스가 비 미리보기 버전을 갖고 있는 경우) 또는 가장 최신의 미리보기 API 버전(서비스가 아직 비 미리보기 API 버전을 갖고 있지 않은 경우)이어야 합니다.

```python
from azure.identity import DefaultAzureCredential

# By default, use the latest supported API version
latest_known_version_client = ExampleClient('https://contoso.com/xmpl',
                                            DefaultAzureCredential())

# ...but allow the caller to specify a specific API version as welll
specific_api_version_client = ExampleClient('https://contoso.com/xmpl',
                                            DefaultAzureCredential(),
                                            api_version='1971-11-01')
```

{% include requirement/MUST id="python-client-constructor-api-version-argument-2" %} 기본적으로 사용되는 서비스 API 버전을 문서화하세요.

{% include requirement/MUST id="python-client-constructor-api-version-argument-3" %} 만약 모든 서비스 API 버전이 해당 기능(함수 또는 매개변수)을 지원하지 않는다면, 해당 기능이 도입된 API 버전을 문서화하세요.

{% include requirement/MAY id="python-client-constructor-api-version-argument-4" %} 입력된 `api_version` 값이 지원되는 API 버전 목록과 일치하는지 확인하세요.

{% include requirement/MAY id="python-client-constructor-api-version-argument-5" %} 클라이언트 라이브러리에서 지원하는 모든 서비스 API 버전을 ServiceVersion 열거형 값에 포함하세요.

##### Additional constructor parameters

|Name|Description|
|-|-|
|`credential`|서비스 요청을 수행할 때 사용할 자격 증명 ([인증](#인증) 참조)|
|`application_id`|요청을 하는 클라이언트 애플리케이션의 이름. 텔레메트리에 활용|
|`api_version`|서비스 요청 시 사용할 API 버전 ([서비스 버전](#서비스-버전-지정) 참조) |
|`transport`|기본 HTTP 전송 재정의 ([클라이언트 구성](#클라이언트-구성))|

##### 클라이언트 불변성

{% include requirement/MUST id="python-client-immutable-design" %} 클라이언트를 변경할 수 없도록 설계하세요. 이는 읽기 전용 속성을 사용해야 하는 것은 아니지만 (속성은 여전히 허용됨), 호출자가 클라이언트의 속성을 변경해야 하는 시나리오가 없어야 한다는 의미입니다. 클라이언트의 속성/속성을 변경해야 하는 상황이 발생하지 않도록 주의하세요.

#### Service methods

##### Naming

{% include requirement/SHOULD id="python-client-service-verbs" %} 메서드 이름에는 우선적으로 선호하는 동사 중 하나를 사용하는 것이 좋습니다. 특정 작업에 대해 다른 동사를 사용하는 경우에는 그에 대한 명확한 이유가 있어야 합니다. 이러한 작업에 대한 대체 동사를 사용하는 이유를 명확하게 이유를 설명해야 합니다.

|동사|매개변수|반환값|설명|
|-|-|-|-|
|`create_\<noun>`|key, item, `[allow_overwrite=False]`|생성된 item|새로운 항목을 생성합니다. 이미 해당 항목이 존재하는 경우 실패합니다.|
|`upsert_\<noun>`|key, item|item|새로운 항목을 생성하거나 기존 항목을 업데이트합니다. 주로 데이터베이스와 유사한 서비스에서 사용됩니다.|
|`set_\<noun>`|key, item|item|새로운 항목을 생성하거나 기존 항목을 업데이트합니다. 이 동사는 주로 서비스의 사전(dictionary) 형태의 속성들에 사용됩니다. |
|`update_\<noun>`|key, partial item|item|해당 항목이 존재하지 않는 경우 실패합니다. |
|`replace_\<noun>`|key, item|item|기존 항목을 완전히 대체합니다. 해당 항목이 존재하지 않는 경우 실패합니다. |
|`append_\<noun>`|item|item|컬렉션에 항목을 추가합니다. 항목은 마지막에 추가됩니다. |
|`add_\<noun>`|index, item|item|컬렉션에 항목을 추가합니다. 항목은 지정된 인덱스에 추가됩니다. |
|`get_\<noun>`|key|item|항목이 존재하지 않는 경우 예외를 발생시킵니다. |
|`list_\<noun>`||`azure.core.ItemPaged[Item]`|`Item`들의 반복 가능한(iterable) 항목을 반환합니다. 항목이 없을 경우, 아이템이 없는 반복 가능한(iterable) 객체를 반환합니다 (`None`을 반환하거나 예외를 발생시키지 않습니다).|
|`\<noun>\_exists`|key|`bool`|만약 항목이 존재한다면 True를 반환합니다. 만약 메서드가 항목이 존재하는지 여부를 결정하는 데 실패한 경우 (예: 서비스가 HTTP 503 응답을 반환한 경우), 예외를 발생시켜야 합니다.|
|`delete_\<noun>`|key|`None`|기존 항목을 삭제합니다. 항목이 존재하지 않아도 반드시 성공해야 합니다.|
|`remove_\<noun>`|key|제거된 item or `None`|컬렉션에서 항목에 대한 참조를 제거합니다. 이 메서드는 실제 항목을 삭제하지 않고 참조만 제거합니다.|

{% include requirement/MUST id="python-client-standardize-verbs" %} 여러 언어 SDK에 걸쳐 주어진 서비스에 대해 선호하는 동사 목록 외부의 동사 접두사를 표준화합니다. 만약 동사를 한 언어로 `download`라고 한다면 다른 언어로 `fetch`라고 명명하는 것은 피해야 합니다.

{% include requirement/MUST id="python-lro-prefix" %} [긴 실행 시간](#장시간-실행-작업을-호출하는-방법)이 필요한 작업에 대해 메서드를 begin_으로 접두사를 붙이세요.

{% include requirement/MUST id="python-paged-prefix" %} 리소스를 열거(목록)하는 메서드에 대해 `list_`를 사용하는 접두사 메서드

##### 반환 타입

서비스에 대한 요청은 두 가지 기본 그룹, 즉 단일 논리적 요청을 만드는 메서드 또는 일련의 결정적 요청으로 나뉩니다. 단일 논리적 요청의 예로는 작업 내부에서 다시 시도할 수 있는 요청이 있습니다. 결정론적 요청 시퀀스의 예로는 페이징 작업이 있습니다.

논리적 개체는 응답의 프로토콜 중립적 표현입니다. HTTP의 경우, 논리적 개체는 헤더, 본문 및 상태 라인의 데이터를 결합할 수 있습니다. 예를 들어, `ETag` 헤더를 논리적 개체의 `etag` 속성으로 노출시키고 싶을 수 있습니다. 자세한 내용은 [모델 타입](#모델-타입)을 참조하세요.

{% include requirement/MUST id="python-response-logical-entity" %} 총 99% 이상의 경우에 필요한 정보를 나타내는 논리적 개체를 반환하기 위해 최적화하세요.

{% include requirement/MUST id="python-response-exception-on-failure" %} 사용자가 지정한 작업을 수행하지 못한 경우, 메소드 호출이 실패한 경우에는 예외를 발생시키세요. 이는 서비스가 명시적으로 실패로 응답한 경우뿐만 아니라 응답을 받지 못한 경우에도 해당됩니다. 자세한 내용은 [예외](#예외)를 참조하세요.

```python
client = ComputeClient(...)

try:
    # Please note that there is no status code etc. as part of the response.
    # If the call fails, you will get an exception that will include the status code
    # (if the request was made)
    virtual_machine  = client.get_virtual_machine('example')
    print(f'Virtual machine instance looks like this: {virtual_machine}')
except azure.core.exceptions.ServiceRequestError as e:
    print(f'Failed to make the request - feel free to retry. But the specifics are here: {e}')
except azure.core.exceptions.ServiceResponseError as e:
    print(f'The request was made, but the service responded with an error. Status code: {e.status_code}')
```

에러를 나타내기 위해 `None`이나 `boolean` 값을 반환하지 마세요.:

```python
# Yes
try:
    resource = client.create_resource(name)
except azure.core.errors.ResourceExistsException:
    print('Failed - we need to fix this!')

# No
resource = client.create_resource(name):
if not resource:
    print('Failed - we need to fix this!')
```

{% include requirement/MUSTNOT id="python-errors-normal-responses" %} "일반적인 응답"에 대해서는 예외를 발생시키세요.

예를 들어 `exists` 메소드의 경우, 서비스가 클라이언트 오류인 404/NotFound를 반환한 경우와 요청 자체를 수행하지 못한 경우를 구분해야 합니다.

```python
# Yes
try:
    exists = client.resource_exists(name):
    if not exists:
        print("The resource doesn't exist...")
except azure.core.errors.ServiceRequestError:
    print("We don't know if the resource exists - so it was appropriate to throw an exception!")

# No
try:
    client.resource_exists(name)
except azure.core.errors.ResourceNotFoundException:
    print("The resource doesn't exist... but that shouldn't be an exceptional case for an 'exists' method")
```

##### 작업 취소

{% include requirement/MUST id="python-client-cancellation-sync-methods" %} 메소드가 완료될 때까지 대기할 수 있는 시간을 호출자가 지정할 수 있도록 선택적 키워드 인수 `timeout`을 제공하세요. `timeout`은 초 단위로 표시되며, 최대한 지정된 시간을 준수해야 합니다.

 {% include requirement/MUST id="python-client-cancellation-async-methods" %}  비동기 메소드를 취소하기 위해 표준 [asyncio.Task.cancel](https://docs.python.org/3/library/asyncio-task.html#asyncio.Task.cancel) 메소드를 사용하세요.

#### 서비스 메서드 매개변수

{% include requirement/MUST id="python-client-optional-arguments-keyword-only" %} 작업별 선택적 인수를 키워드 전용으로 제공하세요. 자세한 내용은 [positional and keyword-only arguments]를 참조하세요.

{% include requirement/MUST id="python-client-service-per-call-args" %} 요청별 정책 옵션을 덮어쓰는 키워드 전용 인수를 제공하세요. 매개 변수의 이름은 클라이언트 생성자 또는 팩토리 메서드에 제공된 인수의 이름을 반영해야 합니다.

파이프라인 정책 및 전송 구성(클라이언트 생성자 및 서비스 작업별)에 사용되는 지원되는 선택적 인수의 전체 목록은 [Azure Core 개발자 설명서](https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/core/azure-core/CLIENT_LIBRARY_DEVELOPER.md)를 참조하세요.

{% include requirement/MUST id="python-client-service-args-conflict" %} 만약 서비스의 매개변수 이름이 문서화된 파이프라인 정책이나 모든 서비스 작업 및 클라이언트 생성자와 함께 사용되는 전송 구성 옵션 중 어떤 것과 충돌한다면, 해당 서비스 매개변수 이름을 구체화하세요.

```python
# Set the default number of retries to 18 and timeout to 2s for this client instance.
client = ExampleClient('https://contoso.com/xmpl', DefaultAzureCredential(), max_retries=18, timeout=2)

# Override the client default timeout for this specific call to 32s (but max_retries is kept to 18)
client.do_stuff(timeout=32)
```

##### 매개변수 유효성 검사

서비스 클라이언트에는 서비스에 요청을 보내는 여러 가지 방법이 있습니다. **서비스 매개 변수**는 와이어를 통해 Azure 서비스로 직접 전달됩니다. **클라이언트 매개 변수**는 서비스에 직접 전달되지 않지만 클라이언트 라이브러리 내에서 요청을 수행하는 데 사용됩니다. URI 또는 업로드할 파일을 구성하는 데 사용되는 매개 변수는 클라이언트 매개 변수의 예입니다.

{% include requirement/MUST id="python-params-client-validation" %} 클라이언트 매개 변수의 유효성을 검사합니다. 잘못된 형식의 URL은 클라이언트 라이브러리가 잘못된 엔드포인트를 호출하게 되므로 URL을 구축하는 데 사용되는 매개 변수에 대해 유효성 검사가 특히 중요합니다.

```python
# No:
def get_thing(name: "str") -> "str":
    url = f'https://<host>/things/{name}'
    return requests.get(url).json()

try:
    thing = get_thing('') # Ooops - we will end up calling '/things/' which usually lists 'things'. We wanted a specific 'thing'.
except ValueError:
    print('We called with some invalid parameters. We should fix that.')

# Yes:
def get_thing(name: "str") -> "str":
    if not name:
        raise ValueError('name must be a non-empty string')
    url = f'https://<host>/things/{name}'
    return requests.get(url).json()

try:
    thing = get_thing('')
except ValueError:
    print('We called with some invalid parameters. We should fix that.')
```

{% include requirement/MUSTNOT id="python-params-service-validation" %} 서비스 매개 변수의 유효성을 검사하세요. 서비스 매개 변수에 대해 null 검사, 빈 문자열 검사 또는 기타 일반적인 유효성 검사 조건에 대한 체크는 수행하지 마세요. 모든 요청 매개 변수의 유효성에 대해서는 서비스가 검사하도록 합니다.

{% include requirement/MUST id="python-params-devex" %} 유효하지 않은 서비스 매개변수일 때 개발자 경험이 적절한 오류 메시지가 서비스에 의해 생성되도록 확인하세요. 만약 서비스 측의 오류 메시지로 인해 개발자 경험이 저하된다면, 서비스 팀과 협력하여 문제를 해결하세요.

##### 공통 서비스 작업 매개변수

{% include requirement/MUST id="python-client-service-args" %} 일반적인 서비스 작업에 대한 인수를 지원합니다:

|Name|Description|Applies to|Notes|
|-|-|-|-|
|`timeout`|초 단위의 타임아웃을 지원합니다.|모든 서비스 메소드|
|`headers`|서비스 요청에 포함할 사용자 정의 헤더를 지원합니다.|All requests|해당 메소드에 의해 직접적으로 또는 간접적으로 수행되는 모든 요청에 헤더가 추가됩니다.|
|`client_request_id`|요청의 호출자 지정 식별을 지원합니다.|클라이언트가 클라이언트 생성 상관 ID를 전송할 수 있는 서비스에 대한 서비스 운영에 적용됩니다.|예시로 `x-ms-client-request-id` 헤더가 있습니다.|클라이언트 라이브러리는 제공된 경우 이 값을 **반드시** 사용해야하며, 지정되지 않은 경우 각 요청에 대해 고유한 값을 생성해야 합니다.|
|`response_hook`|각 작업에 대해 (응답, 헤더)와 함께 호출되는 `호출 가능한` 함수입니다|모든 서비스 메소드|

{% include requirement/MUST id="python-client-splat-args" %} 매개변수에 대해 직렬화된 모델 객체와 동일한 형태로 `dict`와 유사한 매핑(Mapping) 객체를 받아들입니다.

```python
# Yes:
class Thing(object):

    def __init__(self, name, size):
        self.name = name
        self.size = size

def do_something(thing: "Thing"):
    ...

do_something(Thing(name='a', size=17)) # Works
do_something({'name': 'a', 'size', '17'}) # Does the same thing...
```

{% include requirement/MUST id="python-client-flatten-args" %} `update_` 메서드에 대해 "flattened"된 이름을 가진 인자를 사용하세요. **필요한 경우** 모델 인스턴스 전체를 명명된 매개변수로 추가로 사용할 수 있습니다. 호출자가 모델 인스턴스와 개별 키=값 매개변수를 함께 전달하는 경우, 명시적으로 지정된 키=값 매개변수가 모델 인스턴스에 지정된 값보다 우선합니다.

```python
class Thing(object):

    def __init__(self, name, size, description):
        self.name = name
        self.size = size
        self.description = description

    def __repr__(self):
        return json.dumps({
            "name": self.name, "size": self.size, "description": self.description
        })[:1024]

class Client(object):

    def update_thing(self, name=None, size=None, thing=None): ...

thing = Thing(name='hello', size=4711, description='This is a description...')

client.update_thing(thing=thing, size=4712) # Will send a request to the service to update the model's size to 4712
thing.description = 'Updated'
thing.size = -1
# Will send a request to the service to update the model's size to 4713 and description to 'Updated'
client.update_thing(name='hello', size=4713, thing=thing)
```

#### 컬렉션 반환 메서드 (페이징):

서비스는 큰 컬렉션 내의 아이템 전체 세트를 검색하기 위해 여러 요청을 필요로 할 수 있습니다. 이는 일반적으로 서비스가 부분 결과를 반환하고 응답에 클라이언트가 아이템 세트 외에도 다음 배치의 응답을 검색하는 데 사용할 수 있는 토큰 또는 링크를 제공함으로써 이루어집니다.

Azure SDK for Python 클라이언트 라이브러리에서는 이를 사용자에게 [ItemPaged](#python-core-protocol-paged) 프로토콜을 통해 제공합니다. ItemPaged 프로토콜은 사용자가 기본 페이징을 처리하는 것이 아니라 아이템 전체 세트를 검색하는 것에 최적화되어 있습니다.

Azure SDK for Python 클라이언트 라이브러리에서는 이를 사용자에게 [ItemPaged]프로토콜(#python-core-protocol-paged)을 통해 제공합니다. `ItemPaged` 프로토콜은 사용자가 기본 페이징을 처리하는 것이 아니라 아이템 전체 세트를 검색하는 것에 최적화되어 있습니다.

{% include requirement/MUST id="python-response-paged-protocol" %} 컬렉션을 반환하는 작업에 대해 [ItemPaged 프로토콜](#python-core-protocol-paged)을 구현하는 값을 반환합니다. [ItemPaged 프로토콜](#python-core-protocol-paged)은 사용자가 반환된 컬렉션의 모든 아이템을 반복하도록 하며, 개별 페이지에 접근할 수 있는 메소드를 제공합니다.
return a value that implements the [ItemPaged protocol]

```python
client = ExampleClient(...)

# List all things - paging happens transparently in the
# background.
for thing in client.list_things():
    print(thing)

# The protocol also allows you to list things by page...
for page_no, page in enumerate(client.list_things().by_page()):
    print(page_no, page)
```

{% include requirement/MAY id="python-response-paged-results" %} 서비스에서 지원하는 경우(예: OData `$top` 쿼리 파라미터) `results_per_page` 키워드 전용 매개변수를 노출합니다.

{% include requirement/SHOULDNOT id="python-response-paged-continuation" %} `list_` 클라이언트 메소드에서 연속성 매개변수를 노출합니다 - 이는 `by_page()` 함수에서 지원됩니다.

```python
client = ExampleClient(...)

# No - don't pass in the continuation token directly to the method...
for thing in client.list_things(continuation_token='...'):
    print(thing)

# Yes - provide a continuation_token to in the `by_page` method...
for page in client.list_things().by_page(continuation_token='...'):
    print(page)
```

{% include requirement/MUST id="python-paged-non-server-paged-list" %} 서비스 API가 현재 서버 주도 페이징을 지원하지 않더라도 [ItemPaged 프로토콜](#python-core-protocol-paged)을 구현하는 값을 반환합니다. 이를 통해 클라이언트 라이브러리에서 파괴적인 변경을 도입하지 않고도 서비스 API에 서버 주도 페이징을 추가할 수 있습니다.

#### 장시간 실행 작업을 호출하는 방법

서비스에서 지속 시간이 오래 걸리는 작업들 (현재 [Microsoft REST API 지침](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#141-principles)에서는 P99 기준으로 0.5초 이상 소요되는 작업)은 장기 실행 작업으로 서비스에 의해 모델링됩니다.

Python 클라이언트 라이브러리는 [Long running operation Poller 프로토콜](#python-core-protocol-lro-poller)을 사용하여 장기 실행 작업을 추상화합니다. 서비스 API가 명시적으로 장기 실행 작업으로 구현되지 않은 경우라도, 일반적인 사용 패턴에서 고객이 대기하거나 상태를 폴링해야 하는 경우 - 이러한 API들은 여전히 Poller 프로토콜을 사용하여 SDK에서 표현되어야 합니다.

{% include requirement/MUST id="python-lro-poller" %} 장기 실행 작업에 대해 [Poller 프로토콜](#python-core-protocol-lro-poller)을 구현하는 객체를 반환합니다.

{% include requirement/MUST id="python-lro-poller-begin-naming" %} 모든 장기 실행 작업에 대해 `begin_` 접두사를 사용합니다.

#### 조건부 요청 메서드

{% include requirement/MUST id="python-method-conditional-request" %} 서비스 메소드에서 조건부 요청을 지원하는 경우 키워드 전용 `match_condition` 매개변수를 추가합니다. 이 매개변수는 `azure-core`에서 정의된 `azure.core.MatchConditions` 타입을 입력으로 지원해야 합니다.

{% include requirement/MUST id="python-method-conditional-request-etag" %} 조건부 요청을 지원하는 서비스 메소드에 키워드 전용 `etag` 매개변수를 추가합니다. `etag` 속성을 가진 모델 인스턴스를 인자로 받는 서비스 메소드의 경우, 명시적으로 전달된 `etag` 값이 모델 인스턴스 내의 값보다 우선합니다.

```python
class Thing(object):

    def __init__(self, name, etag):
        self.name = name
        self.etag = etag

thing = client.get_thing('theName')

# Uses the etag from the retrieved thing instance....
client.update_thing(thing, name='updatedName', match_condition=azure.core.MatchConditions.IfNotModified)

# Uses the explicitly provided etag.
client.update_thing(thing, name='updatedName2', match_condition=azure.core.MatchConditions.IfNotModified, etag='"igotthisetagfromsomewhereelse"')
```

#### 계층형 클라이언트

많은 서비스에서는 중첩된 하위 리소스를 가진 리소스가 있습니다. 예를 들어, Azure Storage는 0개 혹은 그 이상의 컨테이너를 포함하는 계정이 있고, 각 컨테이너는 0개 혹은 그 이상의 블롭을 포함할 수 있습니다.

{% include requirement/MUST id="python-client-hierarchy" %} 각 계층에 해당하는 클라이언트 타입을 생성하되, 단말 리소스 타입에 대해서는 클라이언트 타입을 생성하지 않아도 됩니다. 단말 노드 리소스에 대한 클라이언트 타입 생성은 선택사항입니다.

{% include requirement/MUST id="python-client-hier-creation" %} 각 계층의 클라이언트를 직접적으로 생성할 수 있도록 만듭니다. 생성자는 직접 호출하거나 부모를 통해 호출할 수 있습니다.

```python
class ChildClient:
    # Yes:
    __init__(self, parent, name, credentials, **kwargs) ...

class ChildClient:
    # Yes:
    __init__(self, url, credentials, **kwargs) ...
```

{% include requirement/MUST id="python-client-hier-vend" %} `get_<child>_client(self, name, **kwargs)` 메소드를 제공하여 지정된 이름의 하위 클라이언트를 가져올 수 있도록 합니다. 이 메소드는 하위 자원의 존재를 확인하기 위해 네트워크 호출을 수행하지 않아야 합니다.

{% include requirement/MUST id="python-client-hier-create" %} `create_<child>(...) 메소드`를 제공하여 하위 리소스를 생성합니다. 이 메소드는 새로 생성된 하위 리소스의 클라이언트를 반환해야 합니다.

{% include requirement/SHOULD id="python-client-hier-delete" %} `delete_<child>(...)` 메소드를 제공하여 하위 리소스를 삭제합니다.

### 지원하는 타입:

#### 모델 타입

클라이언트 라이브러리에서는 Azure 서비스로 전송되는 엔티티를 모델 타입으로 나타냅니다. 일부 타입은 서비스와의 왕복 통신에 사용됩니다. 이러한 타입은 서비스로 보내지거나 (추가 또는 업데이트 작업으로) 서비스로부터 검색될 수 있습니다 (get 작업으로). 이러한 타입은 해당 타입에 따라 명명되어야 합니다. 예를 들어, App Configuration의 `ConfigurationSetting`이나 Azure Resource Manager의 `VirtualMachine`과 같은 타입입니다.

모델 타입 내의 데이터는 일반적으로 두 부분으로 분할될 수 있습니다. 서비스의 주요 시나리오 중 하나를 지원하는 데 사용되는 데이터와 그보다 중요하지 않은 데이터입니다. 타입 `Foo`가 주어진 경우, 중요하지 않은 세부 정보는 `FooDetails`라는 타입으로 수집하고 `Foo`에 `details` 속성으로 첨부할 수 있습니다.

{% include requirement/MUST id="python-models-input-dict" %} 모델 타입 대신 대안으로 딕셔너리를 입력으로 지원합니다.

{% include requirement/MUST id="python-models-input-constructor" %} 사용자에 의해 인스턴스화되는 모델을 위해 생성자를 작성합니다. 이 생성자는 필수 정보를 최소한으로 받고 선택적인 정보를 키워드 전용 인자로 받습니다.

{% include requirement/MAY id="python-models-generated" %} 가이드라인을 충족하는 경우, 생성된 레이어의 모델을 노출하기 위해 루트 `__init__.py` (및 `__all__`)에 추가합니다.

{% include requirement/MUSTNOT id="python-models-async" %} 루트와 `aio` 네임스페이스 간에 모델을 중복하여 생성합니다.

응답의 왕복 흐름을 용이하게 하기 위해 (일반적으로 리소스 가져오기 -> 조건부 리소스 수정 -> 리소스 설정 작업에서 사용되는 경우) 출력 모델 타입은 가능한 경우 입력 모델 타입 (예: `ConfigurationSetting`)을 사용해야 합니다. `ConfigurationSetting` 타입은 입력 모델로 사용될 때 무시되지만, 서버에서 생성된 (읽기 전용) 속성을 포함해야 합니다.

- 모델의 부분 스키마를 반환하는 경우, 열거형이 각 항목에 대해 `<model>Item`을 사용합니다. 예를 들어, GetBlobs()는 BlobItem의 열거형을 반환하는데, 이는 Blob의 이름과 메타데이터를 포함하지만 Blob의 내용은 포함하지 않습니다.
- 작업 결과에 대해서는 `<operation>Result`를 사용합니다. `<operation>`은 특정 서비스 작업과 연결됩니다. 동일한 결과가 여러 작업에 사용될 수 있는 경우, 적절한 명사-동사 구문을 사용합니다. 예를 들어, `UploadBlob`의 결과에는 `UploadBlobResult`를 사용하지만, Blob 컨테이너를 변경하는 다양한 메소드의 결과에는 `ContainerChangeResult`를 사용합니다.

{% include requirement/MUST id="python-models-dict-result" %} 다른 API의 입력 매개변수로 `<operation>Result` 클래스를 사용하지 않는 경우, `<operation>Result` 클래스를 생성하는 대신 간단한 Mapping (예: `dict`)을 사용합니다.

다음 표는 가능한 모델을 열거합니다:

| 타입                | 예시                | 사용법                                                            |
|---------------------|---------------------|-------------------------------------------------------------------|
| \<model>            | Secret              | 리소스의 전체 데이터                                              |
| \<model>Details     | SecretDetails       | 리소스에 대한 상세 정보. \<model>.details에 첨부됩니다.            |
| \<model>Item        | SecretItem          | 열거를 위해 반환된 데이터의 일부분                                  |
| \<operation>Result  | AddSecretResult     | 단일 작업을 위한 일부 또는 다른 데이터 집합                       |
| \<model>\<verb>Result | SecretChangeResult | 모델에 대한 여러 작업을 위한 일부 또는 다른 데이터 집합           |

```python
# An example of a model type.
class ConfigurationSetting(object):
    """Model type representing a configuration setting

    :ivar name: The name of the setting
    :vartype name: str
    :ivar value: The value of the setting
    :vartype value: object
    """

    def __init__(self, name, value):
        # type: (str, object) -> None
        self.name = name
        self.value = value

    def __repr__(self):
        # type: () -> str
        return json.dumps(self.__dict__)[:1024]
```

#### 열거형

{% include requirement/MUST id="python-models-enum-string" %} 확장 가능한 열거형(Extensible Enumerations)을 사용하세요.

{% include requirement/MUST id="python-models-enum-name-uppercase" %} 열거형 이름에는 대문자로 된 이름을 사용하세요

```python

# Yes
class MyGoodEnum(str, Enum):
    ONE = 'one'
    TWO = 'two'

# No
class MyBadEnum(str, Enum):
    One = 'one' # No - using PascalCased name.
    two = 'two' # No - using all lower case name.
```

### 예외

{% include requirement/SHOULD id="python-errors-azure-exceptions" %} [`azure-core` 패키지에서 기존 예외 타입](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/index.html#azure-core-library-exceptions)을 별도로 생성하는 대신, 기존 예외 타입을 사용하는 것을 선호합니다.

{% include requirement/MUSTNOT id="python-errors-use-standard-exceptions" %} [기본 제공 예외 타입](https://docs.python.org/3/library/exceptions.html)으로 충분한 경우라면 새로운 예외 타입을 생성하지 마세요.

{% include requirement/SHOULDNOT id="python-errors-new-exceptions" %} 개발자가 프로그램적으로 오류를 처리할 수 있는 경우가 아니라면, 새로운 예외 타입을 생성하지 마세요. 서비스 작업 실패와 관련된 특수화된 예외 타입은 [`azure-core`](https://azuresdkdocs.blob.core.windows.net/$web/python/azure-core/1.9.0/index.html#azure-core-library-exceptions) 패키지의 기존 예외 타입을 기반으로 해야 합니다.

여러 개의 HTTP 요청을 사용하는 상위 수준 메서드의 경우, 마지막 예외 또는 모든 실패의 집합적인 예외(aggregated exception)를 생성해야 합니다.

{% include requirement/MUST id="python-errors-rich-info" %} 예외에는 서비스별 오류 정보를 포함해야 합니다. 서비스별 오류 정보는 서비스별 속성이나 필드에 제공되어야 합니다.

{% include requirement/MUST id="python-errors-documentation" %} 각 메서드에서 생성되는 오류를 문서화해야 합니다. 일반적으로 Python에서 문서화되지 않는 일반적으로 발생하는 오류 (예: `ValueError`, `TypeError`, `RuntimeError` 등)를 문서화하지 마세요.

### 인증

{% include requirement/MUST id="python-auth-credential-azure-core" %} use the credentials classes in `azure-core` whenever possible.

{% include requirement/MUST id="python-auth-policy-azure-core" %} 가능한 경우, `azure-core`의 인증(credentials) 클래스를 사용하세요.

{% include requirement/MAY  id="python-auth-service-credentials" %} 서비스에 필요한 경우, [Architecture board]의 지침에 따라 필요한 경우 추가적인 인증(credentials) 타입을 추가하세요.

{% include requirement/MUST id="python-auth-service-support" %} 서비스가 지원하는 모든 인증(authentication) 방법을 지원하세요.

### 네임스페이스

다음 가이드라인에서 "네임스페이스(namespace)"라는 용어는 파이썬 패키지 또는 모듈을 의미합니다. 이는 코드에서 가져올 수 있는 것을 가리킵니다. "배포 패키지(distribution package)"라는 용어는 패키지 매니저를 통해 게시하고 설치하는 아티팩트를 설명하는 데 사용됩니다. 이는 일반적으로 pip install을 통해 설치하는 것을 의미합니다.

{% include requirement/MUST id="python-namespaces-prefix" %} 라이브러리를 구현할 때, `azure` 루트 네임스페이스의 하위 패키지로 라이브러리를 구성하세요.

> 참고: 루트 네임스페이스로 `microsoft`를 사용해서는 안 됩니다. 다른 프로젝트 확장을 위한 정책 요구 사항(예: `opentelemetry`와 같은) 때문에 `microsoft`를 네임스페이스에 포함해야 하는 경우, 해당 패키지의 고유한 네임스페이스와 밑줄(_)로 연결하여 사용해야 합니다 (예: `microsoft_myservice`). 이러한 경우에도 `microsoft-myservice`를 배포 패키지 이름으로 사용할 수 있습니다.

{% include requirement/MUST id="python-namespaces-naming" %} 소비자가 사용 중인 서비스와 네임스페이스를 연결할 수 있는 패키지 이름을 선택하세요. 기본적으로, 네임스페이스의 끝에 압축된 서비스 이름을 사용하세요. 네임스페이스는 제품의 브랜딩이 변경되어도 변경되지 않습니다. 변경될 수 있는 마케팅용 이름 사용을 피하세요.

압축된 서비스 이름은 공백이 없는 서비스 이름입니다. 압축된 버전이 커뮤니티에서 잘 알려진 경우, 더 축약될 수도 있습니다. 예를 들어, "Azure Media Analytics"는 압축된 서비스 이름으로 `mediaanalytics`가 될 수 있고, "Azure Service Bus"는 `servicebus`로 변환됩니다. 필요한 경우 단어를 언더스코어(_)로 구분하세요. 예를 들어, `mediaanalytics`는 `media_analytics`로 구분될 수 있습니다.

{% include requirement/MAY id="python-namespaces-grouping" %} 서비스 또는 서비스 그룹이 공통된 동작 (예: 공유 인증 타입)을 가지는 경우, 네임스페이스에 그룹 이름 세그먼트를 포함하세요. 예를 들어, `azure.<group>.<servicename>`과 같이 그룹 이름 세그먼트를 네임스페이스에 포함시킬 수 있습니다.

{% include requirement/MUST id="python-namespaces-grouping-dont-introduce-new-packages" %} 동일한 이름만 다른 새로운 배포 패키지를 도입하지 않도록 주의하세요. 기존 패키지의 경우, 그룹 이름을 도입하기 위해 패키지 이름을 변경하지 않아야 합니다.

그룹 이름 세그먼트를 사용하려면 다음 중 하나의 그룹을 사용하세요:

{% include tables/data_namespaces.md %}

{% include requirement/MUST id="python-namespaces-mgmt" %} Azure Resource Manager (관리) API는 `mgmt` 그룹에 위치시킵니다. 네임스페이스에는 `azure.mgmt.<servicename>` 그룹을 사용합니다. 데이터 평면 API보다 제어 평면 API를 요구하는 서비스가 많기 때문에 다른 네임스페이스는 명시적으로 제어 평면에만 사용될 수 있습니다.

{% include requirement/MUST id="python-namespaces-register" %} 선택한 네임스페이스를 [아키텍처 위원회]에 등록하십시오. 네임스페이스 요청을 위해 [이슈]를 열어주세요. 현재 등록된 네임스페이스 목록은 등록된 네임스페이스 목록에서 확인하실 수 있습니다.

{% include requirement/MUST id="python-namespaces-async" %} 동기 클라이언트의 네임스페이스에 `.aio` 접미사를 추가하여 비동기 클라이언트를 사용하세요.

Example:

```python
# Yes:
from azure.exampleservice.aio import ExampleServiceClient

# No: Wrong namespace, wrong client name...
from azure.exampleservice import AsyncExampleServiceClient
```

#### 네임스페이스 예시

다음은 이 가이드라인을 만족하는 네임스페이스의 예시입니다:

- `azure.storage.blob`
- `azure.keyvault.certificates`
- `azure.ai.textanalytics`
- `azure.mgmt.servicebus`

### 비동기 지원

`asyncio` 라이브러리는 Python 3.4부터 사용 가능하였고, `async`/`await` 키워드는 Python 3.5에서 도입되었습니다. 그럼에도 불구하고, 대부분의 파이썬 개발자들은 비동기 메서드만을 제공하는 라이브러리를 사용하는데 익숙하지 않거나 편하지 않습니다.

{% include requirement/MUST id="python-client-sync-async" %} 당신의 API는 동기 버전과 비동기 버전 모두를 제공해야 합니다.

{% include requirement/MUST id="python-client-async-keywords" %} `async`/`await` 키워드를 사용하세요 (Python 3.5 이상 필요). [yield from coroutine or asyncio.coroutine](https://docs.python.org/3.4/library/asyncio-task.html) 구문은 사용하지 마세요.

{% include requirement/MUST id="python-client-separate-sync-async" %} 동기 작업과 비동기 작업을 위해 두 개의 별도의 클라이언트 클래스를 제공하세요. 동기 작업과 비동기 작업을 같은 클래스에 결합하지 마세요.

```python
# Yes
# In module azure.example
class ExampleClient(object):
    def some_service_operation(self, name, size) ...

# In module azure.example.aio
class ExampleClient:
    # Same method name as sync, different client
    async def some_service_operation(self, name, size) ...

# No
# In module azure.example
class ExampleClient(object):
    def some_service_operation(self, name, size) ...

class AsyncExampleClient: # No async/async pre/postfix.
    async def some_service_operation(self, name, size) ...

# No
# In module azure.example
class ExampleClient(object): # Don't mix'n match with different method names
    def some_service_operation(self, name, size) ...
    async def some_service_operation_async(self, name, size) ...

```

{% include requirement/MUST id="python-client-same-name-sync-async" %} 동기와 비동기 패키지에 대해 동일한 클라이언트 이름을 사용하세요.

예시:

|동기/비동기|네임스페이스|배포 패키지 이름|클라이언트 이름|
|-|-|-|-|
|동기|`azure.sampleservice`|`azure-sampleservice`|`azure.sampleservice.SampleServiceClient`|
|비동기|`azure.sampleservice.aio`|`azure-sampleservice-aio`|`azure.sampleservice.aio.SampleServiceClient`|

{% include requirement/MUST id="python-client-namespace-sync" %} 동기화 클라이언트에 대해 `.aio`가 추가된 패키지의 동기화 버전과 동일한 네임스페이스를 사용하세요.

예시:

```python
from azure.storage.blob import BlobServiceClient # Sync client

from azure.storage.blob.aio import BlobServiceClient # Async client
```

{% include requirement/SHOULD id="python-client-separate-async-pkg" %} 비동기 버전이 추가적인 종속성을 필요로 한다면, 비동기 지원을 위한 별도의 패키지를 제공하세요.

{% include requirement/MUST id="python-client-same-pkg-name-sync-async" %} 패키지의 비동기 버전에 `-aio`가 추가된 패키지의 동기 버전과 동일한 이름을 사용합니다.

{% include requirement/MUST id="python-client-async-http-stack" %} 비동기 작업에 대한 기본 HTTP 스택으로 [`aiohttp`](https://aiohttp.readthedocs.io/en/stable/)를 사용하세요. 비동기 클라이언트에 대한 기본 `전송` 타입으로 `azure.core.pipeline.transport.AioHttpTransport`를 사용하세요.

## Azure SDK 배포 패키지

### 패키징

{% include requirement/MUST id="python-packaging-name" %} 메인 클라이언트 클래스의 네임스페이스에 따라 패키지 이름을 지으세요. 예를 들어, 메인 클라이언트 클래스가 `azure.data.tables` 네임스페이스에 있다면, 패키지 이름은 azure-data-tables이어야 합니다.

{% include requirement/MUST id="python-packaging-name-allowed-chars" %} 패키지 이름에는 모두 소문자를 사용하고, 분리자로 대시(-)를 사용하세요.

{% include requirement/MUSTNOT id="python-packaging-name-disallowed-chars" %} 패키지 이름에는 밑줄 (_) 또는 마침표 (.)를 사용하지 마세요. 네임스페이스에 밑줄이 포함되어 있다면, 배포 패키지 이름에서 밑줄을 대시(-)로 바꾸세요.

{% include requirement/MUST id="python-packaging-follow-repo-rules" %} [azure-sdk-packaging wiki](https://github.com/Azure/azure-sdk-for-python/wiki/Azure-packaging) 위키에서의 특정 패키지 지침을 따르세요.

{% include requirement/MUST id="python-packaging-follow-python-rules" %} Python 3.x를 대상으로 하는 패키지의 경우, [Python 3.x를 위한 네임스페이스 패키지 권장사항](https://docs.python.org/3/reference/import.html#namespace-packages)을 따르세요.

{% include requirement/MUST id="python-general-supply-sdist" %} 소스 배포(sdist)와 휠(wheels)을 모두 제공하세요.

{% include requirement/MUST id="python-general-pypi" %} 소스 배포(`sdist`)와 휠(wheels) 모두를 PyPI에 게시하세요.

{% include requirement/MUST id="python-general-wheel-behavior" %} [순수](https://packaging.python.org/guides/distributing-packages-using-setuptools/#id75) 파이썬 및 [범용](https://packaging.python.org/guides/distributing-packages-using-setuptools/#universal-wheels) 파이썬 휠(wheels)의 올바른 동작을 CPython과 PyPy 양쪽에서 테스트하세요.

{% include requirement/MUST id="python-packaging-nspkg" %} Python 2.x의 경우 azure-nspkg에 의존하세요.

{% include requirement/MUST id="python-packaging-group-nspkg" %} Python 2.x에서 `네임스페이스 그룹`을 사용하는 경우 `azure-<group>-nspkg`에 의존하세요.

{% include requirement/MUST id="python-packaging-init" %} sdist에는 네임스페이스의 `__init__.py`를 포함하세요.

#### 서비스별 공통 라이브러리 코드

일부 경우에는 여러 클라이언트 라이브러리 간에 공통 코드를 공유해야 할 때가 있습니다. 예를 들어, 일련의 협력 클라이언트 라이브러리들은 예외나 모델과 같은 공통 코드를 공유하길 원할 수 있습니다.

{% include requirement/MUST id="python-commonlib-approval" %} 공통 라이브러리를 구현하기 전에 [아키텍처 위원회]의 승인을 받으세요.

{% include requirement/MUST id="python-commonlib-minimize-code" %} 공통 라이브러리 내의 코드를 최소화하세요. 공통 라이브러리 내의 코드는 클라이언트 라이브러리의 소비자에게 사용 가능하며, 동일한 네임스페이스 내에서 여러 클라이언트 라이브러리에서 공유됩니다.

공통 라이브러리는 다음 조건을 충족하는 경우 승인됩니다:

- 공유되지 않는 라이브러리의 사용자가 공통 라이브러리 내의 객체를 직접 사용해야 한다.
- 정보가 여러 클라이언트 라이브러리 간에 공유될 것이다.
두 가지 예시를 살펴보겠습니다:

두 개의 Cognitive Services 클라이언트 라이브러리를 구현하는 경우, 두 라이브러리가 동일한 비즈니스 로직에 의존한다면, 공통 라이브러리를 선택하는 것이 적절합니다.

두 개의 Cognitive Services 클라이언트 라이브러리에는 모델 (데이터 클래스)의 구조가 동일하지만, 관련된 로직이 없거나 매우 적을 경우, 이는 공유 라이브러리로 적합하지 않습니다. 대신에 두 개의 별도 클래스를 구현하세요.

### 패키지 버전 관리

{% include requirement/MUST id="python-versioning-semver" %} 패키지에는 [semantic versioning](https://semver.org)을 사용하세요.

{% include requirement/MUST id="python-versioning-beta" %} beta 릴리스를 위해 bN을 [베타 릴리스 세그먼트](https://www.python.org/dev/peps/pep-0440/#pre-releases)로 사용하세요.

[PEP440](https://www.python.org/dev/peps/pep-0440)에서 정의된 것 이외의 사전 릴리스 세그먼트 (`aN`, `bN`, `rcN`) 외에는 사전 릴리스 세그먼트를 사용하지 마세요. 빌드 도구, 게시 도구 및 인덱스 서버는 버전을 올바르게 정렬하지 못할 수 있습니다.

{% include requirement/MUST id="python-versioning-changes" %} 라이브러리에 `어떤 변경 사항이든` 발생하는 경우 버전 번호를 변경하세요.

{% include requirement/MUST id="python-versioning-patch" %} 만약 패키지에 버그 수정만 추가된다면, 패치 버전을 증가시키세요.

{% include requirement/MUST id="python-verioning-minor" %} 만약 패키지에 새로운 기능이 추가된다면, 마이너 버전을 증가시키세요.

{% include requirement/MUST id="python-versioning-apiversion" %} 만약 기본 REST API 버전이 변경된다면, 라이브러리의 공개 API 변경이 없더라도 적어도 마이너 버전은 증가시키세요.

{% include requirement/MUSTNOT id="python-versioning-api-major" %} 파이썬 라이브러리 자체에서 API 변경이 필요하지 않은 경우, 새로운 REST API 버전에 대해서는 주 버전을 증가시키세요.

{% include requirement/MUST id="python-versioning-major" %} 패키지에 중대한 변경 사항이 있는 경우, 주 버전을 증가시키세요. 중대한 변경 사항은 [아키텍처 위원회]의 사전 승인을 필요로 합니다.

{% include requirement/MUST id="python-versioning-major-cross-languages" %} 다른 스코프나 언어에서 해당 서비스에 대해 릴리스된 Track 1 패키지의 가장 높은 버전 번호보다 큰 버전 번호를 선택하세요.

GA(일반 사용 가능) 클라이언트 라이브러리의 경우, 중요한 변경 사항을 도입하기 위한 장벽은 매우 높습니다. 다이아몬드 종속성 문제를 피하기 위해 새로운 이름의 패키지를 생성할 수도 있습니다.

### 의존성

{% include requirement/MUST id="python-dependencies-approved-list" %} 단일 기능을 위한 외부 종속성은 다음 목록의 잘 알려진 패키지 중에서만 선택하세요:

{% include_relative approved_dependencies.md %}

{% include requirement/MUSTNOT id="python-dependencies-external" %} 잘 알려진 종속성 목록 이외의 외부 종속성을 사용하지 마세요. 새로운 종속성을 추가하려면 [아키텍처 위원회]에 문의하세요.

{% include requirement/MUSTNOT id="python-dependencies-vendor" %} [아키텍처 위원회]의 승인이 없는 한, 벤더 종속성을 사용하지 마세요.

Python에서 종속성을 벤더링할 때, 다른 패키지의 소스 코드를 자신의 패키지 일부인 것처럼 포함시킵니다.

{% include requirement/MUSTNOT id="python-dependencies-pin-version" %} 의존성의 버전에 버그가 있는 경우가 아니라면, 해당 종속성의 특정 버전을 지정하세요. 그 버전의 버그를 우회하기 위한 유일한 방법인 경우에만 다른 방법을 사용하세요.

정확한 종속성을 고정하는 것은 응용 프로그램에만 해당됩니다. 라이브러리는 그렇지 않습니다. 라이브러리는 종속성에 대해 [호환되는 릴리스](https://www.python.org/dev/peps/pep-0440/#compatible-release) 식별자를 사용해야 합니다.

### 바이너리 확장 (네이티브 코드)

{% include requirement/MUST id="python-native-approval" %} 바이너리 확장을 구현하기 전에 [아키텍처 위원회]의 승인을 받으세요.

{% include requirement/MUST id="python-native-plat-support" %} Windows, Linux (manylinux), 그리고 MacOS를 지원하세요. 가능한 최초의 manylinux를 지원하여 가능한 많은 플랫폼에서 작동하도록 하세요. 이에 대한 자세한 내용은 [PEP513](https://www.python.org/dev/peps/pep-0513/)와 [PEP571](https://www.python.org/dev/peps/pep-0571/)을 참조하세요.

{% include requirement/MUST id="python-native-arch-support" %} x86 및 x64 아키텍처를 모두 지원하세요.

{% include requirement/MUST id="python-native-charset-support" %} CPython 2.7의 Unicode 및 ASCII 버전을 모두 지원하세요.

### Docstrings

{% include requirement/MUST id="python-docstrings-pydocs" %} 이 문서에서 명시적으로 다른 내용이 없는 한, [문서 작성 지침](http://aka.ms/pydocs)을 따르세요.

{% include requirement/MUST id="python-docstrings-all" %} 모든 공개 모듈, 타입, 상수 및 함수에 대해 docstring을 제공하세요.

{% include requirement/MUST id="python-docstrings-kwargs" %} 메서드에서 직접 사용되는 `**kwargs`에 대해서는 문서화하세요. 만약 `**kwargs`가 전달되는 경우에는 호출된 메서드의 시그니처를 참조해도 됩니다.

예시:
```python
def request(method, url, headers, **kwargs): ...

def get(*args, **kwargs):
    "Calls `request` with the method "GET" and forwards all other arguments."
    return request("GET", *args, **kwargs)
```

{% include requirement/MUST id="python-docstrings-exceptions" %} 메서드에서 명시적으로 발생할 수 있는 예외와 호출된 메서드에서 발생하는 예외에 대해 문서화하세요.

#### 코드 스니펫

{% include requirement/MUST id="python-snippets-include" %} 라이브러리 코드와 함께 예제 코드 스니펫을 리포지토리 내에 포함하세요. 스니펫은 대부분의 개발자가 라이브러리를 사용하여 수행해야 하는 작업을 명확하고 간결하게 보여줘야 합니다. 일반적인 작업에 대한 스니펫은 모두 포함하며, 특히 복잡하거나 라이브러리를 처음 사용하는 사용자에게 어려울 수 있는 작업에 대한 스니펫을 추가하세요. 최소한 라이브러리의 핵심 시나리오에 해당하는 스니펫을 포함하세요.

{% include requirement/MUST id="python-snippets-build" %} 예제 코드 스니펫을 빌드하고 테스트하세요. 이를 위해 리포지토리의 지속적인 통합 (CI)을 사용하여 스니펫이 계속 작동하는지 확인하세요.

{% include requirement/MUST id="python-snippets-docstrings" %} 라이브러리의 docstring에 예제 코드 스니펫을 포함하여 API 참조에 나타나도록 하세요. 언어와 해당 도구가 지원하는 경우, docstring 내에서 직접 API 참조로 스니펫을 가져옵니다. 각 예제는 유효한 `pytest` 형식이어야 합니다.

라이브러리의 Python docstring에는 `literalinclude` 지시문을 사용하여 Sphinx에게 [스니펫을 자동으로 가져오도록][1] 지시하세요.

{% include requirement/MUSTNOT id="python-snippets-combinations" %} 코드 스니펫에 여러 작업을 조합하지 마세요. 작업의 타입이나 멤버를 보여주거나, 기존의 원자적 작업을 보완하는 경우가 아니라면 여러 작업을 포함하지 않도록 하세요. 예를 들어, Cosmos DB 코드 스니펫에는 계정 및 컨테이너 생성 작업이 모두 포함되어서는 안됩니다. 대신, 계정 생성을 위한 하나의 스니펫과 컨테이너 생성을 위한 다른 스니펫을 만드세요.

## 저장소 가이드

### 문서 작성 스타일

클라이언트 라이브러리에 포함되거나 동반되어야 하는 여러 가지 문서 전달물이 있습니다. 코드 자체 내에서의 완전하고 유용한 API 문서 (docstring) 이외에도, 훌륭한 README와 기타 지원 문서가 필요합니다.

* `README.md` - SDK 리포지토리 내에서 라이브러리 디렉터리의 루트에 위치하며, 패키지 설치 및 클라이언트 라이브러리 사용 정보를 포함합니다. ([예시][https://github.com/Azure/azure-sdk-for-python/blob/main/sdk/appconfiguration/azure-appconfiguration/README.md])
* `API reference` - 라이브러리의 코드 내의 docstring에서 생성되며, docs.microsoft.com에 게시됩니다.
* `Code snippets` - 라이브러리의 README, docstring 및 Quickstart에 포함된, 싱글(원자적) 작업을 보여주는 짧은 코드 예제입니다. 이는 라이브러리의 주요 시나리오를 위해 식별한 예시입니다.
* `Quickstart` - README 내용과 유사하지만 확장된 docs.microsoft.com의 문서입니다. 일반적으로 해당 서비스의 콘텐츠 개발자가 작성합니다.
* `Conceptual` - Quickstarts, 튜토리얼, 사용 가이드 및 기타 docs.microsoft.com의 긴 형식 문서입니다. 일반적으로 해당 서비스의 콘텐츠 개발자가 작성합니다.

{% include requirement/MUST id="python-docs-content-dev" %} 라이브러리에 대한 adparch 검토에 서비스의 콘텐츠 개발자를 포함하세요. 작업할 콘텐츠 개발자를 찾으려면 팀의 프로그램 관리자와 확인하세요.

{% include requirement/MUST id="python-docs-contributor-guide" %} [Azure SDK Contributors Guide]를 따르세요. (마이크로소프트 내부용)

{% include requirement/MUST id="python-docs-style-guide" %} 공개 문서 작성 시 Microsoft 스타일 가이드에 명시된 사양을 준수하세요. 이는 README와 코드의 docstring과 같은 긴 형식의 문서에 적용됩니다. (마이크로소프트 내부용)

* [Microsoft Writing Style Guide].
* [Microsoft Cloud Style Guide].

{% include requirement/SHOULD id="python-docs-into-silence" %} 문서화를 통해 라이브러리의 사용 방법에 대한 질문을 미리 예방하고, API를 명확히 설명함으로써 GitHub 이슈를 최소화하세요. docstring에서 서비스 제한 사항과 발생할 수 있는 오류에 대한 정보, 그리고 해당 오류를 피하고 복구하는 방법을 명시적으로 포함시키세요.

코드를 작성할 때, 그 코드에 대한 *문서를 작성하여 다른 사람들로부터 문의를 최소화*하세요. 클라이언트 라이브러리에 대해 받아야 하는 질문이 줄어들면, 해당 서비스의 새로운 기능을 구축하는 데 더 많은 시간을 할애할 수 있습니다.

### 샘플

코드 샘플은 클라이언트 라이브러리와 관련된 특정 기능을 보여주는 작은 응용 프로그램입니다. 샘플은 개발자가 클라이언트 라이브러리의 전체 사용 요구 사항을 빠르게 이해할 수 있도록 도와줍니다. 코드 샘플은 해당 기능을 보여주기에 필요한 만큼 복잡하지 않아야 합니다. 전체 애플리케이션을 작성하지 마세요. 샘플은 유용한 코드와 관련 없는 이유로 발생하는 보일러플레이트 코드 사이에 높은 신호 대 잡음비를 가져야 합니다.

{% include requirement/MUST id="python-samples-include-them" %} 라이브러리의 코드와 함께 리포지토리 내에 코드 샘플을 포함하세요. 이 샘플은 개발자가 라이브러리를 사용하여 작성해야 하는 코드를 명확하고 간결하게 보여줍니다. 모든 일반적인 작업에 대한 샘플을 포함하세요. 라이브러리를 처음 사용하는 사용자에게 복잡하거나 어려울 수 있는 작업에 특히 주의하세요. 라이브러리의 핵심 시나리오에 해당하는 샘플을 포함하세요.

{% include requirement/MUST id="python-samples-location" %} 코드 샘플을 클라이언트 라이브러리 루트 디렉터리 내의 /samples 디렉터리에 배치하세요. 샘플은 최종 배포 패키지에 포함됩니다.

{% include requirement/MUST id="python-samples-runnable" %} 각 샘플 파일이 실행 가능하도록 보장하세요.

{% include requirement/MUST id="python-samples-coding-style" %} 샘플을 작성할 때는 Python 3 관례를 따르세요. [`six`](https://six.readthedocs.io)와 같은 Python 2 호환 코드를 포함하지 마세요. 이는 제시하려는 내용에서 주의를 분산시킬 수 있습니다. Python 3의 기준 버전 이후에 나온 기능을 사용하는 것도 피하세요. 현재 지원되는 Python 버전은 3.6입니다.

{% include requirement/MUST id="python-samples-grafting" %} 문서에서 코드 샘플을 사용자의 애플리케이션에 쉽게 통합할 수 있도록 하세요. 예를 들어, 다른 샘플에서 변수 선언에 의존하지 않도록 주의하세요.

{% include requirement/MUST id="python-samples-readability" %} 코드의 읽기와 이해를 용이하게 하는 데 초점을 맞추어 코드 샘플을 작성하세요. 코드의 간결성과 효율성보다 가독성을 중시하세요.

{% include requirement/MUST id="python-samples-platform-support" %} 샘플이 Windows, macOS, 그리고 Linux 개발 환경에서 실행될 수 있도록 보장하세요.

{% include requirement/MUSTNOT id="python-snippets-no-combinations" %} 타입 또는 멤버를 보여주기 위해 필요한 경우가 아니라면, 여러 시나리오를 하나의 코드 샘플에 결합하지 마세요. 예를 들어, Cosmos DB 코드 샘플에는 계정 생성 및 컨테이너 생성 작업이 동시에 포함되어서는 안됩니다. 대신 계정 생성을 위한 샘플과 컨테이너 생성을 위한 다른 샘플을 생성하세요.

결합된 시나리오는 현재 집중하고 있는 작업과는 관련이 없는 추가 작업에 대한 지식이 필요합니다. 개발자는 먼저 작업 중인 시나리오 주변의 코드를 이해해야 하며, 코드 샘플을 그대로 복사하여 프로젝트에 붙여넣을 수 없습니다.

{% include refs.md %}
{% include_relative refs.md %}
