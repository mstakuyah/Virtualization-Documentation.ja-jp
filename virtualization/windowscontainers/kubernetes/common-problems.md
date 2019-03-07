---
title: Kubernetes のトラブルシューティング
author: gkudra-msft
ms.author: gekudray
ms.date: 11/02/2018
ms.topic: troubleshooting
ms.prod: containers
description: Kubernetes の展開と Windows ノードの参加で発生する一般的な問題の解決方法。
keywords: kubernetes、1.12、linux コンパイルします。
ms.openlocfilehash: 30bb0c064c96ff4bd0b6e1c078221b2d9170d4e7
ms.sourcegitcommit: 817a629f762a4a5d4bcff58302f2bc2408bf8be1
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/07/2019
ms.locfileid: "9149922"
---
# <a name="troubleshooting-kubernetes"></a>Kubernetes のトラブルシューティング #
このページでは、Kubernetes のセットアップ、ネットワーク、および展開に関する一般的な問題について説明します。

> [!tip]
> PR を[ドキュメント リポジトリ](https://github.com/MicrosoftDocs/Virtualization-Documentation/)に投稿することで、よく寄せられる質問の項目をご提案ください。

このページは、次のカテゴリに分割します。
1. [一般的な質問](#general-questions)
2. [ネットワークの一般的なエラー](#common-networking-errors)
3. [Windows の一般的なエラー](#common-windows-errors)
4. [一般的な Kubernetes マスター エラー](#common-kubernetes-master-errors)

## <a name="general-questions"></a>一般的な質問 ##

### <a name="how-do-i-know-startps1-on-windows-completed-successfully"></a>Windows が正常に完了 start.ps1 はどうすればよいですか。 ###
Kubelet が表示 kube プロキシを (選択した場合 Flannel ネットワーク ソリューションとして) と flanneld ホスト エージェント プロセスが動作し、ノード、上に表示されているログを実行している分割 PoSh windows します。 これに加えは、Windows ノードが Kubernetes クラスターで「準備が」として登録されている必要があります。

### <a name="can-i-configure-to-run-all-of-this-in-the-background-instead-of-posh-windows"></a>このすべてを PoSh の windows ではなく、バック グラウンドで実行するを構成することができますか。 ###
Kubernetes バージョン 1.11 から始めて、ネイティブ[Windows サービス](https://kubernetes.io/docs/getting-started-guides/windows/#kubelet-and-kube-proxy-can-now-run-as-windows-services)として kubelet & kube プロキシが実行できます。 常に、 [nssm.exe](https://nssm.cc/)などの別のサービス管理者を使用して、常に (flanneld、kubelet & kube プロキシ)、これらのプロセスがバック グラウンドで実行することができます。 [Kubernetes の Windows サービス](./kube-windows-services.md)の例の手順を参照してください。

### <a name="i-have-problems-running-kubernetes-processes-as-windows-services"></a>Windows サービスとして Kubernetes プロセスを実行している問題があります。 ###
初期のトラブルシューティング、出力ファイルに出力し、出力をリダイレクトするのには、 [nssm.exe](https://nssm.cc/)で次のフラグを使用できます。
```
nssm set <Service Name> AppStdout C:\k\mysvc.log
nssm set <Service Name> AppStderr C:\k\mysvc.log
```
詳細については、正式な[nssm 使用](https://nssm.cc/usage)ドキュメントを参照してください。

## <a name="common-networking-errors"></a>ネットワークの一般的なエラー ##

### <a name="my-windows-pods-do-not-have-network-connectivity"></a>[Windows ポッド ネットワーク接続がありません。 ###
仮想マシンを使用している場合は、MAC のスプーフィングが VM のすべてのネットワーク アダプターで有効になっていることを確認します。 詳細については、[スプーフィング対策保護](./getting-started-kubernetes-windows.md#disable-anti-spoofing-protection)を参照してください。


### <a name="my-windows-pods-cannot-ping-external-resources"></a>外部リソースの ping を実行できない、Windows ポッド ###
Windows ポッドには、今日 ICMP プロトコルのプログラミング外部ルールはありません。 ただし、TCP/UDP はサポートされています。 クラスターの外部でのリソースへの接続を示すしようとすると、置き換える`ping <IP>`に対応する`curl <IP>`コマンド。

問題が引き続き直面した場合は、最も可能性が高い[cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf)で、ネットワークの構成余分な注意にふさわしいします。 この値が固定されたファイルの編集は常に、構成は、新しく作成された Kubernetes リソースに適用されます。

どうしてでしょうか。
Kubernetes ネットワーク要件の 1 つは、クラスターへの通信 nat 内部で発生するです ( [Kubernetes モデル](https://kubernetes.io/docs/concepts/cluster-administration/networking/)を参照してください)。 この要求を受け入れるがあるすべての通信を[ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20)場所たくないに実行されるように NAT を送信します。 ただし、つまり、ExceptionList からクエリを実行しようとして外部 ip アドレスを除外する必要があること。 のみ、[は、Windows ポッド元トラフィックする SNAT'ed 正しく外部から返信されます。 このに関してで、ExceptionList`cni.conf`は次のようになります。
```
                "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
```

### <a name="my-windows-node-cannot-access-a-nodeport-service"></a>[Windows ノード NodePort サービスにアクセスできません。 ###
ノードからローカルの NodePort アクセスができなくなります。 これは既知の制限です。 NodePort アクセスが他のノードまたは外部のユーザーから有効です。

### <a name="after-some-time-vnics-and-hns-endpoints-of-containers-are-being-deleted"></a>いくつかの時間、Vnic、コンテナーの端点を HNS は削除されます。 ###
場合にこの問題が発生することが、 `hostname-override` [kube](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)プロキシ パラメーターが渡されていません。 その問題を解決するには、ユーザーは kube プロキシにホスト名を次のように渡す必要があります。
```
C:\k\kube-proxy.exe --hostname-override=$(hostname)
```

### <a name="on-flannel-vxlan-mode-my-pods-are-having-connectivity-issues-after-rejoining-the-node"></a>Flannel (vxlan) モードでは、[マイ ポッドがある接続の問題ノードに再度参加後に ###
削除済みノードをクラスターに参加するが、常に flannelD はノードに新しいポッド サブネットを割り当てるしようとします。 ユーザーは、次のパスで古いポッド サブネット構成ファイルを削除する必要があります。
```powershell
Remove-Item C:\k\SourceVip.json
Remove-Item C:\k\SourceVipRequest.json
```

### <a name="after-launching-startps1-flanneld-is-stuck-in-waiting-for-the-network-to-be-created"></a>Start.ps1 を起動した後、Flanneld が「を作成するネットワークの待機中」に残っています。 ###
この問題を調査していますが多数のレポートがあります。最も可能性が高い flannel ネットワークの管理 ip アドレスが設定されている場合のタイミングの問題を勧めします。 単に start.ps1 を再実行または次のように手動で再起動には、問題を回避します。
```
PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
```

現在レビュー中には、この問題を解決するための[現在価値](https://github.com/coreos/flannel/pull/1042)もあります。

### <a name="my-windows-pods-cannot-launch-because-of-missing-runflannelsubnetenv"></a>/Run/flannel/subnet.env 不足しているため、Windows ポッドを起動できません。 ###
Flannel が正常に起動していないことを示しています。 Flanneld.exe を再起動してか、またはファイルを手動でからコピー`/run/flannel/subnet.env`に Kubernetes マスターで`C:\run\flannel\subnet.env`Windows ワーカー ノードを変更して、`FLANNEL_SUBNET`別の番号に行します。 たとえば、ノード サブネット 10.244.4.1/24 が必要な場合。
```
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.4.1/24
FLANNEL_MTU=1500
FLANNEL_IPMASQ=true
```

### <a name="my-endpointsips-are-leaking"></a>自分のエンドポイント/ip アドレスがリークします。 ###
リーク端点が発生する可能性がある 2 の既知の問題が存在します。 
1.  最初の[既知の問題](https://github.com/kubernetes/kubernetes/issues/68511)は、Kubernetes 1.11 のバージョンの問題です。 Kubernetes 1.11.0 - 1.11.2 のバージョンを使用してしないようにしてください。
2. エンドポイントの記憶域の同時実行の問題には、2 番目の[既知の問題](https://github.com/docker/libnetwork/issues/1950)リーク エンドポイントの原因となります。 修正プログラムを受信するには、Docker EE 18.09 を使用する必要があります以上します。

### <a name="my-pods-cannot-launch-due-to-network-failed-to-allocate-for-range-errors"></a>期限を自分のポッドを起動できません"ネットワーク: 範囲の割り当てに失敗しました"エラー ###
これノードの IP アドレス スペースが使用されていることを示します。 [リーク端点](#my-endpointsips-are-leaking)をクリーンアップすると、すべてのリソースに影響を受けるノード & の次のコマンドが実行を移行してください。
```
c:\k\stop.ps1
Get-HNSEndpoint | Remove-HNSEndpoint
Remove-Item -Recurse c:\var
```

### <a name="my-windows-node-cannot-access-my-services-using-the-service-ip"></a>Windows ノードがサービス IP を使用してサービスにアクセスできない ###
これは、Windows の現在のネットワーク スタックの既知の制限です。 Windows*ポッド***は**ただしサービス IP をアクセスできるようにします。

### <a name="no-network-adapter-is-found-when-starting-kubelet"></a>Kubelet を起動するときにネットワーク アダプターが見つからない ###
Kubernetes ネットワークが機能するには、Windows ネットワーク スタックで仮想アダプターが必要です。 次のコマンドが (管理シェルに) 結果を返さない場合、仮想ネットワークの作成 &mdash; Kubelet が機能するのに必要な前提条件 &mdash; に失敗しています。

```powershell
Get-HnsNetwork | ? Name -ieq "cbr0"
Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
```

多くの場合のホストのネットワーク アダプターが「イーサネット」をされていない場合、start.ps1 スクリプト[InterfaceName](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/start.ps1#L6)パラメーターを変更することをお勧めします。 出力を確認して、それ以外の場合、`start-kubelet.ps1`スクリプトを使用して仮想ネットワークの作成時にエラーがあるかどうか。 

### <a name="pods-stop-resolving-dns-queries-successfully-after-some-time-alive"></a>しばらく稼働した後、ポッドが正常な DNS クエリの解決を停止する ###
Windows Server のネットワーク スタックに懸案事項をキャッシュ既知の DNS が、1803 とその下のバージョンが生じる DNS の要求が失敗することもあります。 この問題を回避するには、0 以下のレジストリ キーを使用して TTL キャッシュの最大値を設定できます。

```Dockerfile
FROM microsoft/windowsservercore:<your-build>
SHELL ["powershell', "-Command", "$ErrorActionPreference = 'Stop';"]
New-ItemProperty -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxCacheTtl -Value 0 -Type DWord 
New-ItemPropery -Path 'HKLM:\SYSTEM\CurrentControlSet\Services\Dnscache\Parameters' -Name MaxNegativeCacheTtl -Value 0 -Type DWord
```

### <a name="i-am-still-seeing-problems-what-should-i-do"></a>それでも問題が表示されます。 どうしたらいいでしょう。 ### 
ネットワークまたはホストに追加の制約があり、ノード間で特定の種類の通信が妨げられている場合があります。 次のことを確認してください。
  - 選択された[ネットワーク トポロジ](./network-topologies.md)が正しく構成されています。
  - ポッドからと思われるトラフィックが許可されていること
  - HTTP トラフィックが許可されていること (Web サービスを展開する場合)
  - さまざまなプロトコル (ie ICMP と TCP/UDP) からパケットが削除されません。


## <a name="common-windows-errors"></a>Windows の一般的なエラー ##

### <a name="my-kubernetes-pods-are-stuck-at-containercreating"></a>使用している Kubernetes ポッドが "ContainerCreating" で停止する ###
この問題には多くの原因が考えられますが、最も一般的な原因の 1 つは、pause イメージが正しく構成されていないことです。 これは、次の問題の大きな兆候です。


### <a name="when-deploying-docker-containers-keep-restarting"></a>展開するときに、Docker コンテナーが再起動を繰り返す ###
pause イメージが、OS のバージョンと互換性があることを確認します。 [手順](./deploying-resources.md)は、OS とコンテナーの両方のバージョン 1803 がいると仮定します。 Insider ビルドなど、新しいバージョンの Windows を使用する場合は、イメージを調整する必要があります。 イメージについては、マイクロソフトの [Docker リポジトリ](https://hub.docker.com/u/microsoft/)を参照してください。 いずれの場合も、pause イメージ Dockerfile とサンプル サービスの両方で、イメージに `:latest` にタグ付けされている必要があります。


## <a name="common-kubernetes-master-errors"></a>一般的な Kubernetes マスター エラー ##
Kubernetes マスターのデバッグは、(発生しやすい順で) 次の 3 つのカテゴリに分類されます。

  - Kubernetes システム コンテナーに問題がある。
  - `kubelet` の実行状態に問題がある。
  - システムに問題がある。

Kubernetes によるポッドの作成を確認するには、`kubectl get pods -n kube-system` を実行します。これにより、いずれかがクラッシュしていたり正しく開始していないとわかることがあります。   次に `docker ps -a` を実行して、これらのポッドの背後にあるすべての未加工コンテナーを表示します。 最後に、問題の原因となっている疑いがあるコンテナーに対して `docker logs [ID]` を実行し、プロセスの未加工出力を確認します。


### <a name="cannot-connect-to-the-api-server-at-httpsaddressport"></a>`https://[address]:[port]` で API サーバーに接続できない ###
このエラーはほとんどの場合、証明書の問題を示しています。 構成ファイルを正しく生成していること、その中の IP アドレスがホストに対応していること、API サーバーによってマウントされたディレクトリにコピー済みであることを確認します。

[手順](./creating-a-linux-master.md)に従う場合は、次を検索するその他のヒントに配置します。   
* `~/kube/kubelet/`
* `$HOME/.kube/config`
*  `/etc/kubernetes/admin.conf`

 それ以外の場合、マウント ポイントをチェックする API サーバーのマニフェストのファイルを参照してください。