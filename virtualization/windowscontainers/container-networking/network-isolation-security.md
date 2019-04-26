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
ms.openlocfilehash: 1c0a3fd25a5572604db59e0c68d8b4a3d84b00e9
ms.sourcegitcommit: 0deb653de8a14b32a1cfe3e1d73e5d3f31bbe83b
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/26/2019
ms.locfileid: "9576313"
---
# <a name="network-isolation-and-security"></a>ネットワークの分離とセキュリティ

## <a name="isolation-with-network-namespaces"></a>ネットワークの名前空間に分離

各コンテナーのエンドポイントは、自身の__ネットワーク名前空間__内に置かれます。 管理ホストの vNIC とホスト ネットワーク スタックは、既定のネットワーク名前空間に置かれます。 Windows Server コンテナーごとに、同じホストのコンテナーの間のネットワークの分離を強制するために、ネットワークの名前空間が作成され、コンテナーにコンテナーのネットワーク アダプターがインストールされている HYPER-V 分離を実行します。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 HYPER-V 分離では、統合 VM NIC (ユーティリティ VM に公開されない) を使用して、仮想スイッチに添付します。

![テキスト](media/network-compartment-visual.png)

```powershell
Get-NetCompartment
```

## <a name="network-security"></a>ネットワークのセキュリティ

使用されているコンテナーとネットワーク ドライバーによっては、Windows ファイアウォールと [VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/) の組み合わせでポート ACL が適用されます。

### <a name="windows-server-containers"></a>Windows Server コンテナー

これらは、Windows ホストのファイアウォール (ネットワークの名前空間に対応) と共に VFP を使用します。

* 既定の発信: ALLOW ALL
* 既定の着信: (TCP、UDP、ICMP、IGMP の) 未承諾ネットワーク トラフィックに対して ALLOW ALL 
  * これらのプロトコル以外のネットワーク トラフィックに対して DENY ALL

  >[!NOTE]
  >Windows Server、バージョン 1709 と Windows 10 秋の作成者の更新プログラムの場合は、前に既定の受信の規則はすべて拒否されました。 これらの以前のリリースを実行しているユーザーが受信できるようにするルールを作成できます``docker run -p``(ポート転送) します。

### <a name="hyper-v-isolation"></a>Hyper-V による分離

HYPER-V 分離で実行されているコンテナーでは、独自の分離カーネルし、そのため構成は以下の Windows ファイアウォールの独自のインスタンスを実行します。

* Windows ファイアウォール (ユーティリティ VM 内で実行) と VFP の両方で既定 ALLOW ALL

![テキスト](media/windows-firewall-containers.png)

### <a name="kubernetes-pods"></a>Kubernetes ポッド

[Kubernetes ポッド](https://kubernetes.io/docs/concepts/workloads/pods/pod/)に両端のシートが関連付けられているインフラストラクチャ コンテナーが最初に作成されます。 インフラストラクチャおよび作業者のコンテナーなど、同じポッドに属しているコンテナーでは、一般的なネットワーク名前空間 (同じ ip アドレスやポート領域) を共有します。

![テキスト](media/pod-network-compartment.png)

### <a name="customizing-default-port-acls"></a>既定のポート ACL のカスタマイズ

既定のポート Acl、最初に、ネットワーク サービスをホストのドキュメントを参照してください (すぐに追加するリンク) を変更する場合。 次のコンポーネント内のポリシーを更新する必要があります。

>[!NOTE]
>透明と NAT モードでの HYPER-V 分離、する現在できない再設定既定のポート Acl します。 このことは、次の表の中で "X" として示されています。

| ネットワーク ドライバー | Windows Server コンテナー | Hyper-V による分離  |
| -------------- |-------------------------- | ------------------- |
| 透過 | Windows ファイアウォール | X |
| NAT | Windows ファイアウォール | X |
| L2Bridge | 両方 | VFP |
| L2Tunnel | 両方 | VFP |
| オーバーレイ  | 両方 | VFP |