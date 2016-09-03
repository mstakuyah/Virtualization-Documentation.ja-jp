---
title: "Windows コンテナー イメージ"
description: "Windows コンテナーでコンテナー イメージを作成および管理します。"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 08/22/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: d8163185-9860-4ee4-9e96-17b40fb508bc
translationtype: Human Translation
ms.sourcegitcommit: 7b5cf299109a967b7e6aac839476d95c625479cd
ms.openlocfilehash: 8b9ec6370d1f9f9187fbb6d74168e9e88391b657

---

# Windows コンテナー イメージ

**この記事は暫定的な内容であり、変更される可能性があります。** 

>Windows コンテナーは、Docker を使用して管理されます。 Windows コンテナー ドキュメントは、[docker.com](https://www.docker.com/) のドキュメントを補完するものです。

コンテナー イメージは、コンテナーを展開するために使用します。 これらのイメージには、アプリケーションとすべてのアプリケーションの依存関係を含めることができます。 たとえば、Nano Server、IIS、および IIS で実行するアプリケーションによって事前に構成されたコンテナー イメージを開発できます。 このコンテナー イメージは、後で使用するためにコンテナー レジストリに保存したり、任意の Windows コンテナー ホスト (オンプレミス、クラウド、またはコンテナー サービス) に展開したり、新しいコンテナー イメージのベースとして使用したりすることができます。

### イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 基本イメージは、基になるオペレーティング システムとして Windows Server Core と Nano Server の両方で使用できます。 サポートされる構成については、[Windows コンテナーのシステム要件](../deployment/system_requirements.md)に関する記事を参照してください。

Windows Server Core 基本イメージをインストールするには、次のコマンドを実行します。

```none
docker pull microsoft/windowsservercore
```

Nano Server 基本イメージをインストールするには、次のコマンドを実行します。

```none
docker pull microsoft/nanoserver
```

### イメージの一覧表示

```none
docker images

REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
microsoft/windowsservercore   latest              02cb7f65d61b        9 weeks ago         7.764 GB
microsoft/nanoserver          latest              3a703c6e97a2        9 weeks ago         969.8 MB
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

### Docker Pull

Docker Hub からイメージをダウンロードするには、`docker pull` を使用します。 詳細については、[Docker.com の Docker Pull のページ](https://docs.docker.com/engine/reference/commandline/pull/)を参照してください。

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

> Docker Pull が失敗した場合は、最新の累積的な更新プログラムがコンテナー ホストに適用されていることを確認します。 TP5 更新プログラムは [KB3157663]( https://support.microsoft.com/en-us/kb/3157663) にあります。

### Docker Push

コンテナー イメージを Docker Hub または Docker Trusted Registry にアップロードすることもできます。 いったんアップロードされたイメージは、ダウンロードすることができ、さまざまな Windows コンテナー環境で再利用できます。

Docker Hub にコンテナー イメージをアップロードするには、まずレジストリにログインします。 詳細については、[Docker.com の Docker Login のページ]( https://docs.docker.com/engine/reference/commandline/login/)を参照してください。

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: username
Password:

Login Succeeded
```

Docker Hub または Docker Trusted Regstry にログインしたら、`docker push` を使用してコンテナー イメージをアップロードします。 コンテナー イメージは名前または ID で参照できます。 詳細については、[Docker.com の Docker Push のページ]( https://docs.docker.com/engine/reference/commandline/push/)を参照してください。

```none
docker push username/containername

The push refers to a repository [docker.io/username/containername]
b567cea5d325: Pushed
00f57025c723: Pushed
2e05e94480e9: Pushed
63f3aa135163: Pushed
469f4bf35316: Pushed
2946c9dcfc7d: Pushed
7bfd967a5e43: Pushed
f64ea92aaebc: Pushed
4341be770beb: Pushed
fed398573696: Pushed
latest: digest: sha256:ae3a2971628c04d5df32c3bbbfc87c477bb814d5e73e2787900da13228676c4f size: 2410
```

この時点で、コンテナー イメージが使用可能になり、`docker pull` を使用してアクセスできます。






<!--HONumber=Aug16_HO4-->


