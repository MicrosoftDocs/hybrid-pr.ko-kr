---
title: Azure 및 Azure Stack Hub에서 클라우드 간에 스케일링되는 앱 배포
description: Azure 및 Azure Stack Hub에서 클라우드 간에 스케일링되는 앱을 배포하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895417"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Azure 및 Azure Stack Hub를 사용하여 클라우드 간에 스케일링되는 앱 배포

트래픽 관리자를 통해 자동 스케일링을 사용하여 Azure Stack Hub 호스팅 웹앱에서 Azure 호스팅 웹앱으로 전환하는 수동으로 트리거되는 프로세스를 제공하는 클라우드 간 솔루션을 만드는 방법을 알아봅니다. 이 프로세스를 사용하면 부하를 받을 때 유연하게 스케일링 가능한 클라우드 유틸리티를 만들 수 있습니다.

이 패턴을 사용하면 테넌트가 앱을 퍼블릭 클라우드에서 즉시 실행하지 못할 수 있습니다. 그러나 앱의 급증하는 수요를 처리하기 위해 온-프레미스 환경에 필요한 용량을 유지하기가 경제적으로 어려운 기업도 있습니다. 테넌트에서 탄력적인 퍼블릭 클라우드를 온-프레미스 솔루션에 활용할 수 있습니다.

이 솔루션에서는 다음을 수행하는 샘플 환경을 빌드합니다.

> [!div class="checklist"]
> - 다중 노드 웹앱을 만듭니다.
> - CD(지속적인 배포) 프로세스를 구성하고 관리합니다.
> - 웹앱을 Azure Stack Hub에 게시합니다.
> - 릴리스를 만듭니다.
> - 배포를 모니터링하고 추적하는 방법을 알아봅니다.

> [!Tip]  
> ![하이브리드 핵심 요소 다이어그램](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub는 Azure의 확장입니다. Azure Stack Hub는 온-프레미스 환경에 클라우드 컴퓨팅의 민첩성과 혁신을 제공하여 어디서나 하이브리드 앱을 빌드하고 배포할 수 있는 유일한 하이브리드 클라우드를 사용하도록 설정합니다.  
> 
> [하이브리드 앱 디자인 고려 사항](overview-app-design-considerations.md) 문서는 하이브리드 앱 디자인, 배포 및 운영에 대한 소프트웨어 품질(배치, 확장성, 가용성, 복원력, 관리 효율성 및 보안)의 핵심 요소를 검토합니다. 디자인 고려 사항은 하이브리드 앱 디자인을 최적화하고 프로덕션 환경에서 문제를 최소화하는 데 도움이 됩니다.

## <a name="prerequisites"></a>필수 구성 요소

- 동작합니다. 필요한 경우 시작하기 전에 [체험 계정](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)을 만듭니다.
- Azure Stack Hub 통합 시스템 또는 ASDK(Azure Stack Development Kit) 배포
  - Azure Stack Hub를 설치하는 방법에 대한 지침은 [ASDK 설치](/azure-stack/asdk/asdk-install)를 참조하세요.
  - ASDK 배포 후 자동화 스크립트는 [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)에서 찾을 수 있습니다.
  - 이 설치를 완료하는 데 몇 시간 정도 걸릴 수 있습니다.
- [App Service](/azure-stack/operator/azure-stack-app-service-deploy) PaaS 서비스를 Azure Stack Hub에 배포합니다.
- Azure Stack Hub 환경 내에서 [계획/제안을 만듭니다](/azure-stack/operator/service-plan-offer-subscription-overview).
- Azure Stack Hub 환경 내에서 [테넌트 구독을 만듭니다](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm).
- 테넌트 구독 내에서 웹앱을 만듭니다. 나중에 사용할 수 있도록 새 웹앱 URL을 기록해 둡니다.
- 테넌트 구독 내에서 Azure Pipelines VM(가상 머신)을 배포합니다.
- .NET 3.5가 설치된 Windows Server 2016 VM이 필요합니다. 이 VM은 Azure Stack Hub의 테넌트 구독에서 프라이빗 빌드 에이전트로 빌드됩니다.
- [SQL 2017 VM 이미지가 있는 Windows Server 2016](/azure-stack/operator/azure-stack-add-vm-image)은 Azure Stack Hub Marketplace에서 사용할 수 있습니다. 이 이미지를 사용할 수 없는 경우 Azure Stack Hub 운영자에게 문의하여 환경에 추가해 달라고 요청합니다.

## <a name="issues-and-considerations"></a>문제 및 고려 사항

### <a name="scalability"></a>확장성

클라우드 간 스케일링의 핵심 구성 요소는 퍼블릭 클라우드와 온-프레미스 클라우드 인프라 간에 즉시 스케일링 및 주문형 스케일링을 제공하여 일관적이고 안정적인 서비스를 제공하는 기능입니다.

### <a name="availability"></a>가용성

로컬로 배포된 앱이 온-프레미스 하드웨어 구성 및 소프트웨어 배포를 통해 고가용성으로 구성되도록 합니다.

### <a name="manageability"></a>관리 효율

클라우드 간 솔루션은 환경 간에 원활한 관리 및 친숙한 인터페이스를 보장합니다. PowerShell은 플랫폼 간 관리에 권장되는 도구입니다.

## <a name="cross-cloud-scaling"></a>클라우드 간 크기 조정

### <a name="get-a-custom-domain-and-configure-dns"></a>사용자 지정 도메인 가져오기 및 DNS 구성

도메인에 대한 DNS 영역 파일을 업데이트합니다. Azure AD는 사용자 지정 도메인 이름의 소유권을 확인합니다. Azure 내에서 Azure/Microsoft 365/외부 DNS 레코드에 대해 [Azure DNS](/azure/dns/dns-getstarted-portal)를 사용하거나 [서로 다른 DNS 등록 기관](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider)에서 DNS 항목을 추가합니다.

1. 사용자 지정 도메인을 퍼블릭 등록자에 등록합니다.
2. 도메인에 대한 도메인 이름 등록 기관에 로그인합니다. 승인된 관리자가 DNS를 업데이트해야 할 수 있습니다.
3. Azure AD에서 제공하는 DNS 항목을 추가하여 도메인에 대한 DNS 영역 파일을 업데이트합니다. (DNS 항목은 이메일 라우팅 또는 웹 호스팅 동작에 영향을 주지 않습니다.)

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Azure Stack Hub에서 기본 다중 노드 웹앱 만들기

웹앱을 Azure 및 Azure Stack Hub에 배포하고 두 클라우드의 변경 내용을 자동으로 푸시하는 하이브리드 CI/CD(연속 통합 및 지속적인 배포)를 설정합니다.

> [!Note]  
> 실행을 위한 적절한 이미지(Windows Server 및 SQL)가 신디케이트된 Azure Stack Hub 및 App Service를 배포해야 합니다. 자세한 내용은 App Service 설명서 [Azure Stack Hub에 App Service를 배포하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started)을 검토하세요.

### <a name="add-code-to-azure-repos"></a>Azure Repos에 코드 추가

Azure Repos

1. Azure Repos에 프로젝트를 만드는 권한이 있는 계정으로 Azure Repos에 로그인합니다.

    하이브리드 CI/CD는 앱 코드와 인프라 코드 모두에 적용할 수 있습니다. 프라이빗 클라우드와 호스트된 클라우드 개발 모두에 [Azure Resource Manager 템플릿](https://azure.microsoft.com/resources/templates/)을 사용합니다.

    ![Azure Repos에서 프로젝트에 연결](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. 기본 웹앱을 만들고 열어 **리포지토리를 복제** 합니다.

    ![Azure 웹앱에서 리포지토리 복제](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>두 클라우드의 App Services에 대한 자체 포함 웹앱 배포 만들기

1. **WebApplication.csproj** 파일을 편집합니다. `Runtimeidentifier`를 선택하고 `win10-x64`를 추가합니다. ([자체 포함 배포](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조하세요.)

    ![웹앱 프로젝트 파일 편집](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. 팀 탐색기를 사용하여 코드를 Azure Repos에 체크 인합니다.

3. 앱 코드가 Azure Repos에 체크 인되었는지 확인합니다.

## <a name="create-the-build-definition"></a>빌드 정의 만들기

1. Azure Pipelines에 로그인하여 빌드 정의를 만드는 기능을 확인합니다.

2. **-r win10-x64** 코드를 추가합니다. .NET Core를 사용하여 자체 포함 배포를 트리거하려면 이 코드를 추가해야 합니다.

    ![웹앱에 코드 추가](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. 빌드를 실행합니다. [자체 포함 배포 빌드](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행되는 아티팩트를 게시합니다.

## <a name="use-an-azure-hosted-agent"></a>Azure 호스트된 에이전트 사용

Azure Pipelines에서 호스트된 빌드 에이전트를 사용하면 간편하게 웹앱을 빌드하고 배포할 수 있습니다. Microsoft Azure에서 유지 관리 및 업그레이드를 자동으로 수행하므로 지속적이고 중단 없는 개발 주기가 가능합니다.

### <a name="manage-and-configure-the-cd-process"></a>CD 프로세스 관리 및 구성

Azure Pipelines 및 Azure DevOps Services는 개발, 준비, QA, 프로덕션 환경 등의 여러 환경에 릴리스할 수 있도록 매우 유연하게 구성하고 쉽게 관리할 수 있는 파이프라인을 제공하며, 특정 단계에서 승인을 요구합니다.

## <a name="create-release-definition"></a>릴리스 정의 만들기

1. Azure DevOps Services의 **빌드 및 릴리스** 섹션에 있는 **릴리스** 탭 아래에서 **+** 단추를 선택하여 새 릴리스를 추가합니다.

    ![릴리스 정의 만들기](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Azure App Service 배포 템플릿을 적용합니다.

   ![Azure App Service 배포 템플릿 적용](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. **아티팩트 추가** 에서 Azure 클라우드 빌드 앱에 대한 아티팩트를 추가합니다.

   ![Azure 클라우드 빌드에 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. [파이프라인] 탭에서 환경의 **단계, 작업** 링크를 선택하고 Azure 클라우드 환경 값을 설정합니다.

   ![Azure 클라우드 환경 값 설정](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. **환경 이름** 을 설정하고 Azure 클라우드 엔드포인트에 대한 **Azure 구독** 을 선택합니다.

      ![Azure 클라우드 엔드포인트에 대한 Azure 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. **앱 서비스 이름** 에서 필요한 Azure 앱 서비스 이름을 설정합니다.

      ![Azure 앱 서비스 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. **에이전트 큐** 에서 Azure 클라우드 호스팅 환경으로 "호스트된 VS2017"을 입력합니다.

      ![Azure 클라우드 호스팅 환경의 에이전트 큐 설정](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. [Azure App Service 배포] 메뉴에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다. **폴더 위치** 에 대해 **확인** 을 선택합니다.
  
      ![Azure App Service 환경에 사용할 패키지 또는 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![폴더 선택기 대화 상자 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. 모든 변경 내용을 저장하고 **릴리스 파이프라인** 으로 돌아갑니다.

    ![릴리스 파이프라인의 변경 내용 저장](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. 새 아티팩트를 추가하고 Azure Stack Hub 앱에 대한 빌드를 선택합니다.

    ![Azure Stack Hub 앱에 대한 새 아티팩트 추가](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Azure App Service 배포를 적용하여 환경을 하나 더 추가합니다.

    ![Azure App Service 배포에 환경 추가](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. 새 환경의 이름을 "Azure Stack"으로 지정합니다.

    ![Azure App Service 배포에서 환경의 이름 지정](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. **작업** 탭에서 Azure Stack 환경을 찾습니다.

    ![Azure Stack 환경](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Azure Stack 엔드포인트의 구독을 선택합니다.

    ![Azure Stack 엔드포인트의 구독 선택](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. 앱 서비스 이름으로 Azure Stack 웹앱 이름을 설정합니다.
    ![Azure Stack 웹앱 이름 설정](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Azure Stack 에이전트를 선택합니다.

    ![Azure Stack 에이전트 선택](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. [Azure App Service 배포] 섹션에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다. 폴더 위치에 대해 **확인** 을 선택합니다.

    ![Azure App Service 배포에 사용할 폴더 선택](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![폴더 선택기 대화 상자 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. [변수] 탭에서 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`라는 변수를 추가하고 해당 값을 **true**, 범위를 Azure Stack으로 설정합니다.

    ![Azure 앱 배포에 변수 추가](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. 두 아티팩트 모두에서 **지속적인** 배포 트리거 아이콘을 선택하고 **지속적인** 배포 트리거를 사용하도록 설정합니다.

    ![지속적인 배포 트리거 선택](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Azure Stack 환경에서 **배포 전** 조건 아이콘을 선택하고 트리거를 **릴리스 후** 로 설정합니다.

    ![배포 전 조건 선택](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. 모든 변경 내용을 저장합니다.

> [!Note]  
> 템플릿에서 릴리스 정의를 만들 때 작업에 대한 일부 설정이 자동으로 [환경 변수](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)로 정의되었을 수 있습니다. 이러한 설정은 작업 설정에서 수정할 수 없습니다. 이러한 설정을 편집하려면 부모 환경 항목을 선택해야 합니다.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Visual Studio를 통해 Azure Stack Hub에 게시

Azure DevOps Services 빌드는 엔드포인트를 만들어서 Azure Stack Hub에 Azure 서비스 앱을 배포할 수 있습니다. Azure Pipelines는 빌드 에이전트에 연결하고, 빌드 에이전트는 Azure Stack Hub에 연결합니다.

1. Azure DevOps Services에 로그인하고 앱 설정 페이지로 이동합니다.

2. **설정** 에서 **보안** 을 선택합니다.

3. **VSTS 그룹** 에서 **엔드포인트 작성자** 를 선택합니다.

4. **구성원** 탭에서 **추가** 를 선택합니다.

5. **사용자 및 그룹 추가** 에서 사용자 이름을 입력하고 사용자 목록에서 해당 사용자를 선택합니다.

6. **변경 내용 저장** 을 선택합니다.

7. **VSTS 그룹** 목록에서 **엔드포인트 관리자** 를 선택합니다.

8. **구성원** 탭에서 **추가** 를 선택합니다.

9. **사용자 및 그룹 추가** 에서 사용자 이름을 입력하고 사용자 목록에서 해당 사용자를 선택합니다.

10. **변경 내용 저장** 을 선택합니다.

이제 엔드포인트 정보가 있으므로 Azure Pipelines에서 Azure Stack Hub에 연결할 수 있습니다. Azure Stack Hub의 빌드 에이전트는 Azure Pipelines에서 명령을 가져온 다음, Azure Stack Hub와 통신할 수 있도록 엔드포인트 정보를 전달합니다.

## <a name="develop-the-app-build"></a>앱 빌드 개발

> [!Note]  
> 실행을 위한 적절한 이미지(Windows Server 및 SQL)가 신디케이트된 Azure Stack Hub 및 App Service를 배포해야 합니다. 자세한 내용은 [Azure Stack Hub에 App Service를 배포하기 위한 필수 조건](/azure-stack/operator/azure-stack-app-service-before-you-get-started)을 참조하세요.

Azure Repos의 웹앱 코드와 같은 [Azure Resource Manager 템플릿](https://azure.microsoft.com/resources/templates/)을 사용하여 두 클라우드를 배포합니다.

### <a name="add-code-to-an-azure-repos-project"></a>Azure Repos 프로젝트에 코드 추가

1. Azure Stack Hub에 프로젝트를 만드는 권한이 있는 계정으로 Azure Repos에 로그인합니다.

2. 기본 웹앱을 만들고 열어 **리포지토리를 복제** 합니다.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>두 클라우드의 App Services에 대한 자체 포함 웹앱 배포 만들기

1. 다음과 같이 **WebApplication.csproj** 파일을 편집합니다. `Runtimeidentifier`를 선택한 다음, `win10-x64`를 추가합니다. 자세한 내용은 [자체 포함 배포](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 설명서를 참조하세요.

2. 팀 탐색기를 사용하여 코드를 Azure Repos에 체크 인합니다.

3. 앱 코드가 Azure Repos에 체크 인되었는지 확인합니다.

### <a name="create-the-build-definition"></a>빌드 정의 만들기

1. 빌드 정의를 만들 수 있는 계정으로 Azure Pipelines에 로그인합니다.

2. 프로젝트에 대한 **웹 애플리케이션 빌드** 페이지로 이동합니다.

3. **인수** 에서 **-r win10-x64** 코드를 추가합니다. .NET Core를 사용하여 자체 포함 배포를 트리거하려면 이 코드를 추가해야 합니다.

4. 빌드를 실행합니다. [자체 포함 배포 빌드](/dotnet/core/deploying/deploy-with-vs#simpleSelf) 프로세스는 Azure 및 Azure Stack Hub에서 실행 가능한 아티팩트를 게시합니다.

#### <a name="use-an-azure-hosted-build-agent"></a>Azure 호스트된 빌드 에이전트 사용

Azure Pipelines에서 호스트된 빌드 에이전트를 사용하면 간편하게 웹앱을 빌드하고 배포할 수 있습니다. Microsoft Azure에서 유지 관리 및 업그레이드를 자동으로 수행하므로 지속적이고 중단 없는 개발 주기가 가능합니다.

### <a name="configure-the-continuous-deployment-cd-process"></a>CD(지속적인 배포) 프로세스를 구성합니다.

Azure Pipelines 및 Azure DevOps Services는 개발, 준비, QA, 프로덕션 환경 등의 여러 환경에 릴리스할 수 있도록 매우 유연하게 구성하고 쉽게 관리할 수 있는 파이프라인을 제공합니다. 이 프로세스는 앱 수명 주기의 특정 단계에서 승인이 필요할 수 있습니다.

#### <a name="create-release-definition"></a>릴리스 정의 만들기

앱 빌드 프로세스의 마지막 단계는 릴리스 정의 만들기입니다. 이 릴리스 정의는 릴리스를 만들고 빌드를 배포하는 데 사용됩니다.

1. Azure Pipelines에 로그인하고 프로젝트에 대한 **빌드 및 릴리스** 로 이동합니다.

2. **릴리스** 탭에서 **[+]** 기호를 선택한 다음, **릴리스 정의 만들기** 를 선택합니다.

3. **템플릿 선택** 에서 **Azure App Service 배포** 를 선택한 다음, **적용** 을 선택합니다.

4. **아티팩트 추가** 의 **원본(빌드 정의)** 에서 Azure 클라우드 빌드 앱을 선택합니다.

5. **파이프라인** 탭에서 **단계 1**, **작업 1** 링크를 선택하여 **환경 작업을 살펴봅니다**.

6. **작업** 탭에서 **환경 이름** 으로 Azure를 입력하고, **Azure 구독** 목록에서 AzureCloud Traders-Web EP를 선택합니다.

7. **Azure 앱 서비스 이름** 을 입력합니다. 다음 화면 캡처에서는 `northwindtraders`입니다.

8. 에이전트 단계의 경우 **에이전트 큐** 목록에서 **호스트된 VS2017** 을 선택합니다.

9. **Azure App Service 배포** 에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다.

10. **파일 또는 폴더** 에서 **위치** 에 대해 **확인** 을 선택합니다.

11. 모든 변경 내용을 저장하고 **파이프라인** 으로 돌아갑니다.

12. **파이프라인** 탭에서 **아티팩트 추가** 를 선택하고, **원본(빌드 정의)** 목록에서 **NorthwindCloud Traders-Vessel** 을 선택합니다.

13. **템플릿 선택** 에서 또 다른 환경을 추가합니다. **Azure App Service 배포** 를 선택한 다음, **적용** 을 선택합니다.

14. **환경 이름** 으로 `Azure Stack Hub`를 입력합니다.

15. **작업** 탭에서 Azure Stack Hub를 찾아 선택합니다.

16. **Azure 구독** 목록에서 Azure Stack Hub 엔드포인트로 **AzureStack Traders-Vessel EP** 를 선택합니다.

17. **앱 서비스 이름** 으로 Azure Stack Hub 웹앱 이름을 입력합니다.

18. **에이전트 선택** 의 **에이전트 큐** 목록에서 **AzureStack -b Douglas Fir** 을 선택합니다.

19. **Azure App Service 배포** 에서 해당 환경의 유효한 **패키지 또는 폴더** 를 선택합니다. **파일 또는 폴더 선택** 에서 폴더 **위치** 에 대해 **확인** 을 선택합니다.

20. **변수** 탭에서 `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`라는 변수를 찾습니다. 변수 값을 **true** 로 설정하고, 범위를 **Azure Stack Hub** 로 설정합니다.

21. **파이프라인** 탭에서 NorthwindCloud Traders-Web 아티팩트에 대한 **지속적인 배포 트리거** 아이콘을 선택하고, **지속적인 배포 트리거** 를 **사용** 으로 설정합니다. **NorthwindCloud Traders-Vessel** 아티팩트에 대해 동일한 작업을 수행합니다.

22. Azure Stack Hub 환경에서 **배포 전 조건** 아이콘을 선택하고, 트리거를 **릴리스 후** 로 설정합니다.

23. 모든 변경 내용을 저장합니다.

> [!Note]  
> 템플릿에서 릴리스 정의를 만들 때 작업에 대한 일부 설정이 자동으로 [환경 변수](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables)로 정의됩니다. 이러한 설정은 작업 설정에서 수정할 수 없고, 부모 환경 항목에서 수정할 수 있습니다.

## <a name="create-a-release"></a>릴리스 만들기

1. **파이프라인** 탭에서 **릴리스** 목록을 열고 **릴리스 만들기** 를 선택합니다.

2. 릴리스에 대한 설명을 입력하고 올바른 아티팩트가 선택되었는지 확인한 다음, **만들기** 를 선택합니다. 잠시 후 새 릴리스가 생성되었다는 배너가 나타나고 릴리스 이름이 링크로 표시됩니다. 이 링크를 선택하여 릴리스 요약 페이지를 살펴봅니다.

3. 릴리스 요약 페이지에는 릴리스에 대한 세부 정보가 표시됩니다. "릴리스-2"의 다음 화면 캡처에서 **환경** 섹션에는 Azure **배포 상태** 가 "진행 중"으로 표시되고, Azure Stack Hub의 상태는 "성공"으로 표시됩니다. Azure 환경의 배포 상태가 "성공"으로 변경되면 릴리스를 승인할 준비가 되었다는 배너가 표시됩니다. 배포가 보류 중이거나 실패한 경우 파란색 **(i)** 정보 아이콘이 표시됩니다. 아이콘을 마우스로 가리키면 지연 또는 실패 이유를 설명하는 팝업이 표시됩니다.

4. 릴리스 목록과 같은 기타 보기에도 승인이 보류 중이라는 아이콘이 표시됩니다. 이 아이콘의 팝업은 환경 이름 및 배포와 관련된 자세한 정보를 표시합니다. 관리자는 손쉽게 전반적인 릴리스 진행률을 확인하고 승인 대기 중인 릴리스를 볼 수 있습니다.

## <a name="monitor-and-track-deployments"></a>배포 모니터링 및 추적

1. **릴리스-2** 요약 페이지에서 **로그** 를 선택합니다. 배포하는 동안 이 페이지에는 에이전트의 실시간 로그가 표시됩니다. 왼쪽 창에는 각 환경에 대한 각 배포 작업의 상태가 표시됩니다.

2. 배포 전 또는 배포 후 승인에 대한 **작업** 열에서 사람 아이콘을 선택하여 배포를 승인(또는 거부)한 사람과 그 사람이 제공한 메시지를 확인합니다.

3. 배포가 완료되면 전체 로그 파일이 오른쪽 창에 표시됩니다. 왼쪽 창에서 원하는 **단계** 를 선택하여 **작업 초기화** 같은 단일 단계에 대한 로그 파일을 살펴봅니다. 개별 로그를 확인하는 기능을 사용하면 전체 배포의 일부를 쉽게 추적하고 디버그할 수 있습니다. 단계에 대한 로그 파일을 **저장** 하거나 **모든 로그를 zip으로 다운로드** 합니다.

4. **요약** 탭을 열고 릴리스에 대한 일반 정보를 확인합니다. 이 보기에는 빌드, 빌드가 배포된 환경, 배포 상태에 대한 자세한 정보와 릴리스에 대한 기타 정보가 표시됩니다.

5. 환경 링크(**Azure** 또는 **Azure Stack Hub**)를 선택하여 특정 환경의 기존 배포 및 보류 중인 배포를 확인합니다. 이러한 보기를 사용하여 동일한 빌드가 두 환경에 배포되었는지 신속하게 확인할 수 있습니다.

6. 브라우저에서 **배포된 프로덕션 앱** 을 엽니다. 예를 들어 Azure App Service 웹 사이트의 경우 URL `https://[your-app-name\].azurewebsites.net`을 엽니다.

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>Azure와 Azure Stack Hub를 통합하여 확장성 있는 클라우드 간 솔루션 제공

유연하고 강력한 다중 클라우드 서비스는 데이터 보안, 백업 및 중복성, 일관적이고 빠른 가용성, 확장성 있는 스토리지 및 배포, 지리적 규격 라우팅을 제공합니다. 수동으로 트리거되는 이 프로세스를 통해 호스트된 웹앱 간에 안정적이고 효율적으로 부하를 전환하고 중요한 데이터의 가용성을 즉시 확보할 수 있습니다.

## <a name="next-steps"></a>다음 단계

- Azure Cloud 패턴에 대해 자세히 알아보려면 [클라우드 디자인 패턴](/azure/architecture/patterns)을 참조하세요.
