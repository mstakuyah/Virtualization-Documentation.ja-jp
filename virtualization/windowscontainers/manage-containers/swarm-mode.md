---
title: "swarm モードの概要"
description: "swarm クラスターの開始、オーバーレイ ネットワークの作成、ネットワークへのサービスのアタッチ。"
keywords: "docker,コンテナー, swarm, オーケストレーション"
author: kallie-b
ms.date: 02/9/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5ceb9626-7c48-4d42-81f8-9c936595ad85
ms.openlocfilehash: 9acb433e0165d0ca97012dc73363036804298d2d
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2017
---
# swarm モードの概要 

## "swarm モード" とは
swarm モードは、Docker ホストのネイティブなクラスタリングやコンテナー ワークロードのスケジューリングなどのオーケストレーション機能を提供する Docker 機能です。 複数の Docker ホストの Docker エンジンを “swarm モード” で連携して実行すると、それらの Docker ホストで “swarm” クラスターが形成されます。 swarm モードについて詳しくは、[Docker のメイン ドキュメント サイト](https://docs.docker.com/engine/swarm/)を参照してください。

## マネージャー ノードとワーカー ノード
1 つの swarm は、*マネージャー ノード*と*ワーカー ノード*の 2 種類のコンテナー ホストで構成されます。 すべての swarm はマネージャー ノードを介して初期化され、swarm を制御および監視するためのすべての Docker CLI コマンドは、いずれかのマネージャー ノードから実行する必要があります。 マネージャー ノードは、swarm の状態の ”管理ノード” と考えることができます。マネージャー ノードは連携してコンセンサス・グループを形成し、その swarm で実行されているサービスの状態を常に監視します。マネージャー ノードの役目は、swarm の実際の状態が、開発者や管理者によって定義される意図した状態に常に一致するように維持することがです。 

>   **注:** swarm は複数のマネージャー ノードを持つことができますが、*1 つ以上*のマネージャー ノードが必要です。 

ワーカー ノードは、マネージャー ノードを介して Docker swarm によってオーケストレーションされます。 ワーカー ノードを swarm に参加させるには、swarm の初期化時にマネージャー ノードで生成された "join トークン" を使う必要があります。 ワーカー ノードは、マネージャー ノードから単純にタスクを単に受信して実行するノードであるため、swarm の状態を認識 (および保持) する必要はありません。

## swarm モードのシステム要件

**Windows 10 Creators Update** または **Windows Server 2016** *とすべての最新の更新プログラム\**を実行し、コンテナー ホストとしてセットアップされた (Windows 10 で Docker コンテナーを使い始める方法について詳しくは、「[Windows 10 の Windows コンテナー](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)」または「[Windows Server の Windows コンテナー](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-server)」をご覧ください) 1 台以上の物理または仮想コンピューター システム (swarm の機能をフルに活用するには、複数ノードの使用をお勧めします)。

\***注**: Windows Server 2016 で Docker swarm を使用するには [KB4015217](https://support.microsoft.com/en-us/help/4015217/windows-10-update-kb4015217) が必要です。

**Docker Engine v1.13.0 以降**

空きポート: 各ホストの次のポートが利用できる必要があります。 システムによっては、これらのポートは既定で空きポートです。
- TCP ポート 2377 (クラスター管理通信用)
- TCP および UDP ポート 7946 (ノード間通信用)
- UDP ポート 4789 (オーバーレイ ネットワーク トラフィック用)

## swarm クラスターの初期化

swarm を初期化するには、コンテナー ホストのいずれかから次のコマンドを実行するだけです (\<HOSTIPADDRESS\> は、ホスト マシンのローカル IPv4 アドレスと置き換えます)。

```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```
特定のコンテナーのホストからこのコマンドを実行すると、そのホスト上の Docker エンジンがマネージャー ノードとして swarm モードで実行を開始します。

## swarm へのノードの追加

> **注:** swarm モードやオーバーレイ ネットワーク機能の使用には、複数のノードは*必要ありません*。 すべての swarm 機能やオーバーレイ機能は、swarm ノードで実行する 1 つのホスト (つまり、`docker swarm init`コマンドで swarm モードに設定したマネージャー ノード) で使用できます。

### swarm へのワーカーの追加
マネージャー ノードから swarm を初期したら、他のホストをワーカーとして swarm に追加できます。これには次のような簡単なコマンドを使います。

```none
C:\> docker swarm join --token <WORKERJOINTOKEN> <MANAGERIPADDRESS>
```

ここで、\<MANAGERIPADDRESS\> は swarm マネージャー ノードのローカル IP アドレスです。\<WORKERJOINTOKEN\> はマネージャー ノードから `docker swarm init` コマンドを実行した結果、出力として提供されたワーカー join トークンです。 join トークンは、swarm を初期化した後、マネージャー ノードから次のコマンドのいずれかを実行しても入手できます。

```none
# Get the full command required to join a worker node to the swarm
C:\> docker swarm join-token worker

# Get only the join-token needed to join a worker node to the swarm
C:\> docker swarm join-token worker -q
```

### swarm へのマネージャーの追加
swarm クラスターには、次のコマンドを使って別のマネージャー ノードを追加できます。

```none
C:\> docker swarm join --token <MANAGERJOINTOKEN> <MANAGERIPADDRESS>
```

ここでも、\<MANAGERIPADDRESS\> は swarm マネージャー ノードのローカル IP アドレスです。 join トークン \<MANAGERJOINTOKEN\> は、swarm *マネージャー*の join トークンです。これは既存のマネージャー ノードから次のコマンドのいずれかを実行して取得できます。

```none
# Get the full command required to join a **manager** node to the swarm
C:\> docker swarm join-token manager

# Get only the join-token needed to join a **manager** node to the swarm
C:\> docker swarm join-token manager -q
```

## オーバーレイ ネットワークの作成

swarm クラスターを構成したら、swarm 上にオーバーレイ ネットワークを作成できます。 オーバーレイ ネットワークは、swarm マネージャー ノードから、次のコマンドを実行して作成できます。

```none
# Create an overlay network 
C:\> docker network create --driver=overlay <NETWORKNAME>
```

ここで、\<NETWORKNAME\> は指定するネットワーク名です。

## swarm へのサービスの展開
オーバーレイ ネットワークを作成したら、サービスを作成してネットワークに接続できます。 サービスは、次の構文で作成されます。

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

ここで、\<SERVICENAME\> は、指定するサービス名です。この名前は、サービス ディスカバリでサービスを参照するために使います (サービス ディスカバリには、Docker のネイティブな DNS サーバーが使われます)。 \<NETWORKNAME\> は、このサービスを接続するネットワークの名前です (例: "myOverlayNet")。 \<CONTAINERIMAGE\> は、サービスを定義するコンテナー イメージの名前です。

> **注:** このコマンドの 2 番目の引数 `--endpoint-mode dnsrr` は、Docker エンジンに対し、サービス コンテナー エンドポイント間でのネットワーク トラフィックの負荷分散に、DNS ラウンド ロビン ポリシーが使用されることを指定するために必要です。 DNS ラウンド ロビンは、現在 Windows でサポートされている唯一の負荷分散方法です。Windows Docker ホスト用の[ルーティング メッシュ](https://docs.docker.com/engine/swarm/ingress/)はまだサポートされていませんが、まもなく追加される予定です。 他の負荷分散方法を使用する場合は、外部のロード バランサー (NGINXなど) を設定したうえで、swarm の[公開ポート モード](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)を使って、負荷分散の際に経由するコンテナー ホスト ポートを公開します。

## サービスの縮小拡大
swarm クラスターにサービスが展開されると、そのサービスを構成するコンテナー インスタンスがクラスターに展開されます。 既定では、サービスをサポートするコンテナー インスタンスの数 ("レプリカ"、つまりサービス用の "タスク" の数) は 1 つです。 ただし、`docker service create` コマンドで `--replicas`オプションを使って、複数のタスクを持つサービスを作成したり、サービスの作成後にサービスを拡大縮小したりすることができます。

サービスのスケーラビリティは、Docker Swarm が提供する重要なメリットであり、この機能もまた次の 1 つの Docker コマンドで利用できます。

```none
C:\> docker service scale <SERVICENAME>=<REPLICAS>
```

ここで、\<SERVICENAME\> は拡大縮小するサービスの名前であり、\<REPLICAS\> は拡大縮小後のタスク (つまりコンテナー インスタンス) の数です。


## swarm の状態の表示

swarm と swarm 上で実行しているサービスの状態は、いくつかの便利なコマンドを使って表示できます。

### swarm ノードの一覧表示
swarm に現在参加しているノードと各ノードの状態情報を一覧表示するには、次のコマンドを使います。 このコマンドは、**マネージャー ノード**から実行する必要があります。

```none
C:\> docker node ls
```

このコマンドの出力では、1 つのノードにアスタリスク (*) が付いています。このアスタリスクは、単純に現在のノード、つまり `docker node ls` コマンドが実行されたノードを示しています。

### ネットワークの一覧表示
特定のノードに存在するネットワークの一覧を表示するには、次のコマンドを使います。 オーバーレイ ネットワークを表示するには、swarm モードで動作する**マネージャー ノード**からこのコマンドを実行する必要があります。

```none
C:\> docker network ls
```

### サービスの一覧表示
swarm で現在実行しているサービスとそれらの状態を一覧表示するには、次のコマンドを使います。

```none
C:\> docker service ls
```

### サービスを定義するコンテナー インスタンスの一覧表示
特定のサービスに対して実行されているコンテナー インスタンスの詳細を表示するには、次のコマンドを使います。 このコマンドの出力では、各コンテナーが実行されている ID とノードに加え、それぞれのコンテナーの状態に関する情報が表示されます。  

```none
C:\> docker service ps <SERVICENAME>
```
## Linux+Windows 混在 OS クラスター

最近、チームのメンバーの 1 人が、Docker Swarm を使用して Windows+Linux 混在 OS アプリケーションを設定する方法について、短い 3 部構成のデモを投稿しました。 Docker Swarm を初めて使用する場合や、Docker Swarm を使用して混在 OS アプリケーションを実行するのが初めてである場合は、このデモを見ることをお勧めします。 今すぐ確認してください。
- [Docker Swarm を使用して Windows+Linux のコンテナー化されたアプリケーションを実行する (パート 1/3)](https://www.youtube.com/watch?v=ZfMV5JmkWCY&t=170s)
- [Docker Swarm を使用して Windows+Linux のコンテナー化されたアプリケーションを実行する (パート 2/3)](https://www.youtube.com/watch?v=VbzwKbcC_Mg&t=406s)
- [Docker Swarm を使用して Windows+Linux のコンテナー化されたアプリケーションを実行する (パート 3/3)](https://www.youtube.com/watch?v=I9oDD78E_1E&t=354s)

### Linux+Windows 混在 OS クラスターの初期化
ファイアウォール規則が正しく構成されており、ホストが相互にアクセスできる限り、混在 OS の swarm クラスターの初期化は簡単です。Linux ホストを swarm に追加するのに必要なものは、`docker swarm join` コマンドのみです。
```none
C:\> docker swarm join --token <JOINTOKEN> <MANAGERIPADDRESS>
```
Windows ホストから swarm を初期化する場合に実行するのと同じコマンドを使って、Linux ホストから swarm を初期化することもできます。
```none
# Initialize a swarm 
C:\> docker swarm init --advertise-addr=<HOSTIPADDRESS> --listen-addr <HOSTIPADDRESS>:2377
```

### swarm ノードへのラベルの追加
混在 OS swarm クラスターに対して Docker サービスを起動するには、そのサービスの対象となる OS を実行している swarm ノードと、それ以外のノードを判別する方法が必要です。 [Docker オブジェクト ラベル](https://docs.docker.com/engine/userguide/labels-custom-metadata/)は、ノードにラベル付けするのに便利な方法であり、OS に一致するノードでのみ実行されるように、サービスを作成および構成できます。 

> 注: [Docker オブジェクト ラベル](https://docs.docker.com/engine/userguide/labels-custom-metadata/)は、さまざまな Docker オブジェクト (コンテナー イメージ、コンテナー、ボリューム、ネットワークなど) にメタデータを適用するために使用できるほか、さまざまな目的に使用できます。たとえば、ラベルを使用してアプリケーションの "フロントエンド" と "バックエンド" のコンポーネントを区別することで、フロントエンドのマイクロサービスを "front-end" のラベルが付けられたノードでのみスケジュールし、バックエンドのマイクロサービスを "back-end" のラベルが付けられたノードでのみスケジュールすることができます。 ここでは、ノードのラベルを使用して、Windows OS ノードと Linux OS ノードを区別します。

既存の swarm ノードにラベルを付けるには、次の構文を使います。

```none
C:\> docker node update --label-add <LABELNAME>=<LABELVALUE> <NODENAME>
```

ここで、`<LABELNAME>` は作成するラベルの名前です。この例では、OS によってノードを区別するため、ラベルの論理名を "os" とすることができます。 `<LABELVALUE>` は、ラベルの値です。この例では、"windows" や "linux" などの値を使用できます  (もちろん、一貫性がある限り、ラベルとラベルの値には任意の名前付けを選択できます)。 `<NODENAME>` は、ラベルを付けるノードの名前です。`docker node ls` を実行することによって、ノードの名前を確認できます。 

**たとえば**、クラスター内に 4 つの swarm ノードがあり、2 つが Windows ノードで、2 つが Linux ノードである場合、ラベルの更新コマンドは次のようになります。

```none
# Example -- labeling 2 Windows nodes and 2 Linux nodes in a cluster...
C:\> docker node update --label-add os=windows Windows-SwarmMaster
C:\> docker node update --label-add os=windows Windows-SwarmWorker1
C:\> docker node update --label-add os=linux Linux-SwarmNode1
C:\> docker node update --label-add os=linux Linux-SwarmNode2
```

### 混在 OS swarm へのサービスの展開
swarm ノードのラベルを使用する場合、クラスターへのサービスの展開は簡単です。[`docker service create`](https://docs.docker.com/engine/reference/commandline/service_create/) コマンドで `--constraint` オプションを使用するだけです。

```none
# Deploy a service with swarm node constraint
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> --constraint node.labels.<LABELNAME>=<LABELVALUE> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

上記の例のラベルとラベル値の命名を使用する場合、一連のサービスの作成コマンド (1 つは Windows ベースのサービス用、1 つは Linux ベースのサービス用) は次のようになります。

```none
# Example -- using the 'os' label and 'windows'/'linux' label values, service creation commands might look like these...

# A Windows service
C:\> docker service create --name=win_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==windows' microsoft/nanoserver:latest powershell -command { sleep 3600 }

# A Linux service
C:\> docker service create --name=linux_s1 --endpoint-mode dnsrr --network testoverlay --constraint 'node.labels.os==linux' redis
```

## 制限事項
現在、Windows 上の swarm モードには次の制限があります。
- データプレーンの暗号化 (`--opt encrypted`オプションを使用したコンテナー間のトラフィック) はサポートされていません。
- Windows Docker ホスト用の[ルーティング メッシュ](https://docs.docker.com/engine/swarm/ingress/)はまだサポートされていませんがまもなく公開予定です。 他の負荷分散方法を使用する場合は、外部のロード バランサー (NGINXなど) を設定したうえで、swarm の[公開ポート モード](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)を使って、負荷分散の際に経由するコンテナー ホスト ポートを公開します。 以下で詳しく説明します。

## サービス エンドポイントのポートの公開
Docker Swarm の[ルーティング メッシュ](https://docs.docker.com/engine/swarm/ingress/)機能は、Windows ではまだサポートされていませんが、サービス エンドポイントのポートを公開する方法を探している場合は、現在でもポート公開モードを使用して行うことができます。 

サービスを定義するタスク/コンテナー エンドポイントのそれぞれについて、ホスト ポートを公開するには、`docker service create` コマンドで `--publish mode=host,target=<CONTAINERPORT>` 引数を使用します。

```none
# Create a service for which tasks are exposed via host port
C:\ > docker service create --name=<SERVICENAME> --publish mode=host,target=<CONTAINERPORT> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

たとえば、次のコマンドでは、サービス 's1' が作成され、そのサービスについて、コンテナー ポート 80 とランダムに選択されたホスト ポート経由で各タスクが公開されます。

```none
C:\ > docker service create --name=s1 --publish mode=host,target=80 --endpoint-mode dnsrr web_1 powershell -command {echo sleep; sleep 360000;}
```

ポート公開モードを使用してサービスを作成した後、サービスに照会して各サービス タスクのポート マッピングを表示できます。

```none
C:\ > docker service ps <SERVICENAME>
```
上記のコマンドは、(すべての swarm ホストにわたって) サービス用に実行されているすべてのコンテナー インスタンスに関する詳細を返します。 出力の列の 1 つである "ports" 列には、\<HOSTPORT\>->\<CONTAINERPORT\>/tcp の形式で各ホストのポート情報が含まれます。 各コンテナーはそれぞれのホスト ポートで公開されるため、\<HOSTPORT\> の値は各コンテナー インスタンスによって異なります。


## ヒントと情報 

#### *既存の透過ネットワークによって swarm の初期化/オーバーレイ ネットワークの作成がブロックされる場合がある* 
Windows では、オーバーレイ ネットワーク ドライバーと透過ネットワーク ドライバーはいずれも、外部 vSwitch が (仮想) ホスト ネットワーク アダプターにバインドされている必要があります。 オーバーレイ ネットワークが作成されると、新しいスイッチが作成され、オープンなネットワーク アダプターに接続されます。 透過ネットワーク モードでも、ホスト ネットワーク アダプターが使用されます。 さらに、特定のネットワーク アダプターは一度に 1 つのスイッチのみバインドできます。ホストのネットワーク アダプターが 1 つだけである場合、ネットワーク アダプターは、vSwitch がオーバーレイ ネットワーク用であるか、透過ネットワーク用であるかに関係なく、一度に 1 つの外部 vSwitch にのみ接続できます。 

したがって、コンテナー ホストのネットワーク アダプターが 1 つだけである場合、透過ネットワークによってオーバーレイ ネットワークの作成がブロックされる (またはその逆の) 問題が発生する可能性があります。これは、透過ネットワークが、現在、ホストの唯一の仮想ネットワーク インターフェイスを占有しているためです。

この問題を回避するには、次の 2 つの方法があります。
- *オプション 1 -既存の透過ネットワークを削除する:* swarm を初期化する前に、コンテナー ホストに既存の透過ネットワークがないことを確認します。 透過ネットワークを削除して、ホストに空いている仮想ネットワーク アダプターがあり、オーバーレイのネットワークの作成に使用できることを確認します。
- *オプション2 - ホストで追加の (仮想) ネットワーク アダプターを作成する:* ホスト上の透過ネットワークを削除する代わりに、オーバーレイ ネットワークの作成に使用する追加のネットワーク アダプターをホストに作成できます。 これを行うには、(PowerShell または Hyper-V マネージャーを使用して) 新しい外部ネットワーク アダプターを作成するだけです。新しいインターフェイスが作成されると、swarm が初期化されるときに、ホスト ネットワーク サービス (HNS) が自動的にホスト上のインターフェイスを認識し、そのインターフェイスを使用してオーバーレイ ネットワーク作成用の外部 vSwitch をバインドします。



