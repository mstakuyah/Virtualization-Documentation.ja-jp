---
title: Windows 10 の Windows コンテナー
description: コンテナー展開のクイック スタート
keywords: Docker, コンテナー
author: taylorb-microsoft
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa69739697e9e2c668aaf5026c6f5ddc69d43c4d
ms.sourcegitcommit: f6f53fd0da65ac44d16fb793c5aa1af56c14d90e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/23/2018
ms.locfileid: "5428994"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 の Windows コンテナー

この演習では、Windows 10 Professional または Enterprise (Anniversary Edition) の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 完了後、Docker for Windows がインストールされ、シンプルなコンテナーが実行されます。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイック スタートは、Windows 10 に固有です。 このページの左側の目次に追加のクイック スタート文書があります。

***Hyper-V による分離:*** 運用環境と同じカーネル バージョンと構成を開発者に提供するためには、Windows 10 の Windows Server コンテナーを Hyper-V によって分離する必要があります。これについて詳しくは、「[Windows コンテナーについて](../about/index.md)」のページをご覧ください。

**前提条件: **

- Windows 10 Anniversary Edition または Creators Update (Professional または Enterprise) を実行する 1 台の物理コンピューター システム。   
- このクイック スタートは Windows 10 仮想マシンで実行できますが、入れ子の仮想化を有効にする必要があります。 詳細については、[入れ子になった仮想化に関するガイド](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/user_guide/nesting)を参照してください。

> Windows コンテナーが機能するには、重要な更新プログラムがインストールされている必要があります。
> 使用している OS のバージョンを確認するには、`winver.exe` を実行し、表示されたバージョンを「[Windows 10 の更新履歴](https://support.microsoft.com/en-us/help/12387/windows-10-update-history)」と比較してください。

> 14393.222 以降であることを確認してから、次に進んでください。  このバージョンは、1607 上にあるすべてのバージョンを完全にサポートされる必要があるために、Windows 10 版 1607 に対応します。

## <a name="1-install-docker-for-windows"></a>1. Docker for Windows をインストールする

[Docker for Windows をダウンロード](https://download.docker.com/win/stable/InstallDocker.msi)し、インストーラーを実行します。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

## <a name="2-switch-to-windows-containers"></a>2. Windows コンテナーに切り替える

インストール後、Docker for Windows の既定では、Linux コンテナーが実行されます。 Docker のシステム トレイのメニューを使用するか、PowerShell プロンプトで `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon` コマンドを実行して、Windows コンテナーに切り替えます。

![](./media/docker-for-win-switch.png)

## <a name="3-install-base-container-images"></a>3. コンテナーの基本イメージをインストールする

Windows コンテナーは、基本イメージから作成されます。 次のコマンドは、Nano Server の基本イメージをプルします。

```
docker pull microsoft/nanoserver
```

イメージのプルが完了した後、`docker images` を実行するとインストール済みのイメージのリストが返されます。この場合は Nano Server のイメージです。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> Windows Containers OS Image 使用許諾契約書 (EULA) を参照してください。こちらの「[EULA](../images-eula.md)」に掲載されています。

## <a name="4-run-your-first-container"></a>4. 最初のコンテナーを実行する

この簡単な例では、"Hello World" というコンテナー イメージを作成して展開します。 最適なパフォーマンスでご利用いただくために、管理者特権で起動した Windows CMD シェルまたは PowerShell で以下のコマンドを実行します。

> Windows PowerShell ISE は、コンテナーとの対話型セッションには機能しません。 コンテナーが実行されている場合でも、ハングしているように見えます。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが起動すると、コンテナーからコマンド シェルが表示されます。  

```
docker run -it microsoft/nanoserver cmd
```

コンテナー内で、"Hello World" という簡単なスクリプトを作成することにします。

```
powershell.exe Add-Content C:\helloworld.ps1 'Write-Host "Hello World"'
```   

スクリプトが完成したら、コンテナーを終了します。

```
exit
```

次に、変更したコンテナーから新しいコンテナー イメージを作成します。 コンテナーの一覧を表示するために、次を実行します。該当するコンテナー ID をメモします。

```
docker ps -a
```

次のコマンドを実行して、"HelloWorld" というイメージを新たに作成します。 <containerid> を目的のコンテナーの ID に置き換えます。

```
docker commit <containerid> helloworld
```

操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、次のコマンドで確認できます。

```
docker images
```

最後に、コンテナーを実行するには、`docker run` コマンドを使用します。

```
docker run --rm helloworld powershell c:\helloworld.ps1
```

`docker run` コマンドを実行すると、結果的に、Hyper-V コンテナーが "Hello World" イメージから作成され、サンプル スクリプト "Hello World" が実行されます (出力がシェルにエコーされます)。次に、コンテナーが停止し、削除されます。
後続の Windows 10 とコンテナーのクイック スタートでは、Windows 10 のコンテナーでアプリケーションを作成し、展開する方法について説明します。

## <a name="next-steps"></a>次の手順

次のチュートリアルに進み、[サンプル アプリのビルド](./building-sample-app.md)の例をご覧ください。
