---
title: Kubernetes のトラブルシューティング
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Kubernetes の展開と Windows ノードの参加で発生する一般的な問題の解決方法。
keywords: kubernetes、1.14、linux、compile
ms.openlocfilehash: 54396f4b350fa7dfe59e073601f41b0a73f06dca
ms.sourcegitcommit: 76dce6463e820420073dda2dbad822ca4a6241ef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/25/2019
ms.locfileid: "10307265"
---
# Kubernetes のトラブルシューティング #
このページでは、Kubernetes のセットアップ、ネットワーク、および展開に関する一般的な問題について説明します。

> [!tip]
> PR を[ドキュメント リポジトリ](https://github.com/MicrosoftDocs/Virtualization-Documentation/)に投稿することで、よく寄せられる質問の項目をご提案ください。

このページは、次のカテゴリに分類されます。
1. [一般的な質問](#general-questions)
2. [一般的なネットワークエラー](#common-networking-errors)
3. [Windows の一般的なエラー](#common-windows-errors)
4. [一般的な Kubernetes マスターのエラー](#common-kubernetes-master-errors)

## 一般的な質問 ##

### Windows での起動が正常に完了したことを確認するにはどうすればよいですか? ###
Kubelet、kube、および (ネットワークソリューションとしてフランネルを選択した場合は) ノードで実行されているログが個別の PoSh ウィンドウに表示されている状態で、ホストエージェントのプロセスを表示する必要があります。 さらに、Windows ノードが Kubernetes クラスターに "Ready" として表示されるようにする必要があります。

### このすべてを、PoSh windows ではなくバックグラウンドで実行するように設定できますか? ###
Kubernetes バージョン1.11 以降、kubelet & kube-proxy は、ネイティブの[Windows サービス](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)として実行できます。 また、 [nssm](https://nssm.cc/)のような代替のサービスマネージャーを使って、常にバックグラウンドでこれらのプロセス (flanneld、& kubelet、kube-proxy) を実行することもできます。 手順の例については、「 [Kubernetes の Windows サービス](./kube-windows-services.md)」を参照してください。

### Windows サービスとしての Kubernetes プロセスの実行で問題が発生する ###
最初のトラブルシューティングでは、 [nssm](https://nssm.cc/)で次のフラグを使って、stdout と stderr を出力ファイルにリダイレクトすることができます。
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
詳細については、「公式[nssm の使用](https://nssm.cc/usage)に関するドキュメント」を参照してください。

## 一般的なネットワークエラー ##

### ロードバランサーがクラスターノード間で一貫性がない ###
(既定) kube プロキシ構成では、100 + ロードバランサーを含むクラスターは、利用可能な一時的 TCP ポートを使用しなくなる可能性があります (別名 動的ポート範囲 (DSR 以外の) ロードバランサーの各ノードで予約されているポートの数が多いため、通常は、ポート 49152 ~ 65535 を対象としています。 これにより、次のような kube のエラーが表示されることがあります。
```
Policy creation failed: hcnCreateLoadBalancer failed in Win32: The specified port already exists.
```

ユーザーがこの問題を特定するには、 [Collectlogs. ps1](https://github.com/microsoft/SDN/blob/master/Kubernetes/windows/debug/collectlogs.ps1)スクリプト`*portrange.txt`を実行して、ファイルを確認します。

また`CollectLogs.ps1` 、では、一時的な TCP ポートの範囲でポートプールの割り当ての可用性をテストし、の`reservedports.txt`成功/失敗を報告するために、HNS の割り当てロジックにも似ています。 このスクリプトは、10個の 64 TCP 短期ポート (HNS の動作をエミュレートするため) を予約して、予約の成功 & 失敗し、割り当てられたポート範囲を解放します。 成功数が10未満の場合、短期プールの空き領域が不足していることを示します。 Heuristical の64ブロックポート予約の数の概要は、に`reservedports.txt`も生成されます。

この問題を解決するには、いくつかの手順を実行します。
1.  永続的な解決策としては、kube の負荷分散を[DSR モード](https://techcommunity.microsoft.com/t5/Networking-Blog/Direct-Server-Return-DSR-in-a-nutshell/ba-p/693710)に設定する必要があります。 残念ながら、DSR モードは新しい[Windows Server Insider build 18945](https://blogs.windows.com/windowsexperience/2019/07/30/announcing-windows-server-vnext-insider-preview-build-18945/#o1bs7T2DGPFpf7HM.97) (またはそれ以降) のみに完全に実装されています。
2. 回避策として、ユーザーは、などのコマンドを使用して、利用可能な一時的`netsh int ipv4 set dynamicportrange TCP <start_port> <port_count>`なポートの既定の Windows 構成を増やすこともできます。 *警告:* 既定の動的ポートの範囲を上書きすると、ホスト上の非短期範囲の利用可能な TCP ポートに依存する他のプロセス/サービスに影響を与える可能性があるため、この範囲は慎重に選ぶ必要があります。
3. また、2020年第1四半期の累積的な更新プログラムでリリースされる予定のインテリジェントポートプール共有を使用して、非 DSR モードのロードバランサーのスケーラビリティを強化しています。

### HostPort publishing が機能していない ###
現時点では、Kubernetes `containers.ports.hostPort`フィールドを使ってポートを公開することはできません。このフィールドは Windows CNI プラグインによって適用されるわけではありません。 ノードでポートを公開する時間には、NodePort publishing を使用してください。

### 「Win32 での実行に失敗しました。」というエラーが表示される (ドライブに間違ったフロッピーディスクがあります)。 ###
このエラーは、古い HNS オブジェクトを切断せずに、HNS オブジェクトに対してカスタムの変更を行ったり、新しい Windows Update をインストールして、HNS に変更を加えるときに発生する可能性があります。 更新前に以前に作成された HNS オブジェクトが、現在インストールされている HNS バージョンと互換性がないことを示します。

Windows Server 2019 (および以下) では、ユーザーは、HNS のデータファイルを削除して、HNS オブジェクトを削除できます。 
```
Stop-Service HNS
rm C:\ProgramData\Microsoft\Windows\HNS\HNS.data
Start-Service HNS
```

ユーザーは、互換性のない HNS エンドポイントまたはネットワークを直接削除できる必要があります。
```
hnsdiag list endpoints
hnsdiag delete endpoints <id>
hnsdiag list networks 
hnsdiag delete networks <id>
Restart-Service HNS
```

Windows Server バージョン1903のユーザーは、次のレジストリの場所に移動し、ネットワーク名で始まるすべての Nic を削除`vxlan0`する`cbr0`ことができます (例:)。
```
\\Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\vmsmp\parameters\NicList
```

### Azure でのフランネルのホストのコンテナーの展開がインターネットに接続できない ###
Azure でフランネルをホスト-gw モードで展開する場合、パケットは Azure 物理ホスト vSwitch を通過する必要があります。 ユーザーは、ノードに割り当てられている各サブネットに対して、種類が "仮想アプライアンス" の[ユーザー定義のルート](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-udr-overview#user-defined)をプログラムする必要があります。 これは、Azure ポータル ([ここで](https://docs.microsoft.com/en-us/azure/virtual-network/tutorial-create-route-table-portal)は例を参照) または azure `az` CLI 経由で行うことができます。 次に示すのは、IP 10.0.0.4 とそれぞれの pod subnet 10.244.0.0/24 を持つノードに対して az コマンドを使った "MyRoute" という名前の UDR の例です。
```
az network route-table create --resource-group <my_resource_group> --name BridgeRoute 
az network route-table route create  --resource-group <my_resource_group> --address-prefix 10.244.0.0/24 --route-table-name BridgeRoute  --name MyRoute --next-hop-type VirtualAppliance --next-hop-ip-address 10.0.0.4 
```

>[!TIP]
> 他のクラウドプロバイダーから Azure または IaaS Vm に Kubernetes を展開している場合は、代わりに[オーバーレイネットワーク](./network-topologies.md#flannel-in-vxlan-mode)を使うこともできます。

### Windows ポッドが外部リソースに ping できない ###
Windows ポッドには、現在 ICMP プロトコル用にプログラムされた送信ルールはありません。 ただし、TCP/UDP はサポートされています。 クラスター外のリソースへの接続を示す場合は、対応する`ping <IP>` `curl <IP>`コマンドに置き換えてください。

それでも問題が解決されない場合は、 [cni](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)のネットワーク構成で、十分な注意を払う必要があります。 この静的ファイルはいつでも編集できます。構成は、新しく作成された Kubernetes リソースに適用されます。

どうしてでしょうか。
Kubernetes のネットワーク要件の1つ ( [Kubernetes モデル](https://kubernetes.io/docs/concepts/cluster-administration/networking/)を参照) は、内部で NAT を使わずにクラスター通信を行うために使用されます。 この要件を満たすために、送信 NAT が行われないようにするすべての通信のための "の追加されていない[" という](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)情報が含まれています。 ただし、これは、クエリを実行しようとしている外部 IP を例外除外から除外する必要があることも意味します。 その後、Windows ポッドから発信されたトラフィックは、外部の世界からの応答を受信するために正しく送信されます。 この点を考慮して、 `cni.conf`の場合は次のようにします。
```conf
"ExceptionList": [
  "10.244.0.0/16",  # Cluster subnet
  "10.96.0.0/12",   # Service subnet
  "10.127.130.0/24" # Management (host) subnet
]
```

### Windows ノードが NodePort サービスにアクセスできない ###
ノード自体からのローカルの NodePort へのアクセスは失敗します。 これは既知の制限です。 NodePort access は、他のノードまたは外部クライアントから機能します。

### しばらくすると、コンテナーの vNICs と HNS のエンドポイントが削除されます。 ###
この問題は、 `hostname-override`パラメーターが[kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)に渡されない場合に発生する可能性があります。 これを解決するには、次のようにして、ユーザーのホスト名を kube プロキシに渡す必要があります。
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### フランネル (vxlan) モードでは、ノードを再び参加した後に、ポッドに接続の問題が発生しています。 ###
以前に削除されたノードが再びクラスターに再参加するたびに、flannelD は新しい pod サブネットをノードに割り当てようとします。 ユーザーは、次のパスにある古い pod サブネット構成ファイルを削除する必要があります。
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### Flanneld を起動すると、"ネットワークの作成を待っています" という画面が止まってしまいます。 ###
この問題については、多数のレポートが調査中です。ほとんどの場合、フランネルネットワークの管理 IP が設定されている場合のタイミングの問題である可能性があります。 回避策として、次の手順に従って、単に ps1 を再起動するか、手動で再起動する必要があります。
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

また、[現在のレビュー] でこの問題に対処する[PR](https://github.com/coreos/flannel/pull/1042)もあります。


### /Run/flannel/subnet.env が見つからないため、Windows ポッドを起動できません。 ###
これは、フランネルが正常に起動されなかったことを示します。 Flanneld を再起動するか、Kubernetes マスターから`/run/flannel/subnet.env` Windows ワーカーノード`C:\run\flannel\subnet.env`に手動でファイルをコピーして、割り当てられたサブネットに`FLANNEL_SUBNET`行を変更することができます。 たとえば、ノードサブネット 10.244.4.1/24 が割り当てられている場合は、次のようになります。
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```
Flanneld を使うと、このファイルを自動的に生成できます。


### VSphere で実行されている Kubernetes クラスターでホスト間の pod 間接続が切断される 
VSphere とフランネルは両方ともオーバーレイネットワーク用にポート 4789 (既定 VXLAN ポート) を予約しているため、パケットが傍受される可能性があります。 VSphere がオーバーレイネットワークに使用されている場合、4789を解放するために別のポートを使用するように構成する必要があります。  


### エンドポイント/IPs でリークが発生しています ###
エンドポイントでリークが発生する可能性のある既知の問題が2つ存在します。 
1.  Kubernetes バージョン1.11 では、最初の[既知の問題](https://github.com/kubernetes/kubernetes/issues/68511)が発生しています。 Kubernetes バージョン 1.11.0-1.11.2 を使用しないようにしてください。
2. エンドポイントのリークを引き起こす可能性のある2番目の[既知の問題](https://github.com/docker/libnetwork/issues/1950)は、エンドポイントのストレージでの同時実行の問題です。 修正プログラムを受け取るには、Docker EE 18.09 以降を使用する必要があります。

### "ネットワーク: 範囲に割り当てられませんでした" というエラーが原因で pod が起動できない ###
これは、ノードの IP アドレス空間が使用されていることを示します。 リークした[エンドポイント](#my-endpointsips-are-leaking)をクリーンアップするには、影響を受けるノードのすべてのリソースを移行して、次のコマンドを実行 & ます。
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### Windows ノードがサービス IP を使用してサービスにアクセスできない ###
これは、Windows の現在のネットワーク スタックの既知の制限です。 ただし、Windows*ポッド***は**サービス IP にアクセスできます。

### Kubelet を起動するときにネットワーク アダプターが見つからない ###
Kubernetes ネットワークが機能するには、Windows ネットワーク スタックで仮想アダプターが必要です。 次のコマンドが (管理シェルに) 結果を返さない場合、仮想ネットワークの作成 &mdash; Kubelet が機能するのに必要な前提条件 &mdash; に失敗しています。

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

多くの場合、ホストのネットワークアダプターが "Ethernet" でない場合に、 [InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)パラメーターを変更することをお勧めします。 それ以外の場合は、仮想`start-kubelet.ps1`ネットワークの作成中にエラーが発生しているかどうかを確認するために、スクリプトの出力を参照してください。 

### しばらく稼働した後、ポッドが正常な DNS クエリの解決を停止する ###
Windows Server のネットワークスタック、バージョン1803以降に既知の DNS キャッシュの問題がある場合、DNS 要求が失敗する可能性があります。 この問題を回避するには、次のレジストリキーを使用して、最大 TTL キャッシュの値を0に設定します。

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### まだ問題が発生しています。 どうしたらいいでしょう。 ### 
ネットワークまたはホストに追加の制約があり、ノード間で特定の種類の通信が妨げられている場合があります。 次のことを確認してください。
  - 選択した[ネットワークトポロジ](./network-topologies.md)が適切に構成されている
  - ポッドからと思われるトラフィックが許可されていること
  - HTTP トラフィックが許可されていること (Web サービスを展開する場合)
  - さまざまなプロトコル (ie ICMP と TCP/UDP) からのパケットがドロップされない

>[!TIP]
> その他のセルフヘルプリソースについては、Windows 用の Kubernetes のトラブルシューティングガイドも[参照](https://techcommunity.microsoft.com/t5/Networking-Blog/Troubleshooting-Kubernetes-Networking-on-Windows-Part-1/ba-p/508648)してください。

## Windows の一般的なエラー ##

### 使用している Kubernetes ポッドが "ContainerCreating" で停止する ###
この問題には多くの原因が考えられますが、最も一般的な原因の 1 つは、pause イメージが正しく構成されていないことです。 これは、次の問題の大きな兆候です。


### 展開するときに、Docker コンテナーが再起動を繰り返す ###
pause イメージが、OS のバージョンと互換性があることを確認します。 この[手順](./deploying-resources.md)では、OS とコンテナーの両方がバージョン1803であることを前提としています。 Insider ビルドなど、新しいバージョンの Windows を使用する場合は、イメージを調整する必要があります。 イメージについては、マイクロソフトの [Docker リポジトリ](https://hub.docker.com/u/microsoft/)を参照してください。 いずれの場合も、pause イメージ Dockerfile とサンプル サービスの両方で、イメージに `:latest` にタグ付けされている必要があります。


## 一般的な Kubernetes マスターのエラー ##
Kubernetes マスターのデバッグは、(発生しやすい順で) 次の 3 つのカテゴリに分類されます。

  - Kubernetes システム コンテナーに問題がある。
  - `kubelet` の実行状態に問題がある。
  - システムに問題がある。

Kubernetes によるポッドの作成を確認するには、`kubectl get pods -n kube-system` を実行します。これにより、いずれかがクラッシュしていたり正しく開始していないとわかることがあります。   次に `docker ps -a` を実行して、これらのポッドの背後にあるすべての未加工コンテナーを表示します。 最後に、問題の原因となっている疑いがあるコンテナーに対して `docker logs [ID]` を実行し、プロセスの未加工出力を確認します。


###  で API サーバーに接続できない `https://[address]:[port]` ###
このエラーはほとんどの場合、証明書の問題を示しています。 構成ファイルを正しく生成していること、その中の IP アドレスがホストに対応していること、API サーバーによってマウントされたディレクトリにコピー済みであることを確認します。

この[手順](./creating-a-linux-master.md)に従う場合は、次のことを確認してください。   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 それ以外の場合は、API サーバーのマニフェストファイルを参照して、マウントポイントを確認します。
