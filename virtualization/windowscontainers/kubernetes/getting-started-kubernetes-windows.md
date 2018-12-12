---
title: Windows で使用する Kubernetes
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: get-started-article
ms.prod: containers
description: Kubernetes v1.12 を使用するには、Windows ノードを結合できます。
keywords: kubernetes、1.12、windows、作業の開始
ms.assetid: 3b05d2c2-4b9b-42b4-a61b-702df35f5b17
ms.openlocfilehash: 0e43b2ac5b19d16721c1ba0dd1f34e339223bdaf
ms.sourcegitcommit: 8e9252856869135196fd054e3cb417562f851b51
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/08/2018
ms.locfileid: "6178905"
---
# <a name="kubernetes-on-windows"></a>Windows で使用する Kubernetes #
このページでは、Linux ベースのクラスターを Windows ノードを結合して Kubernetes Windows で使用を開始する概要についてとして機能します。 Windows Server の[バージョン 1803](https://docs.microsoft.com/en-us/windows-server/get-started/whats-new-in-windows-server-1803#kubernetes)ベータ版の Kubernetes 1.12 のリリースでユーザーの活用できます[最新機能](https://kubernetes.io/docs/getting-started-guides/windows/#supported-features)の Kubernetes windows:

  - **簡素化されたネットワークの管理**: Flannel ホスト ゲートウェイ モードで管理を使用する自動ルーティング ノード間
  - **拡張性の向上**: [Windows Server コンテナーのデバイスを使わない Vnic](https://blogs.technet.microsoft.com/networking/2018/04/27/network-start-up-and-performance-improvements-in-windows-10-spring-creators-update-and-windows-server-version-1803/)該当高速化と信頼性の高いコンテナー起動にかかる時間を活用します。
  - **ハイパー v 分離 (アルファ)**: ([Windows コンテナーの種類を参照してください](https://docs.microsoft.com/en-us/virtualization/windowscontainers/about/#windows-container-types)) セキュリティを強化するための訳分離[ハイパー v コンテナー](https://kubernetes.io/docs/getting-started-guides/windows/#hyper-v-containers)を調整します。
  - **記憶域プラグイン**: Windows コンテナーの中小企業や iSCSI サポート[FlexVolume 記憶域のプラグイン](https://github.com/Microsoft/K8s-Storage-Plugins)を使用

> [!TIP] 
> Azure でクラスターを展開する場合は、オープン ソースの ACS Engine ツールを使用すると簡単です。 手順を説明した[チュートリアル](https://github.com/Azure/acs-engine/blob/master/docs/kubernetes/windows.md)をご利用ください。

## <a name="prerequisites"></a>前提条件 ##

### <a name="plan-ip-addressing-for-your-cluster"></a>自分のクラスターの IP アドレスの指定を計画します。 ###
<a name="definitions"></a>Kubernetes クラスターでは、新しいサブネット ポッドやサービスを導入、環境に他の既存のネットワークと競合していることを確認するのには重要です。 Kubernetes を正常に展開するためを解放する必要があるすべてのアドレス スペースを紹介します。

| サブネット/アドレス範囲 | 説明 | 既定値 |
| --------- | ------------- | ------------- |
| <a name="service-subnet-def"></a>**サービス サブネット** | ポッドで uniformally かけるネットワーク トポロジはせずにサービスへのアクセスに使用されるルーティング不可能な仮想純サブネットします。 ノードで実行されている `kube-proxy` によって、ルーティング可能なアドレス空間との間で変換が行われます。 | 「10.96.0.0/12」 |
| <a name="cluster-subnet-def"></a>**クラスター サブネット** |  これは、グローバル サブネット クラスター内のすべてのポッドで使用されています。 各ノードの小さい/24 が割り当てられているを使用するには、そのポッドに対してこのからサブネットします。 自分のクラスターで使用されているすべてのポッドに対応するために十分な大きさがある場合があります。 *最小*サブネットのサイズを計算するには。 `(number of nodes) + (number of nodes * maximum pods per node that you configure)` <p/>ノードごとに 100 ポッドの 5 つのノードの例:`(5) + (5 *  100) = 505`します。  | 「10.244.0.0/16」 |
| **Kubernetes DNS サービス IP** | DNS の解像度とクラスター サービスの検索用に使用される"kube dns"サービスの IP アドレス。 | 「10.96.0.10」 |
> [!NOTE]
> Docker のインストール時に既定で作成される他の Docker ネットワーク (NAT) があります。 代わりにクラスター サブネットの ip アドレスを割り当てることと、windows Kubernetes を操作するには必要ありません。

### <a name="disable-anti-spoofing-protection"></a>スプーフィング対策保護を無効にします。 ###
> [!Important] 
> すべてのユーザーが正常に仮想マシンを使用して、windows Kubernetes を今日に展開するために必要になると、このセクションをよくお読みください。

MAC アドレス スプーフィングことを確認し、[Windows コンテナー ホスト (ゲスト) 仮想マシンの仮想化を有効にします。 これを実現するには、仮想マシン (HYPER-V で指定されたなど) をホストしているコンピューターに管理者として、次を実行する必要があります。

```powershell
Set-VMProcessor -VMName "<name>" -ExposeVirtualizationExtensions $true 
Get-VMNetworkAdapter -VMName "<name>" | Set-VMNetworkAdapter -MacAddressSpoofing On
```
> [!TIP]
> 仮想化ニーズに合わせてを VMware ベースの製品を使用している場合は、MAC なりすまし要件の[無作為検出モード](https://kb.vmware.com/s/article/1004099)を有効にするのに確認してください。

>[!TIP]
> 独自に展開する Kubernetes Azure IaaS 仮想マシンで、この要件の[入れ子になった仮想化](https://azure.microsoft.com/en-us/blog/nested-virtualization-in-azure/)をサポートする仮想マシンを参照してください。

## <a name="what-you-will-accomplish"></a>作業内容 ##

このガイドでは、次のことを行います。

> [!div class="checklist"]
> * [Kubernetes マスター](./creating-a-linux-master.md)ノードを作成します。  
> * [ネットワーク ソリューション](./network-topologies.md)を選択します。  
> * [Windows ワーカー ノード](./joining-windows-workers.md)または[Linux ワーカー ノード](./joining-linux-workers.md)を結合します。  
> * [サンプル Kubernetes リソース](./deploying-resources.md)を展開します。  
> * [一般的な問題とよくある誤り](./common-problems.md)を理解する。

## <a name="next-steps"></a>次のステップ ##
ここでは、[重要な前提条件と windows Kubernetes を今日正常に展開するために必要な前提条件について話しします。 引き続き Kubernetes マスター シェイプをセットアップする方法について説明します。

> [!div class="nextstepaction"]
> [Kubernetes マスター シェイプを作成します。](./creating-a-linux-master.md)