---
title: Windows コンテナー イメージ
description: Windows コンテナーでコンテナー イメージを作成および管理します。
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
---

# Windows コンテナー イメージ

**この記事は暫定的な内容であり、変更される可能性があります。** 

コンテナー イメージは、コンテナーを展開するために使用します。 これらのイメージには、オペレーティング システム、アプリケーション、およびすべてのアプリケーションの依存関係を含めることができます。 たとえば、Nano Server、IIS、および IIS で実行するアプリケーションによって事前に構成されたコンテナー イメージを開発できます。 このコンテナー イメージは、後で使用するためにコンテナー レジストリに保存したり、任意の Windows コンテナー ホスト (オンプレミス、クラウド、またはコンテナー サービス) に展開したり、新しいコンテナー イメージのベースとして使用したりすることができます。

2 つの種類のコンテナー イメージがあります。

**ベース OS イメージ** - Microsoft によって提供されます。コア OS コンポーネントが含まれます。 

**コンテナー イメージ** - ベース OS イメージから派生したカスタム コンテナー イメージです。

## ベース OS イメージ

### イメージのインストール

コンテナー OS イメージは、ContainerImage PowerShell モジュールを使用して検索およびインストールできます。 このモジュールを使用するには、先にインストールする必要があります。 モジュールをインストールするために、次のコマンドを使用できます。 Container Image OneGet PowerShell モジュールの使用方法については、[コンテナー イメージ プロバイダー](https://github.com/PowerShell/ContainerProvider)を参照してください。 

```none
Install-PackageProvider ContainerImage -Force
```

インストールすると、`Find-ContainerImage` を使用してベース OS イメージの一覧を返せるようになります。

```none
Find-ContainerImage

Name                 Version          Source           Summary
----                 -------          ------           -------
NanoServer           10.0.14300.1010  ContainerImag... Container OS Image of Windows Server 2016 Technical...
WindowsServerCore    10.0.14300.1000  ContainerImag... Container OS Image of Windows Server 2016 Technical...
```

Nano Server ベースの OS イメージをダウンロードしてインストールするには、次を実行します。 `-version` パラメーターはオプションです。 ベース OS イメージのバージョンを指定しない場合は、最新のバージョンがインストールされます。

```none
Install-ContainerImage -Name NanoServer -Version 10.0.14300.1010
```

このコマンドは、Windows Server Core ベース OS イメージも同様にダウンロードしてインストールします。 `-version` パラメーターはオプションです。 ベース OS イメージのバージョンを指定しない場合は、最新のバージョンがインストールされます。

```none
Install-ContainerImage -Name WindowsServerCore -Version 10.0.14300.1000
```

`docker images` コマンドを使用して、イメージがインストールされていることを確認します。 

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     40356b90dc80        2 weeks ago         793.3 MB
windowsservercore   10.0.14304.1000     7837d9445187        2 weeks ago         9.176 GB
```  

インストールした後に、イメージに ’latest’ タグを付けることもできます。 これらの手順の詳細については、以下のタグに関するセクションを参照してください。

> ベース OS イメージがダウンロードされているのに `docker images` の実行時に表示されない場合は、[サービス] コントロール パネル アプレットを使用するか、コマンド 'sc stop docker' および 'sc start docker' を順番に使用して、Docker サービスを再起動します

### タグ イメージ

名前でコンテナー イメージを参照すると、Docker エンジンは最新バージョンのイメージを検索します。 最新のバージョンを特定できない場合は、次のエラーがスローされます。

```none
docker run -it windowsservercore cmd

Unable to find image 'windowsservercore:latest' locally
Pulling repository docker.io/library/windowsservercore
C:\Windows\system32\docker.exe: Error: image library/windowsservercore not found.
```

Windows Server Core または Nano Server のベース OS イメージのインストール後、これらのイメージには 'latest' バージョンのタグが付けられる必要があります。 この場合は、`docker tag` コマンドを使用します。 

`docker tag` の詳細については、[docker.com のイメージのタグ付け、プッシュ、およびプルに関するページ](https://docs.docker.com/mac/step_six/)を参照してください。 

```none
docker tag <image id> windowsservercore:latest
```

タグ付けすると、`docker images` の出力により同じイメージの 2 つのバージョンが表示されます。1 つはイメージのバージョンのタグが付けられたもの、もう 1 つは 'latest' のタグが付けられたものです。 これで、イメージを名前で参照できるようになりました。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nanoserver          10.0.14300.1010     df03a4b28c50        2 days ago          783.2 MB
windowsservercore   10.0.14300.1000     290ab6758cec        2 days ago          9.148 GB
windowsservercore   latest              290ab6758cec        2 days ago          9.148 GB
```

### オフライン インストール

インターネットに接続しないでベース OS イメージをインストールすることもできます。 この場合、インターネット接続を使用してコンピューターにイメージをダウンロードし、ターゲット システムにコピーし、`Install-ContainerOSImages` コマンドを使用してイメージをインポートします。

ベース OS イメージをダウンロードする前に、次のコマンドを実行して、コンテナー イメージ プロバイダーで**インターネット接続された**システムを準備します。

```none
Install-PackageProvider ContainerImage -Force
```

PowerShell OneGet パッケージ マネージャーからイメージの一覧が返されます。

```none
Find-ContainerImage
```

出力:

```none
Name                 Version                 Description
----                 -------                 -----------
NanoServer           10.0.14300.1010         Container OS Image of Windows Server 2016 Techn...
WindowsServerCore    10.0.14300.1000         Container OS Image of Windows Server 2016 Techn...
```

イメージをダウンロードするには、`Save-ContainerImage` コマンドを使用します。

```none
Save-ContainerImage -Name NanoServer -Path c:\container-image
```

ダウンロードしたコンテナー イメージは、**オフライン コンテナー ホスト**にコピーし、`Install-ContainerOSImage` コマンドを使用してインストールできるようになります。

```none
Install-ContainerOSImage -WimPath C:\container-image\NanoServer.wim -Force
```

### OS イメージのアンインストール

ベース OS イメージをアンインストールするには、`Uninstall-ContainerOSImage` コマンドを使用します。 次の例では、NanoServer のベース OS イメージをアンインストールします。

```none
Uninstall-ContainerOSImage -FullName CN=Microsoft_NanoServer_10.0.14304.1003
```

## コンテナー イメージ

### イメージの一覧表示

```none
docker images

REPOSITORY             TAG                 IMAGE ID            CREATED              VIRTUAL SIZE
windowsservercoreiis   latest              ca40b33453f8        About a minute ago   44.88 MB
windowsservercore      10.0.14300.1000     6801d964fda5        2 weeks ago          0 B
nanoserver             10.0.14300.1010     8572198a60f1        2 weeks ago          0 B
```

### 新しいイメージの作成

既存のコンテナーから新しいコンテナー イメージを作成できます。 この場合は、`docker commit` コマンドを使用します。 次の例では、‘windowsservercoreiis’ という名前で新しいコンテナー イメージを作成します。

```none
docker commit 475059caef8f windowsservercoreiis
```

### イメージの削除

いずれかのコンテナーに、それが停止状態であっても、イメージへの依存関係がある場合は、コンテナー イメージを削除できません。

docker でイメージを削除する場合、イメージをイメージ名または id で参照できます。

```none
docker rmi windowsservercoreiis
```

### イメージの依存関係

Docker によってイメージの依存関係を表示するには、`docker history` コマンドを使用できます。

```none
docker history windowsservercoreiis

IMAGE               CREATED             CREATED BY          SIZE                COMMENT
2236b49aaaef        3 minutes ago       cmd                 171.2 MB
6801d964fda5        2 weeks ago                             0 B
```

### Docker Hub

Docker Hub レジストリには、事前構築されたイメージが格納され、これをコンテナー ホストにダウンロードできます。 これらのイメージがダウンロードされていると、これらを Windows コンテナー アプリケーションのベースとして使用することができます。

Docker Hub から使用できるイメージの一覧を表示するには、`docker search` コマンドを使用します。 注: Windows Serve Core または Nano Server のベース OS イメージをインストールしてからでないと、これらの依存するコンテナー イメージを Docker Hub からプルできません。

このようなイメージの多くは、Windows Server Core バージョンや Nano Server バージョンです。 特定のバージョンを取得するには、":windowsservercore" または ":nanoserver" というタグを追加します。 "latest" タグが指定されていると、Nano Server バージョンが使用できる場合以外は、既定で Windows Server Core バージョンが返されます。

> "nano-" で始まるイメージは、Nano Server のベース OS イメージに依存しています。

```none
docker search *

NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
microsoft/sample-django  Django installed in a Windows Server Core ...   1                    [OK]
microsoft/dotnet35       .NET 3.5 Runtime installed in a Windows Se...   1         [OK]       [OK]
microsoft/sample-golang  Go Programming Language installed in a Win...   1                    [OK]
microsoft/sample-httpd   Apache httpd installed in a Windows Server...   1                    [OK]
microsoft/iis            Internet Information Services (IIS) instal...   1         [OK]       [OK]
microsoft/sample-mongodb MongoDB installed in a Windows Server Core...   1                    [OK]
microsoft/sample-mysql   MySQL installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-nginx   Nginx installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-node    Node installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-python  Python installed in a Windows Server Core ...   1                    [OK]
microsoft/sample-rails   Ruby on Rails installed in a Windows Serve...   1                    [OK]
microsoft/sample-redis   Redis installed in a Windows Server Core b...   1                    [OK]
microsoft/sample-ruby    Ruby installed in a Windows Server Core ba...   1                    [OK]
microsoft/sample-sqlite  SQLite installed in a Windows Server Core ...   1                    [OK]
```

Docker Hub からイメージをダウンロードするには、`docker pull` を使用します。

```none
docker pull microsoft/aspnet

Using default tag: latest
latest: Pulling from microsoft/aspnet
f9e8a4cc8f6c: Pull complete

b71a5b8be5a2: Download complete
```

`docker images` を実行すると、イメージが表示されるようになります。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
microsoft/aspnet    latest              b3842ee505e5        5 hours ago         101.7 MB
windowsservercore   10.0.14300.1000     6801d964fda5        2 weeks ago         0 B
windowsservercore   latest              6801d964fda5        2 weeks ago         0 B
```



<!--HONumber=May16_HO4-->


