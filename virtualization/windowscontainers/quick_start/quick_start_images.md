---
title: "コンテナー展開のクイック スタート - イメージ"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: enderb-ms
ms.date: 09/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 479e05b1-2642-47c7-9db4-d2a23592d29f
translationtype: Human Translation
ms.sourcegitcommit: ffdf89b0ae346197b9ae631ee5260e0565261c55
ms.openlocfilehash: f856499eea09efd4aa26cba6c7e437d61eb8b80b

---

# Windows Server のコンテナー イメージ

前の Windows Server クイック スタートでは、以前に作成した .NET Core サンプルから Windows コンテナーを作成しました。 この演習では、カスタム コンテナー イメージを手動で作成する方法、Dockerfile を使用してコンテナー イメージの作成を自動化する方法、Docker Hub パブリック レジストリにコンテナー イメージを保存する方法について詳しく説明します。

このクイック スタートは、Windows Server 2016 上の Windows Server コンテナー固有の内容です。Windows Server Core コンテナー基本イメージを使用します。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

- Windows Server 2016 を実行している 1 台のコンピューター システム (物理または仮想)。
- Windows コンテナー機能と Docker でこのシステムを構成します。 これらの手順のチュートリアルについては、「[Windows Containers on Windows Server](./quick_start_windows_server.md)」 (Windows Server の Windows コンテナー) を参照してください。
- Docker ID。コンテナー イメージを Docker Hub にプッシュするために使用されます。 Docker ID がない場合は、[Docker Cloud]( https://cloud.docker.com/) でサインアップしてください。

## 1.コンテナー イメージ - 手動

この演習は Windows コマンド シェル (cmd.exe) から実行すると、最も効果的です。

コンテナー イメージを手動で作成するための最初の手順はコンテナーを展開することです。 この例では、事前に作成した IIS イメージから IIS イメージを展開します。 コンテナーが展開されたら、コンテナー内からシェル セッションで操作します。 対話型セッションが `-it` フラグで開始されます。 Docker Run コマンドの詳細については、Docker.com の「[Docker Run Reference]( https://docs.docker.com/engine/reference/run/)」を参照してください。 

> Windows Server Core 基本イメージのサイズにより、この手順には時間がかかる可能性があります。

```none
docker run -it -p 80:80 microsoft/iis cmd
```

ダウンロードが完了すると、コンテナーが起動し、シェル セッションが開始されます。

次に、コンテナーが変更されます。 次のコマンドを実行して、IIS スプラッシュ画面を削除します。

```none
del C:\inetpub\wwwroot\iisstart.htm
```

次のコマンドを実行して、既定の IIS サイトを新しい静的サイトに置き換えます。

```none
echo "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

別のシステムから、コンテナー ホストの IP アドレスを参照します。 ‘Hello World’ アプリケーションが表示されるはずです。

**注:** Azure で作業している場合、ポート 80 経由のトラフィックを許可するために、ネットワーク セキュリティ グループ ルールを設定する必要があります。 詳細については、「[既存の NSG に規則を作成する]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)」をご覧ください。

![](media/hello.png)

コンテナーに戻り、対話型のコンテナー セッションを終了します。

```none
exit
```

これで、変更されたコンテナーを新しいコンテナー イメージにキャプチャできます。 それには、コンテナー名が必要になります。 これを見つけるには、`docker ps -a` コマンドを使用します。

```none
docker ps -a

CONTAINER ID     IMAGE                             COMMAND   CREATED             STATUS   PORTS   NAMES
489b0b447949     microsoft/iis   "cmd"     About an hour ago   Exited           pedantic_lichterman
```

新しいコンテナー イメージを作成するには、`docker commit` コマンドを使用します。 Docker コミットの形式は “docker commit container-name new-image-name” になります。 注記 – この例のコンテナー名を実際のコンテナー名に置き換えます。

```none
docker commit pedantic_lichterman modified-iis
```

新しいイメージが作成されたことを確認するには、`docker images` コマンドを使用します。  

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
modified-iis        latest              3e4fdb6ed3bc        About a minute ago   10.17 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago          9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago          9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago          9.344 GB
```

これで、このイメージを展開できます。 結果的に生成されるコンテナーには、キャプチャされたすべての変更が含まれます。

## 2.コンテナー イメージ - Dockerfile

前回の演習によって、コンテナーが手動で作成され、変更されて、新しいコンテナー イメージにキャプチャされています。 Docker には、このプロセスを自動化するためのメソッドが含まれており、Dockerfile を使用します。 この演習の結果は前回とほぼ同じになりますが、今回はプロセスが自動化されます。 この演習には Docker ID が必要です。 Docker ID がない場合は、[Docker Cloud]( https://cloud.docker.com/) でサインアップしてください。

コンテナー ホストで、ディレクトリ `c:\build` を作成し、このディレクトリ内に `Dockerfile` という名前のファイルを作成します。 注記 – このファイルにはファイル拡張子を与えません。

```none
powershell new-item c:\build\Dockerfile -Force
```

Dockerfile をメモ帳で開きます。

```none
notepad c:\build\Dockerfile
```

Dockerfile に次のテキストをコピーし、ファイルを保存します。 これらのコマンドは、`microsoft/iis` を基礎として使用し、新しいイメージを作成するように Docker に指示します。 dockerfile は次に、`RUN` の指示に指定されているコマンドを実行します。この場合、index.html ファイルが新しいコンテンツで更新されます。 

Dockerfile の詳細については、「[Dockerfiles on Windows](../docker/manage_windows_dockerfile.md)」 (Windows 上の Dockerfile) を参照してください。

```none
FROM microsoft/iis
RUN echo "Hello World - Dockerfile" > c:\inetpub\wwwroot\index.html
```

`docker build` コマンドはイメージ ビルド プロセスを開始します。 `-t` パラメーターは、新しいイメージに `iis-dockerfile` という名前を付けるようにビルド プロセスに指示します。 **'user' は Docker アカウントのユーザー名で置き換えます**。 Docker にアカウントがない場合は、[Docker Cloud]( https://cloud.docker.com/) でサインアップしてください。

```none
docker build -t <user>/iis-dockerfile c:\Build
```

完了したら、`docker images` コマンドを使用して、イメージが作成されたことを確認できます。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
iis-dockerfile      latest              8d1ab4e7e48e        2 seconds ago       9.483 GB
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

次のコマンドでコンテナーを展開します (ここでも user を Docker ID で置き換えます)。

```none
docker run -d -p 80:80 <user>/iis-dockerfile ping -t localhost
```

コンテナーが作成されたら、コンテナー ホストの IP アドレスを参照します。 hello world アプリケーションが表示されるはずです。

![](media/dockerfile2.png)

コンテナー ホストに戻り、`docker ps` を使用してコンテナーの名前を取得し、`docker rm` を使用してコンテナーを削除します。 注記 – この例のコンテナー名を実際のコンテナー名に置き換えます。

コンテナー名を取得します。

```none
docker ps

CONTAINER ID   IMAGE            COMMAND               CREATED              STATUS              PORTS                NAMES
c1dc6c1387b9   iis-dockerfile   "ping -t localhost"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp   cranky_brown
```

コンテナーを削除します。

```none
docker rm -f <container name>
```

## 3.Docker Push

Docker コンテナー イメージはコンテナー レジストリに保存できます。 イメージをレジストリに保存すると、後で多数のコンテナー ホストから取得して使用できるようになります。 Docker には、[Docker Hub](https://hub.docker.com/) にコンテナー イメージを保存できるパブリック レジストリがあります。

この演習では、カスタムの hello world イメージを Docker Hub の自分のアカウントにプッシュします。

まず、`docker login command` を使用して Docker アカウントにログインします。

```none
docker login

Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.

Username: user
Password: Password

Login Succeeded
```

ログインしたら、コンテナー イメージを Docker Hub にプッシュできます。 この場合は、`docker push` コマンドを使用します。 **'user' は Docker ID で置き換えます**。 

```none
docker push <user>/iis-dockerfile
```

これで、`docker pull` を使用して、コンテナー イメージを Docker Hub から任意の Windows コンテナー ホストにダウンロードできるようになります。 このチュートリアルでは、既存のイメージを削除し、Docker Hub からプルします。 

```none
docker rmi <user>/iis-dockerfile
```

`docker images` を実行すると、イメージが削除されたことが表示されます。

```none
docker images

REPOSITORY                TAG                 IMAGE ID            CREATED             SIZE
modified-iis              latest              51f1fe8470b3        5 minutes ago       7.69 GB
microsoft/iis             latest              e4525dda8206        3 hours ago         7.61 GB
```

最後に、Docker Pull を使用して、イメージをコンテナー ホストにプルすることができます。 'user' は Docker アカウントのユーザー名で置き換えます。 

```none
docker pull <user>/iis-dockerfile
```

## 次の手順

[Windows 10 の Windows コンテナー](./quick_start_windows_10.md)



<!--HONumber=Oct16_HO4-->


