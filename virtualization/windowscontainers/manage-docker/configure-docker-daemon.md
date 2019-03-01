---
title: Windows で Docker を構成する
description: Windows で Docker を構成する
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 08/23/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: bbc405fc2a490cfe5082be112fde724707e24785
ms.sourcegitcommit: 21d93e5febd9b1b47ae1aa59d08086e6ec1691e0
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/28/2019
ms.locfileid: "9121054"
---
# <a name="docker-engine-on-windows"></a>Windows 上の Docker エンジン

Docker エンジンとクライアントは、Windows に含まれていないし、インストールされ、個別に構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントは、インストールして、Docker エンジンを構成する方法の詳細も頻繁に使われる構成の例をいくつか用意されています。


## <a name="install-docker"></a>Docker のインストール
Windows コンテナーを使用するには Docker が必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべてのものをインストールする最も簡単な方法は、クイック スタート ガイドで説明しています。 ヘルプのすべてのアイテムを取得することを設定し、最初のコンテナーを実行します。 

* [Windows Server 2019 の Windows コンテナー](../quick-start/quick-start-windows-server.md)
* [Windows 10 の Windows コンテナー](../quick-start/quick-start-windows-10.md)

スクリプト化したインストールについては、「[Use a script to install Docker EE](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)」 (スクリプトを使用して Docker EE をインストールする) をご覧ください。

Docker を使用する前にコンテナーの画像をインストールする必要があります。 詳しくは、[イメージを使用するためのクイック スタート ガイド](../quick-start/quick-start-images.md)をご覧ください。

## <a name="configure-docker-with-configuration-file"></a>構成ファイルを使用して Docker を構成する

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 このファイルがまだ存在していない場合は、作成できます。

注: Windows 上の Docker では、使用できない Docker 構成オプションもあります。 たとえば、次のオプションは使用できません。 Docker エンジン構成の詳細なドキュメントについては、[Docker デーモン構成ファイルのページ](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

```
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
    "data-root": "",
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

```
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、このサンプルでは、代替パスにイメージとコンテナーを保持するように Docker デーモンが構成されます。 指定しない場合、既定は c:\programdata\docker です。

```
{    
    "data-root": "d:\\docker"
}
```

同様に、このサンプルでは、セキュリティで保護された接続をポート 2376 でのみ受け入れるように Docker デーモンを構成しています。

```
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker サービスで Docker を構成する

`sc config` を使用して Docker サービスを変更することで、Docker エンジンを構成することもできます。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。


```
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

注: daemon.json ファイルに `"hosts": ["tcp://0.0.0.0:2375"]` というエントリが既に含まれている場合は、このコマンドを実行する必要はありません。

## <a name="common-configuration"></a>一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### <a name="default-network-creation"></a>既定のネットワークの作成 

既定の NAT ネットワークが作成されないように Docker エンジンを構成するには、次のコマンドを使用します。 詳細については、「[コンテナーのネットワーク](../container-networking/network-drivers-topologies.md)」を参照してください。

```
{
    "bridge" : "none"
}
```

### <a name="set-docker-security-group"></a>Docker セキュリティ グループの設定

Docker ホストにログインし、Docker コマンドをローカルで実行すると、名前付きパイプ経由でコマンドが実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```
{
    "group" : "docker"
}
```

## <a name="proxy-configuration"></a>プロキシの構成

`docker search` と `docker pull` のプロキシ情報を設定するには、`HTTP_PROXY` または `HTTPS_PROXY` という名前と、プロキシ情報の値を使用して Windows 環境変数を作成します。 そのためには、PowerShell で次のようなコマンドを実行します。

```powershell
[Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://username:password@proxy:port/", [EnvironmentVariableTarget]::Machine)
```

変数が設定されたら、Docker サービスを再起動します。

```powershell
Restart-Service docker
```

詳細については、[Docker.com の Windows 構成ファイルのページ](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)をご覧ください。

## <a name="uninstall-docker"></a>Docker のアンインストール
*Docker をアンインストールし、Windows 10 または Windows Server 2016 のシステムから Docker システム コンポーネントの完全クリーンアップを行うには、このセクションの手順を使用します。*

> 注: 以下の手順のコマンドはすべて、**管理者特権**による PowerShell セッションで実行する必要があります。

### <a name="step-1-prepare-your-system-for-dockers-removal"></a>手順 1: システムで Docker 削除の準備を行う 
Docker を削除する前に、システム上でコンテナーが実行されていないことを確認するようお勧めします (まだの場合)。 これを行うには、以下のコマンドが便利です。
```
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```
また、Docker を削除する前に、システムからのすべてのコンテナー、コンテナー イメージ、ネットワーク、およびボリュームを削除するようお勧めします。
```
docker system prune --volumes --all
```

### <a name="step-2-uninstall-docker"></a>手順 2: Docker のアンインストール 

#### ***<a name="steps-to-uninstall-docker-on-windows-10"></a>Windows 10 で Docker をアンインストールする手順は次のとおりです。10:***
- Windows 10 のコンピューターで、**[設定] > [アプリ]** の順に移動します。
- **[アプリと機能]** で、**[Docker for Windows]** を見つけます。
- **[Docker for Windows] > [アンインストール]** をクリックします。

#### ***<a name="steps-to-uninstall-docker-on-windows-server-2016"></a>Windows Server 2016 で Docker をアンインストールする手順は次のとおりです。16:***
管理者特権による PowerShell セッションで、コマンドレット `Uninstall-Package` および `Uninstall-Module` を使用して、Docker モジュールおよび対応するパッケージ管理プロバイダーをシステムから削除します。 
> ヒント: Docker をインストールするために使用したパッケージ プロバイダーを見つけるには、次のコマンドを使用します。 `PS C:\> Get-PackageProvider -Name *Docker*`

*例*: 
```
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

### <a name="step-3-cleanup-docker-data-and-system-components"></a>手順 3: Docker のデータおよびシステム コンポーネントをクリーンアップする
Docker を削除した後、システム上に構成が残らないよう、Docker の*既定ネットワーク*を削除します。
```
Get-HNSNetwork | Remove-HNSNetwork
```
Docker の*プログラム データ*をシステムから削除します。
```
Remove-Item "C:\ProgramData\Docker" -Recurse
```
Windows 上の Docker/コンテナーに関連付けられている *Windows オプション機能*も必要に応じて削除します。 

これには少なくとも、Windows 10 または Windows Server 2016 で Docker をインストールすると自動的に有効になる "コンテナー" 機能が含まれます。 また、"Hyper-V" 機能も含まれる可能性があります。これは、Windows 10 では Docker のインストール時に自動的に有効になりますが、Windows Server 2016 では明示的に有効にする必要がある機能です。

> **Hyper-V の無効化に関する重要な注意:** [Hyper-V 機能](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/about/)には、単なるコンテナーを大幅に超える全般的な仮想化機能が含まれます。 Hyper-V 機能を無効にする前に、この機能を必要とするその他の仮想化コンポーネントがシステム上にないことを確認してください。

#### ***<a name="steps-to-remove-windows-features-on-windows-10"></a>Windows 10 で Windows 機能を削除する手順は次のとおりです。10:***
- Windows 10 のコンピューターで、**[コントロール パネル] > [プログラム] > [プログラムと機能] > [Windows の機能の有効化または無効化]** に移動します。
- 無効にする機能の名前として、この場合は **[コンテナー]** および (必要に応じて) **"[Hyper-V]** を見つけます。
- 無効にする機能の名前の横にあるチェック ボックスを**オフ**にします。
- **[OK]** をクリックします。

#### ***<a name="steps-to-remove-windows-features-on-windows-server-2016"></a>Windows Server 2016 で Windows 機能を削除する手順は次のとおりです。16:***
管理者特権による PowerShell セッションで以下のコマンドを使用して、システムから **[コンテナー]** および (必要に応じて) **[Hyper-V]** 機能を無効にします。
```
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V 
```

### <a name="step-4-reboot-your-system"></a>手順 4: システムを再起動する
これらのアンインストール/クリーンアップ手順を完了するには、管理者特権による PowerShell セッションで以下のコマンドを実行します。
```
Restart-Computer -Force
```
