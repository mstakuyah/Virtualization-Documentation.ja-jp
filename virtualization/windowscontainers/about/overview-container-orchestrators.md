---
title: Windows コンテナーオーケストレーションの概要
description: Windows コンテナーオーケストレーションについて説明します。
keywords: Docker, コンテナー
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 99a3b47a9d80e21c246fb3b4f61d650557eb37fa
ms.sourcegitcommit: e9dda81f1f68359ece9ef132a184a30880bcdb1b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/27/2019
ms.locfileid: "10161789"
---
# <a name="windows-container-orchestration-overview"></a>Windows コンテナーオーケストレーションの概要

サイズとアプリケーションの向きが小さいため、コンテナーは、アジャイル配信環境と microservice ベースアーキテクチャに最適です。 ただし、コンテナーと microservers を使う環境では、数百または数千のコンポーネントを追跡することができます。 数十の仮想マシンまたは物理サーバーを手動で管理することもできますが、自動化されていない実稼働規模のコンテナー環境を適切に管理する方法はありません。 このタスクは、多数のコンテナーを自動化して管理し、互いの相互作用をどのように処理するかを示すプロセスである、orchestrator に分類されます。

オーケストレーションは、次のタスクを実行します。

- スケジュール: コンテナーイメージとリソース要求を指定すると、orchestrator はコンテナーを実行する適切なコンピューターを見つけます。
- アフィニティ/許可-アフィニティ: 一連のコンテナーがパフォーマンスを向上させるか、または可用性を確保するために互いに離れている必要があるかを指定します。
- 正常性の監視: コンテナー エラーを監視し、自動的にスケジュールを再調整します。
- フェールオーバー: 各コンピューターで何が実行されているかを追跡し、障害が発生したコンピューターから正常なノードにコンテナーのスケジュールを変更します。
- スケーリング: 要求に合わせて手動または自動でコンテナーインスタンスを追加または削除します。
- ネットワーク: 複数のホストコンピューター間で通信するためにコンテナーを調整するオーバーレイネットワークを提供します。
- サービスの検出: コンテナーがホスト コンピューター間で移動され、IP アドレスが変更された場合でも、自動的に互いを見つけることができるようにします。
- アプリケーション アップグレード調整: アプリケーションのダウンタイムを回避し、問題が発生した場合はロールバックできるように、コンテナーのアップグレードを管理します。

## <a name="orchestrator-types"></a>Orchestrator の種類

Azure には、Azure Kubernetes Service (AKS) と Service Fabric という2つのコンテナーオーケストレーションが用意されています。

[Azure Kubernetes Service (AKS)](/azure/aks/)は、コンテナー化されたアプリケーションを実行するように事前に構成された仮想マシンのクラスターの作成、構成、管理を簡単に行うことができます。 これにより、既存のスキルを使用して、Microsoft Azure のコンテナーベースのアプリケーションを展開して管理するために、コミュニティの専門知識の大規模と成長を促すことができます。 AKS を使用することで、Azure のエンタープライズグレードの機能を利用しながら、Kubernetes と Docker のイメージ形式を使用して、アプリケーションの移植性を維持できます。

[Azure Service Fabric](/azure/service-fabric/) は、スケーラブルで信頼性の高いマイクロサービスとコンテナーを容易にパッケージ化、展開、管理できる分散システム プラットフォームです。 Service Fabric によって、クラウド ネイティブ アプリケーションを開発、管理する際の重要な問題を解決します。 開発者や管理者は、複雑なインフラストラクチャの問題を回避し、ミッション クリティカルで要求の厳しいワークロードを、スケーラブルで、信頼性が高く、管理しやすい方法で実装することに集中できます。 Service Fabric は、このようなエンタープライズ クラス、階層 1、クラウド規模のアプリケーションを構築および管理するための次世代プラットフォームです。

## <a name="getting-started"></a>開始するには

Azure Kubernetes サービスの展開を開始するには、「 [Kubernetes のセットアップガイド](../kubernetes/getting-started-kubernetes-windows.md)」を参照してください。

Azure Service Fabric の展開を開始するには、 [Service fabric のクイックスタート](/azure/service-fabric/service-fabric-quickstart-containers.md)を参照してください。