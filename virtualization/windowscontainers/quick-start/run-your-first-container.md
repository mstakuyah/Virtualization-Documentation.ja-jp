---
title: Windows 10 の windows と Linux のコンテナー
description: コンテナー展開のクイック スタート
keywords: docker、コンテナー、LCOW
author: cwilhit
ms.date: 09/11/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: bb9bfbe0-5bdc-4984-912f-9c93ea67105f
ms.openlocfilehash: 3d651a4a68acefa25f1b647b1b33618bbfb91ae9
ms.sourcegitcommit: 868a64eb97c6ff06bada8403c6179185bf96675f
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/13/2019
ms.locfileid: "10129372"
---
# <a name="get-started-run-your-first-container"></a>はじめに: 最初のコンテナーを実行する

前の[セグメント](./set-up-environment.md)で、コンテナーを実行するための環境を構成しました。 この演習では、コンテナーイメージを取得して実行する方法について説明します。

## <a name="install-container-base-image"></a>コンテナーベースイメージをインストールする

すべてのコンテナーはから`container images`インスタンス化されます。 Microsoft には、いくつかの "スターター `base images`" 画像 (呼び出されます) が用意されています。 次のコマンドは、Nano Server の基本イメージをプルします。

```console
docker pull mcr.microsoft.com/windows/nanoserver:1809
```

> [!TIP]
> "という`no matching manifest for unknown in the manifest list entries`エラーメッセージが表示される場合は、Docker が Linux コンテナーを実行するように構成されていないことを確認します。

画像が引き出されたら、ローカルの docker イメージリポジトリを照会することによって、その画像がコンピューターに存在することを確認できます。 このコマンド`docker images`を実行すると、インストールされている画像の一覧が返されます。この場合は、Nano Server のイメージです。

```console
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
microsoft/nanoserver   latest              105d76d0f40e        4 days ago          652 MB
```

> [!IMPORTANT]
> Windows コンテナ OS イメージ使用[許諾契約書](../images-eula.md)をお読みください。

## <a name="run-your-first-windows-container"></a>Windows の最初のコンテナーを実行する

この簡単な例では、"Hello World" コンテナーイメージを作成し、展開します。 最適なエクスペリエンスを実現するには、昇格された Windows CMD shell または PowerShell で次のコマンドを実行します。

> Windows PowerShell ISE は、コンテナーとの対話型セッションには機能しません。 コンテナーが実行されている場合でも、ハングしているように見えます。

まず、`nanoserver` イメージから対話型セッションでコンテナーを起動します。 コンテナーが開始されると、コンテナー内からコマンドシェルが表示されます。  

```console
docker run -it mcr.microsoft.com/windows/nanoserver:1809 cmd.exe
```

コンテナー内で、シンプルな "Hello World" テキストファイルを作成します。

```cmd
echo "Hello World!" > Hello.txt
```   

スクリプトが完成したら、コンテナーを終了します。

```cmd
exit
```

変更したコンテナーから新しいコンテナーイメージを作成します。 実行されている、または終了しているコンテナーの一覧を表示するには、次を実行して、コンテナー id をメモします。

```console
docker ps -a
```

次のコマンドを実行して、"HelloWorld" というイメージを新たに作成します。 `<containerid>` を目的のコンテナーの ID に置き換えます。

```console
docker commit <containerid> helloworld
```

操作を完了すると、hello world スクリプトを含むカスタム イメージが作成されます。 これは、次のコマンドで確認できます。

```console
docker images
```

最後に、 `docker run`コマンドを使用してコンテナーを実行します。

```console
docker run --rm helloworld cmd.exe /s /c type Hello.txt
```

`docker run`コマンドの結果として、コンテナーが ' HelloWorld ' イメージから作成されたため、コンテナーで cmd のインスタンスが開始され、ファイルの読み取り (シェルへのエコーが発生) が実行され、次にコンテナーが停止および削除されます。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリの containerize 方法について](./building-sample-app.md)
