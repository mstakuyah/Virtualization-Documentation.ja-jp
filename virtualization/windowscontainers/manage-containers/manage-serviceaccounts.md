---
title: Windows コンテナー用の gMSAs を作成する
description: Windows コンテナーのグループ管理サービスアカウント (gMSAs) を作成する方法について説明します。
keywords: docker、コンテナー、active directory、gmsa、グループ管理サービスアカウント、グループ管理サービスアカウント
author: rpsqrd
ms.date: 09/10/2019
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 9ed9029e534d56bfe1830281d0bfd3ddde0cee9e
ms.sourcegitcommit: 5d4b6823b82838cb3b574da3cd98315cdbb95ce2
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "10079666"
---
# <a name="create-gmsas-for-windows-containers"></a>Windows コンテナー用の gMSAs を作成する

Windows ベースのネットワークでは、一般に Active Directory (AD) を使用して、ユーザー、コンピューター、その他のネットワークリソース間の認証と承認を容易にします。 エンタープライズアプリケーションの開発者は、アプリが、ドメインに参加しているサーバーで動作するように設計されているため、統合された Windows 認証を利用できるようになっています。これにより、ユーザーや他のサービスが、自動的に透過的にサインインすることが簡単になります。id を持つアプリケーション。

Windows コンテナーはドメインに参加することはできませんが、Active Directory ドメイン id を使って、さまざまな認証シナリオをサポートすることができます。

これを実現するために、[グループ管理サービスアカウント](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)(gMSA) で実行するように windows コンテナーを構成できます。これは、windows Server 2012 で導入された特別な種類のサービスアカウントであり、複数のコンピューターで id を共有する必要はありません。を選び、パスワードを確認します。

GMSA を使ってコンテナーを実行すると、コンテナーホストは Active Directory ドメインコントローラーから gMSA パスワードを取得し、それをコンテナーインスタンスに提供します。 コンピューターアカウント (システム) がネットワークリソースにアクセスする必要がある場合、コンテナーは gMSA の資格情報を使用します。

この記事では、Windows コンテナーで Active Directory グループ管理サービスアカウントの使用を開始する方法について説明します。

## <a name="prerequisites"></a>前提条件

グループ管理サービスアカウントを使用して Windows コンテナーを実行するには、次のものが必要です。

- Windows Server 2012 以降を実行している1つ以上のドメインコントローラーを持つ Active Directory ドメイン。 GMSAs を使用するためのフォレストまたはドメインの機能レベルの要件はありませんが、gMSA パスワードは Windows Server 2012 以降を実行しているドメインコントローラーでのみ配布できます。 詳細については、「 [gMSAs の Active Directory の要件](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/getting-started-with-group-managed-service-accounts#BKMK_gMSA_Req)」を参照してください。
- GMSA アカウントを作成するためのアクセス許可。 GMSA アカウントを作成するには、ドメイン管理者であるか、または "*グループの作成*" という権限の作成を委任されたアカウントを使用している必要があります。
- CredentialSpec PowerShell モジュールをダウンロードするには、インターネットにアクセスしてください。 接続されていない環境で作業している場合は、インターネットアクセスのあるコンピューターに[モジュールを保存](https://docs.microsoft.com/powershell/module/powershellget/save-module?view=powershell-5.1)して、開発用コンピューターまたはコンテナーのホストにコピーできます。

## <a name="one-time-preparation-of-active-directory"></a>Active Directory の1回限りの準備

ドメインで gMSA をまだ作成していない場合は、キー配布サービス (KDS) のルートキーを生成する必要があります。 KDS は、承認されたホストへの gMSA パスワードの作成、回転、解放を担当します。 コンテナーホストが gMSA を使ってコンテナーを実行する必要がある場合は、現在のパスワードを取得するために KDS に連絡します。

KDS ルートキーが既に作成されているかどうかを確認するには、次の PowerShell コマンドレットを、ドメインコントローラーまたはドメインメンバーのドメイン管理者として実行します。これには、AD PowerShell ツールがインストールされています。

```powershell
Get-KdsRootKey
```

コマンドがキー ID を返す場合は、すべて設定されています。また、「[グループ管理サービスアカウントを作成](#create-a-group-managed-service-account)する」セクションに進むこともできます。 それ以外の場合は、続行して KDS ルートキーを作成します。

複数のドメインコントローラーを含む運用環境またはテスト環境では、KDS ルートキーを作成するために、PowerShell の次のコマンドレットをドメイン管理者として実行します。

```powershell
# For production environments
Add-KdsRootKey -EffectiveImmediately
```

このコマンドは、キーをすぐに有効にすることを意味しますが、KDS ルートキーが複製され、すべてのドメインコントローラーで使用できるようになるまで10時間待つ必要があります。

ドメインにドメインコントローラーが1つしかない場合は、10時間前に有効にするようにキーを設定することで、プロセスを迅速化できます。

>[!IMPORTANT]
>この手法は運用環境では使わないでください。

```powershell
# For single-DC test environments ONLY
Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
```

## <a name="create-a-group-managed-service-account"></a>グループ管理サービスアカウントを作成する

統合 Windows 認証を使うすべてのコンテナーには、少なくとも1つの gMSA が必要です。 プライマリ gMSA は、システムまたはネットワークサービスとして実行されているアプリがネットワーク上のリソースにアクセスするたびに使用されます。 GMSA の名前は、コンテナーに割り当てられているホスト名に関係なく、ネットワーク上のコンテナーの名前になります。 コンテナーコンピューターアカウントとは別の id としてコンテナーでサービスまたはアプリケーションを実行する場合は、コンテナーを追加の gMSAs 構成することもできます。

GMSA を作成する場合は、複数の異なるコンピューターで同時に使用できる共有 id も作成します。 GMSA パスワードへのアクセスは、Active Directory のアクセス制御リストで保護されています。 パスワードへのアクセスを制限するには、各 gMSA account のセキュリティグループを作成し、関連するコンテナーホストをセキュリティグループに追加することをお勧めします。

また、コンテナーではサービスプリンシパル名 (SPN) が自動的に登録されないため、少なくとも gMSA アカウント用のホスト SPN を手動で作成する必要があります。

通常、ホストまたは http SPN は、gMSA アカウントと同じ名前を使って登録されますが、クライアントがコンテナー内のアプリケーションにアクセスするときに、ロードバランサーの背後または gMSA 名とは異なる DNS 名を使用している場合は、別のサービス名の使用が必要になることがあります。

たとえば、gMSA アカウントが "WebApp01" という名前で、ユーザーがそのサイト`mysite.contoso.com`にアクセスする場合は、gMSA `http/mysite.contoso.com`アカウントで SPN を登録する必要があります。

アプリケーションによっては、固有のプロトコル用に追加の Spn が必要になることがあります。 たとえば、SQL Server には`MSSQLSvc/hostname` SPN が必要です。

次の表では、gMSA を作成するために必要な属性を示します。

|gMSA プロパティ | 必須の値 | 例 |
|--------------|----------------|--------|
|名前 | 任意の有効なアカウント名。 | `WebApp01` |
|DnsHostName | アカウント名に追加されたドメイン名。 | `WebApp01.contoso.com` |
|ServicePrincipalNames | 少なくともホストの SPN を設定し、必要に応じて他のプロトコルを追加します。 | `'host/WebApp01', 'host/WebApp01.contoso.com'` |
|PrincipalsAllowedToRetrieveManagedPassword | コンテナーホストが含まれているセキュリティグループ。 | `WebApp01Hosts` |

GMSA の名前を決めたら、PowerShell で次のコマンドレットを実行して、セキュリティグループと gMSA を作成します。

> [!TIP]
> 以下のコマンドを実行するには、**ドメイン管理者**セキュリティグループに属しているか、または、 **GroupManagedServiceAccount オブジェクトの作成**アクセス許可を委任されているアカウントを使用する必要があります。
> [新しい-ADServiceAccount](https://docs.microsoft.com/powershell/module/addsadministration/new-adserviceaccount?view=win10-ps)コマンドレットは、[リモートサーバー管理ツール](https://aka.ms/rsat)の AD PowerShell ツールに含まれています。

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
Add-ADGroupMember -Identity "WebApp01Hosts" -Members "ContainerHost01", "ContainerHost02", "ContainerHost03"
```

開発環境、テスト環境、および運用環境用に、個別の gMSA アカウントを作成することをお勧めします。

## <a name="prepare-your-container-host"></a>コンテナーホストを準備する

GMSA で Windows コンテナーを実行する各コンテナーホストは、ドメインに参加しており、gMSA パスワードを取得するためのアクセス権を持っている必要があります。

1. Active Directory ドメインにコンピューターを参加します。
2. ホストが gMSA パスワードへのアクセスを制御するセキュリティグループに属していることを確認します。
3. 新しいグループメンバーシップが取得されるように、コンピューターを再起動します。
4. Windows [Server 用](https://docs.docker.com/install/windows/docker-ee/)の windows 10 または docker[用の docker デスクトップ](https://docs.docker.com/docker-for-windows/install/)をセットアップします。
5. 勧めホストが gMSA アカウントを使用できることを確認するには、 [ADServiceAccount](https://docs.microsoft.com/powershell/module/activedirectory/test-adserviceaccount)を実行します。 コマンドが**False**を返す場合は、[トラブルシューティングの手順](gmsa-troubleshooting.md#make-sure-the-host-can-use-the-gmsa)に従います。

    ```powershell
    # To install the AD module on Windows Server, run Install-WindowsFeature RSAT-AD-PowerShell
    # To install the AD module on Windows 10 version 1809 or later, run Add-WindowsCapability -Online -Name 'Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0'
    # To install the AD module on older versions of Windows 10, see https://aka.ms/rsat

    Test-ADServiceAccount WebApp01
    ```

## <a name="create-a-credential-spec"></a>資格情報の仕様を作成する

資格情報スペックファイルは、コンテナーで使用する gMSA account に関するメタデータが含まれた JSON ドキュメントです。 Id 構成をコンテナーイメージと区別することで、コンテナーで使う gMSA を変更することができます。資格情報スペックファイルを交換するだけで、コードの変更は必要ありません。

資格情報スペックファイルは、ドメインに参加しているコンテナーホスト上で[Credentialspec PowerShell モジュール](https://aka.ms/credspec)を使用して作成されます。
ファイルを作成したら、他のコンテナーホストまたはコンテナーオーケストレータにコピーできます。
コンテナーホストはコンテナーの代わりに gMSA を取得するため、資格情報スペックファイルには、gMSA password などのシークレットは含まれません。

Docker は、Docker データディレクトリの**Credentialspecs**ディレクトリで credential spec ファイルを探します。 既定のインストールでは、に`C:\ProgramData\Docker\CredentialSpecs`このフォルダーが表示されます。

コンテナーホスト上に資格情報スペックファイルを作成するには、次の操作を行います。

1. RSAT AD PowerShell ツールをインストールする
    - Windows Server の場合は、 **Install-add-windowsfeature**を実行します。
    - Windows 10 バージョン1809以降の場合は、**追加-ウィンドウ**を実行します。「0.0.1.0」と入力すると、「」と入力します。
    - Windows 10 の以前のバージョンについ<https://aka.ms/rsat>ては、を参照してください。
2. 次のコマンドレットを実行して、 [Credentialspec PowerShell モジュール](https://aka.ms/credspec)の最新バージョンをインストールします。

    ```powershell
    Install-Module CredentialSpec
    ```

    コンテナーホストでインターネットにアクセスできない場合は、インターネット`Save-Module CredentialSpec`に接続されたコンピューターで実行し、モジュールフォルダー `C:\Program Files\WindowsPowerShell\Modules`またはコンテナーホスト`$env:PSModulePath`上の別の場所にコピーします。

3. 新しい資格情報スペックファイルを作成するには、次のコマンドレットを実行します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01
    ```

    既定では、コマンドレットは、指定された gMSA 名をコンテナーのコンピューターアカウントとして使用して、資格を作成します。 ファイルは、gMSA ドメインとファイル名のアカウント名を使用して、Docker CredentialSpecs ディレクトリに保存されます。

    コンテナーでサービスまたはプロセスをセカンダリ gMSA として実行している場合は、追加の gMSA アカウントを含む資格情報の仕様を作成できます。 そのためには、 `-AdditionalAccounts`パラメーターを使用します。

    ```powershell
    New-CredentialSpec -AccountName WebApp01 -AdditionalAccounts LogAgentSvc, OtherSvc
    ```

    サポートされているパラメーターの完全な`Get-Help New-CredentialSpec`一覧については、を実行します。

4. 次のコマンドレットを使用して、すべての資格情報のスペックと完全なパスの一覧を表示できます。

    ```powershell
    Get-CredentialSpec
    ```

## <a name="next-steps"></a>次のステップ

GMSA アカウントの設定が完了したら、次の目的で使用できます。

- [アプリを構成する](gmsa-configure-app.md)
- [コンテナーを実行する](gmsa-run-container.md)
- [オーケストレーションコンテナー](gmsa-orchestrate-containers.md)

セットアップ時に問題が発生した場合は、可能な解決策については、[トラブルシューティングガイド](gmsa-troubleshooting.md)を確認してください。

## <a name="additional-resources"></a>その他の資料

- GMSAs 詳細については、「[グループ管理サービスアカウントの概要](https://docs.microsoft.com/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)」を参照してください。
- ビデオのデモについては、Ignite 2016 から記録した[デモ](https://youtu.be/cZHPz80I-3s?t=2672)をご覧ください。
