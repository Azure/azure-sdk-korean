---
title: "정책: 리뷰 프로세스"
permalink: policies_reviewprocess.html
folder: policies
sidebar: general_sidebar
---

## 소개

Azure Developer Platform (ADP) 아키텍처 리뷰 위원회는 Java, Python, TS/JS, C#, C, C++, Go, Android, iOS에 전문화된 언어 아키텍트들의 위원회입니다.

**아키텍처 위원회는 오직 Track 2 라이브러리만을 검토합니다.** 정의상, Track 2 라이브러리는 우리의 [Track 2 라이브러리 디자인 가이드라인과 특정 언어 가이드라인을 준수하는 라이브러리](https://azure.github.io/azure-sdk/general_introduction.html)를 의미합니다.

우리는 모든 Azure 클라이언트 라이브러리가 Microsoft에서 생산하는 다른 API (예: .NET API)에 대해 실시하는 것과 유사한 엄격한 SDK API 리뷰를 통과하길 기대합니다. 새로운 라이브러리의 상세한 리뷰 외에도, SDK API에 대한 **모든 변경사항**은 릴리즈 전에 특정 언어의 아키텍트가 승인해야 합니다.

여기서 설명하는 라이브러리 리뷰 프로세스는 현재 **내부적인** 프로세스입니다. 이 문서는 라이브러리를 검토하려는 Azure 서비스 팀들에게 프로세스를 명확하게 하는 데 의도되어 있습니다.

## SDK API 리뷰 프로세스 로드맵

아키텍처 위원회와의 회의는 일반적으로 최소 세 번 이상 진행됩니다:

1. 소개 세션
2. SDK API 리뷰
3. SDK API 승인

라이브러리 서피스와 기타 요인에 따라, 하나 이상의 API 리뷰가 필요할 수 있습니다.

라이브러리 소유자가 아키텍처 위원회와 충분히 일찍 연결하여 수정 및 (때때로 상당한) API 재디자인 시간을 확보하는 것이 중요합니다. 클라이언트 라이브러리 작업의 성격과 범위에 따라, 아키텍처 위원회와의 교류시 따라야 할 일련의 과정은 다음 두 가지 경로 중 하나를 따를 것입니다:

1. **새로운 라이브러리, 대규모 피처 작업, 및/또는 파이프라인 변경**

    이러한 변경사항은 아키텍처 위원회 회의에서 최소 세 번 이상 논의되어야 합니다. 아래의 "리뷰 회의 유형 및 준비물" 섹션을 참조하십시오.

2. **소규모, 목표 지향적인 변경과 버그 수정**

    아래의 "소규모, 목표 지향적인 변경과 버그 수정 승인 받기" 섹션을 참조하십시오.

## 리뷰 회의 유형 및 준비물

소개(Introduction) 회의와 추가(Follow-Up) 회의라는 두 가지 유형의 회의가 예정될 수 있습니다: 

내부 팀의 경우, [일정 도구](https://aka.ms/azsdk/schedulesdkreview)를 사용해 리뷰 세션을 예약하십시오. 검토 위원회에 새로운 서비스를 소개하는 경우("Introduction") 또는 이전의 소개 내용을 따라가거나 SDK API 리뷰 또는 SDK API 승인이 필요한 경우("Follow-Up")를 선택하십시오. 각 회의 유형에 대한 요구 사항은 아래에 자세히 설명되어 있습니다.
 

### 1. 소개 세션

이 세션은 순수하게 정보 제공/교육을 위한 것으로, Azure SDK 아키텍처 위원회가 서비스와 라이브러리/새로운 기능의 동향을 파악할 수 있도록 합니다. 이를 통해 초기 피드백을 받을 수 있으며, 서비스 디자인에 영향을 줄 수 있습니다. API 네임스페이스, 함수 이름, 타입 등의 고수준 주제가 이 첫 번째 논의에서 제안될 것입니다.

**준비물:**

|제목| 중요도 | 간단한 설명 | 예시 및 지원 문서 |
|----------------------|----------------------|----------------------|----------------------|
| 주요 시나리오 | 필수 | 서비스가 활용되는 주요 시나리오들| 주요 시나리오를 식별하는 방법에 대한 가이드라인 - [링크](https://github.com/Azure/azure-sdk-pr/blob/24384df0202021ab86ee37fcb14e9554182cd014/training/azure-sdk-apis/principles/approachable/README.md#hero-scenarios)<br><br> [예시](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/textanalytics/Azure.AI.TextAnalytics/samples/Sample1_DetectLanguage.md) |
| 핵심 개념 | 권장 | 명사 및 동사의 용어집 | [예시](https://github.com/Azure/azure-sdk-pr/blob/main/onboarding/Core_Concepts.pdf) |
| APIView | 권장 | 생성된 SDK용 APIView | [예시](https://apiview.dev/Assemblies/Review/8b7f5312697a458ab9e65c2fd9cdc2dd)  |
| REST API 스펙 | 권장 | [azure/azure-rest-api-specs-pr (직원 액세스)](https://github.com/azure/azure-rest-api-specs-pr) 또는 [azure/azure-rest-api-specs](https://github.com/azure/azure-rest-api-specs) 저장소의 검토된 REST API 사양 정의에 대한 링크 | [예시](https://github.com/Azure/azure-rest-api-specs/blob/main/specification/attestation/data-plane/Microsoft.Attestation/stable/2020-10-01/attestation.json) |

### 2. SDK API 리뷰
API 리뷰 중에는 샘플 코드와 상세 API 목록을 살펴봅니다. 이러한 목록의 예시는 [여기](https://github.com/Azure/azure-sdk/blob/main/docs/dotnet/APIListingExample.md)에서 볼 수 있습니다.

상황과 서비스에 따라 하나 이상의 SDK API 검토가 필요할 수 있습니다 (예를 들어 API 버전 간에 중요한 변경 사항이 있는 경우). 그런 경우에는 팀이 다음 검토를 준비할 준비가 되었을 때 다른 회의를 예약하십시오.  

**모든 SDK API 언어는 stable 릴리스 이전에 승인을 받아야 합니다.** 이전 검토 이후 SDK API에 제한적인 변경이 있었을 경우, *아키텍트는 전체 회의가 필요하지 않은 경우 이메일로 승인*할 수 있습니다.

**준비물:**  

|제목 | 중요도 |간단한 설명 | 예시 및 지원 문서 |
|----------------------|----------------------|----------------------|----------------------|
| APIView | 필수 | 각 SDK에 대한 APIView. 아키텍트가 회의 전에 검토할 시간이 확보되도록 검토 예정일로부터 **적어도 5 영업일 전**에 제공해야 합니다. | [예시](https://apiview.dev/Assemblies/Review/8b7f5312697a458ab9e65c2fd9cdc2dd)  |
| 주요 시나리오 | 권장 | 서비스가 활용되는 최상의 시나리오들. 각 시나리오에 해당하는 코드 샘플이 포함되어 있어야 합니다. 샘플은 APIView에 추가될 수 있습니다. | 주요 시나리오를 식별하는 방법에 대한 가이드라인 - [링크](https://github.com/Azure/azure-sdk-pr/blob/24384df0202021ab86ee37fcb14e9554182cd014/training/azure-sdk-apis/principles/approachable/README.md#hero-scenarios)<br><br> [예시](https://github.com/Azure/azure-sdk-for-net/tree/main/sdk/appconfiguration/Azure.Data.AppConfiguration#examples) |
| REST API 스펙 | 권장 | [azure/azure-rest-api-specs-pr (직원 액세스)](https://github.com/azure/azure-rest-api-specs-pr) 또는 [azure/azure-rest-api-specs](https://github.com/azure/azure-rest-api-specs) 저장소의 검토된 REST API 사양 정의에 대한 링크 | [예시](https://github.com/Azure/azure-rest-api-specs/blob/main/specification/attestation/data-plane/Microsoft.Attestation/stable/2020-10-01/attestation.json) |
| 핵심 개념 | 권장 | 명사 및 동사의 용어집 | [예시](https://github.com/Azure/azure-sdk-pr/blob/main/onboarding/Core_Concepts.pdf) |

**APIView 참고**: 변경 사항에 대한 풀 리퀘스트가 있는 경우, 해당 풀 리퀘스트에서 생성된 자동으로 생성된 [APIView](http://apiview.dev/) 리뷰를 사용하여 아키텍처 위원회와 API를 논의할 수 있습니다. 풀 리퀘스트가 없고 API의 프로토타입이 있는 경우, [여기](https://github.com/Azure/azure-sdk-tools/blob/main/src/dotnet/APIView/APIViewWeb/README.md#how-to-create-an-api-review-manually)에서 언급된대로 [APIView](http://apiview.dev/) 도구에서 API 목록을 생성할 수 있습니다

## 리뷰 중 발생하는 일

### 누가 참석해야 합니까?

SDK API와 서비스에 익숙한 사람들 (보통 엔지니어링 및/또는 PM 리드)이 참석해야 합니다.

### 소개 세션
일반적인 의제는 서비스 팀이 약 30분 동안 서비스를 소개하는 것으로 시작합니다. 그 다음에 주요 시나리오가 제시되고, 각 시나리오에 이어서 논의가 이루어집니다. 이는 대부분의 시간을 차지합니다. 만약 Rest API 스펙이 사용 가능하다면, 시간이 허락할 경우 이에 대해 이야기합니다. 마지막으로 다음 리뷰 회의 전에 수행할 액션 아이템에 대한 짧은 요약이 있을 것입니다.

### SDK API 리뷰

언어 아키텍트들은 리뷰 시점에 제공된 SDK API 목록을 검토하였을 것입니다. 그들은 제공된 SDK API와 샘플에 대한 논의를 바로 시작할 것입니다. 회의는 취할 액션 아이템에 대한 짧은 요약으로 끝날 것입니다.

### API 승인

일반적으로, SDK API에 대한 일부 미결정/논란의 여지가 있는 질문들이 있을 것입니다. 이는 SDK API를 검토한 언어 아키텍트들 또는 발표하는 팀에서 비롯될 수 있습니다. 이 리뷰의 목표는 SDK API를 승인하는 것이기 때문에, 위원회는 보통 이러한 질문에 대한 논의를 바로 시작합니다. 리뷰는 SDK API에 대한 최종 승인이나 SDK API를 승인받기 위해 따라야 하는 항목들로 끝날 것입니다.

### 필요한 합의

SDK API가 승인되기 위해서는 아키텍처 위원회 회의에서 다음 조건들이 충족되어야 합니다:

* 모든 Tier-1 언어 (Java, Python, TS/JS, C#), 그리고 고려 중인 모든 언어의 대표들이 참석해야 합니다.
* 최소한 세 명의 다른 언어 그룹의 아키텍트들이 참석해야 합니다.

만약 언어 아키텍트가 회의에 참석하지 **않는다면**, 부 아키텍트가 해당 언어의 대표가 될 수 있습니다. 언어 대표 목록은 Azure SDK 그룹의 LT에 의해서만 변경될 수 있습니다.

## 리뷰 이후에는 어떻게 되나요?

소개 및 API 리뷰 세션에서는 보통 다음 회의 전에 처리해야 할 작업 목록이 있습니다. 이 작업 항목을 잊지 말고 따르십시오. 때때로, 이러한 작업 항목 중 하나는 아키텍트들이 제안한 변경 사항을 반영한 후 다른 API 리뷰를 예약하는 것일 수 있습니다.

## API 변경 사항 미리보기

SDK API 변경 사항은 일반적으로 GA로 출시되기 전에 일정 기간 베타 버전으로 출시됩니다. 이렇게 하면 고객들이 피드백을 제공할 수 있는 시간이 주어지며, 이 피드백을 바탕으로 GA 이전에 SDK API에 조정이 이루어질 수 있습니다. 바로 GA로 가는 SDK API 변경 사항은 이러한 피드백의 혜택을 받지 못하므로 고객이 사용하기 어렵고 우리가 지원하기 어려울 수 있습니다. 대부분의 경우, 전체 또는 축약된 리뷰 프로세스를 거친 API 변경 사항은 GA 전에 베타로 출시되어야 합니다.

## 작은, 목표 지향적인 변경 사항 및 버그 수정에 대한 승인 받기

SDK API를 수정하는 작은 또는 목표 지향적인 변경 사항 및 버그 수정의 경우, 각 언어의 아키텍트가 결합된/중앙 리뷰 없이 검토하고 승인할 수 있습니다. 이 검토를 가능한 한 빠르게 하는 것을 강력히 권장합니다. 이것은 [APIView](http://apiview.dev/) 차이에 대한 링크를 포함하는 GitHub 이슈를 열어 수행해야 합니다. 모든 아키텍트를 리뷰어로 포함시키십시오. 경우에 따라 API의 작은 변경 사항을 효율성을 위해 배치하는 것이 이치에 맞습니다. 언어 아키텍트가 더 깊은 토론이 필요하다고 판단하면, 해당 아키텍트와의 회의를 예약해야 합니다. 만약 이것이 크로스-언어 토론이라면, 위원회 회의를 예약해야 합니다.

SDK API에 대한 **모든** 변경 사항은 릴리스 전에 언어 아키텍트가 승인해야 함을 기억하십시오.

### 주요 시나리오

주요 시나리오는 클라이언트 라이브러리의 소비자가 일반적으로 수행할 것으로 예상되는 사용 사례입니다. 주요 시나리오는 일반적인 경우에 개발자 경험이 뛰어나도록 보장하는 데 사용됩니다. 주요 시나리오에 대한 전체 코드 샘플 (예를 들어, 에러 처리 등)을 보여줘야 합니다. 또한 라이브러리에서 **인증 워크플로우**가 어떻게 보일지도 보여주세요.

좋은 시나리오는 기술에 구애받지 않는 것입니다 (즉, 고객이 여러 가지 방법으로 동일한 작업을 수행할 수 있음), 그리고 이는 사용자의 20% 이상이 사용할 것으로 예상됩니다.

나쁜 시나리오의 예:
* 클라이언트 생성 (시나리오의 일부이며, 진정한 주요 시나리오에서 충분히 볼 수 있음)
* 이벤트 배치 보내기 (다시, 시나리오의 일부)
* 페이지 블롭 생성 (사용자 기반의 충분한 부분에서 사용되지 않음)

### 퀵스타트 샘플

일반적인 방법을 보여주는 샘플:

* 새로운 리소스 생성
* 리소스 읽기
* 리소스 수정
* 리소스 삭제
* 에러 처리
* 경쟁 조건/동시성 문제 처리


{% include refs.md %}