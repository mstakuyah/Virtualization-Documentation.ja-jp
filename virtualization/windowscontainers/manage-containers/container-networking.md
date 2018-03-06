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
ms.openlocfilehash: 70e662e73693d9fef9d36635289ec9affb8d0331
ms.sourcegitcommit: f542e8c95b5bb31b05b7c88f598f00f76779b519
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/01/2018
---
# <a name="windows-container-networking"></a>Windows コンテナーのネットワーク
> ***一般的な Docker ネットワークのコマンド、オプション、構文については、[Docker コンテナーのネットワークに関するトピック](https://docs.docker.com/engine/userguide/networking/)を参照してください。*** このドキュメントに記載されている場合を除き、Docker ネットワーク コマンドはすべて、Linux と同じ構文を使用して、Windows ですべてサポートされています。 ただし、Windows と Linux のネットワーク スタックは異なっており、そのためいくつかの Linux ネットワーク コマンド (ifconfig など) は、Windows ではサポートされていません。

## <a name="basic-networking-architecture"></a>基本的なネットワーク アーキテクチャ
このトピックでは、Docker で Windows 上にネットワークを作成して管理する方法の概要を示します。 ネットワークに関して言えば、Windows コンテナーの機能は仮想マシンと似ています。 各コンテナーには、Hyper-V 仮想スイッチ (vSwitch) に接続されている仮想ネットワーク アダプター (vNIC) があります。 Windows では、Docker 経由で作成できる、*nat*、*overlay*、*transparent*、*l2bridge*、*l2tunnel* の 5 種類のネットワーク ドライバーまたはモードをサポートしています。 物理ネットワークのインフラストラクチャと単一または複数のホストのネットワーク要件に応じて、ニーズに最適なネットワーク ドライバーを選択する必要があります。

<figure>
  <img src="media/windowsnetworkstack-simple.png">
</figure>  

初めて Docker エンジンを実行したときに、内部 vSwitch と `WinNAT` という名前の Windows コンポーネントを使用する、既定の NAT ネットワーク 'nat' が作成されます。 PowerShell または Hyper-V マネージャーで作成されたホストに既存の外部 vSwitch がある場合、*transparent* ネットワーク ドライバーを使用して Docker でも利用でき、``docker network ls`` コマンドを実行したときに表示できます。  

<figure>
  <img src="media/docker-network-ls.png">
</figure>

> - ***内部*** vSwitch はコンテナー ホスト上のネットワーク アダプターに直接接続されていない vSwitch です。 

> - ***外部*** vSwitch はコンテナー ホスト上のネットワーク アダプターに直接接続されて_いる_ vSwitch です。  

<figure>
  <img src="media/get-vmswitch.png">
</figure>

'nat' ネットワークとは、Windows で実行されているコンテナーの既定のネットワークです。 特定のネットワーク構成を実装するフラグや引数を指定せずに Windows で実行されているすべてのコンテナーは、既定の 'nat' ネットワークに接続され、'nat' ネットワークの内部プレフィックス IP 範囲から自動的に IP アドレスが割り当てられます。 'nat' 用に使用される既定の内部 IP プレフィックスは、172.16.0.0/16 です。 


## <a name="windows-container-network-drivers"></a>Windows コンテナーのネットワーク ドライバー  

Windows で Docker によって作成された既定の 'nat' ネットワークを活用することに加えて、ユーザーはカスタム コンテナー ネットワークを定義できます。 ユーザー定義のネットワークは、Docker CLI の [`docker network create -d <NETWORK DRIVER TYPE> <NAME>`](https://docs.docker.com/engine/reference/commandline/network_create/) コマンドを使用して作成できます。 Windows では、次の種類のネットワーク ドライバーを利用できます。

- **nat**: 'nat' ドライバーを使用して作成されたネットワークに接続されているコンテナーには、ユーザーが指定した (``--subnet``) IP プレフィックスから IP アドレスが割り当てられます。 コンテナー ホストからコンテナー エンドポイントへのポート フォワーディングおよびマッピングがサポートされています。
> 注: Windows 10 Creators Update では、複数の NAT ネットワークがサポートされるようになりました。 

- **transparent**: 'transparent' ドライバーを使用して作成されたネットワークに接続されているコンテナーは、物理ネットワークに直接接続されます。 物理ネットワークの IP は、静的に割り当てることも (ユーザー指定の ``--subnet`` オプションが必要)、外部の DHCP サーバーを使用して動的に割り当てることもできます。 

- **overlay** - __新機能。__  Docker エンジンが [swarm モード](./swarm-mode.md)で動作している場合、オーバーレイ ネットワークに接続されたコンテナーは、複数のコンテナー ホストを越えて、同じネットワークに接続された他のコンテナーと通信できます。 Swarm クラスター上に作成される各オーバーレイ ネットワークは、プライベート IP プレフィックスによって定義される独自の IP サブネットを使って作成されます。 overlay ネットワーク ドライバーは、VXLAN カプセル化を使用します。
> Windows Server 2016 と [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) または Windows 10 Creators Update が必要です。 

- **l2bridge**: 'l2bridge' ドライバーを使用して作成されたネットワークに接続されているコンテナーは、コンテナー ホストと同じ IP サブネット内に存在します。 IP アドレスは、コンテナー ホストと同じプレフィックスから静的に割り当てる必要があります。 ホスト上のすべてのコンテナーのエンドポイントは、入口と出口でのレイヤー 2 のアドレス変換 (MAC の再書き込み) のために同じ MAC アドレスとなります。
> Windows Server 2016 または Windows 10 Creators Update が必要です。

- **l2tunnel** - _このドライバーは Microsoft Cloud スタックでのみ使用する必要があります_。

> Microsoft SDN スタックを使用してオーバーレイの仮想ネットワークをコンテナー エンドポイントに接続する方法については、「[Attaching Containers to a Virtual Network](https://technet.microsoft.com/en-us/windows-server-docs/networking/sdn/manage/connect-container-endpoints-to-a-tenant-virtual-network)」 (仮想ネットワークにコンテナーを接続する) のトピックをご覧ください。

> Windows 10 Creators Update には、実行中のコンテナーに新しいコンテナー エンドポイントを追加する (つまり、ホット アド) ためのプラットフォームのサポートが導入されました。 これにより、エンド ツー エンドで保留される[未処理の Docker のプル要求](https://github.com/docker/libnetwork/pull/1661)を処理できます。

## <a name="network-topologies-and-ipam"></a>ネットワーク トポロジと IPAM
次の表に、各ネットワーク ドライバーの内部 (コンテナー間) および外部接続で、ネットワーク接続がどのように提供されるかを示します。

<figure>
  <img src="media/network-modes-table.png">
</figure>

### <a name="ipam"></a>IPAM 
各ネットワーク ドライバーで、IP アドレスの割り当ては異なります。 Windows では、ホスト ネットワーク サービス (HNS) を使用して nat ドライバーに IPAM を提供し、Docker の Swarm モード (内部 KVS) と連携して overlay に IPAM を提供します。 その他のすべてのネットワーク ドライバーは、外部の IPAM を使用します。

<figure>
  <img src="media/ipam.png">
</figure>

# <a name="details-on-windows-container-networking"></a>Windows コンテナーのネットワークの詳細

## <a name="isolation-namespace-with-network-compartments"></a>ネットワーク コンパートメントによる分離 (名前空間)
各コンテナー エンドポイントは、Linux のネットワーク名前空間に似た、それぞれの__ネットワーク コンパートメント__に配置されます。 管理ホストの vNIC とホスト ネットワーク スタックは、既定のネットワーク コンパートメントに置かれます。 コンテナーのネットワーク アダプターがインストールされている Windows Server と Hyper-V コンテナーには、同じホスト上の複数のコンテナー間でネットワークの分離を強制するために、ネットワーク コンパートメントがそれぞれ作成されます。 Windows Server コンテナーでは、仮想スイッチへの接続に Host vNIC を使用します。 Hyper-V コンテナーでは、仮想スイッチへの接続に (ユーティリティ VM には公開されていない) 統合 VM NIC を使用します。 

<figure>
  <img src="media/network-compartment-visual.png">
</figure>

```powershell 
Get-NetCompartment
```

## <a name="windows-firewall-security"></a>Windows ファイアウォールのセキュリティ

Windows ファイアウォールは、ポートの ACL によってネットワークのセキュリティを強化するために使用されます。

> 注: 既定では、オーバーレイ ネットワークに接続されているすべてのコンテナー エンドポイントについて、すべて許可する規則が作成されます。   

<figure>
  <img src="media/windows-firewall-containers.png">
</figure>

## <a name="container-network-management-with-host-network-service"></a>ホスト ネットワーク サービスによるコンテナーのネットワークの管理

次の図は、ホスト ネットワーク サービス (HNS) とホスト コンピューティング サービス (HCS) が連携してコンテナーを作成し、エンドポイントをネットワークに接続する仕組みを示しています。 

<figure>
  <img src="media/HNS-Management-Stack.png">
</figure>

# <a name="advanced-network-options-in-windows"></a>Windows での高度なネットワーク オプション
Windows に固有の機能を活用するために、いくつかのネットワーク ドライバーのオプションがサポートされています。 

## <a name="switch-embedded-teaming-with-docker-networks"></a>Docker ネットワークを使用したスイッチ埋め込みチーミング

> すべてのネットワーク ドライバーに適用されます。 

Docker で使用するコンテナー ホスト ネットワークを作成するときに、`-o com.docker.network.windowsshim.interface` オプションで複数のネットワーク アダプターを (コンマで区切って) 指定することにより、[スイッチ埋め込みチーミング](https://technet.microsoft.com/en-us/windows-server-docs/networking/technologies/hyper-v-virtual-switch/rdma-and-switch-embedded-teaming#a-namebkmksswitchembeddedaswitch-embedded-teaming-set)を活用できます。 

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2", "Ethernet 3" TeamedNet
```

## <a name="set-the-vlan-id-for-a-network"></a>ネットワークの VLAN ID の設定

> transparent および l2bridge ネットワーク ドライバーに適用されます。 

ネットワークの VLAN ID を設定するには、`docker network create` コマンドに `-o com.docker.network.windowsshim.vlanid=<VLAN ID>` オプションを使用します。 たとえば、次のコマンドを使用して、VLAN ID が 11 である透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.vlanid=11 MyTransparentNetwork
```
ネットワークの VLAN ID を設定すると、そのネットワークに接続されるあらゆるコンテナー エンドポイントの VLAN 分離が設定されます。

> タグ付けされたすべてのトラフィックを正しい VLAN 上でアクセス モードの vNIC (コンテナー エンドポイント) ポートを持つ vSwitch によって処理するためには、(物理的) ホスト ネットワーク アダプターがトランク モードであることを確認します。

## <a name="specify-the-name-of-a-network-to-the-hns-service"></a>HNS サービスへのネットワーク名の指定

> すべてのネットワーク ドライバーに適用されます。 

通常、`docker network create` を使ってコンテナー ネットワークを作成した場合、使用したネットワーク名は Docker サービスでは使われますが、HNS サービスでは使われません。 ネットワークを作成する場合、`docker network create` コマンドで `-o com.docker.network.windowsshim.networkname=<network name>` オプションを使うと、HNS サービスから提供された名前を指定できます。 たとえば、次のコマンドを使用すると、HNS サービスに対して指定された名前を使って透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.networkname=MyTransparentNetwork MyTransparentNetwork
```

## <a name="bind-a-network-to-a-specific-network-interface"></a>特定のネットワーク インターフェイスへのネットワークのバインド

> 'nat' を除くすべてのネットワーク ドライバーに適用されます。  

ネットワーク (Hyper-V 仮想スイッチを通じて接続された) を特定のネットワーク インターフェイスにバインドするには、`docker network create` コマンドで `-o com.docker.network.windowsshim.interface=<Interface>` オプションを使います。 たとえば、次のコマンドを使うと、"Ethernet 2" ネットワーク インターフェイスに接続された透過ネットワークを作成できます。

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" TransparentNet2
```

> 注: *com.docker.network.windowsshim.interface* の値は、ネットワーク アダプターの*名前*です。これは次のコマンドを使って確認できます。

>```
PS C:\> Get-NetAdapter
```
## Specify the DNS Suffix and/or the DNS Servers of a Network

> Applies to all network drivers 

Use the option, `-o com.docker.network.windowsshim.dnssuffix=<DNS SUFFIX>` to specify the DNS suffix of a network, and the option, `-o com.docker.network.windowsshim.dnsservers=<DNS SERVER/S>` to specify the DNS servers of a network. For example, you might use the following command to set the DNS suffix of a network to "example.com" and the DNS servers of a network to 4.4.4.4 and 8.8.8.8:

```
C:\> docker network create -d transparent -o com.docker.network.windowsshim.dnssuffix=abc.com -o com.docker.network.windowsshim.dnsservers=4.4.4.4,8.8.8.8 MyTransparentNetwork
```

## VFP

The Virtual Filtering Platform (VFP) extension is a Hyper-V virtual switch, forwarding extension used to enforce network policy and manipulate packets. For instance, VFP is used by the 'overlay' network driver to perform VXLAN encapsulation and by the 'l2bridge' driver to perform MAC re-write on ingresss and egress. The VFP extension is only present on Windows Server 2016 and Windows 10 Creators Update. To check and see if this is running correctly a user run two commands:

```powershell
Get-Service vfpext

# This should indicate the extension is Running: True 
Get-VMSwitchExtension  -VMSwitchName <vSwitch Name> -Name "Microsoft Azure VFP Switch Extension"
```

## <a name="tips--insights"></a>ヒントと情報
ここでは、Windows コンテナーのネットワークに関してコミュニティに寄せられたよくある質問をまとめた、便利なヒントと情報の一覧を示します。

#### <a name="hns-requires-that-ipv6-is-enabled-on-container-host-machines"></a>HNS では、コンテナーのホスト コンピューターで IPv6 が有効になっている必要がある 
[KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) でも説明されているように、HNSでは、Windows コンテナー ホストで IPv6 が有効になっている必要があります。 次のようなエラーが発生した場合は、ホスト コンピューターで IPv6 が無効になっている可能性があります。
```
docker: Error response from daemon: container e15d99c06e312302f4d23747f2dfda4b11b92d488e8c5b53ab5e4331fd80636d encountered an error during CreateContainer: failure in a Windows system call: Element not found.
```
この問題を自動的に検出/防止するために、プラットフォームの変更に取り組んでいます。 現在、次の回避策によって、ホスト コンピューターで確実に IPv6 を有効にすることができます。

```
C:\> reg delete HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip6\Parameters  /v DisabledComponents  /f
```

#### <a name="moby-linux-vms-use-dockernat-switch-with-docker-for-windows-a-product-of-docker-cehttpswwwdockercomcommunity-edition-instead-of-hns-internal-vswitch"></a>Moby Linux VM は、Docker for Windows ([Docker CE](https://www.docker.com/community-edition) の製品) で、HNS の内部 vSwitch の代わりに、DockerNAT スイッチ を使用する 
Windows 10 上の Docker for Windows (Docker CE エンジン用の Windows ドライバー) は、DockerNAT という内部 vSwitch を使って Moby Linux VM をコンテナー ホストに接続します。 Windows で Moby Linux VM を使用している開発者は、ホストが、HNS サービスによって作成される vSwitch (Windows コンテナーで使用される既定のスイッチ) ではなく、DockerNAT vSwitch を使用していることに注意する必要があります。 

#### <a name="to-use-dhcp-for-ip-assignment-on-a-virtual-container-host-enable-macaddressspoofing"></a>仮想コンテナー ホストでの IP の割り当てに DHCP を使用するには、MACAddressSpoofing を有効にする 
コンテナーのホストが仮想化されており、IP の割り当てに DHCP を使用したい場合は、仮想マシンのネットワーク アダプターで MACAddressSpoofing を有効にする必要があります。 それ以外の場合、Hyper-V ホストでは、複数の MAC アドレスを持つ VM 内のコンテナーからのネットワーク トラフィックをブロックします。 次の PowerShell コマンドを使用して、MACAddressSpoofing を有効にすることができます。
```
PS C:\> Get-VMNetworkAdapter -VMName ContainerHostVM | Set-VMNetworkAdapter -MacAddressSpoofing On
```
VMware をハイパーバイザーとして実行する場合は、正しく実行できるように無作為検出モードを有効にする必要があります。 詳しくは、[こちら](https://kb.vmware.com/s/article/1004099)をご覧ください。
#### <a name="creating-multiple-transparent-networks-on-a-single-container-host"></a>1 つのコンテナー ホストで複数の透過ネットワークを作成する
複数の透過ネットワークを作成する場合は、外部 Hyper-V 仮想スイッチをバインドする (仮想) ネットワーク アダプターを指定する必要があります。 ネットワークのインターフェイスを指定するには、次の構文を使います。
```
# General syntax:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface=<INTERFACE NAME> <NETWORK NAME> 

# Example:
C:\> docker network create -d transparent -o com.docker.network.windowsshim.interface="Ethernet 2" myTransparent2
```

#### <a name="remember-to-specify---subnet-and---gateway-when-using-static-ip-assignment"></a>静的 IP 割り当てを使用する場合は、必ず *--subnet* と *--gateway* を指定する
IP を静的に割り当てる場合、ネットワークの作成時に、*--subnet* および *--gateway* パラメーターが指定されていることを最初に確認する必要があります。 サブネットとゲートウェイの IP アドレスは、コンテナー ホスト (つまり、物理ネットワーク) のネットワーク設定と同じにする必要があります。 たとえば、静的 IP の割り当てを使用して、透過ネットワークを作成し、そのネットワーク上でエンドポイントを実行する方法は次のようになります。

```
# Example: Create a transparent network using static IP assignment
# A network create command for a transparent container network corresponding to the physical network with IP prefix 10.123.174.0/23
C:\> docker network create -d transparent --subnet=10.123.174.0/23 --gateway=10.123.174.1 MyTransparentNet
# Run a container attached to MyTransparentNet
C:\> docker run -it --network=MyTransparentNet --ip=10.123.174.105 windowsservercore cmd
```

#### <a name="dhcp-ip-assignment-not-supported-with-l2bridge-networks"></a>DHCP による IP 割り当ては L2Bridge ネットワークでサポートされない
l2bridge ドライバーを使って作成されたコンテナー ネットワークでは、静的 IP の割り当てのみがサポートされます。 前述のように、静的 IP の割り当て用に構成されたネットワークを作成するには、必ず *--subnet* と *--gateway* パラメーターを使用してください。

#### <a name="networks-that-leverage-external-vswitch-must-each-have-their-own-network-adapter"></a>外部 vSwitch を活用するネットワークでは独自のネットワーク アダプターが必要である
同じコンテナー ホストで、接続に外部 vSwitch を使用する複数のネットワーク (透過、L2 ブリッジ、L2 透過など) が作成されている場合、それぞれに独自のネットワーク アダプターが必要であることに注意してください。 

#### <a name="ip-assignment-on-stopped-vs-running-containers"></a>停止しているコンテナーと実行中のコンテナーでの IP の割り当て
静的な IP の割り当ては、コンテナーのネットワーク アダプターに対して直接行い、コンテナーが停止状態にある場合にのみ実行する必要があります。 コンテナーの実行中は、コンテナーのネットワーク アダプターの "ホット アド" またはネットワーク スタックに対する変更は実行できません (Windows Server 2016 の場合)。
> 注: この動作は、Windows 10 Creators Update では変更され、プラットフォームで "ホット アド" がサポートされるようになりました。 この機能によって、この[未処理の Docker プル要求](https://github.com/docker/libnetwork/pull/1661)が結合された後、E2E が開始されます。

#### <a name="existing-vswitch-not-visible-to-docker-can-block-transparent-network-creation"></a>既存の vSwitch (Docker には表示されない) が透過ネットワークの作成をブロックする場合がある
透過ネットワークの作成中にエラーが発生した場合、Docker で自動検出されなかった外部 vSwitch がシステムに存在し、それが透過ネットワークとコンテナー ホストの外部ネットワーク アダプターの関連付けを阻んでいる可能性があります。 

透過ネットワークを作成するとき、Docker はネットワークの外部 vSwitch を作成し、スイッチと (外部) ネットワーク アダプターの関連付けを試行します。アダプターは VM ネットワーク アダプターまたは物理ネットワーク アダプターになります。 vSwitch が既にコンテナー ホストで作成されており、*Docker に表示される*場合、Windows Docker エンジンは新しいスイッチを作成せず、そのスイッチを使用します。 ただし、vSwitch が帯域外で作成され (すなわち、Hyper-V Manager または PowerShell を利用し、コンテナー ホストで作成され)、Docker にはまだ表示されていない場合、Windows Docker エンジンは新しい vSwitch の作成を試みます。新しいスイッチをコンテナー ホストの外部ネットワーク アダプターに接続することはできません (ネットワーク アダプターが帯域外で作成されたスイッチに接続されるため)。

たとえば、最初に Docker サービスの実行中にホストで新しい vSwitch を作成し、次に透過ネットワークを作成しようとすると、この問題が発生します。 その場合、Docker はユーザーが作成したスイッチを認識せず、透過ネットワークのために新しい vSwitch を作成します。

この問題は 3 つの方法で解決できます。

* 帯域外で作成された vSwitch はもちろん削除できます。削除後、Docker は新しい vSwitch を作成し、それを問題なくホスト ネットワーク アダプターに接続できます。 この方法を選択する前に、帯域外 vSwitch が他のサービス (Hyper-V など) により利用されていないことを確認します。
* あるいは、帯域外で作成された外部 vSwitch を使用する場合、Docker と HNS サービスを再起動し、*Docker にスイッチが表示される*ようにします。
```
PS C:\> restart-service hns
PS C:\> restart-service docker
```
* もう 1 つの選択肢は、'-o com.docker.network.windowsshim.interface' オプションを利用し、透過ネットワークの外部 vSwitch をコンテナー ホストでまだ使用されていない特定のネットワーク アダプター (すなわち、帯域外で作成された vSwitch で利用されているネットワーク アダプター以外のネットワーク アダプター) に関連付けることです。 '-o' オプションに関する説明は、本書の「[透過ネットワーク](https://msdn.microsoft.com/virtualization/windowscontainers/management/container_networking#transparent-network)」セクションに記載されています。


## <a name="unsupported-features-and-network-options"></a>サポートされていない機能とネットワーク オプション 

次のネットワーク オプションは Windows でサポートされていないため、``docker run`` に渡すことができません。
 * コンテナーのリンク (例: ``--link``): _代わりにサービス検出を使用します_
 * IPv6 アドレス (例: ``--ip6``)
 * DNS オプション (例: ``--dns-option``)
 * 複数の DNS 検索ドメイン (例: ``--dns-search``)
 
次のネットワーク オプションおよび機能は Windows でサポートされていないため、``docker network create`` に渡すことができません。
 * --aux-address
 * --internal
 * --ip-range
 * --ipam-driver
 * --ipam-opt
 * --ipv6 

次のネットワーク オプションは、Docker サービスでサポートされていません。
* データプレーンの暗号化 (例: ``--opt encrypted``) 


## <a name="windows-server-2016-work-arounds"></a>Windows Server 2016 の回避策 

マイクロソフトでは新しい機能の追加と開発の促進を継続していますが、これらの機能の一部は以前のプラットフォームには移植されません。 最善の策として、Windows 10 および Windows Server の最新の更新プログラムを適用することをお勧めします。  以下のセクションでは、Windows Server 2016 と以前のバージョンの Windows 10 (つまり 1704 Creators Update より前のバージョン) に適用されるいくつかの回避策と注意事項を示します。

### <a name="multiple-nat-networks-on-ws2016-container-host"></a>WS2016 コンテナー ホスト上の複数の NAT ネットワーク

新しい NAT ネットワークのパーティションは、大規模な内部 NAT ネットワーク プレフィックスの下で作成する必要があります。 PowerShell から次のコマンドを実行し、"InternalIPInterfaceAddressPrefix" フィールドを参照すると、プレフィックスが見つかります。

```
PS C:\> Get-NetNAT
```

たとえば、ホストの NAT ネットワークの内部プレフィックスを 172.16.0.0/16 にします。 その場合、*172.16.0.0/16 プレフィックスのサブセットである限り*、Docker を利用して追加 NAT ネットワークを作成できます。 たとえば、IP プレフィックス 172.16.1.0/24 (ゲートウェイ、172.16.1.1) と 172.16.2.0/24 (ゲートウェイ、172.16.2.1) で 2 つの NAT ネットワークを作成できます。

```
C:\> docker network create -d nat --subnet=172.16.1.0/24 --gateway=172.16.1.1 CustomNat1
C:\> docker network create -d nat --subnet=172.16.2.0/24 --gateway=172.16.1.1 CustomNat2
```

新しく作成されたネットワークは次を利用して一覧表示できます。
```
C:\> docker network ls
```

### <a name="docker-compose"></a>Docker Compose

[Docker Compose](https://docs.docker.com/compose/overview/) を利用し、コンテナー ネットワークをそれらのネットワークを使用するコンテナー/サービスと一緒に定義し、構成できます。 コンテナーの接続先となるネットワークを定義するとき、Compose 'ネットワーク' キーが最上位キーとして使用されます。 たとえば、下の構文では、Docker により作成された既存の NAT ネットワークが特定の Compose ファイルに定義されているすべてのコンテナー/サービスの '既定の' ネットワークになります。

```
networks:
 default:
  external:
   name: "nat"
```

同様に、次の構文を利用し、カスタム NAT ネットワークを定義できます。

> 注: 下の例で定義されている 'カスタム NAT ネットワーク' はコンテナー ホストの既存の NAT 内部プレフィックスのパーティションとして定義されています。 詳細については、上記の「複数の NAT ネットワーク」セクションを参照してください。

```
networks:
  default:
    driver: nat
    ipam:
      driver: default
      config:
      - subnet: 172.16.3.0/24
```

Docker Compose を利用し、コンテナー ネットワークを定義/構成する方法については、「[Compose ファイル参照](https://docs.docker.com/compose/compose-file/)」を参照してください。

### <a name="service-discovery"></a>サービス検出
サービスの検出は、特定の Windows ネットワーク ドライバーについてのみサポートされます。

|  | ローカル サービス検出  | グローバル サービス検出 |
| :---: | :---------------     |  :---                |
| nat | 使用可能 | NA |  
| overlay | 使用可能 | 使用可能 |
| transparent | 使用不可 | 使用不可 |
| l2bridge | 使用不可 | 使用不可 |


