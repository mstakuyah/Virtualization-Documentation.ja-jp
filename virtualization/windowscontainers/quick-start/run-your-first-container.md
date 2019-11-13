---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: a664b5b8eb87adffdf7eba3ffca9f4194128df80
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288130"
---
# <a name="get-started-run-your-first-windows-container"></a>はじめに: Windows の最初のコンテナーを実行する

このトピックでは、「[はじめに: コンテナーのための windows の準備](./set-up-environment.md)」の説明に従って、環境のセットアップ後に初めて windows コンテナーを実行する方法について説明します。 コンテナーを実行するには、最初に基本イメージをインストールします。これは、オペレーティングシステムサービスの基本レイヤーをコンテナーに提供します。 次に、基本イメージに基づいて、コンテナーイメージを作成して実行します。 詳細については、「」を参照してください。

## <a name="install-a-container-base-image"></a>コンテナーの基本イメージをインストールする

すべてのコンテナーは、コンテナイメージから作成されます。 Microsoft では、基本イメージと呼ばれるいくつかのスターターイメージを提供しています (詳しくは、「[コンテナーベースの画像](../manage-containers/container-base-images.md)」をご覧ください)。 この手順では、軽量の Nano Server ベースイメージをプル (ダウンロードおよびインストール) します。

1. コマンドプロンプトウィンドウ (組み込みのコマンドプロンプト、PowerShell、 [Windows ターミナル](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)など) を開き、次のコマンドを実行して基本イメージをダウンロードしてインストールします。

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > "という`no matching manifest for unknown in the manifest list entries`エラーメッセージが表示される場合は、Docker が Linux コンテナーを実行するように構成されていないことを確認します。

2. 画像のダウンロードが完了したら、お使いのお使いの pc で、ローカルの docker イメージリポジトリを照会することによって、使用[許諾契約](../images-eula.md)をお読みください。 このコマンド`docker images`を実行すると、インストールされている画像の一覧が返されます。

   Nano Server のイメージを示す出力の例を次に示します。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Windows コンテナーを実行する

この簡単な例では、"Hello World" コンテナーイメージを作成し、展開します。 最適なエクスペリエンスを実現するには、昇格されたコマンドプロンプトウィンドウで次のコマンドを実行します (ただし、Windows PowerShell ISE は使用しません。コンテナーを使った対話型セッションでは、コンテナーがハングするように見えるため機能しません)。

1. コマンドプロンプトウィンドウで次のコマンドを入力`nanoserver`して、イメージから対話型セッションでコンテナーを開始します。

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. コンテナーが開始されると、コマンドプロンプトウィンドウによってコンテキストがコンテナーに変わります。 コンテナー内で、単純な "Hello World" テキストファイルを作成し、次のコマンドを入力してコンテナーを終了します。

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. 次のように、 [docker ps](https://docs.docker.com/engine/reference/commandline/ps/)コマンドを実行して、終了したコンテナーのコンテナー ID を取得します。

   ```console
   docker ps -a
   ```

4. 最初に実行したコンテナーの変更を含む、新しい "HelloWorld" イメージを作成します。 そのためには、次のように[docker commit](https://docs.docker.com/engine/reference/commandline/commit/)コマンドを実行して、コンテナーの ID に置き換え`<containerid>`ます。

   ```console
   docker commit <containerid> helloworld
   ```

   操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、 [docker images](https://docs.docker.com/engine/reference/commandline/images/)コマンドで表示できます。

   ```console
   docker images
   ```

   この場合の出力例を以下に示します。

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最後に、コマンドライン (cmd.exe) が停止した後に`--rm` 、コンテナーを自動的に削除するパラメーターを使用し[て、新しい](https://docs.docker.com/engine/reference/commandline/run/)コンテナーを実行します。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   その結果、コンテナーは ' HelloWorld ' イメージから作成されました。 cmd.exe のインスタンスは、ファイルを読み取ってシェルにファイルの内容を出力するコンテナーで開始され、コンテナーが停止して削除されました。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリの containerize 方法について](./building-sample-app.md)
