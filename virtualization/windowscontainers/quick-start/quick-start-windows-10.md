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
ms.openlocfilehash: 4202908f8797a2b98ab657c45cd9a6b33191bd6f
ms.sourcegitcommit: 9dfef8d261f4650f47e8137a029e893ed4433a86
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/09/2018
ms.locfileid: "6224901"
---
# <a name="windows-and-linux-containers-on-windows-10"></a>Windows、Windows 10 の Linux コンテナー

練習を作成して、Windows 10 の Windows と Linux コンテナーを実行しているを紹介します。 完了したら、する必要があります。

1. Windows 版の Docker がインストールされています。
2. 単純な Windows コンテナーを実行します。
3. Windows (LCOW) で Linux コンテナーを使用して単純な Linux コンテナーを実行します。

このクイック スタートは、Windows 10 に固有です。 その他のクイック スタート ドキュメントは、このページの左側の目次に記載されています。

***HYPER-V 分離:*** Windows Server コンテナーで同じバージョンのカーネルと運用環境で使用される構成開発者に提供するために Windows 10 で HYPER-V 分離を必要より HYPER-V に関する分離にある [[バージョン情報コンテナー](../about/index.md)のページ。

**前提条件:**

- 物理コンピューター システムを実行している Windows 10 秋作成者の更新プログラム (バージョン 1709) またはそれ以降 (Professional または Enterprise) [HYPER-V を実行できる](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)こと

> このチュートリアルで LCOW を実行するのに表示されていない、Windows のコンテナーが Windows 10 の記念日の更新プログラム (バージョン 1607) またはそれ以降を実行することになります。

## <a name="1-install-docker-for-windows"></a>1. Docker for Windows をインストールする

[Windows 版の Docker をダウンロード](https://store.docker.com/editions/community/docker-ce-desktop-windows)して実行インストーラー (行う必要がありますにログインします。 作成アカウントを既に持っていない場合)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

> インストールされている Docker がある場合 LCOW をサポートするには、18.02 以降のバージョンがあることを確認します。 チェックを実行して`docker -v`や*に関する Docker*をチェックします。

> '実験機能] のオプションで*Docker 設定 > デーモン*LCOW コンテナーを実行するアクティブ化する必要があります。

## <a name="2-switch-to-windows-containers"></a>2. Windows コンテナーに切り替える

インストール後、Docker for Windows の既定では、Linux コンテナーが実行されます。 Docker のシステム トレイのメニューを使用するか、PowerShell プロンプトで `& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon` コマンドを実行して、Windows コンテナーに切り替えます。

![](./media/docker-for-win-switch.png)
> Windows のコンテナーに加えて LCOW コンテナーの Windows コンテナー モードは、注意してください。

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

## <a name="4-run-your-first-windows-container"></a>4.、最初の Windows コンテナーを実行します。

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

## <a name="run-your-first-lcow-container"></a>最初の LCOW コンテナーを実行します。

この例では、BusyBox コンテナーが配置されます。 まず、' Hello World' BusyBox のイメージを実行します。

```
docker run --rm busybox echo hello_world
```

このエラーを返します Docker イメージを取得しようとしたときに注意してください。 これは、Dockers 経由でディレクティブを必要とするため、`--platform`画像とホスト オペレーティング システムが適切に一致することを確認するフラグを設定します。 Windows コンテナー モードで既定のプラットフォームが Windows であるため、追加する`--platform linux`フラグを抽出し、コンテナーを実行します。

```
docker run --rm --platform linux busybox echo hello_world
```

画像が表示された、プラットフォームのとして別途された後、`--platform`フラグが不要になった。 これをテストすることがなくコマンドを実行します。

```
docker run --rm busybox echo hello_world
```

実行`docker images`にインストールされている画像のリストを返します。 この例では、Windows および Linux の両方の画像。

```
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
busybox                latest              59788edf1f3e        4 weeks ago         3.41MB
```

## <a name="next-steps"></a>次のステップ

ボーナス: LCOW を実行して Docker の対応する[ブログ投稿記事](https://blog.docker.com/2018/02/docker-for-windows-18-02-with-windows-10-fall-creators-update/)を参照してください。

次のチュートリアルに進み、[サンプル アプリのビルド](./building-sample-app.md)の例をご覧ください。
