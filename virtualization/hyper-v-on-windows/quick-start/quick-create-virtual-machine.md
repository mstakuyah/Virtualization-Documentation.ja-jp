---
title: "Hyper-V による仮想マシンの作成"
description: "Windows 10 Creators Update で Hyper-V を使用して新しい仮想マシンを作成する"
keywords: windows 10, hyper-v
author: aoatkinson
ms.date: 04/07/2017
ms.topic: article
ms.prod: windows-10-hyperv
ms.assetid: f1e75efa-8745-4389-b8dc-91ca931fe5ae
ms.openlocfilehash: fefef30c3ebbda965f9b1077082a05c8e3ffc6c1
ms.sourcegitcommit: 075985b1e0ee62dc1c16573c40cd1f62427ded3c
ms.translationtype: HT
ms.contentlocale: ja-JP
---
# <a name="create-a-virtual-machine-with-hyper-v"></a>Hyper-V による仮想マシンの作成

仮想マシンを作成し、オペレーティング システムをインストールします。  

実行するオペレーティング システムの .iso ファイルが必要です。 このファイルがない場合は、[TechNet Evaluation Center](http://www.microsoft.com/en-us/evalcenter/) から Windows の評価版を取得します。


> Windows 10 Creators Update では、新しい仮想マシンの作成を効率化する新しい**簡易作成**ツールが導入されました。  
  Windows 10 Creators Update 以降を実行していない場合は、代わりに、仮想マシンの新規作成ウィザードを使用してこれらの手順に従います。  
  [新しい仮想マシンを作成する](create-virtual-machine.md)  
  [仮想ネットワークを作成する](connect-to-network.md)

それでは始めましょう。

![](media/quickcreatesteps_inked.jpg)

1. **Hyper-V マネージャーを開く**  
  Windows キーを押して「Hyper-V マネージャー」と入力して Hyper-V マネージャーのアプリケーションを検索するか、スタート メニューのアプリケーションの一覧をスクロールして [Hyper-V マネージャー] を見つけます。

2. **簡易作成を開く**  
  Hyper-V マネージャーで、右側の **[操作]** メニューの **[簡易作成]** を選択します。

3. **仮想マシンをカスタマイズする**
  * (省略可能) 仮想マシンに名前を付けます。  
    この名前は、Hyper-V が仮想マシンに使用する名前であり、仮想マシン内に展開されるゲスト オペレーティング システムに指定されるコンピューター名ではありません。
  * 仮想マシンのインストール メディアを選択します。 .iso ファイルまたは .vhdx ファイルからインストールできます。  
    仮想マシンに Windows をインストールする場合は、Windows セキュア ブートを有効にすることができます。 それ以外の場合は、オフのままにします。
  * ネットワークを設定します。  
    既存の仮想スイッチがある場合は、ネットワークのドロップダウンで選択できます。 既存のスイッチがない場合は、自動でネットワークを設定するボタンが表示され、これにより外部スイッチが自動的に構成されます。

4. **仮想マシンに接続する**  
  **[接続]** を選択して仮想マシン接続を開始し、仮想マシンを起動します。     
  設定の編集について心配する必要はありません。いつでも前の手順に戻って設定を変更できます。  
  
    "CD または DVD からブートするには、いずれかのキーを押してください" というメッセージが表示される場合があります。 指示に従って操作してください。  CD からインストールしていることが認識されています。

これで、新しい仮想マシンが作成されました。  オペレーティング システムをインストールする準備が整いました。  

仮想マシンは次のように表示されます。  
![](media/OSDeploy_upd.png) 

> **注:** ボリューム ライセンス版の Windows を実行していない限り、仮想マシン内で実行されている Windows のライセンスを区別する必要があります。 仮想マシンのオペレーティング システムは、ホストのオペレーティング システムに依存しません。
