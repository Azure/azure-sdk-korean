---
title: "공통 가이드라인: 설명서"
keywords: guidelines
permalink: general_documentation.html
folder: general
sidebar: general_sidebar
---햣

클라이언트 라이브러리에 반드시 포함되거나 함께 제공돼야 하는 몇 가지의 문서들이 있습니다. 코드 내(Docstrings)에 포함된 완전하고 유용한 API 설명서 외에도, 훌륭한 README 및 기타 지원 설명들이 필요합니다.

* `README.md` - SDK 저장소 내 라이브러리의 루트 디렉토리에 위치합니다; 패키지 설치, 클라이언트 라이브러리 사용 정보가 포함합니다. ([예시][README-EXAMPLE])
* `API 참조` - 코드 내의 Docstrings에서 생성됩니다; docs.microsoft.com에 게시됩니다. 
* `코드 조각` - 라이브러리에 대해 식별된 대표 시나리오의 단일(원자성) 작업을 보여주기 위한 짧은 샘플 코드가 README, Docstrings, 빠른 시작에 포함됩니다. 
* `빠른 시작` -  README의 내용과 유사하지만 확장된 내용의 docs.microsoft.com에서의 항목. 일반적으로 서비스 콘텐츠 개발자가 작성합니다.
* `개념` - 빠른 시작, 자습서, 사용법 가이드 및 docs.microsoft.com의 기타 콘텐츠 같은 장문의 설명서; 일반적으로 서비스 콘텐츠 개발자가 작성합니다. 

{% include requirement/MUST id="general-docs-contentdev" %} 라이브러리의 아키텍처 보드 리뷰에 서비스 콘텐츠 개발자를 포함시킵니다. 함께 작업해야 할 콘텐츠 개발자를 찾으려면 해당 팀의 프로그램 관리자에게 문의하십시오.

{% include requirement/MUST id="general-docs-contributors-guide" %} [Azure SDK 기여자 가이드]를 따릅니다. (MICROSOFT 내부)

{% include requirement/MUST id="general-docs-style-guide" %} 공용 문서를 작성할 때 Microsoft 스타일 가이드에 명시된 사항들을 준수합니다. 이 부분은 README 같은 긴 형식의 문서와 코드 내의 Docstrings 모두에 적용됩니다. (MICROSOFT 내부)

* [Microsoft 작성 스타일 가이드].
* [Microsoft 클라우드 스타일 가이드].

{% include requirement/SHOULD id="general-docs-to-silence" %} 라이브러리를 명료하게 문서화합니다. Docstrings에서 API를 명확하게 설명함으로써 깃허브 이슈를 최소화하고 사용 과정에 따른 개발자들의 질문거리들을 사전에 차단합니다. 서비스 제한 및 발생할 수 있는 오류와 오류를 방지하고 복구하는 방법에 대한 정보를 포함합니다.

코드를 작성할 때, *doc it so you never hear about it again.* 답변해야 하는 클라이언트 라이브러리에 관한 질문이 적을수록, 서비스에 추가될 새로운 기능을 개발하는데 더 많은 시간을 쓸 수 있습니다.

### 코드 조각

{% include requirement/MUST id="general-docs-include-snippets" %} 저장소 내에 라이브러리 코드와 함께 예제 코드 조각들을 포함하십시오.  코드 조각들은 대부분의 개발자가 라이브러리로 수행해야 하는 작업들을 명확하고 간결하게 설명해야 합니다. 코드 조각들은 모든 공통 작업, 특히 라이브러리를 처음 사용하는 이용자에게 복잡하거나 어려울 수 있는 작업들을 포함해야 합니다. 최소한, 라이브러리에 대해 식별한 대표 시나리오에 대한 코드 조각을 포함합니다.

{% include requirement/MUST id="general-docs-build-snippets" %} 저장소의 연속 통합(CI)을 이용해 코드 조각들을 빌드하고 테스트하여 기능이 유지되고 있는지 확인합니다.

{% include requirement/MUST id="general-docs-snippets-in-docstrings" %} 예제 코드 조각들을 라이브러리 Docstrings에 포함시켜 API 참조에 표시합니다
언어나 해당 도구가 이를 지원하는 경우, 이 코드 조각들을 Docstrings에서 API 참조로 직접 수집합니다.
예를 들어, Python Docstrings의 'literalinclude' 지시어를 사용하여 Sphinx가 [코드 조각을 자동으로 수집][1]하도록 지시합니다.

{% include requirement/MUSTNOT id="general-docs-operation-combinations" %} 형식이나 멤버를 보여주는데 필요하지 않거나 원자성 작업을 보여주는 기존 코드 조각에 *추가*되지 않는 한 코드 조각에서 하나 보다 많은 작업을 결합합니다. 예를 들어, 하나의 Cosmos DB 코드 조각에는 계정과 컨테이너를 생성하는 작업이 동시 포함되어서는 안 됩니다--코드 조각을 계정 생성용으로 하나, 컨테이너 생성용으로 하나 총 두 개를 만들어야 합니다.

결합된 작업들은 라이브러리 소비자에게 현재 초점에서 벗어날 수 있는 추가 작업에 대한 지식을 요구하여 불필요한 마찰을 야기합니다. 결합된 작업들은 라이브러리 소비자들이 먼저 작업 중인 작업의 주변 접선 코드를 파악한 다음, 다음 작업에 필요한 코드만 신중하게 추출할 것을 요구합니다. 따라서 개발자는 더는 코드 조각을 복사하여 프로젝트에 붙여넣기 할 수 없습니다.

{% include refs.md %}
[1]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html?highlight=literalinclude#directive-literalinclude
