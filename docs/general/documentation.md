---
title: "공통 가이드라인: 설명서"
keywords: guidelines
permalink: general_documentation.html
folder: general
sidebar: general_sidebar
---

클라이언트 라이브러리에 반드시 포함되거나 함께 제공돼야 하는 몇 가지의 문서들이 있습니다. 코드 내(Docstrings)에 포함된 완전하고 유용한 API 설명서 외에도, 훌륭한 README 및 기타 지원 설명들이 필요합니다.

* `README.md` - SDK 저장소 내 라이브러리의 루트 디렉토리에 위치합니다; 패키지 설치, 클라이언트 라이브러리 사용 정보가 포함합니다. ([예시][README-EXAMPLE])
* `API reference` - 코드 내의 Docstrings에서 생성됩니다; docs.microsoft.com에 게시됩니다. 
* `Code snippets` - 라이브러리에 대해 식별된 우수한 시나리오의 단일(원자성) 작업을 보여주기 위한 짧은 샘플 코드가 README, Docstrings, 빠른 시작에 포함됩니다. 
* `Quickstart` -  README의 내용과 유사하지만 확장된 내용의 docs.microsoft.com에서의 항목. 일반적으로 서비스 콘텐츠 개발자가 작성합니다.
* `Conceptual` - 빠른 시작, 자습서, 사용법 가이드 및 docs.microsoft.com의 기타 콘텐츠 같은 장문의 설명서; 일반적으로 서비스 콘텐츠 개발자가 작성합니다. 

{% include requirement/MUST id="general-docs-contentdev" %} 라이브러리의 아키텍처 보드 리뷰에 서비스 콘텐츠 개발자를 포함시킵니다. 함께 작업해야 할 콘텐츠 개발자를 찾으려면 해당 팀의 프로그램 관리자에게 문의하십시오.

{% include requirement/MUST id="general-docs-contributors-guide" %} [Azure SDK 기여자 가이드]를 따릅니다. (MICROSOFT 내부)

{% include requirement/MUST id="general-docs-style-guide" %} 공용 문서를 작성할 때 Microsoft 스타일 가이드에 명시된 사항들을 준수합니다. 이 부분은 README 같은 긴 형식의 문서와 코드 내의 Docstrings 모두에 적용됩니다. (MICROSOFT 내부)

* [Microsoft 작성 스타일 가이드].
* [Microsoft 클라우드 스타일 가이드].

{% include requirement/SHOULD id="general-docs-to-silence" %} 라이브러리를 명료하게 문서화합니다. Docstrings에서 API를 명확하게 설명함으로써 깃허브 이슈를 최소화하고 사용 과정에 따른 개발자들의 질문거리들을 사전에 차단합니다. 서비스 제한 및 발생할 수 있는 오류와 오류를 방지하고 복구하는 방법에 대한 정보를 포함합니다.

코드를 작성할 때, *doc it so you never hear about it again.* 답변해야 하는 클라이언트 라이브러리에 관한 질문이 적을수록, 서비스에 추가될 새로운 기능을 개발하는데 더 많은 시간을 쓸 수 있습니다.

### 코드 조각

{% include requirement/MUST id="general-docs-include-snippets" %} include example code snippets alongside your library's code within the repository. The snippets should clearly and succinctly demonstrate the operations most developers need to perform with your library. Include snippets for every common operation, and especially for those that are complex or might otherwise be difficult for new users of your library. At a bare minimum, include snippets for the champion scenarios you've identified for the library.

{% include requirement/MUST id="general-docs-build-snippets" %} build and test your example code snippets using the repository's continuous integration (CI) to ensure they remain functional.

{% include requirement/MUST id="general-docs-snippets-in-docstrings" %} include the example code snippets in your library's docstrings so they appear in its API reference. If the language and its tools support it, ingest these snippets directly into the API reference from within the docstrings. For example, use the the `literalinclude` directive in Python docstrings to instruct Sphinx to [ingest the snippets automatically][1].

{% include requirement/MUSTNOT id="general-docs-operation-combinations" %} combine more than one operation in a code snippet unless it's required for demonstrating the type or member, or it's *in addition to* existing snippets that demonstrate atomic operations. For example, a Cosmos DB code snippet should not include both account and container creation operations--create two different snippets, one for account creation, and one for container creation.

Combined operations cause unnecessary friction for a library consumer by requiring knowledge of additional operations which might be outside their current focus. It requires them to first understand the tangential code surrounding the operation they're working on, then carefully extract just the code they need for their task. The developer can no longer simply copy and paste the code snippet into their project.

{% include refs.md %}
[1]: https://www.sphinx-doc.org/en/master/usage/restructuredtext/directives.html?highlight=literalinclude#directive-literalinclude
