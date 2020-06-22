---
title: Azure 및 Azure Stack Hub를 사용 하는 Footfall 검색 패턴
description: Azure 및 Azure Stack Hub를 사용 하 여 소매 매장 트래픽을 분석 하는 AI 기반 footfall 검색 솔루션을 구현 하는 방법을 알아봅니다.
author: BryanLa
ms.topic: article
ms.date: 10/31/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 10/31/2019
ms.openlocfilehash: 0bf07bb38537f530a0adb3569c43d53af13b8d56
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: ko-KR
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911225"
---
# <a name="footfall-detection-pattern"></a><span data-ttu-id="6bdaa-103">Footfall 감지 패턴</span><span class="sxs-lookup"><span data-stu-id="6bdaa-103">Footfall detection pattern</span></span>

<span data-ttu-id="6bdaa-104">이 패턴은 소매 상점에서 방문자 트래픽을 분석 하기 위한 AI 기반 footfall 검색 솔루션을 구현 하는 데 대 한 개요를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-104">This pattern provides an overview for implementing an AI-based footfall detection solution for analyzing visitor traffic in retail stores.</span></span> <span data-ttu-id="6bdaa-105">이 솔루션은 Azure, Azure Stack Hub 및 Custom Vision AI Dev Kit를 사용 하 여 실제 작업에서 통찰력을 생성 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-105">The solution generates insights from real world actions, using Azure, Azure Stack Hub, and the Custom Vision AI Dev Kit.</span></span>

## <a name="context-and-problem"></a><span data-ttu-id="6bdaa-106">컨텍스트 및 문제점</span><span class="sxs-lookup"><span data-stu-id="6bdaa-106">Context and problem</span></span>

<span data-ttu-id="6bdaa-107">Contoso store는 고객이 현재 제품을 받는 방법에 대 한 정보를 상점 레이아웃에 대 한 정보를 얻고 싶습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-107">Contoso Stores would like to gain insights on how customers are receiving their current products in relation to store layout.</span></span> <span data-ttu-id="6bdaa-108">모든 섹션에 직원을 배치 하는 것은 할 수 없으며 분석가 팀이 전체 상점의 카메라 푸티지를 검토 하는 것은 비효율적입니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-108">They're unable to place staff in every section and it's inefficient to have a team of analysts review an entire store's camera footage.</span></span> <span data-ttu-id="6bdaa-109">또한 분석을 위해 모든 카메라에서 클라우드로 비디오를 스트림 하는 데 충분 한 대역폭이 없는 저장소도 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-109">In addition, none of their stores have enough bandwidth to stream video from all their cameras to the cloud for analysis.</span></span>

<span data-ttu-id="6bdaa-110">Contoso는 표시 및 제품을 저장 하는 고객의 인구 통계, 충성도 및 반응을 확인 하는 데 있어 사용자의 정보를 파악 하기 쉬운 방식으로 찾고 싶습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-110">Contoso would like to find an unobtrusive, privacy-friendly way to determine their customers' demographics, loyalty, and reactions to store displays and products.</span></span>

## <a name="solution"></a><span data-ttu-id="6bdaa-111">해결 방법</span><span class="sxs-lookup"><span data-stu-id="6bdaa-111">Solution</span></span>

<span data-ttu-id="6bdaa-112">이 소매점 분석 패턴은 계층화 된 접근 방식을 사용 하 여에 지를 추론 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-112">This retail analytics pattern uses a tiered approach to inferencing at the edge.</span></span> <span data-ttu-id="6bdaa-113">Custom Vision AI Dev Kit를 사용 하 여 인간 얼굴의 이미지만 Azure Cognitive Services를 실행 하는 개인 Azure Stack 허브로 전송 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-113">By using the Custom Vision AI Dev Kit, only images with human faces are sent for analysis to a private Azure Stack Hub that runs Azure Cognitive Services.</span></span> <span data-ttu-id="6bdaa-114">익명화 집계 된 데이터는 Power BI의 모든 매장 및 시각화에서 집계를 위해 Azure에 전송 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-114">Anonymized, aggregated data is sent to Azure for aggregation across all stores and visualization in Power BI.</span></span> <span data-ttu-id="6bdaa-115">Edge와 공용 클라우드를 결합 하면 Contoso에서 최신 AI 기술을 활용할 수 있을 뿐만 아니라 회사 정책에 부합 하 고 고객의 개인 정보를 존중 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-115">Combining the edge and public cloud lets Contoso take advantage of modern AI technology while also remaining in compliance with their corporate policies and respecting their customers' privacy.</span></span>

<span data-ttu-id="6bdaa-116">[![Footfall 검색 패턴 솔루션](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span><span class="sxs-lookup"><span data-stu-id="6bdaa-116">[![Footfall detection pattern solution](media/pattern-retail-footfall-detection/solution-architecture.png)](media/pattern-retail-footfall-detection/solution-architecture.png)</span></span>

<span data-ttu-id="6bdaa-117">다음은 솔루션의 작동 방식에 대 한 요약입니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-117">Here's a summary of how the solution works:</span></span>

1. <span data-ttu-id="6bdaa-118">Custom Vision AI Dev Kit는 IoT Edge 런타임과 ML 모델을 설치 하는 IoT Hub에서 구성을 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-118">The Custom Vision AI Dev Kit gets a configuration from IoT Hub, which installs the IoT Edge Runtime and an ML model.</span></span>
2. <span data-ttu-id="6bdaa-119">모델에서 사람을 볼 경우 그림을 사용 하 여 Azure Stack Hub blob 저장소에 업로드 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-119">If the model sees a person, it takes a picture and uploads it to Azure Stack Hub blob storage.</span></span>
3. <span data-ttu-id="6bdaa-120">Blob service는 Azure Stack 허브에서 Azure 함수를 트리거합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-120">The blob service triggers an Azure Function on Azure Stack Hub.</span></span>
4. <span data-ttu-id="6bdaa-121">Azure 함수는 Face API 있는 컨테이너를 호출 하 여 이미지에서 인구 통계 및 emotion 데이터를 가져옵니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-121">The Azure Function calls a container with the Face API to get demographic and emotion data from the image.</span></span>
5. <span data-ttu-id="6bdaa-122">데이터는 익명화 Azure Event Hubs 클러스터로 전송 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-122">The data is anonymized and sent to an Azure Event Hubs cluster.</span></span>
6. <span data-ttu-id="6bdaa-123">Event Hubs 클러스터는 Stream Analytics 데이터를 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-123">The Event Hubs cluster pushes the data to Stream Analytics.</span></span>
7. <span data-ttu-id="6bdaa-124">Stream Analytics는 데이터를 집계 하 여 Power BI에 푸시합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-124">Stream Analytics aggregates the data and pushes it to Power BI.</span></span>

## <a name="components"></a><span data-ttu-id="6bdaa-125">구성 요소</span><span class="sxs-lookup"><span data-stu-id="6bdaa-125">Components</span></span>

<span data-ttu-id="6bdaa-126">이 솔루션은 다음 구성 요소를 사용 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-126">This solution uses the following components:</span></span>

| <span data-ttu-id="6bdaa-127">계층</span><span class="sxs-lookup"><span data-stu-id="6bdaa-127">Layer</span></span> | <span data-ttu-id="6bdaa-128">구성 요소</span><span class="sxs-lookup"><span data-stu-id="6bdaa-128">Component</span></span> | <span data-ttu-id="6bdaa-129">Description</span><span class="sxs-lookup"><span data-stu-id="6bdaa-129">Description</span></span> |
|----------|-----------|-------------|
| <span data-ttu-id="6bdaa-130">매장 내 하드웨어</span><span class="sxs-lookup"><span data-stu-id="6bdaa-130">In-store hardware</span></span> | [<span data-ttu-id="6bdaa-131">Custom Vision AI Dev Kit</span><span class="sxs-lookup"><span data-stu-id="6bdaa-131">Custom Vision AI Dev Kit</span></span>](https://azure.github.io/Vision-AI-DevKit-Pages/) | <span data-ttu-id="6bdaa-132">는 분석을 위해 사람들의 이미지를 캡처하는 로컬 ML 모델을 사용 하 여 매장 내 필터링을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-132">Provides in-store filtering using a local ML model that only captures images of people for analysis.</span></span> <span data-ttu-id="6bdaa-133">IoT Hub를 통해 안전 하 게 프로 비전 되 고 업데이트 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-133">Securely provisioned and updated through IoT Hub.</span></span><br><br>|
| <span data-ttu-id="6bdaa-134">Azure</span><span class="sxs-lookup"><span data-stu-id="6bdaa-134">Azure</span></span> | [<span data-ttu-id="6bdaa-135">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="6bdaa-135">Azure Event Hubs</span></span>](/azure/event-hubs/) | <span data-ttu-id="6bdaa-136">Azure Event Hubs는 Azure Stream Analytics와 깔끔하게 통합 되는 수집 익명화 데이터를 위한 확장 가능한 플랫폼을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-136">Azure Event Hubs provides a scalable platform for ingesting anonymized data that integrates neatly with Azure Stream Analytics.</span></span> |
|  | [<span data-ttu-id="6bdaa-137">Azure Stream Analytics</span><span class="sxs-lookup"><span data-stu-id="6bdaa-137">Azure Stream Analytics</span></span>](/azure/stream-analytics/) | <span data-ttu-id="6bdaa-138">Azure Stream Analytics 작업은 익명화 데이터를 집계 하 고 시각화를 위해 15 초 windows로 그룹화 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-138">An Azure Stream Analytics job aggregates the anonymized data and groups it into 15-second windows for visualization.</span></span> |
|  | [<span data-ttu-id="6bdaa-139">Microsoft Power BI</span><span class="sxs-lookup"><span data-stu-id="6bdaa-139">Microsoft Power BI</span></span>](https://powerbi.microsoft.com/) | <span data-ttu-id="6bdaa-140">Power BI Azure Stream Analytics의 출력을 볼 수 있는 사용 하기 쉬운 대시보드 인터페이스를 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-140">Power BI provides an easy-to-use dashboard interface for viewing the output from Azure Stream Analytics.</span></span> |
| <span data-ttu-id="6bdaa-141">Azure Stack 허브</span><span class="sxs-lookup"><span data-stu-id="6bdaa-141">Azure Stack Hub</span></span> | [<span data-ttu-id="6bdaa-142">App Service</span><span class="sxs-lookup"><span data-stu-id="6bdaa-142">App Service</span></span>](/azure-stack/operator/azure-stack-app-service-overview.md) | <span data-ttu-id="6bdaa-143">RP (App Service 리소스 공급자)는 웹 앱/a p i 및 함수에 대 한 호스팅 및 관리 기능을 포함 하 여에 지 구성 요소에 대 한 기반을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-143">The App Service resource provider (RP) provides a base for edge components, including hosting and management features for web apps/APIs and Functions.</span></span> |
| | <span data-ttu-id="6bdaa-144">AKS (Azure Kubernetes Service [) 엔진](https://github.com/Azure/aks-engine) 클러스터</span><span class="sxs-lookup"><span data-stu-id="6bdaa-144">Azure Kubernetes Service [(AKS) Engine](https://github.com/Azure/aks-engine) cluster</span></span> | <span data-ttu-id="6bdaa-145">Azure Stack Hub에 배포 된 AKS RP는 Face API 컨테이너를 실행할 수 있는 확장 가능한 복원 력 엔진을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-145">The AKS RP with AKS-Engine cluster deployed into Azure Stack Hub provides a scalable, resilient engine to run the Face API container.</span></span> |
| | <span data-ttu-id="6bdaa-146">Azure Cognitive Services [Face API 컨테이너](/azure/cognitive-services/face/face-how-to-install-containers)</span><span class="sxs-lookup"><span data-stu-id="6bdaa-146">Azure Cognitive Services [Face API containers](/azure/cognitive-services/face/face-how-to-install-containers)</span></span>| <span data-ttu-id="6bdaa-147">Face API 컨테이너가 있는 Azure Cognitive Services RP는 Contoso의 개인 네트워크에 대 한 인구 통계, emotion 및 고유 방문자 검색을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-147">The Azure Cognitive Services RP with Face API containers provides demographic, emotion, and unique visitor detection on Contoso's private network.</span></span> |
| | <span data-ttu-id="6bdaa-148">Blob Storage</span><span class="sxs-lookup"><span data-stu-id="6bdaa-148">Blob Storage</span></span> | <span data-ttu-id="6bdaa-149">AI Dev Kit에서 캡처된 이미지는 Azure Stack 허브의 blob 저장소에 업로드 됩니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-149">Images captured from the AI Dev Kit are uploaded to Azure Stack Hub's blob storage.</span></span> |
| | <span data-ttu-id="6bdaa-150">Azure 기능</span><span class="sxs-lookup"><span data-stu-id="6bdaa-150">Azure Functions</span></span> | <span data-ttu-id="6bdaa-151">Azure Stack Hub에서 실행 되는 Azure 함수는 blob storage에서 입력을 받고 Face API와의 상호 작용을 관리 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-151">An Azure Function running on Azure Stack Hub receives input from blob storage and manages the interactions with the Face API.</span></span> <span data-ttu-id="6bdaa-152">Azure에 있는 Event Hubs 클러스터로 익명화 데이터를 내보냅니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-152">It emits anonymized data to an Event Hubs cluster located in Azure.</span></span><br><br>|

## <a name="issues-and-considerations"></a><span data-ttu-id="6bdaa-153">문제 및 고려 사항</span><span class="sxs-lookup"><span data-stu-id="6bdaa-153">Issues and considerations</span></span>

<span data-ttu-id="6bdaa-154">이 솔루션을 구현 하는 방법을 결정할 때 다음 사항을 고려 하십시오.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-154">Consider the following points when deciding how to implement this solution:</span></span>

### <a name="scalability"></a><span data-ttu-id="6bdaa-155">확장성</span><span class="sxs-lookup"><span data-stu-id="6bdaa-155">Scalability</span></span>

<span data-ttu-id="6bdaa-156">이 솔루션을 여러 카메라 및 위치에서 확장할 수 있도록 하려면 모든 구성 요소에서 증가 된 부하를 처리할 수 있는지 확인 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-156">To enable this solution to scale across multiple cameras and locations, you'll need to make sure that all of the components can handle the increased load.</span></span> <span data-ttu-id="6bdaa-157">다음과 같은 작업을 수행 해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-157">You may need to take actions like:</span></span>

- <span data-ttu-id="6bdaa-158">Stream Analytics 스트리밍 단위의 수를 늘립니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-158">Increase the number of Stream Analytics streaming units.</span></span>
- <span data-ttu-id="6bdaa-159">Face API 배포를 확장 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-159">Scale out the Face API deployment.</span></span>
- <span data-ttu-id="6bdaa-160">Event Hubs 클러스터 처리량을 늘립니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-160">Increase the Event Hubs cluster throughput.</span></span>
- <span data-ttu-id="6bdaa-161">극단적인 경우에는 Azure Functions에서 가상 머신으로 마이그레이션해야 할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-161">For extreme cases, migrate from Azure Functions to a virtual machine may be necessary.</span></span>

### <a name="availability"></a><span data-ttu-id="6bdaa-162">가용성</span><span class="sxs-lookup"><span data-stu-id="6bdaa-162">Availability</span></span>

<span data-ttu-id="6bdaa-163">이 솔루션은 계층화 되므로 네트워킹 또는 전원 오류를 처리 하는 방법을 고려 하는 것이 중요 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-163">Since this solution is tiered, it's important to think about how to deal with networking or power failures.</span></span> <span data-ttu-id="6bdaa-164">비즈니스 요구 사항에 따라 이미지를 로컬로 캐시 한 다음 연결이 반환 될 때 Azure Stack 허브로 전달 하는 메커니즘을 구현할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-164">Depending on business needs, you might want to implement a mechanism to cache images locally, then forward to Azure Stack Hub when connectivity returns.</span></span> <span data-ttu-id="6bdaa-165">위치가 충분히 크면 Face API 컨테이너를 사용 하 여 Data Box Edge를 해당 위치에 배포 하는 것이 더 나을 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-165">If the location is large enough, deploying a Data Box Edge with the Face API container to that location might be a better option.</span></span>

### <a name="manageability"></a><span data-ttu-id="6bdaa-166">관리 효율</span><span class="sxs-lookup"><span data-stu-id="6bdaa-166">Manageability</span></span>

<span data-ttu-id="6bdaa-167">이 솔루션은 많은 장치 및 위치에 걸쳐 있을 수 있으므로 다루기가 어려울 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-167">This solution can span many devices and locations, which could get unwieldy.</span></span> <span data-ttu-id="6bdaa-168">[Azure의 IoT 서비스](/azure/iot-fundamentals/) 를 사용 하 여 자동으로 새 위치 및 장치를 온라인 상태로 전환 하 고 최신 상태로 유지할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-168">[Azure's IoT services](/azure/iot-fundamentals/) can be used to automatically bring new locations and devices online and keep them up to date.</span></span>

### <a name="security"></a><span data-ttu-id="6bdaa-169">보안</span><span class="sxs-lookup"><span data-stu-id="6bdaa-169">Security</span></span>

<span data-ttu-id="6bdaa-170">이 솔루션은 고객 이미지를 캡처하여 보안을 가장 중요 하 게 고려 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-170">This solution captures customer images, making security a paramount consideration.</span></span> <span data-ttu-id="6bdaa-171">모든 저장소 계정에 적절 한 액세스 정책을 사용 하 여 보안을 설정 하 고 키를 정기적으로 회전 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-171">Make sure all storage accounts are secured with the proper access policies and rotate keys regularly.</span></span> <span data-ttu-id="6bdaa-172">저장소 계정 및 Event Hubs 회사 및 정부 개인 정보 규정을 충족 하는 보존 정책을 보유 하 고 있는지 확인 하세요.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-172">Ensure storage accounts and Event Hubs have retention policies that meet corporate and government privacy regulations.</span></span> <span data-ttu-id="6bdaa-173">또한 사용자 액세스 수준도 계층으로 지정 해야 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-173">Also make sure to tier the user access levels.</span></span> <span data-ttu-id="6bdaa-174">계층화를 사용 하면 사용자가 자신의 역할에 필요한 데이터에만 액세스할 수 있습니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-174">Tiering ensures that users only have access to the data they need for their role.</span></span>

## <a name="next-steps"></a><span data-ttu-id="6bdaa-175">다음 단계</span><span class="sxs-lookup"><span data-stu-id="6bdaa-175">Next steps</span></span>

<span data-ttu-id="6bdaa-176">이 문서에서 소개 하는 항목에 대 한 자세한 내용은 다음을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-176">To learn more about the topics introduced in this article:</span></span>

- <span data-ttu-id="6bdaa-177">Footfall 검색 패턴에서 활용 하는 [계층화 된 데이터 패턴](https://aka.ms/tiereddatadeploy)을 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-177">See the [Tiered Data pattern](https://aka.ms/tiereddatadeploy), which is leveraged by the footfall detection pattern.</span></span>
- <span data-ttu-id="6bdaa-178">사용자 지정 비전을 사용 하는 방법에 대해 자세히 알아보려면 [CUSTOM VISION AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) 를 참조 하세요.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-178">See the [Custom Vision AI Dev Kit](https://azure.github.io/Vision-AI-DevKit-Pages/) to learn more about using custom vision.</span></span> 

<span data-ttu-id="6bdaa-179">솔루션 예제를 테스트할 준비가 되 면 [Footfall 검색 배포 가이드](solution-deployment-guide-retail-footfall-detection.md)를 계속 진행 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-179">When you're ready to test the solution example, continue with the [Footfall detection deployment guide](solution-deployment-guide-retail-footfall-detection.md).</span></span> <span data-ttu-id="6bdaa-180">배포 가이드에서는 구성 요소를 배포 및 테스트 하는 방법에 대 한 단계별 지침을 제공 합니다.</span><span class="sxs-lookup"><span data-stu-id="6bdaa-180">The deployment guide provides step-by-step instructions for deploying and testing its components.</span></span>