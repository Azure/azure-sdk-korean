---
title: "스프링 가이드라인"
keywords: guidelines java spring
permalink: java_spring.html
folder: java
sidebar: general_sidebar
---

스프링 생태계에 최상의 환경을 제공하는 것은 매우 중요합니다. 아래 가이드라인은  [자바 표준 디자인 가이드라인](https://azure.github.io/azure-sdk/java_introduction.html) 의 지침을 적절히 재정의하여 확장한 것입니다. 

## 네임스페이스

{% include requirement/MUST id="java-spring-namespaces" %} 모든 자바 패키지들은  `com.azure.spring.<group>.<service>[.<feature>]`과 같은 형식으로 명명되어야 합니다.

{% include requirement/MUST id="java-spring-same-group" %} 그룹, 서비스, 기능은 자바 기본 클라이언트 라이브러리에 사용되는 것과 동일하게 명명합니다.  

{% include requirement/MUST id="java-spring-implementation" %} 모든 논-퍼블릭 API는 루트 네임스페이스에 속한 `implementation` 패키지에 위치시킵니다.

### Maven

{% include requirement/MUST id="java-spring-maven-groupid" %} 그룹 ID는 `com.azure.spring`를 사용합니다.

{% include requirement/MUST id="java-spring-maven-artifactid" %} `artifactId` 는  `azure-spring-boot-starter-<group>-<service>[-<feature>]`의 형태로 지정합니다. `azure-spring-boot-starter-storage-blob` 또는 `azure-spring-boot-starter-security-keyvault-secrets`와 같은 예시가 있습니다. 
스프링 데이터 추상화의 경우, `artifactId`는 `azure-spring-data-<group>-<service>[-<feature>]`와 같은 형식이어야 합니다.
스프링 클라우드 스타터의 경우, `artifactId`는 `azure-spring-cloud-starter-<group>-<service>[-<feature>]`와 같은 형식이어야 합니다.

{% include requirement/MUST id="java-spring-azure-sdk-bom" %} Azure 스프링 라이브러리를 사용하는 사용자가 버전을 선택할 필요 없이 다른 Azure 자바 클라이언트 라이브러리에서 추가적인 종속성들을 가져올 수 있도록 `dependencyManagement` 종속성을 Azure 자바 SDK BOM에 포함합니다.

## 버전 관리

스프링 인티그레이션 모듈 버전 관리는 아래 목표들을 실현하도록 이뤄져야 합니다.

1. 각각의 스프링 인티그레이션 모듈은 다른 릴리스 케이던스에서도 릴리스가 가능해야 합니다.
2. 각각의 스프링 인티그레이션 모듈은 모든 릴리스에서 주 버전, 부 버전, 패치 버전이 모두 포함된 유의적 버전을 가져야 합니다. Azure 스프링 인티그레이션 모듈에서 그랬던 것처럼, 버전 관리가 스프링 종속성 버전에 연결되어 이뤄지면 안 됩니다.
3. 개발자들이 사용할 스프링 인티그레이션 모듈을 쉽게 선택할 수 있도록 해야 합니다.

{% include requirement/MUST id="java-spring-supported-versions" %} 모든 스프링 인티그레이션 모듈은, 릴리스 시점을 기준으로, 상응하는 모든 스프링 API 활성 버전들을 지원해야 합니다.

{% include requirement/MUST id="java-spring-deps" %} 최신 릴리스 버전의 스프링 API 종속성을 스프링 인티그레이션 모듈 POM 파일에 추가해야 합니다. 사용자들은 스스로 스프링 BOM을 통해 스프링 API 버전을 재정의해야 합니다.

{% include requirement/MUST id="java-spring-classifiers" %} 스프링 인티그레이션 모듈이 모든 스프링 API 활성 버전들을 지원할 수 없는 경우 릴리스에 메이븐 분류자를 추가합니다. 예를 들어 스프링 인티그레이션 모듈이 Spring Boot 2.2x와 2.3.x을 지원해야 하지만 기술적인 제약으로 지원하지 못하는 경우
`springboot_2_2`와 `springboot_2_3` 분류자가 있는 두 가지 버전의 스프링 인티그레이션 모듈이 릴리스 돼야 합니다.

{% include requirement/MUST id="java-spring-bom" %} 사용자들을 위해 스프링 인티그레이션 모듈 BOM을 제공해야 합니다. 이 BOM은 같이 동작하는 것으로 알려진 모든 스프링 인티그레이션 모듈들의 버전을 반드시 포함해야 합니다(종속성 버전들에 대한 단일 집합을 포함해야 합니다). 또한 Azure 자바 SDK에 대한 적절한 참조 역시 포함돼야 합니다.

{% include requirement/MUST id="java-spring-bom-docs" %} 사용자들이 메이븐 분류자나 기타 버전 관리 문제들을 신경쓰지 않도록 그들이 선택한 스프링 버전에 대해, 특정 버전의 스프링 인트그레이션 모듈을 사용하기 보다는, 스프링 인티그레이션 모듈 BOM을 사용할 것을 권장해야 합니다.

## 종속성

{% include requirement/MUSTNOT id="java-spring-dependency-approval" %} introduce dependencies on libraries, or change dependency versions, without discussion with the Java architect. Each dependency must receive explicit approval and be added to the dependency allow list before it may be used.

{% include requirement/MUSTNOT id="java-spring-dependency-conflicts" %} introduce dependencies on library versions that conflict with the transitive dependencies of Spring libraries.

{% include requirement/MUST id="java-spring-com-azure-deps" %} make use of `com.azure` client libraries only - do not mix older `com.microsoft.azure` client libraries into the dependency hierarchy.

{% include requirement/MUST id="java-spring-dependency-minimal" %} keep dependencies to the minimal required set.

## 로깅

{% include requirement/MUSTNOT id="java-spring-logging" %} `ClientLogger` 로깅 API를 사용해야 합니다.

## 추적

{% include requirement/MUST id="java-spring-tracing" %} ensure that all Azure Spring libraries fully integrate with the tracing capabilities available in the Azure Java client libraries.

{% include requirement/MUST id="java-spring-tracing-sleuth" %} ensure that all Azure Spring libraries work appropriately with Spring Sleuth, and that tracing information is appropriately exported.

## 성능

{% include requirement/MUST id="java-spring-performance-baseline" %} ensure, through appropriate benchmarks (developed in conjuction with the Java SDK team) that performance of all Spring libraries is at an equivalent level to the same operation being performed directly through the Java client library.
