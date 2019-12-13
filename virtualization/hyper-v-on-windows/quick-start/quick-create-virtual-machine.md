---
title: Hyper-V による仮想マシンの作成
description: Windows 10 Creators Update で Hyper-V を使用して新しい仮想マシンを作成する
keywords: Windows 10, Hyper-V, 仮想マシン
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 970e92def02e5386d38a2e72d5ef921aa8321fdf
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74911402"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Hyper-V による仮想マシンの作成

仮想マシンを作成し、オペレーティング システムをインストールします。

仮想マシンを作成するための新しいツールを作成しています。これにより、過去3回のリリースでの手順が大幅に変更されました。

適切な手順については、該当するオペレーティング システムを選択してください。

* [Windows 10 フォール作成者更新プログラム (v1709) 以降](quick-create-virtual-machine.md#windows-10-fall-creators-update-windows-10-version-1709)
* [Windows 10 クリエーター更新プログラム (v1703)](quick-create-virtual-machine.md#windows-10-creators-update-windows-10-version-1703)
* [Windows 10 記念日更新 (v1607) 以前](quick-create-virtual-machine.md#before-windows-10-creators-update-windows-10-version-1607-and-earlier)

それでは始めましょう。

## <a name="windows-10-fall-creators-update-windows-10-version-1709"></a>Windows 10 の作成者の更新プログラム (Windows 10 バージョン 1709)

Fall Creators Update ではクイック作成が拡張され、Hyper-V マネージャーから個別に起動できる仮想マシン ギャラリーが追加されています。

Fall Creators Update で新しい仮想マシンを作成するには:

1. スタート メニューから [Hyper-V クイック作成] を開きます。

    ![Windows のスタート メニューのクイック作成ギャラリー](media/quick-create-start-menu.png)

1. オペレーティング システムを選択するか、ローカル インストール ソースを使用して独自のイメージを選択します。

    ![ギャラリー ビュー](media/vmgallery.png)

    1. 独自のイメージを使用して仮想マシンを作成する場合は、 **[ローカル インストール ソース]** を選択します。
    1. **[インストール元の変更]** を選択します。
      ローカルインストールソースを使用する ![ボタン](media/change-source.png)
    1. 新しい仮想マシンに変換する .iso または .vhdx を選択します。
    1. イメージが Linux イメージの場合は、セキュア ブート オプションを選択解除します。
      ローカルインストールソースを使用する ![ボタン](media/toggle-secure-boot.png)

1. [仮想マシンの作成] を選択します。

以上で作業は終了です。  残りの操作はクイック作成によって行われます。

## <a name="windows-10-creators-update-windows-10-version-1703"></a>Windows 10 の作成者の更新 (Windows 10 バージョン 1703)

![クイック作成 UI のスクリーン ショット](media/quickcreatesteps_inked.jpg)

1. スタート メニューから [Hyper-V マネージャー] を開きます。

1. Hyper-V マネージャーで、右側の **[操作]** メニューの **[簡易作成]** を選択します。

1. 仮想マシンをカスタマイズします。

    * (省略可能) 仮想マシンに名前を付けます。
    * 仮想マシンのインストール メディアを選択します。 .iso ファイルまたは .vhdx ファイルからインストールできます。
    仮想マシンに Windows をインストールする場合は、Windows セキュア ブートを有効にすることができます。 それ以外の場合は、オフのままにします。
    * ネットワークを設定します。
    既存の仮想スイッチがある場合は、ネットワークのドロップダウンで選択できます。 既存のスイッチがない場合は、自動でネットワークを設定するボタンが表示され、これにより仮想ネットワークが自動的に構成されます。

1. **[作成]** をクリックして、仮想マシンの作成を開始します。 設定の編集について心配する必要はありません。いつでも前の手順に戻って設定を変更できます。

    "CD または DVD からブートするには、いずれかのキーを押してください" というメッセージが表示される場合があります。 指示に従って操作してください。  CD からインストールしていることが認識されています。

これで、新しい仮想マシンが作成されました。  オペレーティング システムをインストールする準備が整いました。

仮想マシンは次のように表示されます。

![仮想マシンの開始画面](media/OSDeploy_upd.png)

> **注:** ボリューム ライセンス版の Windows を実行していない限り、仮想マシン内で実行されている Windows のライセンスを区別する必要があります。 仮想マシンのオペレーティング システムは、ホストのオペレーティング システムに依存しません。

## <a name="before-windows-10-creators-update-windows-10-version-1607-and-earlier"></a>Windows 10 の作成者が更新する前 (Windows 10 バージョン1607およびそれ以前)

Windows 10 Creators Update 以降を実行していない場合は、代わりに、仮想マシンの新規作成ウィザードを使用してこれらの手順に従います。

1. [仮想ネットワークの作成](connect-to-network.md)
1. [新しい仮想マシンを作成する](create-virtual-machine.md)
