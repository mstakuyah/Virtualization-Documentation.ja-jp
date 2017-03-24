---
title: "Windows コンテナーのネットワーク"
description: "Windows コンテナーのネットワークを構成します。"
keywords: "Docker, コンテナー"
author: jmesser81
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 538871ba-d02e-47d3-a3bf-25cda4a40965
translationtype: Human Translation
ms.sourcegitcommit: 23d4b665da627f35cf5fce49c3c9974d0ef287dd
ms.openlocfilehash: e56a5b984cc1c42e27628d00a5cd532788aef11c
ms.lasthandoff: 02/10/2017

---

# コンテナーのネットワーク

ネットワークに関して言えば、Windows コンテナーの機能は仮想マシンと似ています。 それぞれのコンテナーが仮想ネットワーク アダプター (vNIC) を備え、それが仮想スイッチ (vSwitch) に接続され、そこで受信トラフィックと送信トラフィックが転送されます。 コンテナーのネットワーク アダプターがインストールされている Windows Server と HYPER-V コンテナーには、同じホスト上の複数のコンテナーに分離を強制するために、ネットワーク コンパートメントがそれぞれ作成されます。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 HYPER-V コンテナーでは、仮想スイッチへの接続に (ユーティリティ VM には公開されていない) 統合 VM NIC を使用します。

Windows コンテナーは、*nat*、*transparent*、*l2bridge*、および *l2tunnel* の 4 つの異なるネットワーク ドライバーまたはモードをサポートしています。 ネットワーク モードは、物理ネットワークのインフラストラクチャと単一または複数のホストのネットワーク要件に応じて、最適なものを選択する必要があります。

Docker エンジンは、dockerd サービスの初回実行時に、既定で NAT のネットワークを作成します。 作成される既定の内部 IP プレフィックスは、172.16.0.0/12 です。 コンテナーのエンドポイントは、この既定のネットワークに自動的に接続され、その内部プレフィックス内から IP アドレスを割り当てられます。

> 注: コンテナー ホストの IP アドレスが同じプレフィックス内のものである場合、以下に示すように、NAT の内部 IP プレフィックスを変更する必要があります。

別のドライバー (例: transparent、l2bridge) を使用し、同じコンテナー ホストにネットワークを追加作成することができます。 次の表に、内部 (コンテナー間) および外部接続の各モードのネットワーク接続を指定する方法を示します。

- **ネットワーク アドレス変換 (NAT)**: 各コンテナーは、内部のプライベート IP プレフィックス (例: 172.16.0.0/12) から IP アドレスを取得します。 コンテナー ホストからコンテナー エンドポイントへのポート転送およびマッピングがサポートされています。

- **透過**: 各コンテナーのエンドポイントは、物理ネットワークに直接接続されています。 物理ネットワークの IP は、外部の DHCP サーバーを使用して静的または動的に割り当てることができます。

- **[新機能] オーバーレイ**: Docker エンジンが [swarm モード](./swarm-mode.md)で動作している場合に、VXLAN テクノロジに基づくオーバーレイ ネットワークを使用すると、複数のコンテナー ホストにわたってコンテナーのエンドポイントを接続できます。 Swarm クラスター上に作成される各オーバーレイ ネットワークは、プライベート IP プレフィックスによって定義される独自の IP サブネットを使って作成されます。

- **L2 ブリッジ**: 各コンテナーのエンドポイントは、コンテナー ホストと同じ IP サブネット内にあります。 IP アドレスは、コンテナー ホストと同じプレフィックスから静的に割り当てる必要があります。 ホスト上のすべてのコンテナーのエンドポイントは、レイヤー 2 のアドレス変換のために同じ MAC アドレスとなります。

- **L2 トンネル** - _: このモードは Microsoft Cloud スタックでのみ使用する必要があります_。

> Microsoft SDN スタックを使用してオーバーレイの仮想ネットワークをコンテナー エンドポイントに接続する方法については、「[Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)」 (仮想ネットワークにコンテナーを接続する) のトピックをご覧ください。

## 単一ノード

|  | コンテナー間 | コンテナーから外部 |
| :---: | :---------------     |  :---                |
| nat | HYPER-V 仮想スイッチを介したブリッジ接続 | アドレス変換を適用し WinNAT を介してルーティング |
| transparent | Hyper-V 仮想スイッチを介したブリッジ接続 | 物理ネットワークへ直接アクセス |
| overlay | VXLAN カプセル化は、Hyper-V 仮想スイッチの VFP 転送拡張機能で行われ、*内部ホスト*通信は、Hyper-V 仮想スイッチを通じてブリッジ接続経由で行われる | アドレス変換を適用し WinNAT を介してルーティング
| l2bridge | Hyper-V 仮想スイッチを介したブリッジ接続|  MAC アドレス変換を使用し物理ネットワークへアクセス|  



## 複数ノード

|  | コンテナー間 | コンテナーから外部 |
| :---: | :----       | :---------- |
| nat | WinNAT を介してルーティングされた、アドレス変換が適用された、外部のコンテナーのホスト IP およびポートを参照する必要がある | WinNAT を介してルーティングされた、アドレス変換が適用された、外部のホストを参照する必要がある |
| transparent | コンテナー IP エンドポイントを直接参照する必要がある | 物理ネットワークへ直接アクセス |
| overlay | VXLAN カプセル化は、Hyper-V 仮想スイッチの VFP 転送拡張機能で行われ、*内部ホスト*通信は、IP エンドポイントを直接参照します。 | アドレス変換を適用し WinNAT を介してルーティング| 
| l2bridge | コンテナー IP エンドポイントを直接参照する必要がある| MAC アドレス変換を使用し物理ネットワークへアクセス|


## ネットワークの作成

### (既定) NAT ネットワーク

Windows docker エンジンは、IP プレフィックス 172.16.0.0/12 を使用して既定の NAT ネットワーク (Docker 名、'nat') を作成します。 特定の IP プレフィックスを使用して NAT ネットワークを作成するには、Docker config daemon.json ファイル内 (場所: C:\ProgramData\Docker\config\daemon.json) (存在しない場合は作成します) のオプションを次の 2 つの 1 つに変更する必要があります。
 1. _"fixed-cidr": "< IP Prefix > / Mask"_オプション: 指定した IP プレフィックスとマスクを使用して既定の NAT ネットワークを作成します。
 2. _"bridge": "none"_ オプション: 既定のネットワークは作成されず、ユーザーが任意のドライバーを使用して *docker network create -d <driver>* コマンドを使用してユーザー定義のネットワークを作成します。

これらのいずれかの構成オプションの実行する前に、まず Docker サービスを停止して、既存のすべての NAT ネットワークを削除する必要があります。

```none
PS C:\> Stop-Service docker
PS C:\> Get-ContainerNetwork | Remove-ContainerNetwork

...Edit the daemon.json file...

PS C:\> Start-Service docker
```

daemon.json ファイルに "fixed-cidr" オプションを追加した場合、Docker エンジンは指定されたカスタムの IP プレフィックスとマスクでユーザー定義の NAT ネットワークを作成します。 代わりに "bridge:none" オプションが追加された場合、ネットワークを手動で作成する必要があります。

```none
# Create a user-defined NAT network
C:\> docker network create -d nat --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyNatNetwork
```

既定では、コンテナー エンドポイントは既定の 'nat' ネットワークに接続されます。 (daemon.json で "bridge:none" を指定したために) 'nat' ネットワークが作成されなかった場合や別のユーザー定義のネットワークへのアクセスが必要な場合、ユーザーは docker run コマンドと共に、*--network* を指定できます。

```none
# Connect new container to the MyNatNetwork
C:\> docker run -it --network=MyNatNetwork <image> <cmd>
```

#### ポートのマッピング

NAT ネットワークに接続されているコンテナー内で実行されているアプリケーションにアクセスするには、コンテナー ホストとコンテナー エンドポイント間にポート マッピングを作成する必要があります。 これらのマッピングは、コンテナーの作成時またはコンテナーが STOPPED 状態の時に指定する必要があります。

```none
# Creates a static mapping between port TCP:80 of the container host and TCP:80 of the container
C:\> docker run -it -p 80:80 <image> <cmd>

# Creates a static mapping between port 8082 of the container host and port 80 of the container.
C:\> docker run -it -p 8082:80 windowsservercore cmd
```

また、-p パラメーターや、Dockerfile の EXPOSE コマンドで-p パラメーターを使用したポートの動的なマッピングもサポートされています。 指定されていない場合、コンテナー ホストで一時的なポートがランダムに選択され、'docker ps' の実行時に検査されます。

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
 1. サポートされる NAT 内部 IP プレフィックスはコンテナー ホストごとに 1 つだけです。そのため、プレフィックスをパーティショニングし、'複数の' NAT ネットワークを定義する必要があります (本書の「複数の NAT ネットワーク」セクションを参照してください)。
 2. コンテナー エンドポイントには、コンテナーの内部 IP とポートを利用して、コンテナー ホストからのみアクセスできます (この情報は 'docker network inspect <CONTAINER ID>' を利用して検索)。

ネットワークを追加作成するには、別のドライバーを使用します。

> Docker のネットワーク ドライバーは、すべて小文字です。

### 透過ネットワーク

透過ネットワーク モードを使用するには、ドライバー名を 'transparent' にしてコンテナー ネットワークを作成します。

```none
C:\> docker network create -d transparent MyTransparentNetwork
```
> 注: 透過ネットワークの作成中にエラーが発生した場合、Docker で自動検出されなかった外部 vSwitch がシステムに存在し、それが透過ネットワークとコンテナー ホストの外部ネットワーク アダプターの関連付けを阻んでいる可能性があります。 下記の「注意事項と潜在的な問題」の「透過ネットワークの作成を阻む既存の vSwitch」セクションに詳細があります。

コンテナーのホストが仮想化されており、IP の割り当てに DHCP を使用したい場合は、仮想マシンのネットワーク アダプターで MACAddressSpoofing を有効にする必要があります。 それ以外の場合、HYPER-V ホストでは、複数の MAC アドレスを持つ VM 内のコンテナーからのネットワーク トラフィックをブロックします。

```none
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```

> 複数の透過ネットワークを作成する場合は、外部 Hyper-V Virtual Switch (自動作成) をバインドする (仮想) ネットワーク アダプターを指定する必要があります。

透過ネットワークに接続されているコンテナー エンドポイントの IP アドレスは、外部の DHCP サーバーから静的にまたは動的に割り当てることができます。

IP を静的に割り当てる場合、ネットワークの作成時に、*--subnet* および *--gateway* パラメーターが指定されていることを最初に確認する必要があります。 サブネットとゲートウェイの IP アドレスは、コンテナー ホスト (つまり、物理ネットワーク) のネットワーク設定と同じにする必要があります。

```none
# Create a transparent network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 TransparentNet3
```
IP アドレスを指定するには、`docker run` コマンドで *--ip* オプションを使います。

```none
C:\> docker run -it --network=TransparentNet3 --ip 10.123.174.105 <image> <cmd>
```

> この IP アドレスは、物理ネットワーク上の他のネットワーク デバイスに割り当てられていないことを確認する必要があります。

コンテナー エンドポイントは、物理ネットワークに直接接続されているため、ポートのマッピングを指定する必要はありません。

### オーバーレイ ネットワーク

*オーバーレイ ネットワーク モードを使うには、Docker ホストを swarm モードのマネージャー ノードとして実行して使う必要があります。* swarm モードの概要と、swarm マネージャーを初期化する方法について詳しくは、「[swarm モードの概要](./swarm-mode.md)」を参照してください。

オーバーレイ ネットワークを作成するには、**swarm マネージャー ノード**から次のコマンドを実行します。

```none
# Create an overlay network from a swarm manager node, called "myOverlayNet"
C:\> docker network create --driver=overlay myOverlayNet
```

### L2 ブリッジ

L2 ブリッジ ネットワーク モードを使用するには、ドライバー名を 'l2bridge' にして、コンテナー ネットワークを作成します。 物理ネットワークに対応するサブネットおよびゲートウェイも再度指定する必要があります。

```none
C:\> docker network create -d l2bridge --subnet=192.168.1.0/24 --gateway=192.168.1.1 MyBridgeNetwork
```

l2bridge ネットワークでは、静的な IP の割り当てのみがサポートされています。

> SDN ファブリックで l2bridge ネットワークを使用する場合、動的な IP の割り当てのみがサポートされています。 詳しくは「[Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)」 (仮想ネットワークにコンテナーを接続する) をご覧ください。

## その他の操作と構成

> Windows での Docker の機能向上に取り組んでいます。 すべての最新機能に確実にアクセスするためには、最新バージョンの Docker Engine を使用していることを確認してください。 docker のバージョンは、`docker -v` を使用して確認できます。 Docker を構成する方法のガイダンスについては、「[Docker Engine on Windows](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-docker/configure-docker-daemon)」(Windows 上の Docker エンジン) トピックをご覧ください。

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
C:\> docker network rm <network name>
```

これはコンテナー ネットワークが使用したすべての HYPER-V 仮想スイッチと、作成されたすべてのネットワーク アドレス変換 (WinNAT - NetNat インスタンス) をクリーンアップします。

### ネットワーク検査

特定のネットワークに接続されているコンテナー、またこれらのコンテナーのエンドポイントに関連付けられている IP を確認するには、次を実行します。

```none
C:\> docker network inspect <network name>
```

### HNS サービスへのネットワーク名の指定

**通常、`docker network create` を使ってコンテナー ネットワークを作成した場合、使用したネットワーク名は Docker サービスでは使われますが、HNS サービスでは使われません。**

ネットワークを作成する場合、`docker network create` コマンドで `-o com.docker.network.windowsshim.networkname=<network name>` オプションを使うと、HNS サービスから提供された名前を指定できます。 たとえば、次のコマンドを使用すると、HNSサービスに対して指定された名前を使って透過ネットワークを作成できます。

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

#### 例: HNS の既定の名前付け動作

この名前付け動作のオプションをわかりやすくするために、下の画面キャプチャでは、名前付けオプションが*使われていない*場合に、HNS サービスがネットワークに名前を付ける方法を示します。 この例では、`docker network ls` コマンドで示すように、ネットワーク名 "MyTransparentNetwork" が Docker によって認識されます。 しかし、`Get-ContainerNetwork` Windows PowerShell コマンドで示すように、このネットワーク名は HNS サービスには認識されず、その代わりに、大きな英数字のネットワーク名が HNS によって自動的に生成されます。

><figure>
  <img src="media/SpecifyName_Capture.PNG">
  <figcaption>例: HNS サービスに対してネットワーク名が<i>指定されていません</i>。 </figcaption>
</figure>

#### 例: HNS サービスへのネットワーク名の指定

これに対し、`-o com.docker.network.windowsshim.networkname=<network name>` を*使用した場合*、生成された名前の代わりに、指定した名前が HNS によって使われます。 この動作を下の画面キャプチャに示します。

><figure>
  <img src="media/SpecifyName_Capture_2.PNG">
  <figcaption>例: `-o com.docker.network.windowsshim.networkname=<network name>` オプションを使って、ネットワーク名を HNS サービスに指定しています。</figcaption>
</figure>


### 特定のネットワーク インターフェイスへのネットワークのバインド

ネットワーク (Hyper-V 仮想スイッチを通じて接続された) を特定のネットワーク インターフェイスにバインドするには、`docker network create` コマンドで `-o com.docker.network.windowsshim.interface=<Interface>` オプションを使います。 たとえば、次のコマンドを使うと、"Ethernet 2" ネットワーク インターフェイスに接続された透過ネットワークを作成できます。

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注: *com.docker.network.windowsshim.interface* の値は、ネットワーク アダプターの*名前*です。これは次のコマンドを使って確認できます。

>```none
PS C:\> Get-NetAdapter
```

### Set the VLAN ID for a Network

To set a VLAN ID for a network, use the option, `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` to the `docker network create` command. For instance, you might use the following command to create a transparent network with a VLAN ID of 11:

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
ネットワークの VLAN ID を設定すると、そのネットワークに接続されるあらゆるコンテナー エンドポイントの VLAN 分離が設定されます。

**注:** タグ付けされたすべてのトラフィックを正しい VLAN 上でアクセス モードの vNIC (コンテナ エンドポイント) ポートを持つ vSwitch によって処理するためには、(物理的) ホスト ネットワーク アダプターがトランク モードであることを確認します。


### ネットワークの DNS サフィックスや DNS サーバーの指定

ネットワークの DNS サフィックスを指定するには、オプション `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` を使い、ネットワークの DNS サーバーを指定するには、オプション `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` を使います。 たとえば、次のコマンドを使用すると、ネットワークの DNS サフィックスが "example.com" に設定され、ネットワークの DNS サーバーが 4.4.4.4 と 8.8.8.8 に設定されます。

```none
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

### 複数のコンテナー ネットワーク
以下のことに注意して、1 つのコンテナー ホストに複数のコンテナー ネットワークを作成できます。

* 複数のネットワーク (例: 透過、L2 ブリッジ、または L2 トンネル) が接続に外部の vSwitch を使用する場合、それぞれが独自のネットワーク アダプターを使用する必要があります。
* 現在のところ、1 台のコンテナー ホストで複数の NAT ネットワークを作成するための解決策は、既存の NAT ネットワークの内部プレフィックスをパーティショニングすることです。 これに関する詳細は、下の「複数の NAT ネットワーク」セクションを参照してください。

### 複数の NAT ネットワーク
ホストの NAT ネットワークの内部プレフィックスをパーティショニングすることで、1 台のコンテナー ホストに複数の NAT ネットワークを定義できます。

新しい NAT ネットワークのパーティションは、大規模な内部 NAT ネットワーク プレフィックスの下で作成する必要があります。 PowerShell から次のコマンドを実行し、"InternalIPInterfaceAddressPrefix" フィールドを参照すると、プレフィックスが見つかります。

```none
PS C:\> Get-NetNAT
```

たとえば、ホストの NAT ネットワークの内部プレフィックスを 172.16.0.0/12 にします。 その場合、*172.16.0.0/12 プレフィックスの下に入る限り*、Docker を利用して追加 NAT ネットワークを作成できます。 たとえば、IP プレフィックス 172.16.0.0/16 (ゲートウェイ、172.16.0.1) と 172.17.0.0/16 (ゲートウェイ、172.17.0.1) で 2 つの NAT ネットワークを作成できます。

```none
C:\> docker network create -d nat --subnet=172.16.0.0/16 --gateway=172.16.0.1 CustomNat1
C:\> docker network create -d nat --subnet=172.17.0.0/16 --gateway=172.17.0.1 CustomNat2
```

新しく作成されたネットワークは次を利用して一覧表示できます。
```none
C:\> docker network ls
```


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

## Docker Compose とサービス検出

> Docker Compose とサービス検出を利用して複数サービスのスケールアウト アプリケーションを定義する方法の実例は、[仮想化ブログ](https://blogs.technet.microsoft.com/virtualization/)の[投稿](https://blogs.technet.microsoft.com/virtualization/2016/10/18/use-docker-compose-and-service-discovery-on-windows-to-scale-out-your-multi-service-container-application/)でご確認いただけます。

### Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) を利用し、コンテナー ネットワークをそれらのネットワークを使用するコンテナー/サービスと一緒に定義し、構成できます。 コンテナーの接続先となるネットワークを定義するとき、Compose 'ネットワーク' キーが最上位キーとして使用されます。 たとえば、下の構文では、Docker により作成された既存の NAT ネットワークが特定の Compose ファイルに定義されているすべてのコンテナー/サービスの '既定の' ネットワークになります。

```none
networks:
 default:
  external:
   name: "nat"
```

同様に、次の構文を利用し、カスタム NAT ネットワークを定義できます。

> 注: 下の例で定義されている 'カスタム NAT ネットワーク' はコンテナー ホストの既存の NAT 内部プレフィックスのパーティションとして定義されています。 詳細については、上記の「複数の NAT ネットワーク」セクションを参照してください。

```none
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.17.0.0/16
```

Docker Compose を利用し、コンテナー ネットワークを定義/構成する方法については、「[Compose ファイル参照](https://docs.docker.com/compose/compose-file/)」を参照してください。

### サービス検出
Docker にはサービス検出が組み込まれています。これはサービス登録を処理し、コンテナーとサービスを対象に名前を IP (DNS) にマッピングします。サービス検出を利用すれば、すべてのコンテナー エンドポイントが互いを名前 (コンテナー名またはサービス名) で検出できます。 これは特に、複数のコンテナー エンドポイントを利用して 1 つのサービスを定義するような、スケールアウト シナリオで便利です。 そのような場合、サービス検出を利用すると、背後で実行しているコンテナーの数に関係なく、1 つのサービスが 1 つのエンティティとして見なされます。 複数コンテナーのサービスの場合、入ってくるネットワーク トラフィックがラウンドロビン方式で管理されます。この方式では、DNS 負荷分散を利用し、特定のサービスを実行するすべてのコンテナー インスタンスにトラフィックが均一に分散されます。

## オーバーレイ ネットワークと Docker swarm モード (複数ノード コンテナーのネットワー キング)
ネイティブのオーバーレイ ネットワーク ドライバーと Docker swarm モードを組み合わせて、Windows 上で複数ノード (クラスター) のシナリオをサポートできます。 オーバーレイと swarm モードについて詳しくは、Windows 10 Windows Insiders でのオーバーレイと swarm のリリースに伴う[ブログ記事](https://blogs.technet.microsoft.com/virtualization/2017/02/09/overlay-network-driver-with-support-for-docker-swarm-mode-now-available-to-windows-insiders-on-windows-10/)または「[swarm モードの概要](./swarm-mode.md)」のトピックをご覧ください。

## 注意事項と潜在的な問題

### 透過ネットワークの作成を阻む既存の vSwitch

透過ネットワークを作成するとき、Docker はネットワークの外部 vSwitch を作成し、スイッチと (外部) ネットワーク アダプターの関連付けを試行します。アダプターは VM ネットワーク アダプターまたは物理ネットワーク アダプターになります。 vSwitch が既にコンテナー ホストで作成されており、*Docker に表示される*場合、Windows Docker エンジンは新しいスイッチを作成せず、そのスイッチを使用します。 ただし、vSwitch が帯域外で作成され (すなわち、Hyper-V Manager または PowerShell を利用し、コンテナー ホストで作成され)、Docker にはまだ表示されていない場合、Windows Docker エンジンは新しい vSwitch の作成を試みます。新しいスイッチをコンテナー ホストの外部ネットワーク アダプターに接続することはできません (ネットワーク アダプターが帯域外で作成されたスイッチに接続されるため)。

たとえば、最初に Docker サービスの実行中にホストで新しい vSwitch を作成し、次に透過ネットワークを作成しようとすると、この問題が発生します。 その場合、Docker はユーザーが作成したスイッチを認識せず、透過ネットワークのために新しい vSwitch を作成します。

この問題は 3 つの方法で解決できます。

* 帯域外で作成された vSwitch はもちろん削除できます。削除後、Docker は新しい vSwitch を作成し、それを問題なくホスト ネットワーク アダプターに接続できます。 この方法を選択する前に、帯域外 vSwitch が他のサービス (Hyper-V など) により利用されていないことを確認します。
* あるいは、帯域外で作成された外部 vSwitch を使用する場合、Docker と HNS サービスを再起動し、*Docker にスイッチが表示される*ようにします。
```none
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* もう 1 つの選択肢は、'-o com.docker.network.windowsshim.interface' オプションを利用し、透過ネットワークの外部 vSwitch をコンテナー ホストでまだ使用されていない特定のネットワーク アダプター (すなわち、帯域外で作成された vSwitch で利用されているネットワーク アダプター以外のネットワーク アダプター) に関連付けることです。 '-o' オプションに関する説明は、本書の「[透過ネットワーク](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)」セクションに記載されています。

### サポートされていない機能

現在、Docker CLI では次のネットワーク機能はサポートされていません。
 * コンテナーのリンク (例: --link)

現在、Windows Docker では次のネットワーク オプションはサポートされていません。
 * --add-host
 * --dns-opt
 * --dns-search
 * --aux-address
 * --internal
 * --ip-range

