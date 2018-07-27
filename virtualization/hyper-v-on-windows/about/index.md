---
title: Windows 10 の Hyper-V の概要
description: Hyper-V、仮想化、関連テクノロジの概要です。
keywords: windows 10, hyper-v
author: scooley
ms.date: 06/25/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: eb2b827c-4a6c-4327-9354-50d14fee7ed8
ms.openlocfilehash: 95b7b25ffe47f22f2f00e5911195ebbea660a1c0
ms.sourcegitcommit: d625ea23c3eea484d54fa7bec10ac545b0386379
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/26/2018
ms.locfileid: "2093679"
---
# <a name="introduction-to-hyper-v-on-windows-10"></a>Windows 10 の Hyper-V の概要

> Hyper-V によって、Microsoft Virtual PC は置き換えられます。

ソフトウェア開発者や、IT プロフェッショナル、熱心な技術者の方の多くは、複数のオペレーティング システムを実行する必要があります。 Hyper-V を使用すると、Windows 上の仮想マシンとして複数のオペレーティング システムを実行することができます。

![Windows を実行する仮想マシン](media/HyperVNesting.png)

Hyper-V は、具体的には、ハードウェアの仮想化を提供します。  つまり、各仮想マシンが仮想ハードウェア上で実行されます。  Hyper-V を使用して、仮想ハード ドライブや仮想スイッチのほか、仮想マシンに追加できるさまざまな仮想デバイスを作成できます。

## <a name="reasons-to-use-virtualization"></a>仮想化を使用する理由

仮想化によって、次のようなことを実現できます。

* 以前のバージョンの Windows または Windows 以外のオペレーティング システムを必要とするソフトウェアを実行します。

* 他のオペレーティング システムで実験します。 Hyper-V では、さまざまなオペレーティング システムを簡単に作成したり削除したりできます。

* 複数の仮想マシンを使用し、複数のオペレーティング システムでソフトウェアをテストします。 Hyper-V を使用すれば、1 台のデスクトップまたはノート PC でそれらをすべて実行できます。 これらの仮想マシンは、エクスポートして、別の Hyper-V システム (Azure など) にインポートすることができます。

## <a name="system-requirements"></a>システム要件

Hyper-V は、Windows 8 以降の 64 ビット版 Windows Professional、Enterprise、および Education で使用できます。  Hyper-V は Windows Home エディションでは使用できません。

> **[設定]** > **[更新とセキュリティ]** > **[ライセンス認証]** の順に移動して、Windows 10 Home エディションを Windows 10 Professional にアップグレードしてください。 ここで Microsoft Store にアクセスして、アップグレードを購入できます。

ほとんどのコンピューターで Hyper-V を実行できますが、各仮想マシンはまったく別のオペレーティング システムになります。  一般的に、4 GB の RAM を搭載したコンピューターで 1 つまたは複数の仮想マシンを実行できますが、仮想マシンを追加する場合や、ゲーム、ビデオ編集、エンジニアリング設計などのソフトウェアをインストールして実行する場合は、より多くのリソースが必要になります。

Hyper-V のシステム要件と、コンピューター上で Hyper-V が実行されていることを確認する方法について詳しくは、[Hyper-V の要件に関するリファレンス](..\reference\hyper-v-requirements.md)をご覧ください。

## <a name="operating-systems-you-can-run-in-a-virtual-machine"></a>仮想マシンで実行できるオペレーティング システム

Windows 上の Hyper-V では、Linux、FreeBSD、Windows の各種リリースを含むさまざまなオペレーティング システムが仮想マシン内でサポートされます。

なおに、仮想マシンで使用するすべてのオペレーティング システムの有効なライセンスを持っている必要があります。

Windows の Hyper-V でゲストとしてサポートされているオペレーティング システムについては、[サポートされている Windows ゲスト オペレーティング システム](supported-guest-os.md)と[サポートされている Linux ゲスト オペレーティング システム](https://technet.microsoft.com/library/dn531030.aspx)に関する情報をご覧ください。

## <a name="differences-between-hyper-v-on-windows-and-hyper-v-on-windows-server"></a>Windows の Hyper-V と Windows Server の Hyper-V の相違点

一部の機能は、Windows 上の Hyper-V と、Windows Server で実行されている Hyper-V とで動作が異なります。

Windows Server のみで使用できる Hyper-V 機能:

* ホスト間での仮想マシンのライブ マイグレーション
* Hyper-V レプリカ
* 仮想ファイバー チャネル
* SR-IOV ネットワー キング
* 共有 .VHDX

Windows 10 のみで使用できる Hyper-V 機能:

* クイック作成と VM ギャラリー
* 既定のネットワーク (NAT スイッチ)

Windows 上の Hyper-V では、メモリ管理モデルが異なります。 サーバーでは、HYPER-V でメモリをバーチャル マシンのみが、サーバーで実行されていることを前提に管理されます。 Windows 上の Hyper-V では、ほとんどのクライアント コンピューターが仮想マシンだけでなく、ホスト上のソフトウェアも実行している、という前提でメモリが管理されています。

## <a name="limitations"></a>制限事項

特定のハードウェアに依存するプログラムは、仮想マシンでは動作しません。 たとえば、GPU での処理を必要とするゲームまたはアプリケーションは正しく動作しない可能性があります。 また、10 ミリ秒未満のタイマーに依存するアプリケーション (ライブ音楽のミキシング アプリなど) や高い時間精度に依存するアプリケーションの場合、仮想マシンで実行すると問題が発生することがあります。

さらに、Hyper-V を有効にしていると、遅延の影響を受けやすい高精度なアプリは、ホストで実行するときにも問題が発生することがあります。  これは、仮想化を有効にすると、ゲスト オペレーティング システムの場合と同様に、ホスト OS も Hyper-V 仮想化レイヤー上で実行されるためです。 ただし、ゲスト OS とは異なり、ホスト OS は、すべてのハードウェアに直接アクセスできるという点で特殊です。つまり、ホスト OS では、特殊なハードウェア要件を持つアプリケーションでも問題なく実行できます。

## <a name="next-step"></a>次の手順

[Windows 10 上に Hyper-V をインストールする](..\quick-start\enable-hyper-v.md)
