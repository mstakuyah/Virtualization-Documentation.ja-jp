---
title: "Windows コンテナーの進行中の作業"
description: "Windows コンテナーの進行中の作業"
keywords: docker, containers
author: scooley
manager: timlt
ms.date: 05/26/2016
ms.topic: article
ms.prod: windows-containers
ms.service: windows-containers
ms.assetid: 5d9f1cb4-ffb3-433d-8096-b085113a9d7b
redirect_url: ../containers_welcome
ms.sourcegitcommit: e3f5535594123f6b4f8931e41a91d92f3b837814
ms.openlocfilehash: 085bb8c0158aedf4270cf2423114ec1901af1ebd

---

# 進行中の作業

問題の対処方法がここにない場合、またはご質問がございましたら、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)に投稿してください。

-----------------------

## 一般的な機能

### コンテナーとホスト ビルド番号の一致
Windows コンテナーには、ビルドとパッチのレベルに関してコンテナー ホストに一致するオペレーティング システム イメージが必要です。 一致しない場合、コンテナーまたはホストが不安定になったり、予想外の動作が発生したりします。

Windows コンテナー ホストに更新プログラムをインストールする場合、コンテナー ベース OS イメージを更新し、適合する更新プログラムを提供する必要があります。
<!-- Can we give examples of behavior or errors?  Makes it more searchable -->

**回避策:**   
ダウンロードし、コンテナーのホストの OS バージョンとパッチ レベルが一致するコンテナーの基本イメージをインストールします。

### 既定のファイアウォール動作
コンテナー ホストと各コンテナーから構成される環境では、コンテナー ホストにのみファイアウォールが与えられます。 コンテナー ホストで構成されるすべてのファイアウォール ルールがそのすべてのコンテナーに反映されます。

### Windows コンテナーの起動が遅い
コンテナーの起動に 30 秒以上かかる場合、ウイルス スキャンが重複して実行されている可能性があります。

Windows Defender などの多くのマルウェア対策ソリューションは、コンテナー イメージ内のファイル (すべての OS バイナリを含む) およびコンテナー OS イメージ内のファイルを必要以上にスキャンする場合があります。  新しいコンテナーが作成され、マルウェア対策の観点から "コンテナーのファイル" のすべてが以前にスキャンされたことがない新しいファイルのように認識される場合は、必ずこの問題が発生します。  したがって、コンテナー内のプロセスが、これらのファイルを読み取るしようとしています。 マルウェア対策コンポーネント回目のスキャンにファイルへのアクセスを許可する前にします。  実際にはこれらのファイルは、コンテナーのイメージは、インポートまたはサーバーに取得したときに既にスキャンされていました。 Windows Server Technical Preview 5 から、Windows Defender などのマルウェア対策ソフトウェアが前述の状況を認識し、適宜動作し、重複スキャンを回避するように、新しいインフラストラクチャが導入されています。 マルウェア対策ソリューションでは、[ここ](https://msdn.microsoft.com/en-us/windows/hardware/drivers/ifs/anti-virus-optimization-for-windows-containers)に記載されているガイダンスに従って個々のソリューションを更新し、重複スキャンを回避することができます。 

### メモリが 48 MB 未満に制限されている場合に開始/停止が失敗する
メモリが 48 MB 未満に制限されている場合、Windows コンテナーに整合性のないエラーが無作為に発生します。

次の PowerShell を実行し、開始/停止アクションを繰り返すと、開始または停止に失敗します。

```PowerShell
new-container "Test" -containerimagename "WindowsServerCore" -MaximumBytes 32MB
start-container test
stop-container test
```

**回避策:**  
メモリの値を 48 MB に変更します。 


### 4 コア VM でプロセッサ数が 1 または 2 のとき、コンテナーを開始できない

Windows コンテナーが起動に失敗し、次のエラーが表示されます。  
`failed to start: This operation returned because the timeout period expired. (0x800705B4).`

これは、4 コア VM でプロセッサ数が 1 または 2 に設定されている場合に発生します。

``` PowerShell
new-container "Test2" -containerimagename "WindowsServerCore"
Set-ContainerProcessor -ContainerName test2 -Maximum 2
Start-Container test2

Start-Container : 'test2' failed to start.
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4).
'test2' failed to start. (Container ID 133E9DBB-CA23-4473-B49C-441C60ADCE44)
'test2' failed to start: This operation returned because the timeout period expired. (0x800705B4). (Container ID
133E9DBB-CA23-4473-B49C-441C60ADCE44)
At line:1 char:1
+ Start-Container test2
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : OperationTimeout: (:) [Start-Container], VirtualizationException
    + FullyQualifiedErrorId : OperationTimeout,Microsoft.Containers.PowerShell.Cmdlets.StartContainer
PS C:\> Set-ContainerProcessor -ContainerName test2 -Maximum 3
PS C:\> Start-Container test2
```

**回避策:**  
コンテナーが利用できるプロセッサを増やします。コンテナーが利用できるプロセッサは明示的に指定しないでください。または、VM が利用できるプロセッサを減らします。

--------------------------

## ネットワーク

### ネットワーク コンパートメントによる分離とその影響
各コンテナーでは、分離するためにネットワークのコンパートメントを使用します。 指定されたコンテナーに接続されたコンテナー ネットワーク アダプター (エンドポイント) はすべて、同じネットワーク コンパートメント内に置かれます。 使用するネットワーク モード (ドライバー) によっては、同じ IP アドレスまたはポートを使用して 2 つの異なるコンテナー エンドポイントにアクセスできない場合があります。 さらに、Windows ファイアウォール規則はコンパートメントまたはコンテナー対応ではないため、組み込まれたファイアウォール規則はエンドポイントの種類に関係なく、コンテナー ホスト上のすべてのコンテナーに適用されます。

*** 透過ネットワーク ***


***NAT ネットワーク*** エンドポイントごとに適用される NAT ポート転送規則を使用して単一のコンテナーに接続された複数のエンドポイントを公開することができます。 これらの複数の転送規則を同じ内部ポートにマップする場合は、各転送規則で異なる外部ポート (コンテナー ホスト上の) を使用する必要があります。  ただし、前述のように、組み込またファイアウォール規則は、コンテナー ホスト全体にグローバルに適用されます。



### 静的 NAT マッピングは、Docker によるポート マッピングと競合する可能性があります。
Windows Server Technical Preview 5 以降、NAT 作成およびポート マッピングの規則は *ContainerNetwork* コマンドレットと docker コマンドに統合されています。 Windows Host Network Service (HNS) では、NAT を自動的に管理します。 ただし、外部のクライアントが、HNS によって作成された同じ NAT を使用して重複するポート マッピング ルールを作成する可能性があります。


ポート 80 における静的マッピングとの競合と、これが発生した場合に docker によって報告されるエラーの例を次に示します。
```
C:\Users\Administrator>docker run -it -p 80:80 windowsservercore cmd
docker: Error response from daemon: failed to create endpoint berserk_bassi on network nat: hnsCall failed in Win32: The remote procedure call failed. (0x6be).
```


***対応策*** 一般には、NAT は HNS によって管理されるため、これが発生する可能性は極めて低くなります。 ポート転送/マッピング規則はすべて、`docker run -p <external>:<internal>` または ContainerNetworkAdapterStaticMapping を使用して作成する必要があります。 ただし、HNS によってマッピングが自動的にクリーンアップされない場合、PowerShell を使用してポート マッピングを削除することで、このエラーが解決される場合があります。 これにより、上記の例で起こったポート 80 の競合が除去されます。
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows のコンテナーには、ip アドレスがされません。
DHCP VM スイッチで Windows コンテナーに接続する場合、コンテナー ホストが IP を受信し、コンテナーが受信しないという状況が発生します。

コンテナーは 169.254.***.*** APIPA IP アドレスを取得します。

**回避策:** これはカーネル共有の結果、副次的に発生します。  すべてのコンテナーに同じ MAC アドレスが効率よく与えられます。

コンテナー ホスト VM をホストしている物理ホストでの MAC アドレスのスプーフィングを有効にします。

これは PowerShell を使用して実行できます。
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
--------------------------

## アプリケーションの互換性

Windows コンテナーで動作するアプリケーションと動作しないアプリケーションについてはさまざまな問題があります。そのため、アプリケーションの互換性情報には[専用の記事](../reference/app_compat.md)を用意しました。

最も一般的な問題の一部はここにも記載します。

### コンテナー内でイベント ログは使用できません。

`get-eventlog Application` などの PowerShell コマンドやイベント ログを照会する API を実行すると、次のようなエラーが返されます。
```
get-eventlog : Cannot open log Application on machine .. Windows has not provided an error code.
At line:1 char:1
```

回避策として、Dockerfile に次のステップを追加します。 追加されたこのステップでビルドされるイメージでは、イベント ログが有効になります。
```
RUN powershell.exe -command Set-ItemProperty HKLM:\SYSTEM\CurrentControlSet\Control\WMI\Autologger\EventLog-Application Start 1
```


### localdb インスタンスの API メソッドの呼び出しで予想外のエラーが発生する
localdb インスタンスの API メソッドの呼び出しで予想外のエラーが発生する

### RTerm が機能しない
RTerm はインストールされますが、Windows Server コンテナー内で起動しません。

エラー:  
```
'C:\Program' is not recognized as an internal or external command,
operable program or batch file.
```


### コンテナー: Visual C++ ランタイム x64/x86 2015 がインストールされない

監視対象の動作: コンテナー内: 
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

これは重複除去フィルターの相互運用の問題です。 重複除去では、名前変更ターゲットが重複除去されたファイルかどうかを確認します。 "create it issues" が失敗し、`STATUS_IO_REPARSE_TAG_NOT_HANDLED` が表示されます。これは Windows Server コンテナーのフィルターが重複除去に対応していないことによります。


アプリケーションのコンテナー化に関する詳細については、[アプリケーションの互換性に関する記事](../reference/app_compat.md)を参照してください。

--------------------------


## Docker 管理

### 一部の Docker コマンドが機能しない
* Docker exec が Hyper-V コンテナーで失敗します。

この一覧に記載されていないコマンド エラーが発生した場合 (または、想定されているものとは異なるエラーが発生する場合) は、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)でお知らせください。

### 対話型 Docker セッションにコマンドを貼り付けるとき、50 文字に制限される
対話型 Docker セッションに貼り付けられるコマンドは 50 文字に制限されています。  
対話型 Docker セッションにコマンド ラインをコピーする場合、現在のところ、文字数が 50 に制限されています。 貼り付けられた文字列は単純に切り捨てられます。

これは仕様でなく、上限を上げる作業に取り組んでいます。

### net use のエラー
net use でユーザー名やパスワードの入力が求められず、システム エラー 1223 が返されます。

**回避策:**  
net use を実行するときはユーザー名とパスワードを指定します。

``` PowerShell
net use S: \\your\sources\here /User:shareuser [yourpassword]
``` 


--------------------------



## リモート デスクトップ 

Windows コンテナーを TP5 の RDP セッションで管理/操作できません。

--------------------------

### Nano Server コンテナー ホスト内のコンテナーの終了を "exit" で実行できない
Nano Server コンテナー ホスト内のコンテナーを終了する場合、"exit" を使用すると Nano Server コンテナー ホストから切断されるだけで、コンテナーは終了しません。

**回避策:** コンテナーを終了するには代わりに Exit-PSSession を使用します。

機能に関するご要望は[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)にお気軽にお寄せください。 


--------------------------


## ユーザーとドメイン

### ローカル ユーザー
コンテナーで Windows サービスやアプリケーションを実行するためにローカル ユーザー アカウントを作成して使用できます。


### ドメインのメンバーシップ
コンテナーは、Active Directory ドメインに参加できず、ドメイン ユーザー、サービス アカウント、またはコンピューター アカウントとしてサービスまたはアプリケーションを実行することはできません。 

コンテナーは、ほとんど環境に依存しない既知の一貫性のある状態ですばやく起動するように設計されています。 ドメインに参加し、ドメインのグループ ポリシー設定を適用すると、コンテナーの起動時間が長くなり、時間と共に機能が変化し、開発者間および展開間でイメージを移動または共有する機能が制限されます。

マイクロソフトは、サービスとアプリケーションが Active Directory を使用する方法、およびコンテナーへのそれらの展開の共通部分について、フィードバックを慎重に検討しています。 最善な方法の詳細な例をご存知の方は是非、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)でお知らせください。 

これらの種類のシナリオをサポートするソリューションを積極的に探しています。



<!--HONumber=Jun16_HO4-->


