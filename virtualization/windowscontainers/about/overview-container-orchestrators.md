---
title: Windows コンテナーオーケストレーションの概要
description: Windows コンテナーのオーケストレーションについて説明します。
keywords: Docker, コンテナー
author: Heidilohr
ms.author: helohr
ms.date: 05/22/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 23dd1e56ba68a679945779f5e7dbc15225412934
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853908"
---
# <a name="windows-container-orchestration-overview"></a>Windows コンテナーオーケストレーションの概要

サイズとアプリケーションの向きが小さいため、コンテナーはアジャイル配信環境とマイクロサービスベースのアーキテクチャに最適です。 ただし、コンテナーとマイクロサービスを使用する環境では、数百または数千のコンポーネントを追跡することができます。 数十の仮想マシンまたは物理サーバーを手動で管理することもできますが、運用環境のコンテナー環境を自動化せずに適切に管理する方法はありません。 このタスクは orchestrator に分類される必要があります。これは、多数のコンテナーを自動化および管理し、相互に対話する方法を行うプロセスです。

Orchestrators、次のタスクを実行します。

- スケジュール設定: コンテナーイメージとリソース要求を指定すると、orchestrator はコンテナーを実行するのに適したコンピューターを検索します。
- アフィニティ/アンチアフィニティ: 一連のコンテナーがパフォーマンスに関して、または可用性に関して遠く離れた場所で実行する必要があるかどうかを指定します。
- 正常性の監視: コンテナー エラーを監視し、自動的にスケジュールを再調整します。
- フェールオーバー: 各マシンで何が実行されているかを追跡し、障害が発生したマシンから正常なノードにコンテナーを再スケジュールします。
- スケーリング: 要求に合わせて、手動または自動でコンテナーインスタンスを追加または削除します。
- ネットワーク: 複数のホストコンピューター間で通信するようにコンテナーを調整するオーバーレイネットワークを提供します。
- サービスの検出: コンテナーがホスト コンピューター間で移動され、IP アドレスが変更された場合でも、自動的に互いを見つけることができるようにします。
- アプリケーション アップグレード調整: アプリケーションのダウンタイムを回避し、問題が発生した場合はロールバックできるように、コンテナーのアップグレードを管理します。

## <a name="orchestrator-types"></a>Orchestrator の種類

Azure には、Azure Kubernetes Service (AKS) と Service Fabric という2つのコンテナー orchestrators 用意されています。

[Azure Kubernetes Service (AKS)](/azure/aks/)を使用すると、コンテナー化されたアプリケーションを実行するように事前構成された仮想マシンのクラスターを簡単に作成、構成、管理できます。 これにより、既存のスキルを活用し、大規模で成長を続けるコミュニティの専門知識を身につけて、Microsoft Azure にコンテナーベースのアプリケーションをデプロイして管理することができます。 AKS を使用すると、Kubernetes と Docker イメージ形式によってアプリケーションの移植性を維持しながら、エンタープライズレベルの Azure の機能を利用できます。

[Azure Service Fabric](/azure/service-fabric/) は、スケーラブルで信頼性の高いマイクロサービスとコンテナーを容易にパッケージ化、展開、管理できる分散システム プラットフォームです。 Service Fabric によって、クラウド ネイティブ アプリケーションを開発、管理する際の重要な問題を解決します。 開発者や管理者は、複雑なインフラストラクチャの問題を回避し、ミッション クリティカルで要求の厳しいワークロードを、スケーラブルで、信頼性が高く、管理しやすい方法で実装することに集中できます。 Service Fabric は、このようなエンタープライズ クラス、階層 1、クラウド規模のアプリケーションを構築および管理するための次世代プラットフォームです。

## <a name="getting-started"></a>作業の開始

Azure Kubernetes service のデプロイを開始するには、 [Kubernetes セットアップガイド](../kubernetes/getting-started-kubernetes-windows.md)を参照してください。

Azure Service Fabric のデプロイを開始するには、 [Service Fabric のクイックスタート](/azure/service-fabric/service-fabric-quickstart-containers.md)を参照してください。
