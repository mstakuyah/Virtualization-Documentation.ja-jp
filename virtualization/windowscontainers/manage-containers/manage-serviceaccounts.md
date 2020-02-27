---
title: Windows コンテナー用に gMSAs を作成する
description: Windows コンテナーのグループ管理サービスアカウント (gMSAs) を作成する方法。
keywords: docker、コンテナー、active directory、gmsa、グループの管理されたサービスアカウント、グループの管理されたサービスアカウント
author: rpsqrd
ms.date: 01/03/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 36061cfc491dd9dd581d1e6bce92a29e4a6f217d
ms.sourcegitcommit: 530712469552a1ef458883001ee748bab2c65ef7
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/26/2020
ms.locfileid: "77628937"
---
# <a name="create-gmsas-for-windows-containers"></a>Windows コンテナー用に gMSAs を作成する

Windows ベースのネットワークでは、一般的に Active Directory (AD) を使用して、ユーザー、コンピューター、およびその他のネットワークリソース間の認証と承認を容易にします。 エンタープライズアプリケーション開発者は、統合 Windows 認証を利用するために、アプリを AD に統合され、ドメインに参加しているサーバー上で実行するように設計されていることがよくあります。これにより、ユーザーや他のサービスが自動的に透過的にサインインできるようになります。id を持つアプリケーション。

Windows コンテナーをドメインに参加させることはできませんが、Active Directory ドメイン id を使用してさまざまな認証シナリオをサポートすることができます。

これを実現するには、Windows コンテナーを[グループ管理サービスアカウント](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) で実行するように構成します。これは、windows Server 2012 で導入された特別な種類のサービスアカウントで、パスワードを知らなくても複数のコンピューターが id を共有できるように設計されています。

GMSA を使用してコンテナーを実行すると、コンテナーホストは、Active Directory ドメインコントローラーから gMSA パスワードを取得し、コンテナーインスタンスに渡します。 コンテナーは、コンピューターアカウント (システム) がネットワークリソースにアクセスする必要がある場合は常に、gMSA 資格情報を使用します。

この記事では、Windows コンテナーで Active Directory グループの管理されたサービスアカウントの使用を開始する方法について説明します。

## <a name="prerequisites"></a>前提条件

グループの管理されたサービスアカウントを使用して Windows コンテナーを実行するには、次のものが必要です。

- Windows Server 2012 以降を実行しているドメインコントローラーが少なくとも1つある Active Directory ドメイン。 GMSAs を使用するためのフォレストまたはドメインの機能レベルの要件はありませんが、gMSA パスワードは、Windows Server 2012 以降を実行しているドメインコントローラーでのみ配布できます。 詳細については、「 [gMSAs の Active Directory の要件](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)」を参照してください。
- GMSA アカウントを作成するためのアクセス許可。 GMSA アカウントを作成するには、ドメイン管理者であるか、または ""*オブジェクトの作成*アクセス許可を委任されたアカウントを使用する必要があります。
- インターネットにアクセスして、CredentialSpec PowerShell モジュールをダウンロードします。 接続されていない環境で作業している場合は、インターネットにアクセスできるコンピューターに[モジュールを保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)し、開発用コンピューターまたはコンテナーホストにコピーできます。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory の1回限りの準備

ドメインに gMSA をまだ作成していない場合は、キー配布サービス (KDS) のルートキーを生成する必要があります。 KDS は、承認されたホストに対して gMSA パスワードを作成、交換、および解放する役割を担います。 コンテナーホストで gMSA を使用してコンテナーを実行する必要がある場合は、KDS に接続して現在のパスワードを取得します。

KDS ルートキーが既に作成されているかどうかを確認するには、AD PowerShell ツールがインストールされているドメインコントローラーまたはドメインメンバーで、ドメイン管理者として次の PowerShell コマンドレットを実行します。

```powershell
Get-KdsRootKey
```

コマンドからキー ID が返された場合は、すべて設定されているので、「[グループの管理](#create-a-group-managed-service-account)されたサービスアカウントの作成」セクションに進むことができます。 それ以外の場合は、続行して KDS ルートキーを作成します。

複数のドメインコントローラーがある運用環境またはテスト環境では、KDS ルートキーを作成するために、ドメイン管理者として PowerShell で次のコマンドレットを実行します。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

このコマンドは、キーが直ちに有効になることを意味しますが、KDS ルートキーがレプリケートされ、すべてのドメインコントローラーで使用できるようになるまで10時間待つ必要があります。

ドメイン内にドメインコントローラーが1つしかない場合は、10時間前に有効にするようにキーを設定して、プロセスを迅速に実行できます。

>[!IMPORTANT]
>運用環境では、この手法を使用しないでください。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>グループの管理されたサービスアカウントを作成する

統合 Windows 認証を使用するすべてのコンテナーには、少なくとも1つの gMSA が必要です。 プライマリ gMSA は、システムまたはネットワークサービスとして実行されているアプリがネットワーク上のリソースにアクセスするときに使用されます。 GMSA の名前は、コンテナーに割り当てられているホスト名に関係なく、ネットワーク上のコンテナーの名前になります。 コンテナーコンピューターアカウントとは別の id としてコンテナー内のサービスまたはアプリケーションを実行する場合は、追加の gMSAs を使用してコンテナーを構成することもできます。

GMSA を作成するときに、複数の異なるコンピューターで同時に使用できる共有 id も作成します。 GMSA パスワードへのアクセスは、Active Directory Access Control リストによって保護されています。 GMSA アカウントごとにセキュリティグループを作成し、そのセキュリティグループに関連するコンテナーホストを追加して、パスワードへのアクセスを制限することをお勧めします。

最後に、コンテナーはサービスプリンシパル名 (SPN) を自動的に登録しないため、少なくとも gMSA アカウント用のホスト SPN を手動で作成する必要があります。

通常、ホストまたは http SPN は、gMSA アカウントと同じ名前を使用して登録されますが、クライアントが、ロードバランサーの背後からコンテナー化されたアプリケーションにアクセスする場合や、gMSA 名とは異なる DNS 名を使用する場合は、別のサービス名を使用することが必要になる場合があります。

たとえば、gMSA アカウントに "WebApp01" という名前が付けられているのに、ユーザーが `mysite.contoso.com`のサイトにアクセスする場合は、gMSA アカウントに `http/mysite.contoso.com` SPN を登録する必要があります。

アプリケーションによっては、独自のプロトコルに対して追加の Spn が必要になる場合があります。 たとえば、SQL Server には `MSSQLSvc/hostname` SPN が必要です。

次の表に、gMSA を作成するために必要な属性を示します。

|gMSA プロパティ | 必須の値 | 例 |
|--------------|----------------|--------|
|Name | 任意の有効なアカウント名。 | `WebApp01` |
|DnsHostName | アカウント名に追加されたドメイン名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 少なくともホスト SPN を設定し、必要に応じて他のプロトコルを追加します。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | コンテナーホストを含むセキュリティグループ。 | `WebApp01Hosts` |

GMSA の名前を決定したら、PowerShell で次のコマンドレットを実行して、セキュリティグループと gMSA を作成します。

> [!TIP]
> 次のコマンドを実行するには、 **Domain Admins**セキュリティグループに属しているアカウントを使用するか、または、作成中の**GroupManagedServiceAccount オブジェクトの作成**権限を委任されている必要があります。
> [新しい-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps)コマンドレットは、[リモートサーバー管理ツール](https://aka.ms/rsat)の AD PowerShell ツールの一部です。

```powershell
# Replace 'WebApp01' and 'contoso.com' with your own gMSA and domain names, respectively

# To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
# To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
# To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

# Create the security group
New-ADGroup -Name "WebApp01 Authorized Hosts" -SamAccountName "WebApp01Hosts" -GroupScope DomainLocal

# Create the gMSA
New-ADServiceAccount -Name "WebApp01" -DnsHostName "WebApp01.contoso.com" -ServicePrincipalNames "host/WebApp01", "host/WebApp01.contoso.com" -PrincipalsAllowedToRetrieveManagedPassword "WebApp01Hosts"

# Add your container hosts to the security group
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01$", "ContainerHost02$", "ContainerHost03$"
```

開発、テスト、運用の各環境用に個別の gMSA アカウントを作成することをお勧めします。

## <a name="prepare-your-container-host"></a>コンテナーホストを準備する

GMSA を使用して Windows コンテナーを実行する各コンテナーホストは、ドメインに参加していて、gMSA パスワードを取得するためのアクセス権を持っている必要があります。

1. コンピューターを Active Directory ドメインに参加させます。
2. ホストが、gMSA パスワードへのアクセスを制御するセキュリティグループに属していることを確認してください。
3. コンピューターを再起動して、新しいグループメンバーシップを取得します。
4. Windows 10 または[Docker for Windows Server](https://docs.docker.com/install/windows/docker-ee/)[の Docker Desktop](https://docs.docker.com/docker-for-windows/install/)を設定します。
5. しない[テスト ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行して、ホストで gMSA アカウントを使用できることを確認します。 コマンドが**False**を返す場合は、[トラブルシューティングの手順](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)に従います。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>資格情報の仕様を作成する

資格情報の仕様ファイルは、コンテナーで使用する gMSA アカウントに関するメタデータを含む JSON ドキュメントです。 コンテナーイメージとは別に id 構成を保持することで、資格情報の仕様ファイルを交換するだけでコンテナーが使用する gMSA を変更できます。コードの変更は必要ありません。

資格情報仕様ファイルは、ドメインに参加しているコンテナーホストで、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)を使用して作成されます。
作成したファイルは、他のコンテナーホストまたはコンテナー orchestrator にコピーできます。
コンテナーホストがコンテナーの代わりに gMSA を取得するため、資格情報の仕様ファイルには gMSA パスワードなどのシークレットが含まれていません。

Docker は、Docker データディレクトリの**Credentialspecs**ディレクトリにある資格情報仕様ファイルを検索することを想定しています。 既定のインストールでは、このフォルダーは `C:\ProgramData\Docker\CredentialSpecs`にあります。

コンテナーホストで資格情報仕様ファイルを作成するには、次のようにします。

1. RSAT AD PowerShell ツールをインストールする
    - Windows Server の場合は、 **Install-add-windowsfeature-AD-PowerShell**を実行します。
    - Windows 10 バージョン1809以降の場合は、 **"0.0.1.0"** という名前のアドオンを実行します (& ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~ ~)。
    - 以前のバージョンの Windows 10 については、「<https://aka.ms/rsat>」を参照してください。
2. 次のコマンドレットを実行して、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の最新バージョンをインストールします。

    ```powershell
    Install-Module CredentialSpec
    ```

    コンテナーホストでインターネットにアクセスできない場合は、インターネットに接続されたコンピューターで `Save-Module CredentialSpec` を実行し、モジュールフォルダーを `C:\Program Files\WindowsPowerShell\Modules` またはコンテナーホスト上の `$env:PSModulePath` 内の別の場所にコピーします。

3. 次のコマンドレットを実行して、新しい資格情報仕様ファイルを作成します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    既定では、このコマンドレットは、指定された gMSA 名をコンテナーのコンピューターアカウントとして使用して、cred 仕様を作成します。 ファイルは、ファイル名の gMSA ドメインとアカウント名を使用して、Docker CredentialSpecs ディレクトリに保存されます。

    ファイルを別のディレクトリに保存する場合は、`-Path` パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -Path "C:\MyFolder\WebApp01_CredSpec.json"
    ```

    コンテナーでサービスまたはプロセスをセカンダリ gMSA として実行している場合は、追加の gMSA アカウントを含む資格情報仕様を作成することもできます。 これを行うには、`-AdditionalAccounts` パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    サポートされているパラメーターの完全な一覧については、`Get-Help New-CredentialSpec -Full`を実行してください。

4. 次のコマンドレットを使用して、すべての資格情報の仕様と完全なパスの一覧を表示できます。

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>次のステップ:

GMSA アカウントの設定が完了したので、次の目的で使用できます。

- [アプリの構成](gmsa-configure-app.md)
- [コンテナーの実行](gmsa-run-container.md)
- [コンテナーの調整](gmsa-orchestrate-containers.md)

セットアップ中に問題が発生した場合は、[トラブルシューティングガイド](gmsa-troubleshooting.md)で解決策を確認してください。

## <a name="additional-resources"></a>その他のリソース

- GMSAs の詳細については、「グループの管理された[サービスアカウントの概要](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」を参照してください。
- ビデオデモについては、Ignite 2016 からの記録された[デモ](https://youtu.be/cZHPz80I-3s?t=2672)をご覧ください。
