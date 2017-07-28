---
title: "Windows で Docker を構成する"
description: "Windows で Docker を構成する"
keywords: "Docker, コンテナー"
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: f266404f12e47c8605436af44e636c54ec6ef8e5
ms.sourcegitcommit: 65de5708bec89f01ef7b7d2df2a87656b53c3145
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 07/21/2017
---
# Windows 上の Docker エンジン

Docker エンジンとクライアントは Windows に含まれていないため、個別にインポートして構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker エンジンのインストールおよび構成方法について説明します。また、一般的に使用される構成例も紹介します。


## Docker のインストール
Windows コンテナーを使用するには Docker が必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべてのものをインストールする最も簡単な方法は、クイック スタート ガイドで説明しています。 これらのガイドは、すべてのものをセットアップして最初のコンテナーを実行するのに役立ちます。 

* [Windows Server 2016 の Windows コンテナー](../quick-start/quick-start-windows-server.md)
* [Windows 10 の Windows コンテナー](../quick-start/quick-start-windows-10.md)


### 手動インストール
Docker エンジンとクライアントについて開発中バージョンを代わりに使用する場合は、以下の手順を使用してください。 この手順では、Docker エンジンとクライアントの両方をインストールします。 新機能をテストする開発者、または Windows Insider ビルドを使用している開発者である場合は、開発中バージョンの Docker の使用が必要になることがあります。 それ以外の場合は、上記の「Docker のインストール」セクションの手順に従って、最新の製品版を入手してください。

> Docker for Windows が既にインストールされている場合は、ここで説明する手動インストールの手順を実行する前に必ずアンインストールしてください。 

Docker エンジンをダウンロードする

最新バージョンは https://master.dockerproject.org にあります。 このサンプルでは、マスター ブランチにある最新バージョンを使用します。 

```powershell
$version = (Invoke-WebRequest -UseBasicParsing https://raw.githubusercontent.com/docker/docker/master/VERSION).Content.Trim()
Invoke-WebRequest "https://master.dockerproject.org/windows/x86_64/docker-$($version).zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

zip アーカイブを Program Files に展開します。

```powershell
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリをシステム パスに追加します。 完了したら、変更されたパスが認識されるように、PowerShell セッションを再起動します。

```powershell
# Add path to this PowerShell session immediately
$env:path += ";$env:ProgramFiles\Docker"

# For persistent use after a reboot
$existingMachinePath = [Environment]::GetEnvironmentVariable("Path",[System.EnvironmentVariableTarget]::Machine)
[Environment]::SetEnvironmentVariable("Path", $existingMachinePath + ";$env:ProgramFiles\Docker", [EnvironmentVariableTarget]::Machine)
```

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
dockerd --register-service
```

インストールされたら、サービスを開始することができます。

```powershell
Start-Service Docker
```

Docker を使用するには、先にコンテナー イメージをインストールする必要があります。 詳しくは、[イメージを使用するためのクイック スタート ガイド](../quick-start/quick-start-images.md)をご覧ください。

## 構成ファイルを使用して Docker を構成する

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 このファイルがまだ存在していない場合は、作成できます。

注: Windows 上の Docker では、使用できない Docker 構成オプションもあります。 たとえば、次のオプションは使用できません。 Docker エンジン構成の詳細なドキュメントについては、[Docker デーモン構成ファイルのページ](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

```none
{
    "authorization-plugins": [],
    "dns": [],
    "dns-opts": [],
    "dns-search": [],
    "exec-opts": [],
    "storage-driver": "",
    "storage-opts": [],
    "labels": [],
    "log-driver": "", 
    "mtu": 0,
    "pidfile": "",
    "graph": "",
    "cluster-store": "",
    "cluster-advertise": "",
    "debug": true,
    "hosts": [],
    "log-level": "",
    "tlsverify": true,
    "tlscacert": "",
    "tlscert": "",
    "tlskey": "",
    "group": "",
    "default-ulimits": {},
    "bridge": "",
    "fixed-cidr": "",
    "raw-logs": false,
    "registry-mirrors": [],
    "insecure-registries": [],
    "disable-legacy-registry": false
}
```

目的の構成の変更のみを構成ファイルに追加する必要があります。 たとえば、このサンプルでは、ポート 2375 で受信接続を受け入れるように Docker エンジンを構成しています。 その他のすべての構成オプションには、既定値が使用されます。

```none
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、このサンプルでは、代替パスにイメージとコンテナーを保持するように Docker デーモンが構成されます。 指定しない場合、既定は c:\programdata\docker です。

```none
{    
    "graph": "d:\\docker"
}
```

同様に、このサンプルでは、セキュリティで保護された接続をポート 2376 でのみ受け入れるように Docker デーモンを構成しています。

```none
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## Docker サービスで Docker を構成する

`sc config` を使用して Docker サービスを変更することで、Docker エンジンを構成することもできます。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注: daemon.json ファイルに `"hosts": ["tcp://0.0.0.0:2375"]` というエントリが既に含まれている場合は、このコマンドを実行する必要はありません。

## 一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### 既定のネットワークの作成 

既定の NAT ネットワークが作成されないように Docker エンジンを構成するには、次のコマンドを使用します。 詳細については、「[コンテナーのネットワーク](../manage-containers/container-networking.md)」を参照してください。

```none
{
    "bridge" : "none"
}
```

### Docker セキュリティ グループの設定

Docker ホストにログインし、Docker コマンドをローカルで実行すると、名前付きパイプ経由でコマンドが実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```none
{
    "group" : "docker"
}
```

## プロキシの構成

`docker search` と `docker pull` のプロキシ情報を設定するには、`HTTP_PROXY` または `HTTPS_PROXY` という名前と、プロキシ情報の値を使用して Windows 環境変数を作成します。 そのためには、PowerShell で次のようなコマンドを実行します。

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

変数が設定されたら、Docker サービスを再起動します。

```powershell
Restart-Service docker
```

詳細については、[Docker.com の Windows 構成ファイルのページ](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

