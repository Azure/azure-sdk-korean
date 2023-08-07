# Azure SDK Documentation for Korean

(For English, please visit https://github.com/Azure/azure-sdk repository)

Azure SDK는 개발자가 다양한 Azure 서비스를 원하는 프로그래밍 언어로 활용할 수 있는 플랫폼을 제공하고 있습니다. 각 언어에 대한 소스 저장소에 해당 클라이언트 라이브러리에 대한 소스 코드가 있습니다. 이 저장소는 해당 프로그래밍 언어 별 저장소를 가리키며, 한글로 보다 자세한 설명을 공유하고자 합니다. 구체적인 이슈는 각 SDK 저장소를 통해 확인하실 수 있습니다.



| 언어         | 디자인 가이드라인 (영문)                          | 패키지                 | 저장소                            | 문서 (영문)                        |
|:------------|:-------------------------------------------:|:--------------------:|:--------------------------------:|:--------------------------------:|
| General     |[General Design Guidelines]                  |                      |[azure-sdk Repository]            | [Official Azure Documentation]   |
| Android     |[Design Guidelines for Android] (Draft)      |[Android Packages]    |[azure-sdk-for-android Repository]| Coming Soon                      |
| C# /.NET    |[Design Guidelines for .NET]                 |[.NET Packages]       |[azure-sdk-for-net Repository]    | [.NET Documentation]             |
| Go          |[Design Guidelines for Go] (Draft)           |[Go Packages]         |[azure-sdk-for-go Repository]     | [Go Documentation]               |
| C           |[Design Guidelines for C99] (Draft)          |[C Packages]          |[azure-sdk-for-c Repository]      | [C Documentation]                |
| C++         |[Design Guidelines for C++] (Draft)          |[C++ Packages]        |[azure-sdk-for-cpp Repository]    | [C++ Documentation]              |
| iOS         |[Design Guidelines for iOS] (Draft)          |[iOS Packages]        |[azure-sdk-for-ios Repository]    | Coming Soon                      |
| Java        |[Design Guidelines for Java]                 |[Java Packages]       |[azure-sdk-for-java Repository]   | [Java Documentation]             |
| JavaScript  |[Design Guidelines for TypeScript]           |[JavaScript Packages] |[azure-sdk-for-js Repository]     | [JavaScript Documentation]       |
| Python      |[Design Guidelines for Python]               |[Python Packages]     |[azure-sdk-for-python Repository] | [Python Documentation]           |

# 기여하기 (Contributing)

본 저장소는 Microsoft 오픈 소스에 기여하기 위한 가이드라인을 따르고 있습니다.

Note that below is Korean translation of contributing part on [Azure SDK's README.md](https://github.com/Azure/azure-sdk/blob/main/README.md#contributing).    
아래는 [Azure SDK 저장소에 언급된 기여하기 파트](https://github.com/Azure/azure-sdk/blob/main/README.md#contributing)를 한글로 번역한 것입니다. 참고하셨으면 합니다.

이 프로젝트는 기여 및 제안을 환영합니다. 대부분의 기여는 기부자가 기여를 사용할 권리를 가지고 있으며 실제로 그러한 권리를 당사에 부여한다고 선언하는 기부자 라이센스 계약(CLA)에 동의하기를 요구합니다. 자세한 내용은 https://cla.microsoft.com 을 방문하시면 됩니다.

Pull request를 제출하면 CLA 봇이 자동으로 CLA를 제공하고 PR을 적절하게 꾸밀 필요가 있는지 여부를 결정합니다(예: 레이블, 댓글). 봇이 제공하는 지침을 따르면 됩니다. 이 작업은 CLA를 사용하여 모든 저장소에서 한 번만 수행하면 됩니다.

Azure SDK에 기여할 수 있는 방법에 대한 모든 지침은 [contributing guide](CONTRIBUTING.md)를 참조하세요. 각 개별 SDK에 기여하는 방법에 대한 지침은 각 언어 repo의 CONTRIBUTING.md를 확인하세요.

이 프로젝트는 [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) 를 채택했습니다. 자세한 내용은 [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) 를 참조하거나 추가 질문 또는 의견이 있는 경우 [opencode@microsoft.com](mailto:opencode@microsoft.com) 로 문의하시면 됩니다.

[General Design Guidelines]: https://azure.github.io/azure-sdk/general_introduction.html
[Design Guidelines for Android]: https://azure.github.io/azure-sdk/android_design.html
[Design Guidelines for .NET]: https://azure.github.io/azure-sdk/dotnet_introduction.html
[Design Guidelines for Go]: https://azure.github.io/azure-sdk/golang_introduction.html
[Design Guidelines for C99]: https://azure.github.io/azure-sdk/clang_design.html
[Design Guidelines for C++]: https://azure.github.io/azure-sdk/cpp_introduction.html
[Design Guidelines for iOS]: https://azure.github.io/azure-sdk/ios_introduction.html
[Design Guidelines for Java]: https://azure.github.io/azure-sdk/java_introduction.html
[Design Guidelines for TypeScript]: https://azure.github.io/azure-sdk/typescript_introduction.html
[Design Guidelines for Python]: https://azure.github.io/azure-sdk/python_design.html
[revproc]: https://azure.github.io/azure-sdk/policies_reviewprocess.html

[azure-sdk Repository]: https://github.com/Azure/azure-sdk
[azure-sdk-for-android Repository]: https://github.com/Azure/azure-sdk-for-android
[azure-sdk-for-net Repository]: https://github.com/Azure/azure-sdk-for-net
[azure-sdk-for-go Repository]: https://github.com/Azure/azure-sdk-for-go
[azure-sdk-for-c Repository]: https://github.com/Azure/azure-sdk-for-c
[azure-sdk-for-cpp Repository]: https://github.com/Azure/azure-sdk-for-cpp
[azure-sdk-for-ios Repository]: https://github.com/Azure/azure-sdk-for-ios
[azure-sdk-for-java Repository]: https://github.com/Azure/azure-sdk-for-java
[azure-sdk-for-js Repository]: https://github.com/Azure/azure-sdk-for-js
[azure-sdk-for-python Repository]: https://github.com/Azure/azure-sdk-for-python

[Official Azure Documentation]: http://aka.ms/azure-sdk-docs
[.NET Documentation]: http://aka.ms/net-docs
[Go Documentation]: http://aka.ms/go-docs
[Java Documentation]: http://aka.ms/java-docs
[JavaScript Documentation]: http://aka.ms/js-docs
[Python Documentation]: https://aka.ms/python-docs
[C Documentation]: https://aka.ms/c-docs
[C++ Documentation]: https://aka.ms/cpp-docs

[.NET Packages]: https://azure.github.io/azure-sdk/releases/latest/dotnet.html
[Java Packages]: https://azure.github.io/azure-sdk/releases/latest/java.html
[Javascript Packages]: https://azure.github.io/azure-sdk/releases/latest/js.html
[Python Packages]: https://azure.github.io/azure-sdk/releases/latest/python.html
[C Packages]: https://azure.github.io/azure-sdk/releases/latest/c.html
[C++ Packages]: https://azure.github.io/azure-sdk/releases/latest/cpp.html
[Android Packages]: https://azure.github.io/azure-sdk/releases/latest/android.html
[iOS Packages]: https://azure.github.io/azure-sdk/releases/latest/ios.html
[Go Packages]: https://azure.github.io/azure-sdk/releases/latest/go.html
