---
title: Linux Containers on Windows
description: Hyper-v を使用して、ネイティブであるかのように、Windows で Linux コンテナーを実行するさまざまな方法について説明します。
keywords: LCOW、linux コンテナー、docker、コンテナー
author: scooley
ms.date: 09/17/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: edfd11c8-ee99-42d8-9878-efc126fe1826
ms.openlocfilehash: 14445f3e9d292dbdab28986e772d0c045fca1586
ms.sourcegitcommit: 9100d2218c160bbe9fbf24f3524c8ff5e3dd826c
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "10135325"
---
# <a name="linux-containers-on-windows"></a>Windows の Linux コンテナー

Linux コンテナーは、コンテナー全体のエコシステムの大部分を構成しており、開発者エクスペリエンスと運用環境の両方の基盤となっています。  コンテナーは、コンテナーホストとの間でカーネルを共有しているため、Windows で直接 Linux[*](linux-containers.md#other-options-we-considered)のコンテナーを実行する方法はありません。  ここでは、仮想化がピクチャに含まれています。

現在、Windows と Hyper-v の Docker で Linux コンテナーを実行するには、次の2つの方法があります。

- Linux のコンテナーを完全な Linux VM で実行します。これは、現在の Docker です。
- [Hyper-v 分離](../manage-containers/hyperv-container.md)(lcow) を使用して Linux コンテナーを実行します。これは、Docker for Windows の新しいオプションです。

この記事では、各方法のしくみについて説明します。どのソリューションを選択するか、進行中の作業を共有するかについてのガイダンスを示します。

## <a name="linux-containers-in-a-moby-vm"></a>Moby VM の Linux コンテナー

Linux のコンテナーを Linux VM で実行するには、 [Docker の](https://docs.docker.com/docker-for-windows/)「はじめに」ガイドに記載されている手順に従ってください。

Docker は、Hyper-v で実行されている[Linuxkit](https://github.com/linuxkit/linuxkit)ベースの仮想マシンを使用して、最初に 2016 (hyper-v 分離または Linux コンテナーが利用可能になる前に) でリリースされた windows デスクトップで linux コンテナーを実行できました。

このモデルでは、Docker クライアントは Windows デスクトップで実行されますが、Linux VM では Docker デーモンに呼び出されます。

![コンテナーホストとしての Moby VM](media/MobyVM.png)

このモデルでは、すべての Linux コンテナーが、1つの Linux ベースのコンテナーホストとすべての Linux コンテナーを共有します。

* カーネルを相互に、または Moby VM と共有しますが、Windows ホストでは共有できません。
* Linux で実行されている linux コンテナーを使って、一貫したストレージとネットワークのプロパティを設定します (linux のコンテナで実行されているため)。

これは、Linux コンテナホスト (Moby VM) が、Docker デーモンと、すべての Docker デーモンの依存関係を実行している必要があることを意味します。

Moby VM で実行しているかどうかを確認するには、Hyper-v Manager の UI を使用するか、管理者の PowerShell `Get-VM`ウィンドウで実行して、moby Vm の hyper-v manager を確認します。

## <a name="linux-containers-with-hyper-v-isolation"></a>Hyper-v 分離を使用した Linux コンテナー

Windows (LCOW) で Linux コンテナーを試すには、 [windows 10 の linux](../quick-start/quick-start-windows-10-linux.md)コンテナーの linux コンテナ命令に従います。

Hyper-v 分離を搭載した linux のコンテナーでは、最適な Linux VM で、コンテナーを実行するための十分な OS だけで、各 Linux コンテナーを実行します。 Moby VM のアプローチとは異なり、各 Linux コンテナーには独自のカーネルと独自の VM サンドボックスがあります。 また、Windows の Docker によって直接管理されています。

![Hyper-v 分離を備えた Linux コンテナー (LCOW)](media/lcow-approach.png)

コンテナー管理が Moby VM アプローチと LCOW の間でどのように異なるかについて詳しく見ていきます。 LCOW モデルコンテナー管理では、Windows では、LCOW 管理は GRPC とインプレース erd によって実行されます。  これは、LCOW に使用される Linux のディストリビューションコンテナーでは、より少ないインベントリを持つことを意味します。  現時点では、最適化された「ディストリビューション」コンテナーのために LinuxKit を使用していますが、Kata などの他のプロジェクトでも、同様に、高いチューニングの Linux distros (Linux をクリア) が開発されています。

各 LCOW について詳しく見ていきましょう。

![LCOW アーキテクチャ](media/lcow.png)

LCOW を実行しているかどうかを`C:\Program Files\Linux Containers`確認するには、に移動します。 Docker が LCOW を使うように構成されている場合、ここには、Hyper-v 分離で実行されている各コンテナーで実行される最小限の LinuxKit のディストリビューションが含まれているファイルがいくつかあります。  注最適化された VM コンポーネントは、100 MB 未満で、Moby VM の LinuxKit イメージよりも大幅に小さいことに注意してください。

### <a name="work-in-progress"></a>進行中の作業

LCOW は [アクティブな開発] の下にあります。 [GitHub](https://github.com/moby/moby/issues/33850)の Moby プロジェクトで進行中の進行状況を追跡する

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

これらのアプリケーションでは、すべてボリュームマッピングが必要であり、正常に起動または実行されることはありません。

* MySQL
* PostgreSQL
* WordPress
* Jenkins
* MariaDB
* RabbitMQ

### <a name="extra-information"></a>追加情報

[LCOW について説明する Docker ブログ](https://blog.docker.com/2017/11/docker-for-windows-17-11/)

[Linux コンテナビデオ](https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4)

[LinuxKit LCOW-kernel + ビルド手順](https://github.com/linuxkit/lcow)

## <a name="when-to-use-moby-vm-vs-lcow"></a>Moby での VM と LCOW の使い方

### <a name="when-to-use-moby-vm"></a>Moby VM を使用する状況

現時点では、次のユーザーを対象とした Linux コンテナーを実行する Moby VM の方法をお勧めします。

- 安定したコンテナー環境が必要です。  これは、Windows の既定の Docker です。
- Windows または Linux のコンテナーを実行しますが、両方を同時に使うことはほとんどありません。
- Linux コンテナー間の複雑でカスタムネットワーク要件を持つ。
- Linux のコンテナー間では、カーネル分離 (Hyper-v 分離) は必要ありません。

### <a name="when-to-use-lcow"></a>LCOW の用途

現時点では、次のような方々にお勧めします。

- 最新技術をテストしたいと思います。
- Windows と Linux のコンテナーを同時に実行します。
- Linux コンテナー間には、カーネル分離 (Hyper-v 分離) が必要です。

## <a name="other-options-we-considered"></a>検討したその他のオプション

Windows で Linux コンテナーを実行する方法については、WSL と見なされました。 最終的に、Windows 上の Linux コンテナーが linux の linux コンテナーと一貫性が保たれるように、仮想化ベースのアプローチを選択しました。 また、Hyper-v を使用すると、セキュリティが強化されます。 今後、現在は再評価される可能性がありますが、LCOW では引き続き Hyper-v を使用します。

ご意見をお寄せの場合は、GitHub または UserVoice 経由でフィードバックを送信してください。  特に、表示する特定のエクスペリエンスについてのフィードバックを歓迎します。
