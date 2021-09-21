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

Spring integration modules must be versioned in a way that enables the following goals:

1. Each Spring integration module must be able to release at different release cadences.
2. Each Spring integration module must have full semantic versioning for major, minor, and patch versions, in all releases. Versioning must not be tied to the Spring dependency version as in earlier iterations of the Azure Spring integration modules.
3. Allow developers to easily choose Spring integration modules which work together.

{% include requirement/MUST id="java-spring-supported-versions" %} ensure that all releases of a Spring integration module support all active versions (as of the time of release) of the corresponding Spring API.

{% include requirement/MUST id="java-spring-deps" %} add latest release version of Spring API dependencies in the Spring integration module POM files, it is the users responsibility to override the Spring API version via Spring BOM.

{% include requirement/MUST id="java-spring-classifiers" %} add Maven classifiers to releases if a Spring integration module cannot support all active versions of the corresponding Spring API. For example, if a Spring integration module needs to support Spring Boot 2.2.x and 2.3.x, but cannot due to technical contraints, two versions of the Spring integration module must be released, with classifiers `springboot_2_2` and `springboot_2_3`.

{% include requirement/MUST id="java-spring-bom" %} provide a Spring integration modules BOM for users. This BOM must contain versions of all Spring integration modules that are known to work together (and have a single set of dependency versions). It must also include appropriate references to Azure Java SDK.

{% include requirement/MUST id="java-spring-bom-docs" %} encourage users to use the Spring integration modules BOM for their chosen version of Spring rather than specific versions of each Spring integration module, such that they need not worry about Maven classifiers and other versioning issues.

## 종속성

{% include requirement/MUSTNOT id="java-spring-dependency-approval" %} introduce dependencies on libraries, or change dependency versions, without discussion with the Java architect. Each dependency must receive explicit approval and be added to the dependency allow list before it may be used.

{% include requirement/MUSTNOT id="java-spring-dependency-conflicts" %} introduce dependencies on library versions that conflict with the transitive dependencies of Spring libraries.

{% include requirement/MUST id="java-spring-com-azure-deps" %} make use of `com.azure` client libraries only - do not mix older `com.microsoft.azure` client libraries into the dependency hierarchy.

{% include requirement/MUST id="java-spring-dependency-minimal" %} keep dependencies to the minimal required set.

## 로깅

{% include requirement/MUSTNOT id="java-spring-logging" %} use the `ClientLogger` logging APIs.

## 추적

{% include requirement/MUST id="java-spring-tracing" %} ensure that all Azure Spring libraries fully integrate with the tracing capabilities available in the Azure Java client libraries.

{% include requirement/MUST id="java-spring-tracing-sleuth" %} ensure that all Azure Spring libraries work appropriately with Spring Sleuth, and that tracing information is appropriately exported.

## 성능

{% include requirement/MUST id="java-spring-performance-baseline" %} ensure, through appropriate benchmarks (developed in conjuction with the Java SDK team) that performance of all Spring libraries is at an equivalent level to the same operation being performed directly through the Java client library.
