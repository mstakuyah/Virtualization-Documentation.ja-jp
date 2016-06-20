---
title: Windows で Docker を構成する
description: Windows で Docker を構成する
keywords: docker, containers
author: neilpeterson
manager: timlt
ms.date: 06/02/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
---

Docker エンジンは Windows に含まれていないため、個別にインポートして構成する必要があります。 また、Docker デーモンは多くのカスタム構成を受け入れることができます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker デーモンのインストールおよび構成方法について説明します。また、一般的に使用される構成例も紹介します。

## Docker のインストール

Docker は Windows コンテナーで使用するために必要です。 Docker は、Docker エンジンと Docker クライアントで構成されます。 この演習では、両方をインストールします。

Docker の実行可能ファイル用のフォルダーを作成します。

```none
New-Item -Type Directory -Path 'C:\Program Files\docker\'
```

Docker デーモンをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/dockerd -OutFile $env:ProgramFiles\docker\dockerd.exe
```

Docker クライアントをダウンロードします。

```none
Invoke-WebRequest https://aka.ms/tp5/b/docker -OutFile $env:ProgramFiles\docker\docker.exe
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

## Docker 構成ファイル

Windows で Docker デーモンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'c:\ProgramData\docker\config\daemon.json' にあります。 このファイルがまだ存在していない場合は、作成できます。

注: Windows 上の Docker では、使用できない Docker 構成オプションもあります。 たとえば、次のオプションは使用できません。 Linux 用など、Docker デーモン構成の詳細なドキュメントについては、[Docker デーモン]( https://docs.docker.com/v1.10/engine/reference/commandline/daemon/)のページを参照してください。

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

目的の構成の変更のみを構成ファイルに追加する必要があります。 たとえば、このサンプルでは、ポート 2375 で受信接続を受け入れるように Docker デーモンを構成しています。 その他のすべての構成オプションには、既定値が使用されます。

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



## サービス コントロール マネージャ

`sc config` を使用して Docker サービスを変更することで、Docker デーモンを構成することもできます。 この方法の場合、Docker デーモンのフラグは Docker サービスに直接設定されます。


```none
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

## 一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### 既定のネットワークの作成 

既定の NAT ネットワークが作成されないように Docker デーモンを構成するには、次のコマンドを使用します。 詳細については、「[コンテナーのネットワーク](../management/container_networking.md)」を参照してください。

```none
{
    "bridge" : "none"
}
```

### Docker セキュリティ グループの設定

Docker ホストにログインし、Docker コマンドをローカルで実行すると、名前付きパイプ経由でコマンドが実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker デーモンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```none
{
    "group" : "docker"
}
```


<!--HONumber=Jun16_HO2-->


