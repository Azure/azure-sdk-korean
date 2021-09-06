---
title: "파이썬 가이드 라인 : 문서"
keywords: guidelines python
permalink: python_documentation.html
folder: python
sidebar: general_sidebar
---

클라이언트 라이브러리에 포함해야 하는 몇 가지 문서 자료가 있습니다. 
클라이언트 라이브러리에는 코드 자체 내의 유용한 API 설명서 외에도 README 및 기타 지원 문서가 필요합니다.

* `README.md` - SDK 저장소 내 라이브러리 디렉토리의 루트에 위치하며 패키지 설치 및 클라이언트 라이브러리 사용 정보를 포함합니다. ([example][README-EXAMPLE])
* `API reference` - 코드의 the docstrings에서 생성되며 docs.microsoft.com에 게시됩니다. 
* `Code snippets` - 라이브러리에서 식별한 시나리오에 대한 단일(원자) 연산을 보여주는 간단한 코드 예입니다. 이는 README, docstrings, and Quickstart 에 사용됩니다. 
* `Quickstart` - README 와 유사하지만 확장되는 docs.microsoft.com 관련 기사입니다. 일반적으로 서비스 콘텐츠 개발자가 작성합니다.  
* `Conceptual` - Quickstarts, 튜토리얼, 사용 방법 가이드 및 기타 컨텐츠와 같은 긴 형식의 설명서입니다. 일반적으로 서비스 콘텐츠 개발자가 작성합니다. 

{% include requirement/MUST id="python-docs-content-dev" %} 서비스의 콘텐츠 개발자를 라이브러리에 대한 임시 보관소 리뷰어로 포함하시오. 함께 작업해야 할 콘텐츠 개발자를 찾으려면 팀의 프로그램 관리자에게 문의하십시오.

{% include requirement/MUST id="python-docs-contributor-guide" %} [Azure SDK Contributors 가이드라인] 을 따르시오. (MICROSOFT INTERNAL)

{% include requirement/MUST id="python-docs-style-guide" %} 공개 문서를 작성할 때 Microsoft 스타일 가이드에 명시된 사양을 준수하십시오. 이는 README와 같은 긴 양식 설명서와 코드의 docstrings 모두에 적용됩니다(MICROFT INTERNAL).

* [Microsoft 작성 스타일 안내서].
* [Microsoft 클라우드 스타일 안내서].

{% include requirement/SHOULD id="python-docs-into-silence" %} 당신의 라이브러리를 침묵으로 만들려는 시도를 합니다. 문서 문자열에 API를 명확하게 설명하여 개발자의 사용 질문을 사전에 차단하고 GitHub 문제를 최소화합니다. 서비스 제한 및 이러한 오류가 발생할 수 있는 오류, 이러한 오류를 방지하고 복구하는 방법에 대한 정보를 포함합니다.

코드를 작성하면서 더이상 질문을 받지 않을 수 있도록 문서 작성을 하세요. 클라이언트 라이브러리에 관한 질문을 더 적게 받을 수록 서비스를 위한 새로운 기능을 구축해야 하는 시간이 늘어납니다.

## Docstrings

{% include requirement/MUST id="python-docstrings-pydocs" %} 이 문서에서 명시적으로 재정의되지 않은 경우 [문서 작성 가이드라인](http://aka.ms/pydocs) 을 따르세요.

{% include requirement/MUST id="python-docstrings-all" %} 모든 공용 모듈, 유형 및 메서드에 대한 docstrings를 제공합니다.

{% include requirement/MUST id="python-docstrings-kwargs" %} 메소드에서 직접적으로 사용되는 모든  `**kwargs` 를 문서화하고 [core options](https://aka.ms/azsdk/python/options)에 참조 링크를 추가하여 공유 옵션에 대한 소개를 제공합니다. `**kwargs` 가 전달되면,호출된 메소드의 서명을 참조 할 수 있습니다.

예시:
```python
def request(method, url, headers, **kwargs): ...

def get(*args, **kwargs):
    """Calls `request` with the method "GET" and forwards all other arguments.

    :param str method-param: The method-param parameter
    :keyword int method-kwarg: The optional method-kwarg parameter

    For additional request configuration options, please see https://aka.ms/azsdk/python/options.
    """
    return request("GET", *args, **kwargs)
```

{% include requirement/MUST id="python-docstrings-exceptions" %} 메소드안에 명시적으로 제기될 수 있는 document exceptions 및 호출된 방법에 의해 제기되는 예외입니다.

### Code snippets

{% include requirement/MUST id="python-snippets-include" %} 예제 코드 스니펫은 저장소 내에 라이브러리의 코드와 함께 포함합니다. 이러한 코드 스니펫은 대부분의 개발자가 라이브러리에서 수행해야 하는 작업을 명확하고 간결하게 보여 줍니다. 모든 일반 작업, 특히 라이브러리의 새 사용자에게 복잡하거나 어려울 수 있는 작업에 대한 스니펫을 포함합니다. 최소한 라이브러리에 대해 식별한 챔피언 시나리오에 대한 스니펫을 포함합니다.

{% include requirement/MUST id="python-snippets-build" %} 저장소의 CI(Continuous Integration)를 사용하여 예제 코드 조각을 빌드하고 테스트하여 제대로 작동하는지 확인합니다.

{% include requirement/MUST id="python-snippets-docstrings" %} 예제 코드 스니펫을 라이브러리의 문서 문자열에 포함시켜 해당 문서열이 API 참조에 나타나도록 합니다. 언어와 해당 도구가 지원하는 경우 이러한 코드 스니펫을 문서 문자열 내에서 API 참조로 직접 수집합니다. 각 샘플은 유효한 `pytest`여야 합니다.

파이썬 docstrings의 `literalinclude` 지시어를 사용하여 Sphinx가 [자동으로 스니펫을 수집하도록 지시하십시오][1].

{% include requirement/MUSTNOT id="python-snippets-combinations" %} 유형이나 멤버를 시연하는 데 필요한게 아니거나 원자 연산을 시연하는게 아니라면 코드 스니펫에 둘 이상의 작업을 결합합니다. 예를 들어, Cosmos DB 코드 스니펫은 계정 생성 작업과 컨테이너 생성 작업을 모두 포함해서는 안됩니다. 두 개의 다른 스니펫으로, 하나는 계정 생성용, 다른 하나는 컨테이너 생성용으로 작성합니다.

결합된 작업은 라이브러리 사용자에게 현재 초점을 벗어날 수 있는 추가 작업에 대한 지식을 요구하여 어려움을 줄 수 있습니다. 먼저 작업 중인 작업과 관련된 코드를 파악한 다음, 작업에 필요한 코드만 신중하게 추출해야 합니다. 개발자는 더 이상 코드 스니펫만 복사하여 프로젝트에 붙여넣을 수 없습니다.

{% include refs.md %}
{% include_relative refs.md %}
[1]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html?highlight=literalinclude#directive-literalinclude
