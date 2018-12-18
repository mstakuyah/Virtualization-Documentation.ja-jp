---
title: Windows、Windows 10 の Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 11/8/2018
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: dc500a7b6c0f8f078820407e6ed80ca5868bf4f3
ms.sourcegitcommit: 95cec99aa8e817d3e3cb2163bd62a32d9e8f7181
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/18/2018
ms.locfileid: "8973652"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 の Windows コンテナー

> [!div class="op_single_selector"]
> - [Linux Containers on Windows](quick-start-windows-10-linux.md)
> - [Windows 版の Windows コンテナー](quick-start-windows-10.md)

練習を作成して、Windows 10 の Windows コンテナーを実行しているを紹介します。

このクイック スタートでは、次の実行されます。

1. Windows 版の Docker をインストールします。
2. 単純な Windows コンテナーの実行

このクイック スタートは、Windows 10 に固有です。 その他のクイック スタート ドキュメントは、このページの左側の目次に記載されています。

## <a name="prerequisites"></a>前提条件
次の要件を満たしていることを確認してください。
- 1 つの物理コンピューター システムが Windows 10 Professional または Enterprise の記念日の更新プログラム (バージョン 1607) またはそれ以降を実行します。 
- [HYPER-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)が有効になっていることを確認します。

***HYPER-V 分離:*** Windows Server コンテナーで同じバージョンのカーネルと運用環境で使用される構成開発者に提供するために Windows 10 で HYPER-V 分離を必要より HYPER-V に関する分離にある [[バージョン情報コンテナー](../about/index.md)のページ。

> [!NOTE]
> Windows 10 月の更新プログラム 2018 のリリースで実行されている Windows コンテナー プロセス分離モードで Windows 10 のエンタープライズまたは Professional 開発/テスト用のユーザーなった禁止します。 詳細については、[よく寄せられる質問](../about/faq.md)を参照してください。

## <a name="install-docker-for-windows"></a>Windows 版の Docker をインストールします。

[Windows 版の Docker](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードして (にログインする必要があるインストーラーを実行します。 作成アカウントを既に持っていない場合)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

## <a name="switch-to-windows-containers"></a>コンテナーをウィンドウに切り替える

インストール後、Docker for Windows の既定では、Linux コンテナーが実行されます。 Docker トレイ] メニューを使用して、Windows コンテナーに切り替えるか、PowerShell で次のコマンドを実行して、メッセージを表示します。

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>コンテナーの基本イメージのインストール

Windows コンテナーは、基本イメージから作成されます。 次のコマンドは、Nano Server の基本イメージをプルします。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

イメージのプルが完了した後、`docker images` を実行するとインストール済みのイメージのリストが返されます。この場合は Nano Server のイメージです。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> コンテナーの Windows OS イメージ[使用許諾契約書](../images-eula.md)を参照してください。

## <a name="run-your-first-windows-container"></a>初めて Windows コンテナーを実行します。

このシンプルな例では、' Hello World' コンテナーの画像を作成し、展開します。 最適なパフォーマンスでご利用いただくために、管理者特権で起動した Windows CMD シェルまたは PowerShell で以下のコマンドを実行します。

> Windows PowerShell ISE は、コンテナーとの対話型セッションには機能しません。 コンテナーが実行されている場合でも、ハングしているように見えます。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが起動すると、コンテナーからコマンド シェルが表示されます。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

コンテナー内単純な ' Hello World' テキスト ファイルを作成します。

```cmd
echo "Hello World!" > Hello.txt
```   

スクリプトが完成したら、コンテナーを終了します。

```cmd
exit
```

次に、変更したコンテナーから新しいコンテナー イメージを作成します。 コンテナーの一覧を表示するために、次を実行します。該当するコンテナー ID をメモします。

```console
docker ps -a
```

次のコマンドを実行して、"HelloWorld" というイメージを新たに作成します。 <containerid> を目的のコンテナーの ID に置き換えます。

```console
docker commit <containerid> helloworld
```

操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、次のコマンドで確認できます。

```console
docker images
```

最後に、コンテナーを実行するには、`docker run` コマンドを使用します。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

結果、 `docker run` ] コマンドが、HYPER-V コンテナーが 'HelloWorld' イメージから作成された、cmd のインスタンスでコンテナーが開始したファイル (shell エコー出力) し、[停止し、削除、コンテナーの閲覧を実行します。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルのアプリを作成する方法をについてください。](./building-sample-app.md)