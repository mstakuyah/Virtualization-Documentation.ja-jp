---
title: Hyper-V による仮想マシンの作成
description: Windows 10 Creators Update で Hyper-V を使用して新しい仮想マシンを作成する
keywords: Windows 10, Hyper-V, 仮想マシン
author: scooley
ms.date: 04/07/2018
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: 5975f43859368334cab31b6d0eb37f084e9c03ca
ms.sourcegitcommit: 51da93c4548c5df7a9f01e54d46d81b338c874cf
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/28/2019
ms.locfileid: "9031176"
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Hyper-V による仮想マシンの作成

仮想マシンを作成し、オペレーティング システムをインストールします。

過去の 3 つのリリースの上に表示される指示が大幅に変更されたために、仮想マシンを作成するための新しいツール作成したします。

適切な手順については、該当するオペレーティング システムを選択してください。

* [Windows 10 Fall Creators Update 以降](quick-create-virtual-machine.md#windows-10-fall-creators-update)
* [Windows 10 Creators Update](quick-create-virtual-machine.md#windows-10-creators-update)
* [Windows 10 Anniversary Update 以前](quick-create-virtual-machine.md#before-windows-10-creators-update)

それでは始めましょう。

## <a name="windows-10-fall-creators-update"></a>Windows 10 Fall Creators Update

Fall Creators Update ではクイック作成が拡張され、Hyper-V マネージャーから個別に起動できる仮想マシン ギャラリーが追加されています。

Fall Creators Update で新しい仮想マシンを作成するには:

1. スタート メニューから [Hyper-V クイック作成] を開きます。

    ![Windows のスタート メニューのクイック作成ギャラリー](media/quick-create-start-menu.png)

1. オペレーティング システムを選択するか、ローカル インストール ソースを使用して独自のイメージを選択します。

    ![ギャラリー ビュー](media/vmgallery.png)

    1. 独自のイメージを使用して仮想マシンを作成する場合は、**[ローカル インストール ソース]** を選択します。
    1. **[インストール元の変更]** を選択します。
      ![ローカル インストール ソースを使用するためのボタン](media/change-source.png)
    1. 新しいバーチャル マシンに変換する .iso または .vhdx を選択します。
    1. イメージが Linux イメージの場合は、セキュア ブート オプションを選択解除します。
      ![ローカル インストール ソースを使用するためのボタン](media/toggle-secure-boot.png)

1. [仮想マシンの作成] を選択します。

これで完了です。  残りの操作はクイック作成によって行われます。

## <a name="windows-10-creators-update"></a>Windows 10 Creators Update

![クイック作成 UI のスクリーン ショット](media/quickcreatesteps_inked.jpg)

1. スタート メニューから [Hyper-V マネージャー] を開きます。

1. Hyper-V マネージャーで、右側の **[操作]** メニューの **[クイック作成]** を選択します。

1. 仮想マシンをカスタマイズします。

    * (オプション) 仮想マシンに名前を付けます。
    * 仮想マシンのインストール メディアを選択します。 .iso ファイルまたは .vhdx ファイルからインストールできます。
    仮想マシンに Windows をインストールする場合は、Windows セキュア ブートを有効にすることができます。 それ以外の場合は、オフのままにします。
    * ネットワークを設定します。
    既存の仮想スイッチがある場合は、ネットワークのドロップダウンで選択できます。 既存のスイッチがない場合は、自動でネットワークを設定するボタンが表示され、これにより仮想ネットワークが自動的に構成されます。

1. **[作成]** をクリックして、仮想マシンの作成を開始します。 設定の編集について心配する必要はありません。いつでも前の手順に戻って設定を変更できます。

    "CD または DVD からブートするには、いずれかのキーを押してください" というメッセージが表示される場合があります。 指示に従って操作してください。  CD からインストールしていることが認識されています。

これで、新しい仮想マシンが作成されました。  オペレーティング システムをインストールする準備が整いました。

仮想マシンは次のように表示されます。

![仮想マシンの開始画面](media/OSDeploy_upd.png)

> **注意:** ボリューム ライセンス版の Windows を実行している場合を除き、仮想マシン内で実行する Windows については、別途ライセンスが必要です。 仮想マシンのオペレーティング システムは、ホストのオペレーティング システムから独立しています。

## <a name="before-windows-10-creators-update"></a>Windows 10 Creators Update 以前

Windows 10 Creators Update 以降を実行していない場合は、仮想マシンの新規作成ウィザードを使用して以下の手順に従います。

1. [仮想ネットワークを作成する](connect-to-network.md)
1. [新しい仮想マシンを作成する](create-virtual-machine.md)
