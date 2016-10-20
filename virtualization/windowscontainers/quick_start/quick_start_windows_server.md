---
title: "Windows Server の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: af648c1235ab9af181a88a65901401bfbd40656e
ms.openlocfilehash: 791de65ac6e4222c4cae77fe9dd24f4e07e5a936

---

# Windows Server の Windows コンテナー

この演習では、Windows Server の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 完了後、コンテナー ロールがインストールされ、シンプルな Windows Server コンテナーが展開されます。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 情報は[クイック スタートの概要](./quick_start.md)にあります。

このクイック スタートは Windows Server 2016 の Windows Server コンテナーのみに適用されます。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

Windows Server 2016 を実行している 1 台のコンピューター システム (物理または仮想)。 Windows Server 2016 TP5 を使用している場合は、[Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ) に更新してください。 

> Windows コンテナー機能が動作するためには、重要な更新プログラムが必要です。 すべての更新プログラムをインストールしてから、このチュートリアルを進めてください。

## 1.Docker のインストール

Docker をインストールするには、[OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 プロバイダーは、コンピューターでコンテナー機能を有効にして、Docker をインストールします。これには、再起動が必要です。 Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

最初に、OneGet PowerShell モジュールをインストールします。

```none
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
```

次に、OneGet を使用して最新バージョンの Docker をインストールします。
```none
Install-Package -Name docker -ProviderName DockerMsftProvider
```

PowerShell でパッケージ ソース "DockerDefault" を信頼するかどうかの確認を求められたら、「A」と入力してインストールを続行します。 インストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

## 2.最初のコンテナーの展開

この演習では、事前作成された .NET サンプル イメージを Docker Hub レジストリからダウンロードし、.NET Hello World アプリケーションを実行するシンプルなコンテナーを展開します。  

`docker run` を使用し、.NET コンテナーを展開します。 この操作でコンテナー イメージもダウンロードされるので、処理に数分かかる可能性があります。

```none
docker run microsoft/sample-dotnet
```

コンテナーが起動し、hello world メッセージが出力され、終了します。

```none
       Welcome to .NET Core!
    __________________
                      \
                       \
                          ....
                          ....'
                           ....
                        ..........
                    .............'..'..
                 ................'..'.....
               .......'..........'..'..'....
              ........'..........'..'..'.....
             .'....'..'..........'..'.......'.
             .'..................'...   ......
             .  ......'.........         .....
             .                           ......
            ..    .            ..        ......
           ....       .                 .......
           ......  .......          ............
            ................  ......................
            ........................'................
           ......................'..'......    .......
        .........................'..'.....       .......
     ........    ..'.............'..'....      ..........
   ..'..'...      ...............'.......      ..........
  ...'......     ...... ..........  ......         .......
 ...........   .......              ........        ......
.......        '...'.'.              '.'.'.'         ....
.......       .....'..               ..'.....
   ..       ..........               ..'........
          ............               ..............
         .............               '..............
        ...........'..              .'.'............
       ...............              .'.'.............
      .............'..               ..'..'...........
      ...............                 .'..............
       .........                        ..............
        .....
```

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

## 次の手順

[Windows Server のコンテナー イメージ](./quick_start_images.md)

[Windows 10 の Windows コンテナー](./quick_start_windows_10.md)



<!--HONumber=Oct16_HO2-->


