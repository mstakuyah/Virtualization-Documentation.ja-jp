---
title: Windows で使用する Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 02/09/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes v1.13 を使用するには、Windows ノードを結合できます。
keywords: kubernetes、1.13、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 7c3a0111b3d19ae1b513a84665f870bba24ae33d
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576989"
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes

このページでは、Linux ベースのクラスターを Windows ノードを結合して Kubernetes Windows で使用を開始する概要についてとして機能します。 Windows Server[バージョン 1809](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1809#container-networking-with-kubernetes)Kubernetes 1.14 のリリースでユーザーを利用できます、次の機能の Kubernetes で windows。

- **ネットワークを重ねて表示**: 仮想オーバーレイ ネットワークを構成するのには、vxlan モードで Flannel を使用します。
    - インストールされている[KB4489899](https://support.microsoft.com/en-us/help/4489899)または[Windows Server vNext 内部 Preview](https://blogs.windows.com/windowsexperience/tag/windows-insider-program/)ビルド 18317 + いずれかの Windows Server 2019 が必要です。
    - Kubernetes v1.14 が必要です (以上) と`WinOverlay`機能ゲートを有効になっています。
    - Flannel v0.11.0 が必要です (以上)
- **簡素化されたネットワークの管理**: Flannel ホスト ゲートウェイ モードで管理を使用する自動ルーティング ノード間です。
- **拡張性の向上**: [Windows Server コンテナーのデバイスを使わない Vnic](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)該当高速化と信頼性の高いコンテナー起動にかかる時間を享受できます。
- **HYPER-V 分離 (アルファ)**: セキュリティを強化するための訳分離[HYPER-V 分離](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)を調整します。 詳細については、 [Windows コンテナーの種類](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types)。
    - Kubernetes v1.10 が必要です (以上) と`HyperVContainer`機能ゲートを有効にします。
- **記憶域プラグイン**: Windows コンテナーの中小企業や iSCSI サポート[FlexVolume 記憶域のプラグイン](https://github.com/Microsoft/K8s-Storage-Plugins)を使用します。

>[!TIP]
>Azure 上のクラスターを展開する場合は、ファイルを開く AKS エンジン ツールでこの簡単できます。 詳細については、このステップ バイ ステップの[チュートリアル](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)を参照してください。

## <a name="prerequisites"></a>前提条件

### <a name="plan-ip-addressing-for-your-cluster"></a>自分のクラスターの IP アドレスの指定を計画します。

<a name="definitions"></a>Kubernetes クラスターでは、新しいサブネット ポッドやサービスを導入、環境に他の既存のネットワークと競合していることを確認するのには重要です。 Kubernetes を正常に展開するためを解放する必要があるすべてのアドレス スペースを紹介します。

| サブネット/アドレス範囲 | 説明 | 既定値 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**サービス サブネット** | ポッドで uniformally かけるネットワーク トポロジはせずにサービスへのアクセスに使用されるルーティング不可能な仮想純サブネットします。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。 | 「10.96.0.0/12」 |
| <a name="cluster-subnet-def"></a>**クラスター サブネット** |  これは、グローバル サブネット クラスター内のすべてのポッドで使用されています。 各ノードの小さい/24 が割り当てられているを使用するには、そのポッドに対してこのからサブネットします。 自分のクラスターで使用されているすべてのポッドに対応するために十分な大きさがある場合があります。 *最小*サブネットのサイズを計算するには。 `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>ノードごとに 100 ポッドの 5 つのノードの例:`(5) + (5 *  100) = 505`します。  | 「10.244.0.0/16」 |
| **Kubernetes DNS サービス IP** | DNS 解決 & クラスター サービス検索用に使用される"kube dns"サービスの IP アドレス。 | 「10.96.0.10」 |

> [!NOTE]
> Docker のインストール時に既定で作成される他の Docker ネットワーク (NAT) があります。 代わりにクラスター サブネットの ip アドレスを割り当てることと、windows Kubernetes を操作するには必要ありません。

## <a name="what-you-will-accomplish"></a>作業内容

このガイドでは、次のことを行います。

> [!div class="checklist"]
> * [Kubernetes マスター](./creating-a-linux-master.md)ノードを作成します。  
> * [ネットワーク ソリューション](./network-topologies.md)を選択します。  
> * [Windows ワーカー ノード](./joining-windows-workers.md)または[Linux ワーカー ノード](./joining-linux-workers.md)を結合します。  
> * [サンプル Kubernetes リソース](./deploying-resources.md)を展開します。  
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。

## <a name="next-steps"></a>次のステップ

このセクションで話し前提条件の重要な & の前提条件が正常に現在 windows Kubernetes を展開するために必要です。 引き続き Kubernetes マスター シェイプをセットアップする方法について説明します。

>[!div class="nextstepaction"]
>[Kubernetes マスター シェイプを作成します。](./creating-a-linux-master.md)