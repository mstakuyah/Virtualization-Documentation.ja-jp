---
title: Windows で Docker を構成する
description: Windows で Docker を構成する
keywords: Docker, コンテナー
author: PatrickLang
ms.date: 05/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 6885400c-5623-4cde-8012-f6a00019fafa
ms.openlocfilehash: 953dfaf71170de656f4e6ba5e91d524708d5a12a
ms.sourcegitcommit: cdf127747cfcb839a8abf50a173e628dcfee02db
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/07/2019
ms.locfileid: "9998219"
---
# <a name="docker-engine-on-windows"></a>Windows 上の Docker エンジン

Docker エンジンとクライアントは、Windows に含まれていないため、個別にインストールして構成する必要があります。 また、Docker エンジンは多くのカスタム構成に対応できます。 たとえば、デーモンで受信要求を受け入れる方法、既定のネットワークキング オプション、デバッグ/ログの設定の構成などです。 Windows でこのような構成を指定するには、構成ファイルまたは Windows サービス コントロール マネージャーを使用します。 このドキュメントでは、Docker エンジンのインストールと構成の方法について説明し、一般的に使用される構成の例をいくつか示します。

## <a name="install-docker"></a>Docker のインストール

Windows コンテナーを操作するためには、Docker が必要です。 Docker は、Docker エンジン (dockerd.exe) と Docker クライアント (docker.exe) で構成されます。 すべての機能をインストールするには、クイックスタートガイドを参照してください。これは、すべての設定を取得して最初のコンテナーを実行するのに役立ちます。

- [Windows Server 2019 の windows コンテナー](../quick-start/quick-start-windows-server.md)
- [Windows 10 の windows コンテナー](../quick-start/quick-start-windows-10.md)

スクリプトによるインストールの場合は、「スクリプトを使用して[DOCKER EE をインストールする](https://docs.docker.com/install/windows/docker-ee/#use-a-script-to-install-docker-ee)」を参照してください。

Docker を使うには、コンテナイメージをインストールする必要があります。 詳細については、「[画像を使用するためのクイックスタートガイド](../quick-start/quick-start-images.md)」を参照してください。

## <a name="configure-docker-with-a-configuration-file"></a>構成ファイルを使用して Docker を構成する

Windows で Docker エンジンを構成するには、構成ファイルを使用する方法がお勧めです。 構成ファイルは、'C:\ProgramData\Docker\config\daemon.json' にあります。 まだ存在しない場合は、このファイルを作成できます。

>[!NOTE]
>使用可能なすべての Docker 構成オプションが Windows の Docker に適用されるわけではありません。 次の例は、適用される構成オプションを示しています。 Docker Engine 構成の詳細については、「 [docker デーモン構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)」を参照してください。

```json
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

必要な構成変更を構成ファイルに追加するだけです。 たとえば、次のサンプルでは、ポート2375での着信接続を受け入れるように Docker エンジンを構成します。 その他のすべての構成オプションには、既定値が使用されます。

```json
{
    "hosts": ["tcp://0.0.0.0:2375"]
}
```

同様に、次のサンプルでは、Docker デーモンを構成して、イメージとコンテナーを代替パスに保持します。 指定しない場合、既定値`c:\programdata\docker`はです。

```json
{    
    "data-root": "d:\\docker"
}
```

次のサンプルでは、ポート2376経由でセキュリティで保護された接続のみを受け入れるように、Docker デーモンを構成します。

```json
{
    "hosts": ["tcp://0.0.0.0:2376", "npipe://"],
    "tlsverify": true,
    "tlscacert": "C:\\ProgramData\\docker\\certs.d\\ca.pem",
    "tlscert": "C:\\ProgramData\\docker\\certs.d\\server-cert.pem",
    "tlskey": "C:\\ProgramData\\docker\\certs.d\\server-key.pem",
}
```

## <a name="configure-docker-on-the-docker-service"></a>Docker サービスでの Docker の構成

Docker エンジンは、Docker サービスを変更して構成すること`sc config`もできます。 この方法の場合、Docker エンジンのフラグは Docker サービスに直接設定されます。 コマンド プロンプト (PowerShell ではなく cmd.exe) で、次のコマンドを実行します。

```cmd
sc config docker binpath= "\"C:\Program Files\docker\dockerd.exe\" --run-service -H tcp://0.0.0.0:2375"
```

>[!NOTE]
>Daemon.log ファイルに既に`"hosts": ["tcp://0.0.0.0:2375"]`エントリが含まれている場合は、このコマンドを実行する必要はありません。

## <a name="common-configuration"></a>一般的な構成

次に、一般的な Docker 構成の構成ファイル例を示します。 これらの構成を 1 つの構成ファイルにまとめることができます。

### <a name="default-network-creation"></a>既定のネットワークの作成

既定の NAT ネットワークを作成しないように Docker エンジンを構成するには、次の構成を使用します。

```json
{
    "bridge" : "none"
}
```

詳細については、「[コンテナーのネットワーク](../container-networking/network-drivers-topologies.md)」を参照してください。

### <a name="set-docker-security-group"></a>Docker セキュリティグループを設定する

Docker のホストにサインインして、Docker コマンドをローカルで実行している場合、これらのコマンドは名前付きパイプを介して実行されます。 既定では、Administrators グループのメンバーのみが、名前付きパイプ経由で Docker エンジンにアクセスできます。 このアクセス権を持つセキュリティ グループを指定するには、`group` フラグを使用します。

```json
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

詳細については、「 [Docker.com の Windows 構成ファイル](https://docs.docker.com/engine/reference/commandline/dockerd/#/windows-configuration-file)」を参照してください。

## <a name="how-to-uninstall-docker"></a>Docker をアンインストールする方法

このセクションでは、Docker をアンインストールし、Windows 10 または Windows Server 2016 システムから Docker システムコンポーネントの完全なクリーンアップを実行する方法について説明します。

>[!NOTE]
>次の手順では、管理者による PowerShell セッションからすべてのコマンドを実行する必要があります。

### <a name="prepare-your-system-for-dockers-removal"></a>Docker の削除のためにシステムを準備する

Docker をアンインストールする前に、システムでコンテナーが実行されていないことを確認してください。

実行しているコンテナーを確認するには、次のコマンドレットを実行します。

```powershell
# Leave swarm mode (this will automatically stop and remove services and overlay networks)
docker swarm leave --force

# Stop all running containers
docker ps --quiet | ForEach-Object {docker stop $_}
```

また、Docker を削除する前に、システムからコンテナー、コンテナーイメージ、ネットワーク、ボリュームをすべて削除することをお勧めします。 これを行うには、次のコマンドレットを実行します。

```powershell
docker system prune --volumes --all
```

### <a name="uninstall-docker"></a>Docker のアンインストール

次に、Docker を実際にアンインストールする必要があります。

Windows 10 で Docker をアンインストールするには

- Windows 10 コンピューターの [**設定** > ]**アプリ**に移動する
- [**アプリ & 機能**] で、 **Windows 用の Docker**を検索します。
-  > **** **Windows のアンインストールのための Docker**に移動する

Windows Server 2016 で Docker をアンインストールするには、次の操作を行います。

管理者の PowerShell セッションから、**アンインストールパッケージ**と**アンインストールモジュール**のコマンドレットを使用して、次の例に示すように、Docker モジュールとそれに対応するパッケージ管理プロバイダーをシステムから削除します。

```powershell
Uninstall-Package -Name docker -ProviderName DockerMsftProvider
Uninstall-Module -Name DockerMsftProvider
```

>[!TIP]
>Docker をインストールするために使用したパッケージプロバイダーを見つけることができます。 `PS C:\> Get-PackageProvider -Name *Docker*`

### <a name="clean-up-docker-data-and-system-components"></a>Docker データとシステムコンポーネントをクリーンアップする

Docker をアンインストールした後は、docker の既定のネットワークを削除する必要があります。これにより、その構成は、Docker の廃止後もシステムに残ります。 これを行うには、次のコマンドレットを実行します。

```powershell
Get-HNSNetwork | Remove-HNSNetwork
```

次のコマンドレットを実行して、Docker のプログラムデータをシステムから削除します。

```powershell
Remove-Item "C:\ProgramData\Docker" -Recurse
```

Windows 上の Docker/コンテナーに関連付けられている Windows オプション機能も必要に応じて削除します。

これには、Docker がインストールされているときに、Windows 10 または Windows Server 2016 で自動的に有効になる "コンテナー" 機能が含まれます。 また、"Hyper-V" 機能も含まれる可能性があります。これは、Windows 10 では Docker のインストール時に自動的に有効になりますが、Windows Server 2016 では明示的に有効にする必要がある機能です。

>[!IMPORTANT]
>[Hyper-v 機能](https://docs.microsoft.com/virtualization/hyper-v-on-windows/about/)は、コンテナーだけではなく、非常に多くの機能を実現する一般的な仮想化機能です。 Hyper-v 機能を無効にする前に、Hyper-v が必要なその他の仮想化されたコンポーネントがシステムにないことを確認します。

Windows 10 の Windows 機能を削除するには、次の操作を行います。

- **コントロールパネル** > **** > の [**プログラムと機能** > ] に移動して、**Windows の機能をオンまたはオフ**にします。
- 無効にする機能の名前 (この例では、**コンテナー**と (任意) **hyper-v**) を見つけます。
- 無効にする機能の名前の横にあるチェックボックスをオフにします。
- [ **OK** ] を選ぶ

Windows Server 2016 で Windows の機能を削除するには、次の操作を行います。

管理者の PowerShell セッションで、次のコマンドレットを実行して、**コンテナー**を無効にし、(必要に応じて) システムの**hyper-v**機能を無効にします。

```powershell
Remove-WindowsFeature Containers
Remove-WindowsFeature Hyper-V
```

### <a name="reboot-your-system"></a>システムを再起動する

アンインストールとクリーンアップを完了するには、管理者の PowerShell セッションから次のコマンドレットを実行して、システムを再起動します。

```powershell
Restart-Computer -Force
```
