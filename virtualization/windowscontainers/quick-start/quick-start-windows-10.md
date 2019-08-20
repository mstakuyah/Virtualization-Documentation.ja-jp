---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: taylorb-microsoft
ms.date: 08/16/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: aa7a20d914fdb65597c0f31ef6d53b91f6497be4
ms.sourcegitcommit: 2f8fd4b2e7113fbb7c323d89f3c72df5e1a4437e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/20/2019
ms.locfileid: "10044962"
---
# <a name="windows-containers-on-windows-10"></a>Windows 10 の Windows コンテナー

この練習では、Windows 10 での Windows コンテナーの作成と実行について説明します。

このクイックスタートでは、次のことを実行します。

1. Docker デスクトップのインストール
2. 簡単な Windows コンテナーの実行

このクイック スタートは、Windows 10 に固有です。 その他のクイックスタートドキュメントは、このページの左側にある目次に記載されています。

## <a name="prerequisites"></a>前提条件
次の要件を満たしていることを確認してください。
- Windows 10 Professional または Enterprise for 記念日更新プログラム (バージョン 1607) 以降を実行している1つの物理コンピューターシステム。 
- [Hyper-v](https://docs.microsoft.com/virtualization/hyper-v-on-windows/reference/hyper-v-requirements)が有効になっていることを確認します。

***Hyper-v 分離:*** Windows Server コンテナーには、開発者に対して、製造時に使用されるのと同じカーネルバージョンと構成を提供するために、windows 10 の Hyper-v 分離が必要です。 Hyper-v の分離について詳しくは、「 [windows コンテナーのバージョン情報](../about/index.md)」ページをご覧ください。

> [!NOTE]
> Windows 10 年10月の更新プログラム2018のリリースでは、開発者/テスト目的で、windows 10 Enterprise または Professional のプロセス分離モードでユーザーが Windows コンテナーを実行することを許可することはなくなりました。 詳細については、[よく寄せ](../about/faq.md)られる質問を参照してください。

## <a name="install-docker-desktop"></a>Docker デスクトップをインストールする

[Docker デスクトップ](https://store.docker.com/editions/community/docker-ce-desktop-windows)をダウンロードしてインストーラーを実行します (ログインする必要があります)。 まだアカウントを持っていない場合は、アカウントを作成します)。 [インストールの詳しい手順](https://docs.docker.com/docker-for-windows/install)については、Docker のドキュメントを参照してください。

## <a name="switch-to-windows-containers"></a>Windows コンテナーに切り替える

インストール後の Docker デスクトップの既定では、Linux コンテナーが実行されています。 Docker トレイメニューを使用するか、PowerShell プロンプトで次のコマンドを実行して、Windows コンテナーに切り替えます。

```console
& $Env:ProgramFiles\Docker\Docker\DockerCli.exe -SwitchDaemon .
```

![](./media/docker-for-win-switch.png)

## <a name="install-base-container-images"></a>コンテナーの基本イメージのインストール

Windows コンテナーは、基本イメージから作成されます。 次のコマンドは、Nano Server の基本イメージをプルします。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!NOTE]
> 「」という`no matching manifest for unknown in the manifest list entries`エラーメッセージが表示される場合は、Linux コンテナーをプルする予定ではないことを確認します。

イメージのプルが完了した後、`docker images` を実行するとインストール済みのイメージのリストが返されます。この場合は Nano Server のイメージです。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Windows コンテナ OS イメージ使用[許諾契約書](../images-eula.md)をお読みください。

## <a name="run-your-first-windows-container"></a>Windows の最初のコンテナーを実行する

この簡単な例では、"Hello World" コンテナーイメージを作成し、展開します。 最適なパフォーマンスでご利用いただくために、管理者特権で起動した Windows CMD シェルまたは PowerShell で以下のコマンドを実行します。

> Windows PowerShell ISE は、コンテナーとの対話型セッションには機能しません。 コンテナーが実行されている場合でも、ハングしているように見えます。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが起動すると、コンテナーからコマンド シェルが表示されます。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

コンテナー内に、単純な "Hello World" テキストファイルを作成します。

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

この`docker run`コマンドの結果として、hyper-v の分離環境で実行されているコンテナーが ' HelloWorld ' イメージから作成されたため、コンテナーで cmd のインスタンスが開始され、そのファイル (シェルへのエコー) が実行され、次にコンテナーが実行されました。停止して削除されました。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリを作成する方法について説明します。](./building-sample-app.md)
