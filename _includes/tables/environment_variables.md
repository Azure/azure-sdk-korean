| Environment Variable          | Purpose                                                                                    |
|-------------------------------|--------------------------------------------------------------------------------------------|
| **Proxy Settings**            |                                                                                            |
| HTTP_PROXY                    | HTTP 연결에 대한 프록시                                                                         |
| HTTPS_PROXY                   | HTTPS 연결에 대한 프록시                                                                       |
| NO_PROXY                      | 프록시를 사용해서는 안되는 호스트                                                                  |
| ALL_PROXY                     | HTTP_PROXY 및/또는 HTTPS_PROXY 정의되지 않은 경우 HTTP 및/또는 HTTPS 연결에 대한 프록시                |
| **Identity**                  |                                                                                            |
| MSI_ENDPOINT                  | Azure AD MSI 자격 증명                                                                       |
| MSI_SECRET                    | Azure AD MSI 자격 증명                                                                        |
| AZURE_USERNAME                | U/P Auth용 Azure 사용자 이름                                                                   |
| AZURE_PASSWORD                | U/P Auth용 Azure 암호                                                                        |
| AZURE_CLIENT_CERTIFICATE_PATH | Azure 활성 디렉터리                                                                           |
| AZURE_CLIENT_ID               | Azure 활성 디렉터리                                                                           |
| AZURE_CLIENT_SECRET           | Azure 활성 디렉터리                                                                           |
| AZURE_TENANT_ID               | Azure 활성 디렉터리                                                                           |
| AZURE_AUTHORITY_HOST          | Azure 활성 디렉터리                                                                           |
| **Pipeline Configuration**    |                                                                                            |
| AZURE_TELEMETRY_DISABLED      | 원격 분석 비활성화                                                                              |
| AZURE_LOG_LEVEL               | 로그 레벨 설정하여 로깅 사용하도록 설정                                                              |
| AZURE_TRACING_DISABLED        | 추적 비활성화                                                                                  |
| **General SDK Configuration** |                                                                                            |
| AZURE_CLOUD                   | 소버린 클라우드의 이름                                                                          |
| AZURE_SUBSCRIPTION_ID         | Azure 구독                                                                                  |
| AZURE_RESOURCE_GROUP          | Azure 리소스 그룹                                                                             |
