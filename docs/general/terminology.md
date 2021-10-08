---
title: "공통 가이드라인: 용어"
keywords: guidelines
permalink: general_terminology.html
folder: general
sidebar: general_sidebar
---

가이드라인을 통해 다음의 용어들을 이해할 수 있습니다:

아키텍처 보드
: Azure 개발자 경험 Developer Experience [Architecture Board] is comprised of language experts who advise and review client libraries used for accessing Azure services.

애저 SDK
: 애저 서비스에 액세스할 때 사용되는 단일 대상 언어에 대한 _client libraries_ 모음입니다.
The collection of _client libraries_ for a single target language, used for accessing Azure services.

애저 코어
: 많은 클라이언트 라이브러리의 종속성입니다. 애저 코어 라이브러리는 애저 SDK에 전체적으로 적합한 파이프라인, 일반적인 자격 증명 형식, 그리고 다른 유형에 대한 액세스를 제공합니다.
A dependency of many client libraries.  The Azure Core library provides access to the HTTP pipeline, common credential types, and other types that are appropriate to the Azure SDK as a whole.

클라이언트 라이브러리
: 
A library (and associated tools, documentation, and samples) that _consumers_ use to ease working with an Azure service.  There is generally a client library per Azure service and per target language.  Sometimes a single client library will contain the ability to connect to multiple services.

소비자
: 다양한 개발자들을 구분하기 위해 우리는 어떤 개발자가 클라이언트 라이브러리를 
Where appropriate to disambiguate between the various types of developers, we use the term _consumer_ to indicate the developer who is using a client library in an app to connect to an Azure service.

Docstrings
: The comments embedded within the code that describe the API surface being implemented.  The _docstrings_ are extracted and post-processed during the build to generate API reference documentation.

라이브러리 개발자
: Where appropriate to disambiguate between the various types of developers, we use the term _library developer_ to indicate the developer who is writing a client library.

패키지
: A client library after it has been packaged for distribution to consumers.  Packages are generally installed using a package manager from a package repository.

패키지 리포지토리
: Each client library is published separately to the appropriate language-specific package repository.  For example, we distribute JavaScript libraries to [npmjs.org](https://npmjs.org) (also known as the NPM Registry), and Python libraries to [PyPI](https://pypi.org/).  These releases are performed exclusively by the Azure SDK engineering systems team.  Consumers install packages using a package manager.  For example, a JavaScript consumer might use yarn, npm, or similar, whereas a Python consumer will use `pip` to install packages into their project.

프로그레시브 컨셉 공개
: The first interaction with the client library should not rely on advanced service concepts.  As the consumer of the library becomes more adept, we expose the concepts necessary at the point at which the consumer needs those concepts for implementation. [Progressive Disclosure] was first discussed by the Nielson Norman Group as an approach to designing better user interfaces.

## 요구사항

이 문서의 요구사항들은 라벨로 표시되어 있고 상대적 중요성을 나타내기 위해 색상으로 구분되어 있습니다. 중요도가 높은 순으로 작성되었습니다:
Each requirement in this document is labelled and color-coded to show the relative importance.  In order from highest importance to lowest importance:

{% include requirement/MUST %} 클라이언트 라이브러리에 요구사항을 사용해주세요. 만약 예외가 필요하다면 구현전에 [Architecture Board]와 상의해주세요.

{% include requirement/MUST %} adopt this requirement for the client library.  If you feel you need an exception, engage with the [Architecture Board] prior to implementation.

{% include requirement/MUSTNOT %} 클라이언트 라이브러리에 요구사항을 사용하지 말아주세요. 만약 예외가 필요하다면 구현전에 [Architecture Board]와 상의해주세요.

{% include requirement/MUSTNOT %} adopt this requirement for the client library.  If you feel you need an exception, engage with the [Architecture Board] prior to implementation.

{% include requirement/SHOULD %} 클라이언트 라이브러리에 요구사항을 강력하게 고려해야합니다. 만약 이 권장사항을 따르지 않을 경우, **반드시** [Architecture Board] 디자인 리뷰를 할 때 차이를 공개해야합니다.
{% include requirement/SHOULD %} strongly consider this requirement for the client library.  If not following this advice, you **MUST** disclose the variance during the [Architecture Board] design review. 

{% include requirement/SHOULDNOT %} 클라이언트 라이브러리에 요구사항을 강력하게 고려하지 말아야합니다. 만약 이 권장사항을 따르지 않을 경우, **반드시** [Architecture Board] 디자인 리뷰를 할 때 차이를 공개해야합니다.
{% include requirement/SHOULDNOT %} strongly consider this requirement for the client library.  If not following this advice, you **MUST** disclose the variance during the [Architecture Board] design review.

{% include requirement/MAY %} 만약 여러분의 상황에 적절할 경우 이 권장사항을 고려해야합니다. 아키텍처 보드에 통보할 필요가 없습니다. 
{% include requirement/MAY %} consider this advice if appropriate to your situation.  No notification to the architecture board is required.

{% include refs.md %}

[Progressive Disclosure]: https://www.nngroup.com/articles/progressive-disclosure/
