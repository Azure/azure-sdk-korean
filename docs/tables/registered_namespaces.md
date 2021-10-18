---
title: "등록된 네임스페이스 목록"
permalink: registered_namespaces.html
folder: general
sidebar: general_sidebar
---

다음은 등록된 네임스페이스 목록입니다.

| 네임스페이스                     | 서비스 소유자                      |
| :---------------------------- | :----------------------------- |
| `azure.ai.formrecognizer`     | [Form Recognizer]              |
| `azure.ai.textanalytics`      | [Text Analytics]               |
| `azure.data.appconfiguration` | [App Configuration]            |
| `azure.cosmos`                | [Azure Cosmos DB]              |
| `azure.messaging.eventhubs`   | [Event Hubs]                   |
| `azure.messaging.servicebus`  | [Service Bus]                  |
| `azure.search.documents`      | [Azure Search]                 |
| `azure.security.keyvault`     | [Key Vault]                    |
| `azure.storage.blobs`         | [Azure Storage]                |
| `azure.storage.files.shares`  | [Azure Storage]                |
| `azure.storage.files.datalake`| [Azure Storage]                |
| `azure.storage.queues`        | [Azure Storage]                |

우리는 표준 형식으로 네임스페이스를 나타냅니다(각 요소는 모두 소문자이며 `azure` 식별자로 시작합니다). 사용하기 전에 이 표준 양식을 언어별 형식으로 변경해야 합니다. 예를 들어, `azure.security.keyvault`는 다음과 같이 표시됩니다.

* 자바: `com.azure.security.keyvault`
* .NET: `Azure.Security.KeyVault`
* C++: `Azure::Security::KeyVault`

새로운 네임스페이스를 등록하려면, [Architecture Board]에 문의하십시오.

{% include refs.md %}

<!-- Service Links -->
[App Configuration]: https://azure.microsoft.com/services/app-configuration/
[Azure Cosmos DB]: https://azure.microsoft.com/services/cosmos-db/
[Azure Search]: https://azure.microsoft.com/services/search/
[Azure Storage]: https://azure.microsoft.com/services/storage
[Event Hubs]: https://azure.microsoft.com/services/event-hubs/
[Form Recognizer]: https://azure.microsoft.com/services/cognitive-services/form-recognizer/
[Key Vault]: https://azure.microsoft.com/services/key-vault/
[Service Bus]: https://azure.microsoft.com/services/service-bus/
[Text Analytics]: http://azure.microsoft.com/services/cognitive-services/text-analytics/
