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
ms.openlocfilehash: d5081104f1614a91d6441a5e879a439f1df1bf77
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998539"
---
# <a name="network-isolation-and-security"></a>ネットワーク分離とセキュリティ

## <a name="isolation-with-network-namespaces"></a>ネットワーク名前空間を使用した分離

各コンテナーのエンドポイントは、自身の__ネットワーク名前空間__内に置かれます。 管理ホストの vNIC とホスト ネットワーク スタックは、既定のネットワーク名前空間に置かれます。 同じホスト上のコンテナー間にネットワーク分離を適用するには、各 Windows Server コンテナーに対してネットワーク名前空間を作成し、そのコンテナーのネットワークアダプターをインストールする Hyper-v の分離の下で実行します。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 Hyper-v 分離では、合成 VM NIC (ユーティリティ VM に公開されない) を使用して仮想スイッチにアタッチします。

![テキスト](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>ネットワークセキュリティ

使用されているコンテナーとネットワーク ドライバーによっては、Windows ファイアウォールと [VFP](https://www.microsoft.com/research/project/azure-virtual-filtering-platform/) の組み合わせでポート ACL が適用されます。

### <a name="windows-server-containers"></a>Windows Server コンテナー

これらは、Windows ホストのファイアウォール (ネットワークの名前空間に対応) と共に VFP を使用します。

* 既定の発信: ALLOW ALL
* 既定の着信: (TCP、UDP、ICMP、IGMP の) 未承諾ネットワーク トラフィックに対して ALLOW ALL 
  * これらのプロトコル以外のネットワーク トラフィックに対して DENY ALL

  >[!NOTE]
  >Windows Server、バージョン1709、Windows 10 は、作成者が更新される前に、既定の受信規則がすべて拒否されました。 これらの古いリリースを実行しているユーザーは``docker run -p`` 、(ポート転送) を使用して、受信許可ルールを作成できます。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

Hyper-v 分離で実行されているコンテナーは、独自の分離されたカーネルを持っているため、次の構成で Windows ファイアウォールの独自のインスタンスを実行します。

* Windows ファイアウォール (ユーティリティ VM 内で実行) と VFP の両方で既定 ALLOW ALL

![テキスト](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes ポッド

[Kubernetes pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/)では、最初に、エンドポイントがアタッチされるインフラストラクチャコンテナーが作成されます。 インフラストラクチャやワーカーコンテナーなど、同じ pod に属しているコンテナーは、共通のネットワーク名前空間 (同じ IP とポート空間) を共有します。

![テキスト](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>既定のポート ACL のカスタマイズ

既定のポート Acl を変更する場合は、まず、「ホストネットワークサービスのドキュメント (まもなく追加されるリンク)」を参照してください。 ポリシーは、次のコンポーネント内で更新する必要があります。

>[!NOTE]
>Transparent と NAT モードでの Hyper-v の分離の場合、現在、既定のポート Acl を編集することはできません。 このことは、次の表の中で "X" として示されています。

| ネットワークドライバー | Windows Server コンテナー | Hyper-V による分離  |
| -------------- |-------------------------- | ------------------- |
| 透過 | Windows ファイアウォール | X |
| NAT | Windows ファイアウォール | X |
| L2Bridge | 両方 | VFP |
| L2Tunnel | 両方 | VFP |
| オーバーレイ  | 両方 | VFP |