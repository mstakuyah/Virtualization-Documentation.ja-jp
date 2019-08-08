---
title: Windows で使用する Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Windows ノードを Kubernetes クラスターに v 1.14 で参加する。
keywords: kubernetes、1.14、windows、はじめに
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 18734f102042ec951255061dcd82229e18d29a15
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998389"
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes

このページでは、windows ノードを Linux ベースのクラスターに参加させることにより、Windows で Kubernetes を使い始める方法の概要を説明します。 Windows Server[バージョン 1809](https://docs.microsoft.com/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)の Kubernetes 1.14 のリリースでは、ユーザーは Windows の Kubernetes の次の機能を利用できます。

- **オーバーレイネットワーク**: vxlan モードでフランネルを使用して仮想オーバーレイネットワークを構成する
    - Windows Server 2019 で[KB4489899](https://support.microsoft.com/help/4489899)がインストールされているか、 [Windows Server Vnext Insider Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)ビルド 18317 + が必要です。
    - 機能ゲートが有効になって`WinOverlay`いる Kubernetes v 1.14 (以上) が必要です
    - フランネル v 0.11.0 (以上) が必要です
- 簡素化された**ネットワーク管理**: ノード間での自動ルート管理を行うために、フランネルを使用してホストをゲートウェイモードで管理します。
- **スケーラビリティの向上**: [Windows Server コンテナーの deviceless vnics](https://techcommunity.microsoft.com/t5/Networking-Blog/Network-start-up-and-performance-improvements-in-Windows-10/ba-p/339716)により、より高速で信頼性の高いコンテナーをすばやくお楽しみいただけます。
- **Hyper-v 分離 (alpha)**: 強化されたセキュリティを確保するためのカーネルモード分離による[hyper-v 分離](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)の調整。 詳細については、「 [Windows コンテナーの種類](https://docs.microsoft.com/virtualization/windowscontainers/about/#windows-container-types)」を参照してください。
    - 機能ゲートを有効にするに`HyperVContainer`は、Kubernetes v 1.10 (以上) が必要です。
- **ストレージプラグイン**: Windows コンテナー向けの SMB および iSCSI サポートで[flexvolume ストレージプラグイン](https://github.com/Microsoft/K8s-Storage-Plugins)を使用します。

>[!TIP]
>Azure にクラスターを展開する場合は、open source AKS Engine ツールを使用すると簡単です。 詳細については、「ステップバイステップの[チュートリアル](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)」を参照してください。

## <a name="prerequisites"></a>前提条件

### <a name="plan-ip-addressing-for-your-cluster"></a>クラスターの IP アドレス指定を計画する

<a name="definitions"></a>Kubernetes クラスターは、ポッドとサービスの新しいサブネットを導入するため、環境内の他の既存のネットワークと競合しないようにすることが重要です。 Kubernetes を正常に展開するために、解放する必要があるすべてのアドレススペースを次に示します。

| サブネット/アドレス範囲 | 説明 | 既定値 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**サービスサブネット** | ネットワークトポロジについて世話せずに uniformally access services に pod によって使用される、ルーティング不可能な純粋仮想サブネット。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。 | "10.96.0.0/12" |
| <a name="cluster-subnet-def"></a>**クラスターのサブネット** |  これは、クラスター内のすべてのポッドによって使用されるグローバルサブネットです。 各ノードには、pod が使用できるように、この小さいサブネットが割り当てられています。 この値は、クラスターで使用されるすべての pod に対応するのに十分な大きさである必要があります。 *最小*サブネットサイズを計算するには: `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>1ノードあたりの100ポッドの5ノードクラスターの例`(5) + (5 *  100) = 505`:。  | "10.244.0.0/16" |
| **Kubernetes DNS サービス IP** | クラスターサービス検出 & の DNS 解決に使用される "kube-dns" サービスの IP アドレス。 | "10.96.0.10" |

> [!NOTE]
> Docker をインストールすると、既定で作成される、別の Docker ネットワーク (NAT) が存在します。 代わりに、Windows で Kubernetes を操作する必要はありません。代わりに、クラスターサブネットから Ip を割り当てます。

## <a name="what-you-will-accomplish"></a>作業内容

このガイドでは、次のことを行います。

> [!div class="checklist"]
> * [Kubernetes master](./creating-a-linux-master.md)ノードを作成しました。  
> * [ネットワークソリューション](./network-topologies.md)が選択されました。  
> * [Windows ワーカーノード](./joining-windows-workers.md)または[Linux ワーカーノード](./joining-linux-workers.md)に参加しました。  
> * サンプルの[Kubernetes リソース](./deploying-resources.md)を展開した。  
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。

## <a name="next-steps"></a>次のステップ

このセクションでは、今日正常に Windows に Kubernetes を展開するために必要な前提条件 & 前提条件について説明しました。 Kubernetes master のセットアップ方法については、次のページを参照してください。

>[!div class="nextstepaction"]
>[Kubernetes マスターを作成する](./creating-a-linux-master.md)