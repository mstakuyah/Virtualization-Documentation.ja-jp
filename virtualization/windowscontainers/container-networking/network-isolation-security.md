---
title: Windows コンテナー ネットワーク
description: Windows コンテナー内のネットワーク分離とセキュリティ。
keywords: Docker, コンテナー
author: jmesser81
ms.date: 03/27/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
ms.openlocfilehash: 7203989483cb07423b70ff8cc644f715ba4be274
ms.sourcegitcommit: ec186664e76d413d3bf75f2056d5acb556f4205d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 05/11/2018
ms.locfileid: "1876107"
---
# <a name="network-isolation-and-security"></a>ネットワーク分離とセキュリティ

## <a name="isolation-with-network-namespaces"></a>ネットワーク名前空間による分離
各コンテナーのエンドポイントは、自身の__ネットワーク名前空間__内に置かれます。 管理ホストの vNIC とホスト ネットワーク スタックは、既定のネットワーク名前空間に置かれます。 コンテナーのネットワーク アダプターがインストールされている Windows Server と Hyper-V コンテナーには、同じホスト上の複数のコンテナー間でネットワークの分離を実施するために、ネットワーク名前空間がそれぞれ作成されます。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 Hyper-V コンテナーでは、仮想スイッチへの接続に (ユーティリティ VM には公開されていない) 統合 VM NIC を使用します。


![テキスト](media/network-compartment-visual.png)


```powershell 
Get-NetCompartment
```

## <a name="network-security"></a>ネットワーク セキュリティ
使用されているコンテナーとネットワーク ドライバーによっては、Windows ファイアウォールと [VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/) の組み合わせでポート ACL が適用されます。

### <a name="windows-server-containers"></a>Windows Server コンテナー
これらは、Windows ホストのファイアウォール (ネットワークの名前空間に対応) と共に VFP を使用します。
  * 既定の発信: ALLOW ALL
  * 既定の着信: (TCP、UDP、ICMP、IGMP の) 未承諾ネットワーク トラフィックに対して ALLOW ALL 
    * これらのプロトコル以外のネットワーク トラフィックに対して DENY ALL

  > 注意: Windows Server Version 1709 および Windows 10 Fall Creators Update の前は既定の*着信*規則は DENY ALL でした。 これらの古いリリースを実行しているユーザーは、``docker run -p`` (ポート フォワーディング) を使用して着信 ALLOW 規則を作成できます。


### <a name="hyper-v-containers"></a>Hyper-V コンテナー
Hyper-V コンテナーには分離された独自のカーネルがあり、そのため次の構成で独自の Windows ファイアウォール インスタンスが実行されます。
  * Windows ファイアウォール (ユーティリティ VM 内で実行) と VFP の両方で既定 ALLOW ALL


![テキスト](media/windows-firewall-containers.png)


### <a name="kubernetes-pods"></a>Kubernetes ポッド
[Kubernetes ポッド](https://kubernetes.io/docs/concepts/workloads/pods/pod/)では、エンドポイントが関連付けられているインフラストラクチャ コンテナーが最初に作成されます。 同じポッドに属するコンテナー (インフラストラクチャ コンテナーおよびワーカー コンテナーを含む) では、共通のネットワーク名前空間 (同じ IP およびとポート範囲) が使用されます。


![テキスト](media/pod-network-compartment.png)


### <a name="customizing-default-port-acls"></a>既定のポート ACL のカスタマイズ
既定のポート ACL を変更する場合は、HNS ドキュメント (リンクが間もなく追加されます) をご覧ください。 以下のコンポーネント内でポリシーの更新が必要になります。

> 注意: 現在、透過モードと NAT モードの Hyper-V コンテナーについては、既定のポート ACL を再プログラミングすることはできません。 このことは、次の表の中で "X" として示されています。

| ネットワーク ドライバー | Windows Server コンテナー | Hyper-V コンテナー  |
| -------------- |-------------------------- | ------------------- |
| 透過 | Windows ファイアウォール | X |
| NAT | Windows ファイアウォール | X |
| L2Bridge | 両方 | VFP |
| L2Tunnel | 両方 | VFP |
| オーバーレイ  | 両方 | VFP |