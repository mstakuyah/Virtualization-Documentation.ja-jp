---
title: "Windows コンテナーのネットワーク"
description: "Windows コンテナーのネットワークを構成します。"
keywords: "Docker, コンテナー"
author: jmesser81
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 2e26177f3e653e9102dc91070b987e28ef713bed

---

# コンテナーのネットワーク

ネットワークに関して言えば、Windows コンテナーの機能は仮想マシンと似ています。 それぞれのコンテナーが仮想ネットワーク アダプターを備え、それが仮想スイッチに接続され、そこで受信トラフィックと送信トラフィックが転送されます。 コンテナーのネットワーク アダプターがインストールされている Windows Server と HYPER-V コンテナーには、同じホスト上の複数のコンテナーに分離を強制するために、ネットワーク コンパートメントがそれぞれ作成されます。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 HYPER-V コンテナーでは、仮想スイッチへの接続に (ユーティリティ VM には公開されていない) 統合 VM NIC を使用します。

Windows コンテナーは、*nat*、*transparent*、*l2bridge*、および *l2tunnel* の 4 つの異なるネットワーク ドライバーまたはモードをサポートしています。 ネットワーク モードは、物理ネットワークのインフラストラクチャと単一または複数のホストのネットワーク要件に応じて、最適なものを選択する必要があります。 

Docker エンジンは、dockerd サービスの初回実行時に、既定で nat のネットワークを作成します。 作成される既定の内部 IP プレフィックスは、172.16.0.0/12 です。 

> 注: コンテナー ホストの IP アドレスが同じプレフィックス内のものである場合、以下に示すように、NAT の内部 IP プレフィックスを変更する必要があります。

コンテナーのエンドポイントは、この既定のネットワークに接続され、内部プレフィックス内から IP アドレスを割り当てられます。 現在 Windows では、NAT ネットワークは 1 つのみサポートされています (この制限の回避には、保留中の [Pull Request](https://github.com/docker/docker/pull/25097) が役立つ場合があります)。 

別のドライバー (例: transparent、l2bridge) を使用し、同じコンテナー ホストにネットワークを追加作成することができます。 次の表に、内部 (コンテナー間) および外部接続の各モードのネットワーク接続を指定する方法を示します。

- **ネットワーク アドレス変換**: 各コンテナーは、内部のプライベート IP プレフィックス (例: 172.16.0.0/12) から IP アドレスを取得します。 コンテナー ホストからコンテナー エンドポイントへのポート転送およびマッピングがサポートされています。

- **透過**: 各コンテナーのエンドポイントは、物理ネットワークに直接接続されています。 物理ネットワークの IP は、外部の DHCP サーバーを使用して静的または動的に割り当てることができます。

- **L2 ブリッジ**: 各コンテナーのエンドポイントは、コンテナー ホストと同じ IP サブネット内にあります。 IP アドレスは、コンテナー ホストと同じプレフィックスから静的に割り当てる必要があります。 ホスト上のすべてのコンテナーのエンドポイントは、レイヤー 2 のアドレス変換のために同じ MAC アドレスとなります。

- **L2 トンネル** - _: このモードは Microsoft Cloud スタックでのみ使用する必要があります_。

> Microsoft SDN スタックを使用してオーバーレイの仮想ネットワークをコンテナー エンドポイントに接続する方法については、「[Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)」 (仮想ネットワークにコンテナーを接続する) のトピックをご覧ください。

## 単一ノード

|  | コンテナー間 | コンテナーから外部 |
| :---: | :---------------     |  :---                |
| nat | HYPER-V 仮想スイッチを介したブリッジ接続 | アドレス変換を適用し WinNAT を介してルーティング | 
| transparent | HYPER-V 仮想スイッチを介したブリッジ接続 | 物理ネットワークへ直接アクセス | 
| l2bridge | HYPER-V 仮想スイッチを介したブリッジ接続|  MAC アドレス変換を使用し物理ネットワークへアクセス|  



## 複数ノード

|  | コンテナー間 | コンテナーから外部 |
| :---: | :----       | :---------- |
| nat | WinNAT を介してルーティングされた、アドレス変換が適用された、外部のコンテナーのホスト IP およびポートを参照する必要がある | WinNAT を介してルーティングされた、アドレス変換が適用された、外部のホストを参照する必要がある | 
| transparent | コンテナー IP エンドポイントを直接参照する必要がある | 物理ネットワークへ直接アクセス | 
| l2bridge | コンテナー IP エンドポイントを直接参照する必要がある| MAC アドレス変換を使用し物理ネットワークへアクセス| 


## ネットワークの作成 

### (既定) NAT ネットワーク

Windows docker エンジンは、IP プレフィックス 172.16.0.0/12 を使用して既定の 'nat' ネットワークを作成します。 現在、Windows コンテナー ホストでは、nat ネットワークは 1 つのみ許可されています。 特定の IP プレフィックスを使用して nat ネットワークを作成するには、docker config daemon.json ファイル内 (場所: C:\ProgramData\Docker\config\daemon.json) (存在しない場合は作成します) のオプションを次の 2 つの 1 つに変更する必要があります。
 1. _"fixed-cidr": "< IP Prefix > / Mask"_オプション: 指定した IP プレフィックスとマスクを使用して既定の nat ネットワークを作成します。
 2. _"bridge": "none"_ オプション: 既定のネットワークは作成されず、ユーザーが任意のドライバーを使用して *docker network create -d <driver>* コマンドを使用してユーザー定義のネットワークを作成します。

これらのいずれかの構成オプションの実行する前に、まず Docker サービスを停止して、既存のすべての nat ネットワークを削除する必要があります。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

ユーザーが daemon.json ファイルに "fixed-cidr" オプションを追加した場合、docker エンジンは指定したカスタムの IP プレフィックスおよびマスクが使用されたユーザー定義の nat ネットワークを作成します。 代わりに "bridge:none" オプションを追加した場合、ユーザーは手動でネットワークを作成する必要があります。

```none
# Create a user-defined nat network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

既定では、コンテナー エンドポイントは既定の nat ネットワークに接続されます。 (daemon.json で "bridge:none" を指定したために) 既定の nat ネットワークが作成されなかった場合や別のユーザー定義のネットワークへのアクセスが必要な場合、ユーザーは docker run コマンドと共に、*--network* を指定できます。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### ポートのマッピング

NAT ネットワークに接続されているコンテナー内で実行されているアプリケーションにアクセスするには、コンテナー ホストとコンテナー エンドポイント間にポート マッピングを作成する必要があります。 これらのマッピングは、コンテナーの作成時またはコンテナーが STOPPED 状態の時に指定する必要があります。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Create a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

また、-p パラメーターや、Dockerfile の EXPOSE コマンドで-p パラメーターを使用したポートの動的なマッピングもサポートされています。 コンテナー ホストで一時的なポートがランダムに選択され、Docker ps の実行時に検査されます。

```none
C:\> docker run -itd -p 80 windowsservercore cmd

# Network services running on port TCP:80 in this container can be accessed externally on port TCP:14824
C:\> docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                   NAMES
bbf72109b1fc        windowsservercore   "cmd"               6 seconds ago       Up 2 seconds        *0.0.0.0:14824->80/tcp*   drunk_stonebraker

# Container image specified EXPOSE 80 in Dockerfile - publish this port mapping
C:\> docker network 
```
> WS2016 TP5 と 14300 以降の Windows Insider ビルドでは、NAT ポートをマッピングする際に毎回ファイアウォール規則が自動的に作成されます。 このファイアウォール規則は、コンテナー ホストに対してグローバルであり、特定のコンテナーのエンドポイント、またはネットワーク アダプターにはローカライズされません。

Windows NAT (WinNAT) の実装には、いくつかの使用制限があります。ブログ投稿「[WinNAT capabilities and limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)」 (WinNAT の機能と制限事項) をご覧ください。 
 1. 1 つのコンテナー ホストには、1 つの NAT ネットワーク (1 つの内部 IP プレフィックス) のみがサポートされています。
 2. コンテナー エンドポイントは、内部 IP およびポートを使用したコンテナー ホストからのみアクセスできます。

ネットワークを追加作成するには、別のドライバーを使用します。 

> Docker のネットワーク ドライバーは、すべて小文字です。

### 透過ネットワーク

透過ネットワーク モードを使用するには、ドライバー名を 'transparent' にしてコンテナー ネットワークを作成します。 

```none
C:\> docker network create -d transparent MyTransparentNetwork
```

コンテナーのホストが仮想化されており、IP の割り当てに DHCP を使用したい場合は、仮想マシンのネットワーク アダプターで MACAddressSpoofing を有効にする必要があります。 それ以外の場合、HYPER-V ホストでは、複数の MAC アドレスを持つ VM 内のコンテナーからのネットワーク トラフィックをブロックします。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 複数の透過ネットワークを作成する場合は、外部 Hyper-V Virtual Switch (自動作成) をバインドする (仮想) ネットワーク アダプターを指定する必要があります。

特定のネットワーク インターフェイスに (HYPER-V 仮想スイッチを介して接続されている) ネットワークをバインドするには、*-o com.docker.network.windowsshim.interface=<Interface>* オプションを使用します。

```none
# Create a transparent network which is attached to the "Ethernet 2" network interface
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

*com.docker.network.windowsshim.interface* の値は、次の方法で取得したアダプターの *Name* です。 
```none
Get-NetAdapter
```

透過ネットワークに接続されているコンテナー エンドポイントの IP アドレスは、外部の DHCP サーバーから静的にまたは動的に割り当てることができます。

IP を静的に割り当てる場合、ネットワークの作成時に、*--subnet* および *--gateway* パラメーターが指定されていることを最初に確認する必要があります。 サブネットとゲートウェイの IP アドレスは、コンテナー ホスト (つまり、物理ネットワーク) のネットワーク設定と同じにする必要があります。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
IP アドレスを指定するには、docker run コマンドで *--ip* オプションを使用します。

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> この IP アドレスは、物理ネットワーク上の他のネットワーク デバイスに割り当てられていないことを確認する必要があります。

コンテナー エンドポイントは、物理ネットワークに直接接続されているため、ポートのマッピングを指定する必要はありません。

### L2 ブリッジ 

L2 ブリッジ ネットワーク モードを使用するには、ドライバー名を 'l2bridge' にして、コンテナー ネットワークを作成します。 物理ネットワークに対応するサブネットおよびゲートウェイも再度指定する必要があります。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

l2bridge ネットワークでは、静的な IP の割り当てのみがサポートされています。 

> SDN ファブリックで l2bridge ネットワークを使用する場合、動的な IP の割り当てのみがサポートされています。 詳細については、「[Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)」 (仮想ネットワークにコンテナーを接続する) をご覧ください。

## その他の操作と構成

### 使用可能なネットワークのリスト取得

```none
# list container networks
C:\> docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
0a297065f06a        nat                 nat                 local
d42516aa0250        none                null                local
```

### ネットワークの削除

コンテナー ネットワークを削除するには、`docker network rm` を使用します。

```none
C:\> docker network rm "<network name>"
```

これはコンテナー ネットワークが使用したすべての HYPER-V 仮想スイッチと、作成されたすべてのネットワーク アドレス変換 (WinNAT - NetNat インスタンス) をクリーンアップします。

### ネットワーク検査 

特定のネットワークに接続されているコンテナー、またこれらのコンテナーのエンドポイントに関連付けられている IP を確認するには、次を実行します。

```none
C:\> docker network inspect <network name>
```

### 複数のコンテナー ネットワーク

以下のことに注意して、1 つのコンテナー ホストに複数のコンテナー ネットワークを作成できます。
* 1 つのコンテナー ホストには、NAT ネットワークを 1 つのみ作成できます。
* 複数のネットワーク (例: 透過、L2 ブリッジ、または L2 トンネル) が接続に外部の vSwitch を使用する場合、それぞれが独自のネットワーク アダプターを使用する必要があります。

### ネットワークの選択

Windows コンテナーの作成時、コンテナーのネットワーク アダプターを接続するネットワークを指定できます。 ネットワークを指定しない場合、既定 NAT ネットワークが使用されます。

コンテナーを既定以外の NAT ネットワークに接続するには、Docker の run コマンドで --network オプションを使用します。

```none
C:\> docker run -it --network=MyTransparentNet windowsservercore cmd
```

### 静的 IP アドレス

```none
C:\> docker run -it --network=MyTransparentNet --ip=10.80.123.32 windowsservercore cmd
```

静的な IP の割り当ては、コンテナーのネットワーク アダプターに対して直接行い、コンテナーが停止状態にある場合にのみ実行する必要があります。 コンテナーの実行中は、コンテナーのネットワーク アダプターの "ホット アド" またはネットワーク スタックに対する変更は実行できません。


## 注意事項と潜在的な問題

### ファイアウォール

ICMP (Ping) と DHCP を有効にするには、コンテナー ホストに特定のファイアウォール規則を作成する必要があります。 同じホスト上の 2 つのコンテナー間を ping したり、動的に割り当てられた IP アドレスを DHCP を介して受け取ったりするには、Windows Server コンテナーに ICMP と DHCP が必要です。 TP5 では、Install-ContainerHost.ps1 スクリプトを使用してこれらの規則を作成します。 Post-TP5 では、これらの規則は自動的に作成されます。 NAT ポート転送ルールに対応するすべてのファイアウォール規則は自動的に作成され、コンテナーが停止されるときにクリーンアップされます。

### サポートされていない機能

現在、Docker CLI では次のネットワーク機能はサポートされていません。
 * コンテナーのリンク (例: --link)
 * コンテナーの名前およびサービスでの IP 解決 _これは、サービスの更新で近々にサポートされる予定です。_
 * 既定のオーバーレイ ネットワーク ドライバー

現在、Windows Docker では次のネットワーク オプションはサポートされていません。
 * --add-host
 * --dns
 * --dns-opt
 * --dns-search
 * -h､--hostname
 * --net-alias
 * --aux-address
 * --internal
 * --ip-range

 > Windows Server 2016 Technical Preview 5 と最新の Windows Insider Preview (WIP) "flighted" ビルドには既知のバグがあります。このバグでは、新しいビルドにアップグレードすると、コンテナー ネットワークと vSwitch が重複 (「漏洩」) します。 この問題に対処するには、次のスクリプトを実行してください。
```none
$KeyPath = "HKLM:\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\SwitchList"
$keys = get-childitem $KeyPath
foreach($key in $keys)
{
   if ($key.GetValue("FriendlyName") -eq 'nat')
   {
      $newKeyPath = $KeyPath+"\"+$key.PSChildName
      Remove-Item -Path $newKeyPath -Recurse
   }
}
remove-netnat -Confirm:$false
Get-ContainerNetwork | Remove-ContainerNetwork
Get-VmSwitch -Name nat | Remove-VmSwitch # Note: failure is expected
Stop-Service docker
Set-Service docker -StartupType Disabled
```
> ホストを再起動してから、残りのステップを実行します。
```none
Get-NetNat | Remove-NetNat -Confirm $false
Set-Service docker -StartupType automatic
Start-Service docker 
```



<!--HONumber=Aug16_HO4-->


