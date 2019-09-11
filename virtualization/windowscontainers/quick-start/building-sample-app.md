---
title: .NET コアアプリの Containerize
description: コンテナーを含む .NET core アプリのサンプルをビルドする方法について学習する
keywords: Docker, コンテナー
author: cwilhit
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: f5a51fd1211868195126f06d917c0bef6e496c3d
ms.sourcegitcommit: f3b6b470dd9cde8e8cac7b13e7e7d8bf2a39aa34
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "10077473"
---
# <a name="containerize-a-net-core-app"></a>.NET コアアプリの Containerize


このクイックスタートでは、簡単な .NET コアアプリケーションを containerize する方法について説明します。 そうするでしょう：

> [!div class="checklist"]
> * GitHub からサンプルのアプリソースを複製する
> * アプリソースを使ってコンテナーイメージを構築するための dockerfile を作成する
> * ローカルの Docker 環境でコンテナーに含まれる .NET コアアプリをテストする

## <a name="before-you-begin"></a>始める前に

このクイックスタートでは、開発環境がコンテナーを使用するように構成されていることを前提とします。 コンテナー用の環境が構成されていない場合は、 [Windows 10 のクイックスタート](./quick-start-windows-10.md)にアクセスして、使用を開始する方法を確認してください。

コンピューターに Git ソースコントロールシステムがインストールされている必要があります。 ここから取得できます。 [Git](https://git-scm.com/download)

## <a name="getting-started"></a>開始するには

すべてのコンテナーサンプルのソースコードは、と呼ばれる`windows-container-samples`フォルダー内の[仮想化ドキュメント](https://github.com/MicrosoftDocs/Virtualization-Documentation)git リポジトリの下に保持されます。 この git リポジトリを curent のワーキングディレクトリに複製します。

```Powershell
git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
```

で`<directory where clone occured>\Virtualization-Documentation\windows-container-samples\asp-net-getting-started`見つかったサンプルディレクトリに移動し、Dockerfile を作成します。 [Dockerfile](https://docs.docker.com/engine/reference/builder/)は、メイクファイルのようなものであり、コンテナーイメージの構築方法についてコンテナーエンジンに指示する命令の一覧です。

```Powershell
#Create the dockerfile for our project
New-Item -name dockerfile -type file
```

## <a name="write-the-dockerfile"></a>Dockerfile を作成する

作成した dockerfile (任意のテキストエディター) を開き、次の内容を追加します。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

1行おきに分割して、各手順について説明します。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

最初の一連の行では、コンテナー構築の基盤として使う基本イメージを宣言します。 ローカル システムにこのイメージがまだ存在しない場合、docker が自動的に取得を試みます。 は`mcr.microsoft.com/dotnet/core/sdk:2.1` 、.net CORE 2.1 SDK をインストールしてパッケージ化されています。そのため、バージョン2.1 をターゲットとする ASP .net コアプロジェクトを構築することができます。 次の命令は、コンテナー内の作業ディレクトリを変更`/app`するため、このコンテキストに続くすべてのコマンドはこのコンテキストで実行されます。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

次に、これらの命令は、.csproj ファイルを`build-env`コンテナーの`/app`ディレクトリにコピーします。 このファイルをコピーした後、.NET によって読み取りが行われ、プロジェクトで必要なすべての依存関係とツールが取得されます。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET がすべての依存関係を`build-env`コンテナーにプルすると、次の命令では、すべてのプロジェクトソースファイルがコンテナーにコピーされます。 次に、.NET にリリース構成を使ってアプリケーションを公開することを通知し、で出力パスを指定します。

コンパイルは成功します。 これで、最終的な画像を作成する必要があります。 

> [!TIP]
> この quickstart はソースから .NET コアプロジェクトを構築します。 コンテナーイメージを作成する場合は、運用ペイロードとその依存関係_だけ_をコンテナーイメージに含めることをお勧めします。 アプリを構築するために呼び出さ`build-env`れる SDK と共にパッケージ化された一時的なコンテナーを使用するように dockerfile が作成されているため、最終的な画像には .NET core SDK は必要ありません。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

このアプリケーションは ASP.NET であるため、このランタイムに含まれている画像を指定します。 次に、一時コンテナーの出力ディレクトリから、すべてのファイルを最終コンテナーにコピーします。 コンテナーが開始されたときに、新しいアプリがエントリポイントとして実行されるようにコンテナーを構成します。

_複数ステージのビルド_を実行するために、dockerfile を作成しました。 Dockerfile が実行されると、一時コンテナー ( `build-env`.net CORE 2.1 SDK) を使ってサンプルアプリを構築し、.net core 2.1 ランタイムのみを含む別のコンテナーに出力されたバイナリをコピーして、最終的なコンテナー。

## <a name="run-the-app"></a>アプリを実行する

Dockerfile が記述されていると、dockerfile で Docker を参照し、イメージを作成するように指示することができます。 

>[!IMPORTANT]
>以下で実行するコマンドは、dockerfile が存在するディレクトリで実行する必要があります。

```Powershell
docker build -t my-asp-app .
```

コンテナーを実行するには、下のコマンドを実行します。

```Powershell
docker run -d -p 5000:80 --name myapp my-asp-app
```

このコマンドを dissect しましょう。

* `-d` Docker tun はコンテナー ' デタッチ ' を実行します。これは、コンテナー内のコンソールにコンソールがフックされないことを意味します。 コンテナーはバックグラウンドで実行されます。 
* `-p 5000:80` ホスト上のポート5000をコンテナーのポート80にマップするように Docker に指示します。 各コンテナーは、独自の IP アドレスを取得します。 ASP .NET は、既定でポート80でリッスンします。 ポートマッピングを使用すると、マップされたポートでホストの IP アドレスにアクセスでき、Docker はすべてのトラフィックをコンテナー内の宛先ポートに転送します。
* `--name myapp` このコンテナーにクエリのための便利な名前を与えるように Docker に指示します (実行時には、Docker によって割り当てられた、オブジェクトを検索する必要はありません)。
* `my-asp-app` は、Docker で実行する画像です。 これは、 `docker build`プロセスの culmination として作成されたコンテナーイメージです。

Web ブラウザーの web ブラウザーを開き、コンテナー `https://localhost:5000`アプリケーションの表示に移動します。

>![](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>次のステップ

ASP.NET web アプリのコンテナーを正常にコンテナーにしました。 その他のアプリのサンプルと関連付けられた dockerfiles を表示するには、下のボタンをクリックします。

> [!div class="nextstepaction"]
> [その他のコンテナーのサンプルを確認する](../samples.md)
