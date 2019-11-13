---
title: .NET コアアプリの Containerize
description: コンテナーを含む .NET core アプリのサンプルをビルドする方法について学習する
keywords: Docker, コンテナー
author: cwilhit
ms.author: crwilhit
ms.date: 11/12/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.openlocfilehash: fab0dc46ddcc8c82a010d408032e5f3c4cea8d69
ms.sourcegitcommit: e61db4d98d9476a622e6cc8877650d9e7a6b4dd9
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/13/2019
ms.locfileid: "10288070"
---
# <a name="containerize-a-net-core-app"></a>.NET コアアプリの Containerize

このトピックでは、「[はじめに: コンテナー用に windows を準備](set-up-environment.md)する」で説明されているように、windows コンテナーとして展開するために既存のサンプル .net アプリをパッケージ化する方法について説明します。「[最初の Windows コンテナーを実行](run-your-first-container.md)する」で説明するように、最初のコンテナーを実行します。

また、コンピューターに Git ソース管理システムがインストールされている必要があります。 インストールするには、 [Git](https://git-scm.com/download)にアクセスしてください。

## <a name="clone-the-sample-code-from-github"></a>GitHub からサンプルコードの複製を作成する

すべてのコンテナーサンプルのソースコードは、という名前`windows-container-samples`のフォルダーにある[仮想化ドキュメント](https://github.com/MicrosoftDocs/Virtualization-Documentation)の git リポジトリ (リポジトリとも呼ばれます) の下に保持されます。

1. PowerShell セッションを開き、このリポジトリを格納するフォルダーにディレクトリを変更します。 (その他のコマンドプロンプトウィンドウでも動作しますが、この例では PowerShell を使用します)。
2. リポジトリを現在の作業ディレクトリに複製します。

   ```PowerShell
   git clone https://github.com/MicrosoftDocs/Virtualization-Documentation.git
   ```

3. 次のコマンドを使用して`Virtualization-Documentation\windows-container-samples\asp-net-getting-started` 、以下のサンプルディレクトリに移動し、Dockerfile を作成します。

   [Dockerfile](https://docs.docker.com/engine/reference/builder/)は、メイクファイルと似ています。これは、コンテナーイメージの作成方法をコンテナーエンジンに指示する命令の一覧です。

   ```Powershell
   # Navigate into the sample directory
   Set-Location -Path Virtualization-Documentation\windows-container-samples\asp-net-getting-started

   # Create the Dockerfile for our project
   New-Item -Name Dockerfile -ItemType file
   ```

## <a name="write-the-dockerfile"></a>Dockerfile を作成する

任意のテキストエディターを使用して作成した Dockerfile を開き、次の内容を追加します。

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

_複数ステージのビルド_を実行するために、dockerfile を作成しました。 Dockerfile が実行されると、一時的なコンテナーを`build-env`使って .net CORE 2.1 SDK を使ってサンプルアプリを作成し、次に、出力されたバイナリを .net core 2.1 ランタイムのみを含む別のコンテナーにコピーして、最終的なコンテナーのサイズを最小化します。

## <a name="build-and-run-the-app"></a>アプリをビルドして実行する

Dockerfile が記述されていると、Dockerfile で Docker を参照し、イメージを作成して実行するように指示することができます。

1. コマンドプロンプトウィンドウで、dockerfile が格納されているディレクトリに移動し、次に、 [docker build](https://docs.docker.com/engine/reference/commandline/build/)コマンドを実行して dockerfile からコンテナーを構築します。

   ```Powershell
   docker build -t my-asp-app .
   ```

2. 新しく構築されたコンテナーを実行するには、docker の [[実行](https://docs.docker.com/engine/reference/commandline/run/)] コマンドを実行します。

   ```Powershell
   docker run -d -p 5000:80 --name myapp my-asp-app
   ```

   このコマンドを dissect しましょう。

   * `-d` Docker tun はコンテナー ' デタッチ ' を実行します。これは、コンテナー内のコンソールにコンソールがフックされないことを意味します。 コンテナーはバックグラウンドで実行されます。 
   * `-p 5000:80` ホスト上のポート5000をコンテナーのポート80にマップするように Docker に指示します。 各コンテナーは、独自の IP アドレスを取得します。 ASP .NET は、既定でポート80でリッスンします。 ポートマッピングを使用すると、マップされたポートでホストの IP アドレスにアクセスでき、Docker はすべてのトラフィックをコンテナー内の宛先ポートに転送します。
   * `--name myapp` このコンテナーにクエリのための便利な名前を与えるように Docker に指示します (実行時には、Docker によって割り当てられた、オブジェクトを検索する必要はありません)。
   * `my-asp-app` は、Docker で実行する画像です。 これは、 `docker build`プロセスの culmination として作成されたコンテナーイメージです。

3. Web ブラウザーの web ブラウザーを開いて、 `http://localhost:5000`次のスクリーンショットに示すように、コンテナーアプリケーションを表示するには、次の操作を行います。

   >![コンテナー内の localhost から実行されている ASP.NET コア web ページ](media/SampleAppScreenshot.png)

## <a name="next-steps"></a>次のステップ

1. 次の手順では、Azure Container レジストリを使って、コンテナー ASP.NET web アプリをプライベートレジストリに公開します。 これにより、組織に展開することができます。

   > [!div class="nextstepaction"]
   > [プライベートコンテナーレジストリを作成する](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell)

   [コンテナーの画像をレジストリにプッシュ](https://docs.microsoft.com/azure/container-registry/container-registry-get-started-powershell#push-image-to-registry)するセクションに移動したら、パッケージ化した ASP.NET アプリの名前 (`my-asp-app`) とコンテナーのレジストリを指定します (例: `contoso-container-registry`)。

   ```PowerShell
   docker tag my-asp-app contoso-container-registry.azurecr.io/my-asp-app:v1
   ```

   その他のアプリのサンプルと関連する dockerfiles については、「[追加のコンテナーのサンプル](../samples.md)」をご覧ください。

2. アプリをコンテナーレジストリに公開したら、次の手順として、Azure Kubernetes Service で作成した Kubernetes クラスターにアプリを展開します。

   > [!div class="nextstepaction"]
   > [Kubernetes クラスターを作成する](https://docs.microsoft.com/azure/aks/windows-container-cli)
