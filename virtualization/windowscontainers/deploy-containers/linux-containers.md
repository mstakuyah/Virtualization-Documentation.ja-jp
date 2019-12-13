---
title: Linux Containers on Windows
description: Hyper-v を使用して Linux コンテナーを Windows で実行するさまざまな方法については、「ネイティブ」を参照してください。
keywords: LCOW, linux コンテナー, docker, コンテナー
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910572"
---
# <a name="linux-containers-on-windows"></a>Windows 上の Linux コンテナー

Linux コンテナーは、コンテナーエコシステム全体の大部分を占めるものであり、開発者エクスペリエンスと運用環境の両方の基礎となります。  コンテナーはコンテナーホストとカーネルを共有するため、Linux コンテナーを Windows で直接実行することは、オプション[*](linux-containers.md#other-options-we-considered)ではありません。  ここでは、仮想化を図に示します。

現時点では、Docker for Windows と Hyper-v を使用して Linux コンテナーを実行するには、次の2つの方法があります。

- Linux コンテナーを完全な Linux VM で実行する-これは、現在、Docker が行うものです。
- [Hyper-v 分離](../manage-containers/hyperv-container.md)(lcow) を使用して Linux コンテナーを実行する-これは Docker for Windows の新しいオプションです。

この記事では、各方法のしくみについて説明し、どのソリューションを選択するかと、進行中の作業を共有する方法に関するガイダンスを提供します。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM の Linux コンテナー

Linux VM で Linux コンテナーを実行するには、 [Docker の入門ガイド](https://docs.docker.com/docker-for-windows/)に記載されている手順に従ってください。

Docker は、Hyper-v で実行されている[Linuxkit](https://github.com/linuxkit/linuxkit)ベースの仮想マシンを使用して 2016 (hyper-v の分離または windows 上の Linux コンテナーが使用可能になる前) で最初にリリースされたため、windows デスクトップで Linux コンテナーを実行できました。

このモデルでは、Docker クライアントは Windows デスクトップで実行されますが、Linux VM では Docker デーモンを呼び出します。

![コンテナーホストとしての Moby VM](media/MobyVM.png)

このモデルでは、すべての Linux コンテナーが、1つの Linux ベースのコンテナーホストとすべての Linux コンテナーを共有します。

* カーネルと Moby VM を共有しますが、Windows ホストでは共有しません。
* Linux で実行されている linux コンテナー (linux VM で実行されているため) では、ストレージとネットワークのプロパティが一貫しています。

また、Linux コンテナーホスト (Moby VM) は、Docker デーモンとすべての Docker デーモンの依存関係を実行している必要があります。

Moby VM で実行しているかどうかを確認するには、Hyper-v マネージャー UI を使用するか、管理者特権の PowerShell ウィンドウで `Get-VM` を実行して、Moby VM の Hyper-v マネージャーを確認します。

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-v 分離を使用した Linux コンテナー

Windows 上の linux コンテナー (LCOW) を試すには、 [windows 10 の](../quick-start/quick-start-windows-10-linux.md)linux コンテナーの linux コンテナーの手順に従います。

Hyper-v の分離を使用した linux コンテナーは、コンテナーを実行するのに十分な OS を備えた、最適化された Linux VM で各 Linux コンテナーを実行します。 Moby VM の方法とは異なり、各 Linux コンテナーには独自のカーネルと独自の VM サンドボックスがあります。 また、Windows 上の Docker によって直接管理されています。

![Hyper-v 分離を使用した Linux コンテナー (LCOW)](media/lcow-approach.png)

Moby VM のアプローチと LCOW では、コンテナーの管理方法がどのように異なるかを詳しく見ていきます。 LCOW モデルのコンテナーの管理は Windows 上で維持され、各 LCOW 管理は GRPC と格納用の erd を介して行われます。  これは、LCOW に使用される Linux ディストリビューションコンテナーが、より小さな在庫を持つことができることを意味します。  ここでは、最適化されたディストリビューションコンテナーを使用するための LinuxKit を使用していますが、Kata などの他のプロジェクトでも同様の高度に調整された Linux ディストリビューション (Linux のクリア) を作成しています。

各 LCOW を詳しく見てみましょう。

![LCOW アーキテクチャ](media/lcow.png)

LCOW を実行しているかどうかを確認するには、`C:\Program Files\Linux Containers`に移動します。 Docker が LCOW を使用するように構成されている場合は、Hyper-v の分離の下で実行されている各コンテナーで実行される最小の LinuxKit ディストリビューションを含むいくつかのファイルがあります。  最適化された VM コンポーネントが 100 MB 未満であり、Moby VM の LinuxKit イメージよりもはるかに小さくなっていることに注意してください。

### <a name="work-in-progress"></a>進行中の作業

LCOW は、アクティブな開発中です。 [GitHub](https://github.com/moby/moby/issues/33850)の Moby プロジェクトで進行状況を追跡する

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

これらのアプリケーションにはすべてボリュームマッピングが必要であり、起動または正常に実行されません。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>追加情報

[LCOW を説明する Docker ブログ](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux コンテナーのビデオ](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-カーネルとビルドの手順](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby VM vs LCOW を使用する場合

### <a name="when-to-use-moby-vm"></a>Moby VM を使用する場合

現時点では、次のようなユーザーに Linux コンテナーを実行する Moby VM の方法をお勧めします。

- 安定したコンテナー環境を必要とします。  これは Docker for Windows の既定値です。
- Windows または Linux のコンテナーを実行しますが、両方を同時に実行することはほとんどありません。
- Linux コンテナー間に複雑なネットワーク要件やカスタムネットワーク要件があります。
- Linux コンテナー間では、カーネルの分離 (Hyper-v 分離) は必要ありません。

### <a name="when-to-use-lcow"></a>LCOW を使用する場合

現時点では、次のようにすることをお勧めします。

- 最新のテクノロジをテストします。
- Windows と Linux のコンテナーを同時に実行します。
- Linux コンテナー間でのカーネル分離 (Hyper-v 分離) が必要です。

## <a name="other-options-we-considered"></a>検討したその他のオプション

Windows で Linux コンテナーを実行する方法を検討しているときは、WSL と見なされました。 最終的には、Windows 上の Linux コンテナーが linux 上の linux コンテナーと整合するように、仮想化ベースのアプローチを選択しました。 また、Hyper-v を使用すると、セキュリティが強化されます。 今後は再評価されることがありますが、現時点では、LCOW は引き続き Hyper-v を使用します。

ご意見をお持ちの場合は、GitHub または UserVoice からフィードバックをお送りください。  特に、表示する特定のエクスペリエンスに関するフィードバックをお待ちしています。
