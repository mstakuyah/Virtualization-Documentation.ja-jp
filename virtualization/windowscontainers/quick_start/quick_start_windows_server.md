---
title: "Windows Server の Windows コンテナー"
description: "コンテナー展開のクイック スタート"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: e3b2a4dc-9082-4de3-9c95-5d516c03482b
translationtype: Human Translation
ms.sourcegitcommit: 6c7ce9f1767c6c6391cc6d33a553216bd815ff72
ms.openlocfilehash: 4e7123dcff2564bd264c91f228941e86a0723674

---

# Windows Server の Windows コンテナー

**この記事は暫定的な内容であり、変更される可能性があります。**

この演習では、Windows Server の Windows コンテナー機能の基本的な展開と使用について段階的に確認します。 完了後、コンテナー ロールがインストールされ、シンプルな Windows Server コンテナーが展開されます。 このクイック スタートを始める前に、コンテナーの基本的な概念と用語を理解しておいてください。 情報は[クイック スタートの概要](./quick_start.md)にあります。

このクイック スタートは Windows Server 2016 の Windows Server コンテナーのみに適用されます。 このページの左側の目次に追加のクイック スタート文書があります。

**前提条件:**

[Windows Server 2016 Technical Preview 5](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-technical-preview) を実行している 1 台のコンピューター システム (物理または仮想)。

完全に構成された Windows Server イメージは Azure で利用できます。 このイメージを利用するには、下のボタンをクリックして仮想マシンをデプロイします。 デプロイには 10 分ほどかかります。 完了したら、Azure 仮想マシンにログインし、このチュートリアルの手順 4 に進みます。 

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FMicrosoft%2FVirtualization-Documentation%2Fmaster%2Fwindows-server-container-tools%2Fcontainers-azure-template%2Fazuredeploy.json" target="_blank">
    <img src="http://azuredeploy.net/deploybutton.png"/>
</a>

## 1.コンテナー機能のインストール

コンテナー機能は、Windows コンテナーを使用する前に有効にする必要があります。 そのためには、管理者特権の PowerShell セッションで次のコマンドを実行します。

```none
Install-WindowsFeature containers
```

機能のインストールが完了したら、コンピューターを再起動します。

```none
Restart-Computer -Force
```

## 2.Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。

Docker エンジンとクライアントを 1 つの zip アーカイブとしてダウンロードします。

```none
Invoke-WebRequest "https://get.docker.com/builds/Windows/x86_64/docker-1.12.0.zip" -OutFile "$env:TEMP\docker-1.12.0.zip" -UseBasicParsing
```

この zip アーカイブを展開して Program Files に出力します。アーカイブの内容は既に docker ディレクトリに入っています。

```none
Expand-Archive -Path "$env:TEMP\docker-1.12.0.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリをシステム パスに追加します。 完了したら、変更されたパスが認識されるように、PowerShell セッションを再起動します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
& $env:ProgramFiles\docker\dockerd.exe --register-service
```

インストールされたら、サービスを開始することができます。

```none
Start-Service docker
```

## 3.コンテナーの基本イメージのインストール

Windows コンテナーは、テンプレートまたはイメージから展開されます。 コンテナーを展開する前に、基本 OS イメージをダウンロードする必要があります。 次のコマンドは、Windows Server Core のベース イメージをダウンロードします。

まず、コンテナー イメージ パッケージ プロバイダーをインストールします。

```none
Install-PackageProvider ContainerImage -Force
```

次に、Windows Server Core のイメージをインストールします。 このプロセスには時間がかかる場合があるため、しばらく待ち、ダウンロードが完了したら作業に戻ってください。

```none
Install-ContainerImage -Name WindowsServerCore    
```

基本イメージがインストールされたら、Docker サービスを再起動する必要があります。

```none
Restart-Service docker
```

この段階では、`docker images` を実行すると、インストールされたイメージの一覧が返されます。この場合、Windows Server Core のイメージです。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
windowsservercore   10.0.14300.1000     dbfee88ee9fd        7 weeks ago         9.344 GB
```

続行前に、このイメージに '最新' バージョンであることを示すタグを付ける必要があります。 これを行うために、次のコマンドを実行します。

```none
docker tag windowsservercore:10.0.14300.1000 windowsservercore:latest
```

Windows コンテナー イメージの詳細については、[コンテナー イメージの管理](../management/manage_images.md)に関するページを参照してください。

## 4.最初のコンテナーの展開

この演習では、事前作成された IIS イメージを Docker Hub レジストリからダウンロードし、IIS を実行するシンプルなコンテナーを展開します。  

Windows コンテナー イメージの Docker Hub を検索するには、`docker search Microsoft` を実行します。  

```none
docker search microsoft

NAME                                         DESCRIPTION                                     
microsoft/sample-django:windowsservercore    Django installed in a Windows Server Core ...   
microsoft/dotnet35:windowsservercore         .NET 3.5 Runtime installed in a Windows Se...   
microsoft/sample-golang:windowsservercore    Go Programming Language installed in a Win...   
microsoft/sample-httpd:windowsservercore     Apache httpd installed in a Windows Server...   
microsoft/iis:windowsservercore              Internet Information Services (IIS) instal...   
microsoft/sample-mongodb:windowsservercore   MongoDB installed in a Windows Server Core...   
microsoft/sample-mysql:windowsservercore     MySQL installed in a Windows Server Core b...   
microsoft/sample-nginx:windowsservercore     Nginx installed in a Windows Server Core b...  
microsoft/sample-python:windowsservercore    Python installed in a Windows Server Core ...   
microsoft/sample-rails:windowsservercore     Ruby on Rails installed in a Windows Serve...  
microsoft/sample-redis:windowsservercore     Redis installed in a Windows Server Core b...   
microsoft/sample-ruby:windowsservercore      Ruby installed in a Windows Server Core ba...   
microsoft/sample-sqlite:windowsservercore    SQLite installed in a Windows Server Core ...  
```

`docker pull` を利用し、IIS イメージをダウンロードします。  

```none
docker pull microsoft/iis:windowsservercore
```

イメージ ダウンロードは `docker images` コマンドで確認できます。 ここでは、ベース イメージ (windowsservercore) と IIS イメージの両方が表示されることに注意してください。

```none
docker images

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
microsoft/iis       windowsservercore   c26f4ceb81db        2 weeks ago         9.48 GB
windowsservercore   10.0.14300.1000     dbfee88ee9fd        8 weeks ago         9.344 GB
windowsservercore   latest              dbfee88ee9fd        8 weeks ago         9.344 GB
```

`docker run` を使用し、IIS コンテナーを展開します。

```none
docker run -d -p 80:80 microsoft/iis:windowsservercore ping -t localhost
```

このコマンドは IIS イメージをバックグラウンド サービス (-d) として実行し、コンテナー ホストのポート 80 がコンテナーのポート 80 にマッピングされるようにネットワーキングを構成します。
Docker Run コマンドの詳細については、Docker.com の「[Docker Run リファレンス]( https://docs.docker.com/engine/reference/run/)」をご覧ください。


実行中のコンテナーは `docker ps` コマンドで確認できます。 コンテナーの名前をメモしてください。後の手順で使用されます。

```none
docker ps

CONTAINER ID    IMAGE                             COMMAND               CREATED              STATUS   PORTS                NAMES
9cad3ea5b7bc    microsoft/iis:windowsservercore   "ping -t localhost"   About a minute ago   Up       0.0.0.0:80->80/tcp   grave_jang
```

別のコンピューターから、Web ブラウザーを開き、コンテナー ホストの IP アドレスを入力してください。 すべてが正しく構成されていれば、IIS のスプラッシュ画面が表示されます。 これは、Windows コンテナーでホストされている IIS インスタンスからサービスが提供されている状態です。

**注:** Azure を使用している場合、仮想マシンの外部 IP アドレスと、構成されているネットワーク セキュリティが必要になります。 詳細については、「[既存の NSG に規則を作成する]( https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-create-nsg-arm-pportal/#create-rules-in-an-existing-nsg)」をご覧ください。

![](media/iis1.png)

コンテナー ホストに戻り、`docker rm` コマンドを使用してコンテナーを削除します。 注記 – この例のコンテナー名を実際のコンテナー名に置き換えます。

```none
docker rm -f grave_jang
```
## 次の手順

[Windows Server のコンテナー イメージ](./quick_start_images.md)

[Windows 10 の Windows コンテナー](./quick_start_windows_10.md)



<!--HONumber=Aug16_HO1-->


