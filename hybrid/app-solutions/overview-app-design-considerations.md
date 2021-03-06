---
title: Azure 및 Azure Stack Hub의 하이브리드 앱 디자인 고려 사항
description: 배치, 확장성, 가용성 및 복원력을 포함하여 인텔리전트 클라우드 및 인텔리전트 에지용 하이브리드 앱을 빌드할 때 생각해야 할 디자인 고려 사항에 대해 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 06/07/2020
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8b975c7b99807490d446f557e84b6e0eabf34649
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852493"
---
# <a name="hybrid-app-design-considerations"></a>하이브리드 앱 디자인 고려 사항

Microsoft Azure는 유일하게 일관적인 하이브리드 클라우드입니다. 투자한 개발 자산을 재사용하여 글로벌 Azure, 소버린 Azure 클라우드, 데이터 센터의 Azure 확장 기능인 Azure Stack을 포괄하는 앱을 만들 수 있습니다. 클라우드를 포괄하는 앱을 *하이브리드 앱*이라고도 합니다.

[*Azure 애플리케이션 아키텍처 가이드*](/azure/architecture/guide)에서는 확장성과 복원력이 있고 가용성이 뛰어난 앱을 디자인하기 위한 체계적인 방법을 설명합니다. [*Azure 애플리케이션 아키텍처 가이드*](/azure/architecture/guide)에 설명된 고려 사항은 단일 클라우드용으로 설계된 앱과 클라우드를 포괄하는 앱에 동일하게 적용됩니다.

이 문서에서는 하이브리드 앱 디자인에 초점을 맞춘 [*Azure 애플리케이션*](/azure/architecture/guide/) [*아키텍처 가이드*](/azure/architecture/guide/)에 설명된 [*소프트웨어 품질 핵심 요소*](/azure/architecture/guide/pillars)를 보충 설명합니다. 또한 하이브리드 앱은 단일 클라우드 또는 단일 온-프레미스 데이터 센터에만 국한되지 않으므로 *배치* 핵심 요소가 추가되었습니다.

하이브리드 시나리오는 개발에 사용할 수 있는 리소스와 지리, 보안, 인터넷 액세스 등의 기타 고려 사항에 따라 크게 달라집니다. 이 가이드에서 특정 고려 사항을 열거할 수 없지만, 여러분이 준수해야 하는 몇 가지 주요 지침과 모범 사례를 제공해 드릴 수 있습니다. 하이브리드 앱 아키텍처를 성공적으로 설계, 구성, 배포 및 유지 관리하려면 여러 가지 디자인 고려 사항을 염두에 두어야 하며 그 중에는 여러분이 잘 모르는 내용도 있을 수 있습니다.

이 문서는 하이브리드 앱을 구현할 때 가질 수 있는 의문 사항을 종합하고 고려 사항(이와 같은 핵심 요소) 및 고려 사항을 해결하는 모범 사례를 제공하기 위해 작성되었습니다. 디자인 단계에서 이러한 의문 사항을 해결하면 프로덕션에서 발생할 수 있는 문제를 예방할 수 있습니다.

기본적으로 하이브리드 앱을 만들기 전에 이와 같은 의문 사항에 대해 고민해야 합니다. 시작하려면 다음 작업을 수행해야 합니다.

- 앱 구성 요소를 식별하고 평가합니다.
- 앱 구성 요소를 핵심 요소와 비교하여 평가합니다.

## <a name="evaluate-the-app-components"></a>앱 구성 요소 평가

앱의 각 구성 요소는 더 큰 앱 내에서 고유한 특정 역할을 가지며, 모든 디자인 고려 사항과 비교하여 검토해야 합니다. 앱 아키텍처를 결정하는 데 도움이 되도록 각 구성 요소의 요구 사항 및 기능을 이러한 고려 사항과 매핑해야 합니다.

앱의 아키텍처를 연구하고 무엇으로 구성되었는지 확인하여 앱을 구성 요소 단위로 분해합니다. 구성 요소에는 앱이 상호 작용하는 다른 앱이 포함될 수도 있습니다. 구성 요소를 식별할 때 다음과 같은 질문을 하여 구성 요소의 특징에 따라 원하는 하이브리드 작업을 평가합니다.

- 구성 요소의 목적은 무엇인가요?
- 구성 요소 간 상호 종속성이 어떤가요?

예를 들어 앱에서 두 개의 구성 요소로 프런트 엔드 및 백 엔드를 정의할 수 있습니다. 하이브리드 시나리오에서는 프런트 엔드가 한 클라우드에 있고 백 엔드는 다른 클라우드에 있습니다. 앱은 프런트 엔드와 사용자 사이에, 그리고 프런트 엔드와 백 엔드 사이에 통신 채널을 제공합니다.

앱 구성 요소는 다양한 형식과 시나리오에 의해 정의됩니다. 가장 중요한 작업은 구성 요소와 해당 클라우드 또는 온-프레미스 위치를 식별하는 것입니다.

표 1에는 인벤토리에 포함해야 하는 공통 앱 구성 요소가 나와 있습니다.

### <a name="table-1-common-app-components"></a>표 1. 공통 앱 구성 요소

| **구성 요소** | **하이브리드 앱 지침** |
| ---- | ---- |
| 클라이언트 연결 | (모든 디바이스에서) 앱은 다음과 같은 방법을 포함하여 단일 진입점에서 다양한 방법으로 사용자에 액세스할 수 있습니다.<br>- 사용자가 앱과 함께 작동하도록 클라이언트를 설치해야 하는 클라이언트-서버 모델. 브라우저에서 액세스하는 서버 기반 앱입니다.<br>- 클라이언트 연결에는 연결이 끊어질 때 알려주는 알림 또는 로밍 요금이 적용될 때 알려주는 경고가 포함될 수 있습니다. |
| 인증  | 사용자가 앱에 연결하거나 한 구성 요소에서 다른 구성 요소에 연결할 때 인증이 필요할 수 있습니다. |
| API  | API 세트와 클래스 라이브러리를 사용하여 개발자에게 앱에 대한 프로그래밍 방식 액세스를 제공하고, 인터넷 표준 기반의 연결 인터페이스를 제공할 수 있습니다. API를 사용하여 앱을 독립적으로 작동하는 논리 단위로 분해할 수도 있습니다. |
| Services  | 간결한 서비스를 사용하여 앱에 대한 기능을 제공할 수 있습니다. 서비스는 앱이 실행되는 엔진일 수 있습니다. |
| 큐 | 큐를 사용하여 앱 구성 요소의 수명 주기 및 상태를 구성할 수 있습니다. 이러한 큐는 구독 당사자에게 메시지, 알림 및 버퍼링 기능을 제공할 수 있습니다. |
| 데이터 스토리지 | 앱은 상태 비저장 앱일 수도 있고 상태 저장 앱일 수도 있습니다. 상태 저장 앱에는 다양한 형식 및 볼륨을 충족할 수 있는 데이터 스토리지가 필요합니다. |
| 데이터 캐싱  | 디자인의 데이터 캐싱 구성 요소는 대기 시간 문제를 전략적으로 해결하고 클라우드 버스트를 트리거하는 역할을 수행할 수 있습니다. |
| 데이터 수집 | 사용자가 웹 양식을 통해 값을 제출하는 방법부터 지속적인 대량 데이터 흐름까지 다양한 방법으로 데이터를 앱에 제출할 수 있습니다. |
| 데이터 처리 | 보고서, 분석, 일괄 처리 내보내기, 데이터 변환 등의 데이터 처리 작업을 원본에서 처리할 수도 있고 데이터 복사본을 사용하여 별도의 구성 요소에 오프로드할 수도 있습니다. |

## <a name="assess-app-components-for-pillars"></a>앱 구성 요소를 핵심 요소와 비교 평가

각 구성 요소의 특징을 각 핵심 요소와 비교 평가합니다. 각 구성 요소를 모든 핵심 요소와 비교 평가할 때 미처 고려하지 않은 문제가 하이브리드 앱의 디자인에 영향을 주는 것으로 확인될 수 있습니다. 이러한 고려 사항에 따라 조치하면 앱을 더욱 최적화할 수 있습니다. 표 2에서는 하이브리드 앱과 관련된 각 핵심 요소에 대해 설명합니다.

### <a name="table-2-pillars"></a>표 2. 핵심 요소

| **핵심 요소** | **설명** |
| ----------- | --------------------------------------------------------- |
| 배치  | 하이브리드 앱에서 구성 요소의 전략적 포지셔닝입니다. |
| 확장성  | 증가된 부하를 처리하는 시스템의 기능입니다. |
| 가용성  | 하이브리드 앱이 작동하는 시간의 비율입니다. |
| 복원력 | 하이브리드 앱을 복구하는 기능입니다. |
| 관리 효율 | 프로덕션에서 시스템을 실행하는 작업 프로세스입니다. |
| 보안 | 앱과 데이터를 위협으로부터 보호합니다. |

## <a name="placement"></a>배치

하이브리드 앱은 기본적으로 데이터 센터와 마찬가지로 배치를 고려해야 합니다.

배치는 구성 요소를 포지셔닝하는 중요한 작업으로, 하이브리드 앱이 잘 작동하려면 배치가 좋아야 합니다 정의에 따라 하이브리드 앱은 온-프레미스부터 클라우드까지, 그리고 다른 클라우드 사이의 위치까지 포괄합니다. 다음 두 가지 방법으로 앱의 구성 요소를 클라우드에 배치할 수 있습니다.

- **수직적 하이브리드 앱**  
    앱 구성 요소가 여러 위치에 분산됩니다. 각 개별 구성 요소의 여러 인스턴스가 한 위치에만 배치되어야 합니다.

- **수평적 하이브리드 앱**  
    앱 구성 요소가 여러 위치에 분산됩니다. 각 개별 구성 요소의 여러 인스턴스가 여러 위치에 걸쳐 있어도 됩니다.

    어떤 구성 요소는 자신의 위치를 알고, 또 어떤 구성 요소는 자신의 위치 및 배치를 모릅니다. 이와 같은 장점은 추상화 레이어를 사용하여 달성할 수 있습니다. 이 레이어는 마이크로서비스 같은 최신 앱 프레임워크를 사용하여 클라우드의 노드에서 작동하는 앱 구성 요소의 배치를 통해 앱이 서비스되는 방식을 정의할 수 있습니다.

### <a name="placement-checklist"></a>배치 검사 목록

**필요한 위치를 확인합니다.** 작동하려면 앱 또는 앱의 구성 요소를 필요하게 만들거나 특정 클라우드에 대한 인증을 요구합니다. 여기에는 회사 또는 법률에서 요구하는 주권 요구 사항이 포함될 수 있습니다. 또한 특정 위치 또는 로캘에 대한 온-프레미스 작업이 필요한지 확인합니다.

**연결 종속성을 확인합니다.** 필요한 위치 및 기타 요소에 따라 구성 요소 간 연결 종속성이 달라질 수 있습니다. 구성 요소를 배치할 때 구성 요소 간에 원활한 통신이 가능하도록 최적의 연결 및 보안을 확인해야 합니다. [*VPN*,](/azure/vpn-gateway/) [*ExpressRoute*](/azure/expressroute/) 및 [*하이브리드 연결*을 선택할 수 있습니다.](/azure/app-service/app-service-hybrid-connections)

**플랫폼 기능을 평가합니다.** 각 앱 구성 요소와 관련하여, 앱 구성 요소에 필요한 리소스 공급자를 클라우드에서 사용할 수 있는지, 그리고 대역폭이 예상 처리량 및 대기 시간 요구 사항을 수용할 수 있는지 확인합니다.

**이식성에 대한 계획을 세웁니다.** 컨테이너 또는 마이크로서비스 같은 최신 앱 프레임워크를 사용하여 작업을 이동하고 서비스 종속성을 방지할 계획을 세웁니다.

**데이터 주권 요구 사항을 확인합니다.** 하이브리드 앱은 로컬 데이터 센터 등의 데이터 격리를 수용하도록 설계되었습니다. 리소스 배치를 검토하여 이 요구 사항을 수용할 수 있도록 최적화하세요.

**대기 시간에 대한 계획을 세웁니다.** 클라우드 간 작업 시 앱 구성 요소 사이에 물리적 거리가 있을 수 있습니다. 대기 시간을 수용하기 위한 요구 사항을 확인하세요.

**트래픽 흐름을 제어합니다.** 퍼블릭 클라우드의 프런트 엔드에서 액세스할 때 피크 사용량을 처리하고 개인 식별 정보 데이터를 적절하고 안전하게 전달해야 합니다.

## <a name="scalability"></a>확장성

확장성은 늘어나는 앱 부하를 처리하는 시스템 기능이며, 앱의 크기와 범위 외에도 다른 요소와 압력이 대상 크기에 영향을 주기 때문에 시간이 지나면 앱의 부하가 달라질 수 있습니다.

이 핵심 요소에 대한 자세한 내용은 뛰어난 아키텍처의 5가지 핵심 요소에서 [*확장성*](/azure/architecture/guide/pillars#scalability)을 참조하세요.

하이브리드 앱에 수평 스케일링 방법을 사용하면 인스턴스를 추가하여 수요를 충족한 다음, 사용량이 적은 기간에는 인스턴스를 사용하지 않도록 설정할 수 있습니다.

하이브리드 시나리오에서 개별 구성 요소를 스케일 아웃하려면 구성 요소가 클라우드에 분산될 때 추가로 고려할 사항이 있습니다. 앱의 한 부분을 스케일링하려면 다른 부분도 스케일링해야 할 수 있습니다. 예를 들어 클라이언트 연결 수가 늘어나지만 앱의 웹 서비스가 적절하게 스케일 아웃되지 않으면 데이터베이스에 걸리는 부하로 인해 앱이 포화 상태에 빠질 수 있습니다.

어떤 앱 구성 요소는 선형으로 스케일 아웃할 수 있고, 또 어떤 구성 요소는 스케일링 종속성이 있어서 스케일링 범위가 제한될 수 있습니다. 예를 들어 앱 구성 요소 위치에 대한 하이브리드 연결을 제공하는 VPN 터널은 스케일링 가능한 대역폭과 대기 시간이 제한되어 있습니다. 이러한 요구 사항을 충족하도록 앱 구성 요소가 어떻게 스케일링될까요?

### <a name="scalability-checklist"></a>확장성 검사 목록

**스케일링 임계값을 확인합니다.** 앱의 다양한 종속성을 처리하려면 앱을 실행하기 위한 요구 사항을 충족하면서 여러 클라우드의 앱 구성 요소가 서로 독립적으로 스케일링할 수 있는 범위를 결정해야 합니다. 하이브리드 앱은 기능을 처리하기 위해 앱의 특정 영역을 스케일링해야 하는 경우가 자주 있는데, 특정 영역이 나머지 영역과 상호 작용하고 영향을 주기 때문입니다. 예를 들어 프런트 엔드 인스턴스 수를 초과하려면 백 엔드를 스케일링해야 할 수 있습니다.

**스케일링 일정을 정의합니다.** 대부분의 앱은 사용량이 많은 기간이 있으므로, 피크 타임을 일정에 집계하여 최적의 스케일링을 조율해야 합니다.

**중앙 집중식 모니터링 시스템을 사용합니다.** 플랫폼 모니터링 기능은 자동 스케일링을 제공할 수 있지만, 하이브리드 앱에는 시스템 상태 및 부하를 집계하는 중앙 집중식 모니터링 시스템이 필요합니다. 중앙 집중식 모니터링 시스템은 다른 위치에 있는 리소스에 따라 한 위치에 있는 리소스의 스케일링을 시작할 수 있습니다. 또한 중앙 모니터링 시스템은 리소스를 자동 스케일링하는 클라우드와 그렇지 않은 클라우드를 추적할 수 있습니다.

**자동 스케일링 기능을 활용합니다(사용 가능한 경우).** 아키텍처에 자동 스케일링 기능이 포함된 경우 앱 구성 요소를 스케일 업, 다운, 아웃 또는 인해야 하는 조건을 정의하는 임계값을 설정하여 자동 스케일링을 구현합니다. 자동 스케일링의 예로는 용량 증가를 처리하기 위해 한 클라우드에서 자동 스케일링되지만, 다른 클라우드에 분산된 앱의 다른 종속 구성 요소도 함께 스케일링되는 클라이언트 연결을 들 수 있습니다. 이러한 종속 구성 요소의 자동 스케일링 기능을 확인해야 합니다.

자동 스케일링을 사용할 수 없는 경우 중앙 집중식 모니터링 시스템의 임계값에 의해 트리거된 수동 스케일링을 수용할 수 있도록 스크립트 및 기타 리소스를 구현하는 것이 좋습니다.

**위치별 예상 부하를 결정합니다.** 클라이언트 요청을 처리하는 하이브리드 앱은 주로 단일 위치에 의존할 수 있습니다. 클라이언트 요청 부하가 임계값을 초과하면 다른 위치에 추가 리소스를 추가하여 인바운드 요청의 부하를 분산할 수 있습니다. 클라이언트 연결에서 증가하는 부하를 처리할 수 있는지 확인하고, 클라이언트 연결에서 부하를 처리하기 위한 자동화된 절차를 결정해야 합니다.

## <a name="availability"></a>가용성

가용성이란 시스템이 작동하는 시간을 말합니다. 가용성은 작동 시간의 백분율로 측정됩니다. 앱 오류, 인프라 문제 및 시스템 부하로 인해 가용성이 저하될 수 있습니다.

이 핵심 요소에 대한 자세한 내용은 뛰어난 아키텍처의 5가지 핵심 요소에서 [*가용성*](/azure/architecture/framework/)을 참조하세요.

### <a name="availability-checklist"></a>가용성 검사 목록

**연결에 대한 중복성을 제공합니다.** 하이브리드 앱은 앱이 분산된 클라우드 간에 연결이 필요합니다. 하이브리드 연결에 사용할 기술을 선택할 수 있으므로, 선택하는 기본 기술 외에도 기본 기술이 실패할 경우 다른 기술을 사용하여 자동화된 장애 조치(failover) 기능을 통해 중복성을 제공할 수 있습니다.

**장애 도메인을 분류합니다.** 내결함성이 있는 앱에는 여러 장애 도메인이 필요합니다. 장애 도메인은 온-프레미스의 단일 하드 디스크에 오류가 발생하거나, Top-of-Rack 스위치의 작동이 중단되거나 전체 데이터 센터를 사용할 수 없는 경우와 같은 오류 지점을 분리하는 데 도움이 됩니다. 하이브리드 앱에서는 위치를 장애 도메인으로 분류할 수 있습니다. 가용성 요구 사항이 많을수록 단일 장애 도메인을 분류하는 방법을 더 많이 평가해야 합니다.

**업그레이드 도메인을 분류합니다.** 업그레이드 도메인은 앱 구성 요소의 인스턴스 가용성을 확보하는 데 사용되고, 동일한 구성 요소의 다른 인스턴스는 업데이트 또는 기능 업그레이드를 통해 제공됩니다. 장애 도메인과 마찬가지로, 배치되는 위치에 따라 업그레이드 도메인을 분류할 수 있습니다. 앱 구성 요소가 한 위치에서 업그레이드된 후 다른 위치에서 업그레이드되는 것을 수용할 수 있는지 아니면 다른 도메인 구성이 필요한지 결정해야 합니다. 단일 위치 자체에는 여러 개의 업그레이드 도메인이 포함될 수 있습니다.

**인스턴스 및 가용성을 추적합니다.** 고가용성 앱 구성 요소는 부하 분산 및 동기 데이터 복제를 통해 사용할 수 있습니다. 인스턴스 중 몇 개가 오프라인이 되면 서비스를 중단할 것인지 결정해야 합니다.

**자동 복구를 구현합니다.** 문제가 발생하여 앱의 가용성이 중단될 경우 모니터링 시스템에서 이를 감지하여 실패한 인스턴스를 드레이닝하고 다시 배포하는 등의 자동 복구 작업을 시작할 수 있습니다. 이렇게 하려면 하이브리드 CI/CD(연속 통합/지속적인 업데이트) 파이프라인과 통합된 중앙 모니터링 솔루션이 필요할 가능성이 높습니다. 앱이 모니터링 시스템과 통합되어 앱 구성 요소를 다시 배포해야 할 수 있는 문제를 식별합니다. 또한 모니터링 시스템은 하이브리드 CI/CD를 트리거하여 앱 구성 요소 및 잠재적인 다른 종속 구성 요소를 동일한 위치 또는 다른 위치에 다시 배포할 수 있습니다.

**SLA(서비스 수준 계약)를 유지 관리합니다.** 가용성은 고객과 맺은 서비스 및 앱 연결을 유지하는 계약에 있어서 매우 중요합니다. 하이브리드 앱이 의존하는 각 위치에 고유한 SLA가 있을 수 있습니다. 이러한 여러 SLA는 하이브리드 앱의 전반적인 SLA에 영향을 줄 수 있습니다.

## <a name="resiliency"></a>복원력

복원력은 하이브리드 앱 및 시스템이 오류를 복구하여 계속 작동하는 기능입니다. 복원력의 목표는 오류가 발생한 후 앱을 완전히 작동하는 상태로 되돌리는 것입니다. 복원력 전략에는 백업, 복제 및 재해 복구와 같은 솔루션이 포함됩니다.

이 핵심 요소에 대한 자세한 내용은 뛰어난 아키텍처의 5가지 핵심 요소에서 [*복원력*](/azure/architecture/guide/pillars#resiliency)을 참조하세요.

### <a name="resiliency-checklist"></a>복원력 검사 목록

**재해 복구 종속 구성 요소를 찾습니다.** 한 클라우드에서 재해 복구를 수행하려면 다른 클라우드의 앱 구성 요소를 변경해야 할 수 있습니다. 한 클라우드의 하나 또는 여러 구성 요소가 동일한 클라우드의 다른 위치 또는 다른 클라우드로 장애 조치(failover)되는 경우 종속 구성 요소에 이러한 변경 내용을 알려야 합니다. 여기에는 연결 종속성도 포함됩니다. 복원력을 확보하려면 클라우드마다 완전한 테스트를 마친 앱 복구 계획이 필요합니다.

**복구 흐름을 설정합니다.** 효과적인 복구 흐름 디자인은 버퍼, 다시 시도, 실패한 데이터 전송 재시도 그리고 필요한 경우 다른 서비스나 워크플로로 대체하는 것을 수용하는 앱 구성 요소의 기능을 평가했습니다. 사용할 백업 메커니즘, 복원 절차와 관련된 내용 및 테스트 빈도를 결정해야 합니다. 또한 증분 백업과 전체 백업의 빈도를 결정해야 합니다.

**부분 복구를 테스트합니다.** 앱의 일부를 부분 복구하면 앱 전체가 중단된 것은 아니라고 사용자를 안심시킬 수 있습니다. 계획의 이 부분에서는 부분 복구를 사용해도 앱과 상호 작용하는 백업 및 복원 서비스가 백업 전에 정상적으로 종료되는 등 부작용이 없게 해야 합니다.

**재해 복구 실시자를 결정하고 책임을 할당합니다.** 복구 계획에서는 백업 및 복원할 수 있는 항목 외에도 백업 및 복구 작업을 시작할 수 있는 사람과 역할을 설명해야 합니다.

**자동 복구 임계값과 재해 복구를 비교합니다.** 자동 복구 시작을 위한 앱의 자동 복구 기능과 앱의 자동 복구 기능이 실패 또는 성공한 것으로 간주하는 데 필요한 시간을 결정합니다. 각 클라우드의 임계값을 결정합니다.

**복원력 기능의 가용성을 확인합니다.** 각 위치의 복원력 기능 가용성을 확인합니다. 특정 위치에서 필요한 기능을 제공하지 않는 경우 복원력 기능을 제공하는 중앙 집중식 서비스에 해당 위치를 통합할 수 있습니다.

**가동 중지 시간을 확인합니다.** 앱 전체와 앱 구성 요소의 유지 관리로 인해 예상되는 가동 중지 시간을 확인합니다.

**문제 해결 절차를 문서화합니다.** 리소스 및 앱 구성 요소를 다시 배포하기 위한 문제 해결 절차를 정의합니다.

## <a name="manageability"></a>관리 효율

하이브리드 앱을 관리하는 방법에 대한 고려 사항은 아키텍처를 디자인할 때 매우 중요합니다. 잘 관리되는 하이브리드 앱은 일관적인 앱 코드를 공통 개발 파이프라인에 통합할 수 있는 인프라를 코드로 제공합니다. 인프라 변경 내용을 시스템 및 개별 수준에서 일관적으로 테스트할 수 있는 기능을 구현하면 변경 내용이 테스트를 통과할 경우 통합 배포를 제공하여 소스 코드에 병합할 수 있습니다.

이 핵심 요소에 대한 자세한 내용은 뛰어난 아키텍처의 5가지 핵심 요소에서 [*DevOps*](/azure/architecture/framework/#devops)를 참조하세요.

### <a name="manageability-checklist"></a>관리 효율성 검사 목록

**모니터링을 구현합니다.** 클라우드에 분산된 앱 구성 요소를 중앙에서 모니터링하는 시스템을 사용하여 집계된 상태 및 성능 보기를 제공합니다. 이 시스템은 앱 구성 요소와 관련 플랫폼 기능을 모두 모니터링해야 합니다.

앱에서 모니터링이 필요한 부분을 확인합니다.

**정책을 조정합니다.** 하이브리드 앱의 범위에 포함되는 위치마다 허용되는 리소스 종류, 명명 규칙, 태그 및 기타 조건을 관리하는 자체 정책이 있을 수 있습니다.

**역할을 정의하고 사용합니다.** 데이터베이스 관리자는 앱 리소스에 액세스해야 하는 여러 가상 사용자(예: 앱 소유자, 데이터베이스 관리자, 최종 사용자)에게 필요한 권한을 결정해야 합니다. 이러한 권한은 리소스 및 앱 내에서 구성해야 합니다. RBAC(역할 기반 액세스 제어) 시스템을 사용하면 앱 리소스에 대해 이러한 권한을 설정할 수 있습니다. 이러한 액세스 권한은 모든 리소스가 단일 클라우드에 배포될 때에도 관리가 어렵지만 리소스가 클라우드에 분산되는 경우에는 더욱 세심한 주의가 필요합니다. 한 클라우드에서 설정된 리소스 권한은 다른 클라우드에 설정된 리소스에 적용되지 않습니다.

**CI/CD 파이프라인을 사용합니다.** CI/CD(연속 통합 및 지속적인 개발) 파이프라인은 여러 클라우드에 걸쳐 있는 앱을 작성 및 배포하고 인프라 및 앱의 품질을 보증하는 일관적인 프로세스를 제공할 수 있습니다. 이 파이프라인을 사용하면 인프라와 앱을 한 클라우드에서 테스트하고 다른 클라우드에 배포할 수 있습니다. 뿐만 아니라 이 파이프라인을 사용하면 하이브리드 앱의 특정 구성 요소를 한 클라우드에 배포하고 다른 구성 요소를 다른 클라우드에 배포하여 근본적으로 하이브리드 앱 배포를 위한 기반을 구축할 수 있습니다. CI/CD 시스템은 설치하는 동안 앱 구성 요소 간의 종속성을 처리하는 데 필요합니다. 예를 들어 웹앱은 데이터베이스에 대한 연결 문자열이 필요합니다.

**수명 주기를 관리합니다.** 하이브리드 앱의 리소스는 여러 위치로 확장할 수 있으므로 각 단일 위치의 수명 주기 관리 기능을 단일 수명 주기 관리 단위로 집계해야 합니다. 리소스를 만들고, 업데이트하고, 삭제하는 방법을 고려합니다.

**문제 해결 전략을 검사합니다.** 하이브리드 앱의 문제를 해결하려면 단일 클라우드에서 실행되는 동일한 앱보다 많은 앱 구성 요소가 필요합니다. 클라우드 간 연결 외에도, 앱이 하나가 아닌 두 개의 플랫폼에서 실행됩니다. 하이브리드 앱 문제를 해결하는 중요한 작업 중 하나는 집계된 앱 구성 요소의 상태 및 성능 모니터링을 검사하는 것입니다.

## <a name="security"></a>보안

보안은 모든 클라우드 앱의 주요 고려 사항 중 하나이며 하이브리드 클라우드 앱에서는 보안이 더욱 중요합니다.

이 핵심 요소에 대한 자세한 내용은 뛰어난 아키텍처의 5가지 핵심 요소에서 [*보안*](/azure/architecture/guide/pillars#security)을 참조하세요.

### <a name="security-checklist"></a>보안 검사 목록

**위반을 가정합니다.** 앱의 한 부분이 손상될 경우 동일한 위치뿐 아니라 다른 위치로 위반이 확산되는 것을 최소화하는 솔루션을 구비해야 합니다.

**허용되는 네트워크 액세스를 모니터링합니다.** 앱의 네트워크 액세스 정책을 결정하고(예: 특정 서브넷에서만 앱에 액세스 가능), 구성 요소 사이에 앱이 제대로 작동하는 데 필요한 최소한의 포트와 프로토콜만 허용합니다.

**강력한 인증을 사용합니다.** 강력한 인증 체계는 앱의 보안에 매우 중요합니다. Single Sign-On 기능을 제공하고 사용자 이름 및 암호 로그온, 공개 및 프라이빗 키, 2단계 또는 다단계 인증, 신뢰할 수 있는 보안 그룹 중 하나 이상을 사용하는 페더레이션된 ID 공급자를 사용하는 것이 좋습니다. 인증서 유형 및 요구 사항 외에도 앱 인증을 위한 중요한 데이터와 기타 비밀을 저장하는 데 적합한 리소스를 결정합니다.

**암호화를 사용합니다.** 데이터 스토리지 또는 클라이언트 통신과 액세스처럼 앱에서 암호화를 사용하는 영역을 파악합니다.

**보안 채널을 사용합니다.** 클라우드의 보안 채널은 보안 및 인증 검사, 실시간 보호, 격리, 클라우드에서 기타 서비스를 제공하는 데 중요합니다.

**역할을 정의하고 사용합니다.** 리소스 구성에 대한 역할과 모든 클라우드에서 통용되는 단일 ID 액세스를 구현합니다. 앱 및 해당 플랫폼 리소스의 RBAC(역할 기반 액세스 제어) 요구 사항을 확인합니다.

**시스템을 감사합니다.** 시스템 모니터링은 앱 구성 요소 및 관련 클라우드 플랫폼 작업의 데이터를 모두 기록하고 집계할 수 있습니다.

## <a name="summary"></a>요약

이 문서에서는 하이브리드 앱을 작성하고 디자인하는 동안 고려해야 하는 항목의 검사 목록을 제공합니다. 앱을 배포하기 전에 이러한 핵심 요소를 검토하면 이러한 문제로 인해 프로덕션 환경이 중단되고 디자인을 다시 검토해야 하는 사태를 미연에 방지할 수 있습니다.

처음에는 시간이 오래 걸리는 작업처럼 보일 수 있지만, 이러한 핵심 요소를 기반으로 앱을 디자인하면 투자 수익률을 쉽게 달성할 수 있습니다.

## <a name="next-steps"></a>다음 단계

자세한 내용은 다음 자료를 참조하세요.

- [하이브리드 클라우드](https://azure.microsoft.com/overview/hybrid-cloud/)
- [하이브리드 클라우드 앱](https://azure.microsoft.com/solutions/hybrid-cloud-app/)
- [클라우드 일관성을 위한 Azure Resource Manager 템플릿 개발](/azure/azure-resource-manager/templates/templates-cloud-consistency)