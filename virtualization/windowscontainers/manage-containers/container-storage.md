---
title: Windows Server コンテナー記憶域
description: Windows Server コンテナーがホストとその他の種類の記憶域を使用する方法
keywords: コンテナー, ボリューム, 記憶域, マウント, bindmount
author: patricklang
ms.openlocfilehash: 5f8ff4b25ad4a4c34ed2e28683607cfc02891e1e
ms.sourcegitcommit: 62fff5436770151a28b6fea2be3a8818564f3867
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/24/2019
ms.locfileid: "10147225"
---
# <a name="overview"></a>概要

<!-- Great diagram would be great! -->


## <a name="layer-storage"></a>レイヤー記憶域

これは、コンテナーに組み込まれているすべてのファイルです。 コンテナーに対する `docker pull` および `docker run` を実行すると、結果は毎回同じになります。


### <a name="where-layers-are-stored-and-how-to-change-it"></a>レイヤーの保存場所と変更方法

既定のインストールでは、レイヤーが `C:\ProgramData\docker` に格納され、"image" ディレクトリおよび "windowsfilter" ディレクトリに分割されます。 レイヤーの保存場所は、「[Windows 上の Docker エンジン](../manage-docker/configure-docker-daemon.md)」の記載に従い、`docker-root` 構成を使用して変更できます。

> [!NOTE]
> レイヤー記憶域は、NTFS のみでサポートされます。 ReFS はサポートされません。

レイヤー ディレクトリ内のファイルは変更しないでください。これらは以下のようなコマンドを使用して慎重に管理します。

- [docker images](https://docs.docker.com/engine/reference/commandline/images/)
- [docker rmi](https://docs.docker.com/engine/reference/commandline/rmi/)
- [docker pull](https://docs.docker.com/engine/reference/commandline/pull/)
- [docker load](https://docs.docker.com/engine/reference/commandline/load/)
- [docker save](https://docs.docker.com/engine/reference/commandline/save/)

### <a name="supported-operations-in-layer-storage"></a>レイヤー記憶域でサポートされる操作

コンテナーの実行では、ほとんどの NTFS 操作を使用できます (トランザクションを除く)。 これには ACL の設定が含まれており、すべての ACL はコンテナー内でチェックされます。 1 つのコンテナー内で複数のユーザーとしてプロセスを実行するには、`RUN net user /create ...` で `Dockerfile` 内にユーザーを作成し、ファイルの ACL を設定します。次に、[Dockerfile USER ディレクティブ](https://docs.docker.com/engine/reference/builder/#user)を使用して、そのユーザーで実行するプロセスを構成します。


## <a name="image-size"></a>画像のサイズ
Windows アプリケーションでは、新しいファイルをインストールまたは作成する前に、または一時ファイルのクリーンアップのトリガーとして、空きディスク領域の量を照会することが一般的なパターンとなっています。  アプリケーションの互換性を最大限に高める目的で、Windows コンテナーの C: ドライブは 20 GB の空き仮想サイズとなります。  ユーザーによっては、この既定値を上書きして空き領域の増減が必要になることがあります。この操作は、"storage-opt" 構成の "size" オプションで実現できます。

### <a name="examples"></a>例
コマンド ライン: `docker run --storage-opt "size=50GB" microsoft/windowsservercore:1709 cmd`

Docker 構成ファイル
```
"storage-opts": [
    "size=50GB"
  ]
```
> この方法は Docker ビルドに使用できます。
Docker 構成ファイルの変更について詳しくは、[Docker の構成](https://docs.microsoft.com/virtualization/windowscontainers/manage-docker/configure-docker-daemon#configure-docker-with-configuration-file)ドキュメントをご覧ください。


## <a name="persistent-volumes"></a>永続的ボリューム

永続的な記憶域をコンテナーにするには、いくつかの方法があります。

- バインド マウント
- 名前付きボリューム

Docker には、[ボリュームの使用](https://docs.docker.com/engine/admin/volumes/volumes/)に関する概要が用意されているため、まずそのドキュメントをご覧ください。 このページの残りの部分では、Linux と Windows の違いについて説明し、Windows の場合の例を示します。


### <a name="bind-mounts"></a>バインド マウント

[バインド マウント](https://docs.docker.com/engine/admin/volumes/bind-mounts/)を使用すると、コンテナーがディレクトリをホストと共有できます。 この方法は、コンテナーを再起動したときに使用可能なローカル コンピューター上のファイルを保存する場所が必要な場合や、複数のコンテナーと共有する場合に有効です。 コンテナーを複数のコンピューターで実行して同じファイルにアクセスする場合は、代わりに名前付きボリュームまたは SMB マウントを使用する必要があります。

#### <a name="permissions"></a>アクセス許可

バインド マウントに使用されるアクセス許可モデルは、コンテナーの分離レベルによって異なります。

**Hyper-V による分離**を使用するコンテナー (Windows Server Version 1709 上の Linux コンテナーを含む) では、単純な読み取り専用または読み取り/書き込みアクセス許可モデルを使用します。
ホスト上のファイル アクセスには `LocalSystem` アカウントを使用します。 コンテナーに対するアクセスが拒否される場合は、ホスト上のそのディレクトリに `LocalSystem` からのアクセス許可があることを確認してください。
読み取り専用フラグが使用された場合、コンテナー内のボリュームへの変更はホスト上のディレクトリに永続化または表示されません。

**プロセス分離**を使用する Windows Server コンテナーは、データへのアクセスにコンテナー内のプロセス ID を使用する (ファイルの ACL が優先される) ため、やや異なります。
マウントされたボリューム内のファイルとディレクトリへのアクセスには、`LocalSystem` ではなく、コンテナーで実行されているプロセスの ID (既定で Windows サーバー コアの場合は "ContainerAdministrator"、Nano Server コンテナーの場合は "ContainerUser") が使用され、データを使用するにはアクセス権が必要になります。
これらの ID は、ファイルが格納されているホスト上ではなくコンテナーのコンテキスト内でのみ存在するため、コンテナーへのアクセスを許可する ACL を構成する際には、`Authenticated Users` など既知のセキュリティ グループを使用する必要があります。

> [!WARNING]
> `C:\` などの機密性の高いディレクトリは、信頼されていないコンテナーにバインド マウントしないでください。 このようなバインド マウントを行うと、通常はアクセスできないホスト上のファイルへの変更が許可され、セキュリティ侵害が生じる可能性があります。

使用例: 

- `docker run -v c:\ContainerData:c:\data:RO` (読み取り専用アクセス用)
- `docker run -v c:\ContainerData:c:\data:RW` (読み取り/書き込みアクセス用)
- `docker run -v c:\ContainerData:c:\data` (読み取り/書き込みアクセス用、既定)

#### <a name="symlinks"></a>symlink

symlink は、コンテナ内で解決されます。 symlink であるコンテナーまたは symlink を含むコンテナーにホスト パスをバインド マウントしても、コンテナーではこれらの symlink にアクセスできません。

#### <a name="smb-mounts"></a>SMB マウント

Windows Server Version 1709 では、"SMB グローバル マッピング" と呼ばれる新しい機能により、SMB 共有をホストにマウントし、その共有上のディレクトリをコンテナーに渡すことができます。 コンテナーは、特定のサーバー、共有、ユーザー名、またはパスワードを指定して構成する必要はありません。これはすべてホストで処理されます。 コンテナーは、ローカル記憶域がある場合と同様に機能します。

##### <a name="configuration-steps"></a>構成手順

1. コンテナーホストでは、次のように、リモート SMB 共有をグローバルにマップします。
    ```
    $creds = Get-Credential
    New-SmbGlobalMapping -RemotePath \\contosofileserver\share1 -Credential $creds -LocalPath G:
    ```
    このコマンドは、資格情報を使ってリモート SMB サーバーで認証します。 次に、リモート共有パスを G: ドライブ文字にマップします (その他の使用可能なドライブ文字も指定できます)。 このコンテナー ホストで作成されたコンテナーでは、自身のデータ ボリュームを G: ドライブ上のパスにマップできます。

    > [!NOTE]
    > コンテナーの SMB グローバルマッピングを使用している場合、コンテナーホストのすべてのユーザーがリモート共有にアクセスできます。 コンテナー ホストで実行されているすべてのアプリケーションにも、マッピングされたリモート共有へのアクセス権があります。

2. グローバルにマウントされた SMB 共有んいマップされたデータ ボリュームを使用してコンテナーを作成します: docker run -it --name demo -v g:\ContainerData:G:\AppData1 microsoft/windowsservercore:1709 cmd.exe

    コンテナー内で、G:\AppData1 はリモート共有の "ContainerData" ディレクトリにマップされます。 グローバルにマップされたリモート共有に格納されているデータは、コンテナー内のアプリケーションで使用できるようになります。 複数のコンテナーが同じコマンドを使用して、この共有データへの読み取り/書き込みアクセス権を獲得することもできます。

この SMB グローバル マッピングは、互換性のある SMB サーバー上で動作できる SMB クライアント側の機能によってサポートされます。次のようなものがあります。

- 記憶域スペース ダイレクト (S2D) 上のスケール アウト ファイル サーバーまたは従来の SAN
- Azure Files (SMB 共有)
- 従来のファイル サーバー
- SMB プロトコルのサード パーティ実装 (例: NAS アプライアンス)

> [!NOTE]
> SMB グローバル マッピングでは、Windows Server Version 1709 の DFS、DFSN、および DFSR 共有がサポートされていません。

### <a name="named-volumes"></a>名前付きボリューム

名前付きボリュームの機能を使用すると、名前を指定してボリュームを作成し、コンテナーに割り当てて、同じ名前で後で再利用できます。 作成時の実際のパスを追跡する必要はなく、名前だけを使用します。 Windows 上の Docker エンジンには、ローカル コンピューター上にボリュームを作成できる、組み込みの名前付きボリューム プラグインがあります。 複数のコンピューターで名前付きボリュームを使用する場合は、追加のプラグインが必要です。

手順の例:

1. `docker volume create unwound` - 'unwound' という名前のボリュームを作成します。
2. `docker run -v unwound:c:\data microsoft/windowsservercore` - c:\data にマップされているボリュームでコンテナーを開始します。
3. コンテナー内の c:\data にいくつかのファイルを書き込み、コンテナーを停止します。
4. `docker run -v unwound:c:\data microsoft/windowsservercore` - 新しいコンテナーを開始します。
5. 新しいコンテナー内で `dir c:\data` を実行します。ここにはまだファイルが含まれています。

> [!NOTE]
> Windows Server はターゲットのパス名 (コンテナー内のパス) を小文字に変換します。i `-v unwound:c:\MyData`、または`-v unwound:/app/MyData` linux コンテナーでは、コンテナー `c:\mydata`内または`/app/mydata` linux コンテナー内に、マップされている (存在しない場合は作成される) ディレクトリが表示されます。
