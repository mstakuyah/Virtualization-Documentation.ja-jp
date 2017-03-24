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
translationtype: Human Translation
ms.sourcegitcommit: f615c6dd268932a2ff99ac12c4e9ffdcf2cc217e
ms.openlocfilehash: ee6053003b31f226d2cfba8566f274ccc19d97ec
ms.lasthandoff: 03/02/2017

---

# swarm モードの概要 

**重要な注意:*** 現在、swarm モードとオーバーレイ ネットワークのサポートは、[Windows Insider](https://insider.windows.com/) 向けに次回公開予定の Windows 10 Creators Update でのみ利用可能です。それ以降の Windows プラットフォーム向けサポートは近日公開予定です。*

## "swarm モード" とは
swarm モードは、Docker ホストのネイティブなクラスタリングやコンテナー ワークロードのスケジューリングなどのオーケストレーション機能を提供する Docker 機能です。 複数の Docker ホストの Docker エンジンを “swarm モード” で連携して実行すると、それらの Docker ホストで “swarm” クラスターが形成されます。 swarm モードについて詳しくは、[Docker のメイン ドキュメント サイト](https://docs.docker.com/engine/swarm/)を参照してください。

## マネージャー ノードとワーカー ノード
1 つの swarm は、*マネージャー ノード*と*ワーカー ノード*の 2 種類のコンテナー ホストで構成されます。 すべての swarm はマネージャー ノードを介して初期化され、swarm を制御および監視するためのすべての Docker CLI コマンドは、いずれかのマネージャー ノードから実行する必要があります。 マネージャー ノードは、swarm の状態の ”管理ノード” と考えることができます。マネージャー ノードは連携してコンセンサス・グループを形成し、その swarm で実行されているサービスの状態を常に監視します。マネージャー ノードの役目は、swarm の実際の状態が、開発者や管理者によって定義される意図した状態に常に一致するように維持することがです。 

>    **注:** swarm は複数のマネージャー ノードを持つことができますが、*1 つ以上*のマネージャー ノードが必要です。 

ワーカー ノードは、マネージャー ノードを介して Docker swarm によってオーケストレーションされます。 ワーカー ノードを swarm に参加させるには、swarm の初期化時にマネージャー ノードで生成された "join トークン" を使う必要があります。 ワーカー ノードは、マネージャー ノードから単純にタスクを単に受信して実行するノードであるため、swarm の状態を認識 (および保持) する必要はありません。

## swarm モードのシステム要件

**Windows 10 Creators Update** ([Windows Insiders](https://insider.windows.com/) プログラムのメンバーに提供されます) を実行し、コンテナー ホストとしてセットアップされた 1 台以上の物理または仮想コンピューター システム (swarm の機能をフルに活用するには、複数ノードの使用をお勧めします。Windows 10 で Docker コンテナーを使い始める方法について詳しくは、「[Windows 10 の Windows コンテナー](https://docs.microsoft.com/en-us/virtualization/windowscontainers/quick-start/quick-start-windows-10)」をご覧ください)

**Docker Engine v1.13.0 以降**

空きポート: 各ホストの次のポートが利用できる必要があります。 システムによっては、これらのポートは既定で空きポートです。
- TCP ポート 2377 (クラスター管理通信用)
- TCP および UDP ポート 7946 (ノード間通信用)
- TCP および UDP ポート 4789 (オーバーレイ ネットワーク トラフィック用)

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

ここで、\<MANAGERIPADDRESS\>は swarm マネージャー ノードのローカル IP アドレスです。\<WORKERJOINTOKEN\> はマネージャー ノードから `docker swarm init` コマンドを実行した結果、出力として提供されたワーカー join トークンです。 join トークンは、swarm を初期化した後、マネージャー ノードから次のコマンドのいずれかを実行しても入手できます。

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

ここでも、\<MANAGERIPADDRESS\>は swarm マネージャー ノードのローカル IP アドレスです。 join トークン \<MANAGERJOINTOKEN\> は、swarm *マネージャー*の join トークンです。これは既存のマネージャー ノードから次のコマンドのいずれかを実行して取得できます。

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

ここで、\<NETWORKNAME\>は指定するネットワーク名です。

## swarm へのサービスの展開
オーバーレイ ネットワークを作成したら、サービスを作成してネットワークに接続できます。 ネットワークは、次の構文で作成されます。

```none
# Deploy a service to the swarm
C:\> docker service create --name=<SERVICENAME> --endpoint-mode dnsrr --network=<NETWORKNAME> <CONTAINERIMAGE> [COMMAND] [ARGS…]
```

ここで、\<SERVICENAME\> は、指定するサービス名です。この名前は、サービス ディスカバリでサービスを参照するために使います (サービス ディスカバリには、Docker のネイティブな DNS サーバーが使われます)。 \<NETWORKNAME\> は、このサービスを接続するネットワークの名前です (例: "myOverlayNet")。 \<CONTAINERIMAGE\>は、サービスを定義するコンテナー イメージの名前です。

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
C:\ docker node ls
```

このコマンドの出力では、1 つのノードにアスタリスク (*) が付いています。このアスタリスクは、単純に現在のノード、つまり `docker node ls` コマンドが実行されたノードを示しています。

### ネットワークの一覧表示
特定のノードに存在するネットワークの一覧を表示するには、次のコマンドを使います。 オーバーレイ ネットワークを表示するには、swarm モードで動作する**マネージャー ノード**からこのコマンドを実行する必要があります。

```none
C:\ docker network ls
```

### サービスの一覧表示
swarm で現在実行しているサービスとそれらの状態を一覧表示するには、次のコマンドを使います。

```none
C:\ docker service ls
```

### サービスを定義するコンテナー インスタンスの一覧表示
特定のサービスに対して実行されているコンテナー インスタンスの詳細を表示するには、次のコマンドを使います。 このコマンドの出力では、各コンテナーが実行されている ID とノードに加え、それぞれのコンテナーの状態に関する情報が表示されます。  

```none
C:\ docker service ps <SERVICENAME>
```

## 制限事項
現在、Windows 上の swarm モードには次の制限があります。
- 既知のバグ: オーバーレイおよび swarm モードは、イーサネットに接続されているホスト上でのみサポートされます。**Wi-Fi 接続のホストでは動作しません。** この問題は現在修正中です。
- データプレーンの暗号化 (`--opt encrypted`オプションを使用したコンテナー間のトラフィック) はサポートされていません。
- Windows Docker ホスト用の[ルーティング メッシュ](https://docs.docker.com/engine/swarm/ingress/)はまだサポートされていませんがまもなく公開予定です。 他の負荷分散方法を使用する場合は、外部のロード バランサー (NGINXなど) を設定したうえで、swarm の[公開ポート モード](https://docs.docker.com/engine/reference/commandline/service_create/#/publish-service-ports-externally-to-the-swarm--p---publish)を使って、負荷分散の際に経由するコンテナー ホスト ポートを公開します。  



