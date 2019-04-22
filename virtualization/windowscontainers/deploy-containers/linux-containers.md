---
title: Linux Containers on Windows
description: HYPER-V を使用するにはネイティブれている場合に、Windows で Linux コンテナーを実行する方法について説明します。
keywords: LCOW、linux コンテナー、docker、コンテナー
author: scooley
ms.date: 11/02/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 8597a93f035f5e451df8176d1563299120c95cb8
ms.sourcegitcommit: a5ff22c205149dac4fc05325ef3232089826f1ef
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/22/2019
ms.locfileid: "9380176"
---
# <a name="linux-containers-on-windows"></a>Windows の Linux コンテナー

Linux コンテナーしますコンテナー システム全体の大きな % を行い、開発エクスペリエンスと実運用環境の両方に基本的なします。  コンテナーのコンテナー ホスト カーネルと共有するため、Windows 上で直接 Linux コンテナーを実行しているされていないオプション[*](linux-containers.md#other-options-we-considered)します。  これは、図に仮想化になります。

ここでは Docker for Windows とハイパー-v: Linux コンテナーを実行する 2 つの方法があります。

1. Docker 通常は現在は、この Linux の仮想マシンの全画面に Linux コンテナーを実行します。
1. Docker for Windows での新しいオプションは、この[HYPER-V 分離](../manage-containers/hyperv-container.md)(LCOW) - Linux コンテナーを実行します。

この記事では、方法それぞれの方法は、ソリューションを選択するに関するガイダンスを提供し、進行中の作業を共有について説明します。

## <a name="linux-containers-in-a-moby-vm"></a>Moby の仮想マシンに Linux コンテナー

実行するには Linux コンテナー Linux の仮想マシンに[Docker のはじめに-ガイド](https://docs.docker.com/docker-for-windows/)の手順に従います。

Docker を Linux コンテナーを Windows で実行できるようになりましたが最初のリリースからデスクトップ 2016年で (する前に HYPER-V 分離または LCOW 利用可能な) を使用した、 [LinuxKit](https://github.com/linuxkit/linuxkit)ベース仮想マシンの HYPER-V で実行されています。

このモデルで Linux 仮想マシンにおける Docker デーモンに通話が、Windows デスクトップに Docker クライアントを実行します。

![コンテナー ホストとして Moby VM](media/MobyVM.png)

このモデルで Linux ベースの 1 つのコンテナー ホストとすべての Linux コンテナーを共有するすべての Linux コンテナー。

* Windows ホストと Moby 仮想マシンはカーネルを共有します。
* 一貫性のある記憶域があり (Linux の仮想マシンにおける実行されている) ために、Linux で実行されている Linux コンテナーがネットワーク プロパティ。

また、コンテナー、Linux ホスト (Moby VM) Docker デーモンと Docker デーモンの依存関係が実行されている必要があります。

Moby VM で実行しているかどうかを参照するには HYPER-V マネージャーを Moby VM が HYPER-V マネージャー UI を使用するのかを実行して確認`Get-VM`管理者特権の PowerShell ウィンドウにします。

## <a name="linux-containers-with-hyper-v-isolation"></a>HYPER-V 分離 Linux コンテナー

LCOW には、[このはじめに-ガイド](../quick-start/quick-start-windows-10.md)の Linux コンテナー指示に従って、

HYPER-V 分離 Linux コンテナーを最適化された Linux 仮想マシンで、コンテナーを実行するには、十分なだけ OS に各 Linux コンテナー (LCOW) を実行します。  各 LCOW では、Moby VM アプローチとは異なり、専用カーネルと独自 VM サンド ボックスがあります。  Windows Docker によって直接管理がいるもできます。

![HYPER-V 分離 (LCOW) Linux コンテナー](media/lcow-approach.png)

コンテナーの管理を Moby VM アプローチと LCOW 間とは異なる方法で詳しく見ていくで、LCOW モデル コンテナーの管理を常に Windows し GRPC と containerd によって行われ、各 LCOW 管理します。  LCOW の Linux ディストリビューション コンテナーを使用することを意味する量小さい在庫ことができます。  右におを使用している LinuxKit 最適化されたディストリビューション コンテナーを使用してのような高度な調整 Linux ディストリビューション (クリア Linux) も Kata などの他のプロジェクトを作成します。

ここでは、各 LCOW の詳細を確認します。

![LCOW アーキテクチャ](media/lcow.png)

移動する LCOW を使用しているかどうかを表示するには、`C:\Program Files\Linux Containers`します。 LCOW を使用する Docker を構成する場合、次のとおり HYPER-V 分離で実行されている各コンテナーで実行される最低限の LinuxKit ディストリビューションが含まれるいくつかのファイルがあります。  最適化された VM コンポーネントは 100 MB 未満、Moby 仮想マシンに LinuxKit イメージよりもはるかに小さいことを確認します。

### <a name="work-in-progress"></a>進行中の作業

LCOW は、作業中の開発中です。 [GitHub](https://github.com/moby/moby/issues/33850)で Moby プロジェクトの進行状況を追跡します。

#### <a name="bind-mounts"></a>バインド マウント

`docker run -v ...` を使ってボリュームをバインド マウントした場合、Windows NTFS ファイルシステムにファイルが格納されるため、POSIX 操作のための変換が必要になります。 しかしファイル システム操作の中には、現在部分的に、またはまったく実装されていない操作があるため、一部のアプリは互換性がありません。

以下の操作は、現在、バインド マウントされたボリュームには機能しません。

* MkNod
* XAttrWalk
* XAttrCreate
* Lock
* Getlock
* Auth
* Flush
* INotify

完全に実装されていない操作も少数ですが存在します。

* GetAttr: Nlink カウントは、常に 2 と報告されます。
* Open: ReadWrite フラグ、WriteOnly フラグ、および ReadOnly フラグのみが実装されています。

次のアプリケーションすべてボリュームへのマッピングが必要し開始されないか、正常に動作します。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>補足情報

[LCOW を説明する docker ブログ](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux コンテナーのビデオ](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW カーネルとビルド手順](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM vs LCOW を使用する場合

### <a name="when-to-use-moby-vm"></a>Moby VM を使用する場合

右、ことをお勧め Moby VM メソッドの人に Linux コンテナーを実行します。

- コンテナーの安定した環境をしたいです。  これは、Docker for Windows の既定値です。
- Windows または Linux のコンテナーが同時に両方のほとんどを実行します。
- 複雑なまたはカスタム ネットワーク Linux コンテナー間での要件です。
- Linux コンテナー間でカーネル分離 (HYPER-V 分離) は必要ありません。

### <a name="when-to-use-lcow"></a>LCOW を使用する場合

右にことをお勧め LCOW 人にします。

- 最新テクノロジをテストします。
- Windows と Linux コンテナーを同時に実行します。
- Linux コンテナー間でカーネル分離 (HYPER-V 分離) 必要があります。

## <a name="other-options-we-considered"></a>検討しましたが、その他のオプション

Windows 上で Linux コンテナーを実行する方法を見てされた、WSL おと見なされます。 最終的には、選択した仮想化に応じたできるように、Windows の Linux コンテナーが Linux の Linux コンテナーと一致します。 HYPER-V を使用しても、LCOW セキュリティ。 お今後、評価し直すことがありますが、ここでは、LCOW 引き続き HYPER-V を使用します。

アイデアを使っている場合は、GitHub や UserVoice フィードバックを送信してください。  表示するには、特定のエクスペリエンスに関するフィードバックを歓迎特に。
