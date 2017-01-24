---
title: "Windows Server の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 54eff4bb74ac9f4dc870d6046654bf918eac9bb5
ms.openlocfilehash: 31a55ead81e05f7dee1d1f4f8a2101114d1ca017

---

# Windows Server の Windows コンテナー

この演習では、Windows Server 2016 の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 この演習では、コンテナーの役割をインストールし、単純な Windows Server コンテナーを展開します。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 この情報は[クイック スタートの概要](./index.md)にあります。

このクイック スタートは Windows Server 2016 の Windows Server コンテナーのみに適用されます。 このページの左側の目次に、Windows 10 のコンテナーを含む追加のクイック スタート文書があります。

**前提条件:**

Windows Server 2016 を実行している 1 台のコンピューター システム (物理または仮想)。 Windows Server 2016 TP5 を使用している場合は、[Window Server 2016 Evaluation](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2016 ) に更新してください。 

> Windows コンテナー機能が動作するためには、重要な更新プログラムが必要です。 すべての更新プログラムをインストールしてから、このチュートリアルを進めてください。

Azure に展開する場合は、こちらの[テンプレート](https://github.com/Microsoft/Virtualization-Documentation/tree/master/windows-server-container-tools/containers-azure-template)を使用すると展開が簡単です。<br/>
<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>


## 1.Docker のインストール

Docker をインストールするには、[OneGet プロバイダー PowerShell モジュール](https://github.com/oneget/oneget)を使用します。 プロバイダーは、コンピューターでコンテナーの機能を有効にします。 また、再起動が必要になる Docker をインストールします。 Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。

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

## 2.Windows の更新プログラムをインストールする

次を実行して、Windows Server システムが最新の状態であることを確認します。

```none
sconfig
```

これによりテキスト ベースの構成メニューが表示されます。オプション 6 "更新プログラムのダウンロードとインストール" を選択します。

```none
===============================================================================
                         Server Configuration
===============================================================================

1) Domain/Workgroup:                    Workgroup:  WORKGROUP
2) Computer Name:                       WIN-HEFDK4V68M5
3) Add Local Administrator
4) Configure Remote Management          Enabled

5) Windows Update Settings:             DownloadOnly
6) Download and Install Updates
7) Remote Desktop:                      Disabled
...
```

画面の指示に従い、オプション A を選択してすべての更新プログラムをダウンロードします。

## 3.最初のコンテナーの展開

この演習では、事前作成された .NET サンプル イメージを Docker Hub レジストリからダウンロードし、.NET Hello World アプリケーションを実行するシンプルなコンテナーを展開します。  

`docker run` を使用し、.Net コンテナーを展開します。 この操作でコンテナー イメージもダウンロードされるので、処理に数分かかる可能性があります。

```console
docker run microsoft/dotnet-samples:dotnetapp-nanoserver
```

コンテナーが起動し、hello world メッセージが出力され、終了します。

```console
         Dotnet-bot: Welcome to using .NET Core!
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


**Environment**
Platform: .NET Core 1.0
OS: Microsoft Windows 10.0.14393
```

Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。

## 次の手順

[Windows Server のコンテナー イメージ](./quick-start-images.md)

[Windows 10 の Windows コンテナー](./quick-start-windows-10.md)



<!--HONumber=Jan17_HO3-->


