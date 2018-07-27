---
title: Windows コンテナーの Active Directory サービス アカウント
description: Windows コンテナーの Active Directory サービス アカウント
keywords: docker, コンテナー, active directory
author: PatrickLang
ms.date: 11/04/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 9e06ad3a-0783-476b-b85c-faff7234809c
ms.openlocfilehash: 27317dbf5ba5386a3bd555c53c781aac2fc110a7
ms.sourcegitcommit: edc153ffef01094c2324a0da2f9a301b31015a58
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/23/2018
ms.locfileid: "1912669"
---
# <a name="active-directory-service-accounts-for-windows-containers"></a>Windows コンテナーの Active Directory サービス アカウント

ユーザーおよびその他のサービスは、データをセキュリティで保護し続け、不正な使用を回避できるように、アプリケーションとサービスに対して認証された接続を作成する必要があります。 Windows Active Directory (AD) ドメインは、パスワードと証明書認証の両方をネイティブでサポートします。 Windows ドメインに参加しているホストでアプリケーションまたはサービスをビルドするときに、ローカル システムまたはネットワーク サービスとして実行する場合、既定では、そのアプリケーションまたはサービスはホストの ID を使用します。 それ以外の場合は、代わりに別の AD アカウントを認証用に構成する可能性があります。

Windows コンテナーをドメインに参加させることはできませんが、デバイスが領域に参加する場合と同様に、Active Directory ドメインの ID を利用することもできます。 Windows Server 2012 のドメイン コントローラーでは、各サービスでの共有用に設計された、グループ管理サービス アカウント (gMSA) という新しいアカウントが導入されています。 グループ管理サービス アカウント (gMSA) を使用すると、Windows コンテナー自体およびホスト対象のサービスは、ドメイン ID として特定の gMSA を使用するように構成することができます。 ローカル システムまたはネットワーク サービスとして実行しているサービスは、現在、ドメインに参加しているホストの ID を使用しているのと同じように、Windows コンテナーの ID を使用します。 誤って公開される可能性があるコンテナー イメージに、パスワードまたは証明書の秘密キーを保存することはありません。また、コンテナーは、保存されたパスワードや証明書を変更するために再構築することなく、開発環境、テスト環境、および運用環境に再配置することができます。 


# <a name="glossary--references"></a>用語集と参照
- [Active Directory](http://social.technet.microsoft.com/wiki/contents/articles/1026.active-directory-services-overview.aspx) は、Windows 上のユーザー、コンピューター、およびサービス アカウント情報の検出、検索、レプリケーションに使用されるサービスです。 
  - [Active Directory Domain Services](https://technet.microsoft.com/en-us/library/dd448614.aspx) は、コンピューターとユーザーを認証するために使用される Windows Active Directory ドメインを指定します。 
  - デバイスが Active Directory ドメインのメンバーの場合、デバイスは_ドメイン参加済み_になります。 ドメイン参加済みは、ドメイン コンピューター ID でデバイスを示すだけでなく、さまざまなドメイン参加済みサービスも点灯させるデバイスの状態です。
  - [グループ管理サービス アカウント](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx) (略称: gMSA) は、パスワードを共有しないで、Active Directory を使用してサービスのセキュリティを容易に保護できる Active Directory アカウントの種類です。 複数のマシンまたはコンテナーが、サービス間の接続の認証に必要な同じ gMSA を共有します。
- _CredentialSpec_ PowerShell モジュール - このモジュールは、コンテナーで使用されるグループ管理サービス アカウントを構成するために使用されます。 スクリプトのモジュールと例の手順は、[windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) で入手できます。ServiceAccount をご覧ください。

# <a name="how-it-works"></a>動作のしくみ

現在、グループ管理サービス アカウントは通常、1 つのコンピューターまたはサービスと別のコンピューターまたはサービスの接続を、セキュリティで保護するために使用されます。 gMSA を使用する一般的な手順は次のとおりです。

1. gMSA を作成します
2. gMSA として実行するサービスを構成します
3. Active Directory の gMSA シークレットにサービス アクセスを実行しているドメイン参加済みホストを指定します
4. データベースまたはファイル共有などの他のサービスで gMSA へのアクセスを許可します

サービスを起動すると、ドメイン参加済みホストは、Active Directory から gMSA シークレットを自動的に取得し、そのアカウントを使用してサービスを実行します。 そのサービスは gMSA として実行されているため、gMSA が許可されたすべてのリソースにアクセスできます。

Windows コンテナーは同様の手順に従います。

1. gMSA を作成します。 既定では、ドメイン管理者またはアカウント オペレーターが、この操作を行う必要があります。 それ以外の方法として、gMSA を使用するサービスを管理する管理者に、gMSA の作成および管理を行う権限を委任することもできます。 ドメイン内には Windows Server 2012 以降のドメイン コントローラーが 1 台以上必要になりますが、特定のドメイン機能レベルの使用に関する要件はありません。 [gMSA の概要](https://technet.microsoft.com/en-us/library/jj128431(v=ws.11).aspx)をご覧ください
2. gMSA にドメイン参加済みコンテナーのホスト アクセスを与えます
3. データベースまたはファイル共有などの他のサービスで gMSA へのアクセスを許可します
4. [windows-server-container-tools](https://github.com/Microsoft/Virtualization-Documentation/tree/live/windows-server-container-tools) から CredentialSpec PowerShell モジュールを使用して、gMSA を使用するために必要な設定を保存します
5. 追加のオプションを使用して、コンテナーを開始します `--security-opt "credentialspec=..."`

[!NOTE]
[ここ](https://docs.microsoft.com/en-us/windows/device-security/security-policy-settings/network-access-allow-anonymous-sidname-translation)で説明されている手順に従って、コンテナー ホストで匿名の SID と名前の変換を許可する必要があります。許可されていない場合、アカウントを SID に変換できないというエラーが出力されることがあります。

ただし、匿名の SID と名前の変換を許可する必要性を検討する前に、次の対策が実施されていることを確認します。

1. gMSA アカウントの名前は、サービスの名前と一致している必要があります (例:  "myapp")。
2. -h 引数を含めて、コンテナーで起動時に使用する必要があるホスト名を指定します。 
```
docker run --security-opt "credentialspec=file://myapp.json" -d -p 80:80 -h myapp.mydomain.local <imagename>
```
3. gMSA アカウントを作成するときに使用するサービス プリンシパル名 (SPN) は、コンテナーの実行時に使用する -h 引数と一致している必要があります。 作成時に gMSA に SPN を追加しなかった場合は、後で、アカウントのプロパティにそれらを追加できます。

コンテナーを起動すると、ローカル システムまたはネットワーク サービスとして実行しているインストール済みサービスが、gMSA として実行されて表示されます。 これは、コンピューター アカウントの代わりに、gMSA が使用される場合を除いて、これらのアカウントがドメイン参加済みホストで動作する方法と同様です。 

![ダイアグラム - サービス アカウント](media/serviceaccount_diagram.png)


# <a name="example-uses"></a>使用例


## <a name="sql-connection-strings"></a>SQL 接続文字列
サービスがローカル システムまたはコンテナー内のネットワーク サービスとして実行されている場合は、Microsoft SQL Server に接続するために Windows 統合認証を使用できます。

例:

```
Server=sql.contoso.com;Database=MusicStore;Integrated Security=True;MultipleActiveResultSets=True;Connect Timeout=30
```

Microsoft SQL Server では、ドメインと gMSA の名前を使用し、後に $ を付けてログインを作成します。 ログインが作成されると、データベース上のユーザーに追加され、適切なアクセス許可を与えることができます。

例: 

```sql
CREATE LOGIN "DEMO\WebApplication1$"
    FROM WINDOWS
    WITH DEFAULT_DATABASE = "MusicStore"
GO

USE MusicStore
GO
CREATE USER WebApplication1 FOR LOGIN "DEMO\WebApplication1$"
GO

EXEC sp_addrolemember 'db_datareader', 'WebApplication1'
EXEC sp_addrolemember 'db_datawriter', 'WebApplication1'
```

実際の動作を確認するには、Microsoft Ignite 2016 のセッション ”Walk the Path to Containerization - transforming workloads into containers” (コンテナー化への道: ワークロードをコンテナーに変換する) の[録画デモ](https://youtu.be/cZHPz80I-3s?t=2672)をご覧ください。
