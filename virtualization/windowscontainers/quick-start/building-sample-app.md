---
title: サンプル アプリのビルド
description: コンテナーを利用している場合に、サンプル アプリをビルドする方法を説明します。
keywords: Docker, コンテナー
author: cwilhit
ms.date: 07/25/2017
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: 7ffc16e9d5b7c4b4a935a06c012b1d28b5e70f1a
ms.sourcegitcommit: 27e9cd37beaf11e444767699886e5fdea5e1a2d0
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "10058487"
---
# <a name="build-a-sample-app"></a>サンプル アプリのビルド

この演習では、サンプル ASP.net アプリを入手し、それをコンテナーで実行できるように変換する手順を説明します。 Windows 10 でコンテナーを使い始める方法については、「[Windows 10 のクイック スタート](./quick-start-windows-10.md)」をご覧ください。

このクイック スタートは、Windows 10 に固有です。 このページの左側の目次に追加のクイック スタート文書があります。 このチュートリアルの主眼はコンテナーにあるため、コードの記述は別の機会に譲り、コンテナーについてのみ説明します。 チュートリアルをゼロから作成する場合は、[ASP.NET Core のドキュメント](https://docs.microsoft.com/aspnet/core/tutorials/first-mvc-app-xplat/)をご覧ください。

Git ソース管理がまだコンピューターにインストールされていない場合は、[Git のページ](https://git-scm.com/download)から取得できます。

## <a name="getting-started"></a>はじめに

このサンプル プロジェクトは、[VSCode](https://code.visualstudio.com/) を使ってセットアップされています。 また、このチュートリアルでは Powershell も使います。 それでは GitHub からデモ コードを取得しましょう。 Git を使ってリポジトリを複製するか、[SampleASPContainerApp](https://github.com/cwilhit/SampleASPContainerApp) から直接プロジェクトをダウンロードします。

```Powershell
git clone https://github.com/cwilhit/SampleASPContainerApp.git
```

プロジェクト ディレクトリに移動し、Dockerfile を作成します。 [Dockerfile](https://docs.docker.com/engine/reference/builder/) は、makefile に似た、コンテナー イメージの構築方法を記述した命令の一覧です。

```Powershell
#Create the dockerfile for our proj
New-Item C:/Your/Proj/Location/Dockerfile -type file
```

## <a name="writing-our-dockerfile"></a>Dockerfile の作成

それでは、ルート フォルダーに作成した Dockerfile をお好みのテキスト エディターで開き、ロジックを追加しましょう。 その後、このファイルで行われている作業を 1 行ずつ説明していきます。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

最初の一連の行では、コンテナー構築の基盤として使う基本イメージを宣言します。 ローカル システムにこのイメージがまだ存在しない場合、docker が自動的に取得を試みます。 aspnetcore-build には、ここでプロジェクトをコンパイルするための依存関係がパッケージ化されて含まれています。 次に、コンテナー内の作業ディレクトリを "/app" に変更します。これにより、dockerfile 内のすべてのコマンドがこのディレクトリで実行されます。

>[!NOTE]
>プロジェクトを構築する必要があるため、最初に作成する最初のコンテナーは一時的なコンテナーであり、これを実行するために使用するので、最後に破棄します。

```Dockerfile
FROM microsoft/aspnetcore-build:1.1 AS build-env
WORKDIR /app
```

次に、.csproj ファイルを一時コンテナーの ”/app” ディレクトリにコピーします。 これは、.csproj ファイルにプロジェクトで必要なパッケージ参照の一覧が含まれているためです。

このファイルをコピーすると、dotnet によってこのディレクトリからファイルが読み取られ、プロジェクトに必要な依存関係とツールが外部から取得されます。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

それらすべての依存関係が取得されたら、一時コンテナーにコピーします。 次に dotnet でアプリケーションをリリース構成と共に公開し、出力パスを指定します。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

これでプロジェクトのコンパイルが完了しました。 次に完成したコンテナーをビルドする必要があります。 アプリケーションが ASP.NET であるため、イメージとそれらのライブラリをソースとして指定します。 次に、一時コンテナーの出力ディレクトリから、すべてのファイルを最終コンテナーにコピーします。 コンテナーを起動したときに、先ほどコンパイルした新しい .dll を使用してコンテナーが実行されるように構成します。

>[!NOTE]
>この最終的なコンテナーの基本イメージは似ていますが```FROM``` 、上記のコマンドとは似ていますが、ASP.NET アプリを_構築_できるライブラリはありません。実行してください。

```Dockerfile
FROM microsoft/aspnetcore:1.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "MvcMovie.dll"]
```

これで、_マルチステージ ビルド_と呼ばれる作業が完了しました。 ここでは一時コンテナーを使ってイメージをビルドした後、公開した dll を別のコンテナーに移動することで、最終結果のフットプリントを最小限に抑えました。 またこのコンテナーの依存関係を実行に絶対必要な最小限度に抑えました。最初のイメージをそのまま使うと、必須でない (ASP.NET アプリを構築するための) その他のレイヤーがパッケージ化されて含まれるため、イメージのサイズが大きくなります。

## <a name="running-the-app"></a>アプリの実行

dockerfile を作成したら、後はアプリをビルドし、コンテナーを実行するように docker を構成するだけです。 パブリッシュするポートを指定し、コンテナーに "myapp" というタグを付けます。 PowerShell で、以下のコマンドを実行します。

>[!NOTE]
>PowerShell 本体の現在の作業ディレクトリは、上で作成された dockerfile が存在するディレクトリである必要があります。

```Powershell
docker build -t myasp .
docker run -d -p 5000:80 --name myapp myasp
```

このアプリの実行を確認するには、それが実行されているアドレスにアクセスする必要があります。 このコマンドを実行して、IP アドレスを取得してみましょう。

```Powershell
 docker inspect -f "{{ .NetworkSettings.Networks.nat.IPAddress }}" myapp
```

このコマンドを実行すると、実行中のコンテナーの IP アドレスが生成されます。 この出力の例を以下に示します。

```Powershell
 172.19.172.12
```

お好みの Web ブラウザーにこの IP アドレスを入力すると、コンテナーでアプリケーションが正常に実行されている様子が確認できます。

>![](media/SampleAppScreenshot.png)

ナビゲーション バーで [MvcMovie] をクリックすると、映画のエントリを入力、編集、削除できる Web ページが表示されます。

## <a name="next-steps"></a>次の手順

ここでは Docker を使って ASP.NET の Web アプリを正常に入手し、構成、ビルドして実行中のコンテナーに正常に展開しました。 しかし、さらに進んだ手順を実行することもできます。 Web アプリを複数のコンポーネント (Web API を実行するコンテナー、フロント エンドを実行するコンテナー、SQL Server を実行するコンテナーなど) に分割することができます。

これでコンテナーが停止しました。次に、コンテナー内の優れたソフトウェアを構築して構築します。

> [!div class="nextstepaction"]
> [その他のコンテナーのサンプルを確認する](../samples.md)
