---
title: "Windows で Docker を構成する"
description: "Windows で Docker を構成する"
keywords: "Docker, コンテナー"
author: neilpeterson
manager: timlt
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
translationtype: Human Translation
ms.sourcegitcommit: ea86e2dd88413569e4329ab27a8b6a4d3a7afca9
ms.openlocfilehash: d2fe856b9d00c5f7ac33d683f1c2204dc06d4a11

---

# Windows 上の Docker エンジン

Docker エンジンとクライアントは Windows に含まれていないため、個別にインポートして構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker エンジンのインストールおよび構成方法について説明します。また、一般的に使用される構成例も紹介します。


## Docker のインストール
Windows コンテナーを使用するには Docker が必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべてのものをインストールする最も簡単な方法は、クイック スタート ガイドで説明しています。 これらのガイドは、すべてのものをセットアップして最初のコンテナーを実行するのに役立ちます。 

* [Windows Server 2016 の Windows コンテナー](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_server)
* [Windows 10 の Windows コンテナー](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_10)


### 手動インストール
Docker エンジンとクライアントの開発中バージョンを代わりに使用する場合は、以下の手順を使用してください。 この手順では、Docker エンジンとクライアントの両方をインストールします。 それ以外の場合は、次のセクションにスキップしてください。

> Docker for Windows が既にインストールされている場合は、ここで説明する手動インストールの手順を実行する前に必ず削除してください。 

Docker エンジンをダウンロードする

最新バージョンは https://master.dockerproject.org にあります。 このサンプルでは、v1.13-development 分岐にある最新バージョンを使用します。 

```none
Invoke-WebRequest "https://download.docker.com/components/engine/windows-server/cs-1.12/docker.zip" -OutFile "$env:TEMP\docker.zip" -UseBasicParsing
```

zip アーカイブをプログラム ファイルに展開します。

```
Expand-Archive -Path "$env:TEMP\docker.zip" -DestinationPath $env:ProgramFiles
```

Docker ディレクトリをシステム パスに追加します。 完了したら、変更されたパスが認識されるように、PowerShell セッションを再起動します。

```none
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Program Files\Docker", [EnvironmentVariableTarget]::Machine)
```

Windows サービスとして Docker をインストールするには、以下を実行します。

```none
dockerd --register-service
```

インストールされたら、サービスを開始することができます。

```none
Start-Service Docker
```

Docker を使用するには、先にコンテナー イメージをインストールする必要があります。 詳細については、「[コンテナー イメージ](../management/manage_images.md)」を参照してください。

## 構成ファイルを使用して Docker を構成する

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'c:\ProgramData\docker\config\daemon.json' にあります。 このファイルがまだ存在していない場合は、作成できます。

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

既定の NAT ネットワークが作成されないように Docker エンジンを構成するには、次のコマンドを使用します。 詳細については、「[コンテナーのネットワーク](../management/container_networking.md)」を参照してください。

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

```none
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

変数が設定されたら、Docker サービスを再起動します。

```none
restart-service docker
```

詳細については、[Docker.com の Windows 構成ファイルのページ](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。




<!--HONumber=Oct16_HO3-->


