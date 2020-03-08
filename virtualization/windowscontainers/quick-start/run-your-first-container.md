---
title: Windows 10 の windows コンテナーと Linux コンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 551d405d836cfb16b587ef78bc2d5f5abbd8648f
ms.sourcegitcommit: ac923217ee2f74f08df2b71c2a4c57b694f0d7c3
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/06/2020
ms.locfileid: "78853816"
---
# <a name="get-started-run-your-first-windows-container"></a>はじめに: 初めての Windows コンテナーを実行する

このトピックでは、「[作業の開始: コンテナー用に Windows を準備](./set-up-environment.md)する」の説明に従って環境を設定した後に、最初の Windows コンテナーを実行する方法について説明します。 コンテナーを実行するには、最初に基本イメージをインストールします。これにより、コンテナーにオペレーティングシステムサービスの基本層が提供されます。 次に、基本イメージに基づいたコンテナーイメージを作成して実行します。 詳細については、「」を参照してください。

## <a name="install-a-container-base-image"></a>コンテナーの基本イメージをインストールする

すべてのコンテナーは、コンテナー イメージから作成されます。 Microsoft では、基本イメージと呼ばれるいくつかのスターターイメージを提供しています (詳細については、「[コンテナーの基本イメージ](../manage-containers/container-base-images.md)」を参照してください)。 この手順では、ライトウェイト Nano Server 基本イメージをプル (ダウンロードしてインストール) します。

1. コマンドプロンプトウィンドウ (組み込みコマンドプロンプト、PowerShell、 [Windows ターミナル](https://www.microsoft.com/p/windows-terminal-preview/9n0dx20hk701?activetab=pivot:overviewtab)など) を開き、次のコマンドを実行して基本イメージをダウンロードしてインストールします。

   ```console
   docker pull mcr.microsoft.com/windows/nanoserver:1903
   ```

   > [!TIP]
   > `no matching manifest for unknown in the manifest list entries`というエラーメッセージが表示された場合は、Docker が Linux コンテナーを実行するように構成されていないことを確認してください。

2. イメージのダウンロードが完了したら、待機中に[EULA](../images-eula.md)をお読みください。ローカルの docker イメージリポジトリにクエリを実行して、システム上に存在することを確認します。 コマンド `docker images` を実行すると、インストールされているイメージの一覧が返されます。

   Nano Server イメージを示す出力の例を次に示します。

   ```console
   REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
   microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
   ```

## <a name="run-a-windows-container"></a>Windows コンテナーを実行する

この簡単な例では、' Hello World ' コンテナーイメージを作成してデプロイします。 最良の結果を得るには、管理者特権のコマンドプロンプトウィンドウでこれらのコマンドを実行します (ただし、Windows PowerShell ISE は使用しないでください。コンテナーがハングしているように見えますが、コンテナーとの対話型セッションでは機能しません)。

1. コマンドプロンプトウィンドウで次のコマンドを入力して、`nanoserver` イメージから対話型セッションでコンテナーを起動します。

   ```console
   docker run -it mcr.microsoft.com/windows/nanoserver:1903 cmd.exe
   ```
2. コンテナーが起動すると、コマンドプロンプトウィンドウによってコンテキストがコンテナーに変わります。 コンテナー内で、単純な ' Hello World ' テキストファイルを作成し、次のコマンドを入力してコンテナーを終了します。

   ```cmd
   echo "Hello World!" > Hello.txt
   exit
   ```   

3. [Docker ps](https://docs.docker.com/engine/reference/commandline/ps/)コマンドを実行して、先ほど終了したコンテナーのコンテナー ID を取得します。

   ```console
   docker ps -a
   ```

4. 最初に実行したコンテナーの変更を含む新しい "HelloWorld" イメージを作成します。 これを行うには、 [docker commit](https://docs.docker.com/engine/reference/commandline/commit/)コマンドを実行し、`<containerid>` をコンテナーの ID に置き換えます。

   ```console
   docker commit <containerid> helloworld
   ```

   操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、 [docker images](https://docs.docker.com/engine/reference/commandline/images/)コマンドで確認できます。

   ```console
   docker images
   ```

   この場合の出力例を以下に示します。

   ```console
   REPOSITORY                             TAG                 IMAGE ID            CREATED             SIZE
   helloworld                             latest              a1064f2ec798        10 seconds ago      258MB
   mcr.microsoft.com/windows/nanoserver   1903                2b9c381d0911        3 weeks ago         256MB
   ```

5. 最後に、 [docker run](https://docs.docker.com/engine/reference/commandline/run/)コマンドと `--rm` パラメーターを使用して、新しいコンテナーを実行します。コマンドライン (cmd.exe) が停止すると、コンテナーが自動的に削除されます。

   ```console
   docker run --rm helloworld cmd.exe /s /c type Hello.txt
   ```

   結果として、コンテナーが ' HelloWorld ' イメージから作成され、ファイルを読み取るコンテナーで cmd.exe のインスタンスが開始され、ファイルの内容がシェルに出力された後、コンテナーが停止して削除されたことになります。

## <a name="next-steps"></a>次のステップ:

> [!div class="nextstepaction"]
> [サンプルアプリをコンテナー化する方法について説明します](./building-sample-app.md)
