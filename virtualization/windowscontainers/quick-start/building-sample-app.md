---
title: コンテナー化 a .NET Core アプリ
description: コンテナーを使用してサンプル .NET core アプリを構築する方法について説明します
keywords: Docker, コンテナー
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: 1ca9d7562a877c47f227f1a8e6583cb024909749
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/04/2019
ms.locfileid: "74910142"
---
# <a name="containerize-a-net-core-app"></a>コンテナー化 a .NET Core アプリ

このトピックでは、Windows コンテナーとしてデプロイするために既存のサンプル .NET アプリをパッケージ化する方法について説明し[ます。「はじめ](set-up-environment.md)に」の説明に従って環境を設定した後、「[最初の Windows コンテナーを実行](run-your-first-container.md)する」の説明に従って最初のコンテナーを実行します。

また、コンピューターに Git ソース管理システムがインストールされている必要があります。 インストールするには、 [Git](https://git-scm.com/download)にアクセスしてください。

## <a name="clone-the-sample-code-from-github"></a>GitHub からサンプルコードを複製する

すべてのコンテナーサンプルソースコードは、`windows-container-samples`という名前のフォルダーにある[仮想化ドキュメント](https://github.com/MicrosoftDocs/Virtualization-Documentation)git リポジトリ (リポジトリとして知られています) の下に保持されます。

1. PowerShell セッションを開き、このリポジトリを格納するフォルダーに移動します。 (他のコマンドプロンプトウィンドウの種類も同様に機能しますが、コマンド例では PowerShell を使用しています)。
2. 現在の作業ディレクトリにリポジトリを複製します。

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 次のコマンドを使用して、`Virtualization-Documentation\windows-container-samples\asp-net-getting-started` の下にあるサンプルディレクトリに移動し、Dockerfile を作成します。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)はメイクファイルに似ています。これは、コンテナーエンジンにコンテナーイメージを構築する方法を指示する命令のリストです。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile を作成する

先ほど作成した Dockerfile を任意のテキストエディターで開き、次の内容を追加します。

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

1行ずつ分割して、それぞれの手順について説明します。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:2.1 AS build-env
WORKDIR /app
```

最初の一連の行では、コンテナー構築の基盤として使う基本イメージを宣言します。 ローカル システムにこのイメージがまだ存在しない場合、docker が自動的に取得を試みます。 `mcr.microsoft.com/dotnet/core/sdk:2.1` には .NET core 2.1 SDK がインストールされています。そのため、バージョン2.1 をターゲットとする ASP .NET core プロジェクトをビルドする作業は、ここで行います。 次の手順では、コンテナー内の作業ディレクトリを `/app`に変更します。このため、この後に続くすべてのコマンドがこのコンテキストで実行されます。

```Dockerfile
COPY *.csproj ./
RUN dotnet restore
```

次に、これらの手順で .csproj ファイルを `build-env` コンテナーの `/app` ディレクトリにコピーします。 このファイルをコピーすると、.NET はそれを読み取ってから、プロジェクトに必要なすべての依存関係とツールを取得します。

```Dockerfile
COPY . ./
RUN dotnet publish -c Release -o out
```

.NET がすべての依存関係を `build-env` コンテナーにプルすると、次の命令によってすべてのプロジェクトソースファイルがコンテナーにコピーされます。 次に、リリース構成を使用してアプリケーションを発行し、で出力パスを指定するよう .NET に指示します。

コンパイルは成功します。 次に、最終イメージをビルドする必要があります。 

> [!TIP]
> このクイックスタートでは、ソースから .NET core プロジェクトをビルドします。 コンテナーイメージを作成するときは、実稼働ペイロードとその依存関係_のみ_をコンテナーイメージに含めることをお勧めします。 .Net core ランタイムのみが必要であるため、.NET core SDK を最終イメージに含める必要はありません。そのため、dockerfile は、アプリをビルドするために `build-env` という名前の SDK と共にパッケージ化された一時コンテナーを使用するように記述されています。

```Dockerfile
FROM mcr.microsoft.com/dotnet/core/aspnet:2.1
WORKDIR /app
COPY --from=build-env /app/out .
ENTRYPOINT ["dotnet", "asp-net-getting-started.dll"]
```

このアプリケーションは ASP.NET であるため、このランタイムに含まれるイメージを指定します。 次に、一時コンテナーの出力ディレクトリから、すべてのファイルを最終コンテナーにコピーします。 コンテナーの開始時に、新しいアプリを entrypoint として実行するようにコンテナーを構成します。

_複数段階のビルド_を実行するために dockerfile を作成しました。 Dockerfile を実行すると、一時的なコンテナーである `build-env`を .NET core 2.1 SDK と共に使用してサンプルアプリをビルドし、最終的なコンテナーのサイズを最小限に抑えるために、出力されたバイナリを .NET core 2.1 runtime のみを含む別のコンテナーにコピーします。

## <a name="build-and-run-the-app"></a>アプリをビルドして実行する

Dockerfile が記述されたので、Dockerfile で Docker をポイントし、イメージをビルドして実行するように指示できます。

1. コマンドプロンプトウィンドウで、dockerfile が置かれているディレクトリに移動し、 [docker build](https://docs.docker.com/engine/reference/commandline/build/)コマンドを実行して、dockerfile からコンテナーをビルドします。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 新しく構築されたコンテナーを実行するには、 [docker run](https://docs.docker.com/engine/reference/commandline/run/)コマンドを実行します。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   次のコマンドを詳しく分析してみましょう。

   * `-d` によって、Docker tun によってコンテナー ' デタッチ済み ' が実行されます。これは、コンテナー内のコンソールにコンソールがフックされていないことを意味します。 コンテナーはバックグラウンドで実行されます。 
   * `-p 5000:80` は、ホスト上のポート5000をコンテナーのポート80にマップするよう Docker に指示します。 各コンテナーは、独自の IP アドレスを取得します。 ASP .NET は、既定でポート80でリッスンします。 ポートマッピングを使用すると、マップされたポートでホストの IP アドレスに移動し、Docker はすべてのトラフィックをコンテナー内の宛先ポートに転送します。
   * `--name myapp` は Docker に対して、このコンテナーに対してクエリを実行するのに便利な名前を与えるように Docker に指示します (Docker によって実行時に割り当てられたコンテキスト ID を検索する必要はありません)。
   * `my-asp-app` は、Docker で実行するイメージです。 これは、`docker build` プロセスの概念として生成されたコンテナーイメージです。

3. Web ブラウザーの web ブラウザーを開き、`http://localhost:5000` に移動して、次のスクリーンショットに示すように、コンテナー化されたアプリケーションを表示します。

   >![ASP.NET Core web ページ、コンテナー内の localhost から実行する](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>次のステップ

1. 次の手順では、Azure Container Registry を使用して、コンテナー化された ASP.NET web アプリをプライベートレジストリに発行します。 これにより、組織に展開することができます。

   > [!div class="nextstepaction"]
   > [プライベートコンテナーレジストリを作成する](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   [コンテナーイメージをレジストリにプッシュ](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)するセクションが表示されたら、コンテナーレジストリ (`my-asp-app`) と共にパッケージ化した ASP.NET アプリの名前 (例: `contoso-container-registry`) を指定します。

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   アプリのサンプルとそれに関連付けられている dockerfiles の詳細については、[追加のコンテナーのサンプル](../samples.md)を参照してください。

2. アプリをコンテナーレジストリに発行したら、次の手順として、Azure Kubernetes Service で作成した Kubernetes クラスターにアプリをデプロイします。

   > [!div class="nextstepaction"]
   > [Kubernetes クラスターの作成](https://docs.microsoft.com/azure/aks/windows-container-cli)
