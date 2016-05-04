



# 進行中の作業

問題の対処方法がここにない場合、またはご質問がございましたら、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)に投稿してください。

-----------------------


## 一般的な機能

### コンテナーとホスト ビルド番号の一致

Windows コンテナーには、ビルドとパッチのレベルに関してコンテナー ホストに一致するオペレーティング システム イメージが必要です。 一致しない場合、コンテナーまたはホストが不安定になったり、予想外の動作が発生したりします。

Windows コンテナー ホストに更新プログラムをインストールする場合、コンテナー ベース OS イメージを更新し、適合する更新プログラムを提供する必要があります。


**回避策:**   
ダウンロードし、コンテナーのホストの OS バージョンとパッチ レベルが一致するコンテナーの基本イメージをインストールします。

### 既定のファイアウォール動作

コンテナー ホストと各コンテナーから構成される環境では、コンテナー ホストにのみファイアウォールが与えられます。 コンテナー ホストで構成されるすべてのファイアウォール ルールがそのすべてのコンテナーに反映されます。

### Windows コンテナーの起動が遅い

コンテナーの起動に 30 秒以上かかる場合、ウイルス スキャンが重複して実行されている可能性があります。

Windows Defender は、おそらくファイル コンテナー内のイメージが、OS のバイナリ ファイルとファイルのすべてをコンテナーの OS イメージに含めてを不必要にスキャンなどの多くのマルウェア対策ソリューションです。 これは、新しいコンテナーを作成して、以前スキャンされていない新しいファイルのようになります「コンテナーのファイル」のすべてのマルウェアの観点からずっとときに発生します。 したがって、コンテナー内のプロセスが、これらのファイルを読み取るしようとしています。 マルウェア対策コンポーネント回目のスキャンにファイルへのアクセスを許可する前にします。 実際にはこれらのファイルは、コンテナーのイメージは、インポートまたはサーバーに取得したときに既にスキャンされていました。 将来のプレビューでは、新しいインフラストラクチャが置かれ、Windows Defender などのマルウェア対策ソフトウェアは前述の状況を認識し、適宜動作し、重複スキャンを回避します。

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


## ネットワーキング

### ネットワーク コンパートメントの制限

このリリースでは、1 つのコンテナーにつき 1 つのネットワーク コンパートメントを利用できます。 したがって、複数のネットワーク アダプターにコンテナーがある場合は、それぞれのアダプターで、同じネットワーク ポートをアクセスすることはできません (使用や 192.168.0.2:80 など、同じコンテナーに属する)。

**回避策: **  
複数のエンドポイントは、コンテナーによって公開される必要がある場合、は、NAT のポートのマッピングを使用します。


### 静的 NAT マッピングは、Docker によるポート マッピングと競合する可能性があります。

Windows PowerShell を使用してコンテナーを作成し、静的 NAT マッピングを追加する場合、`docker -p &lt;src&gt;:&lt;dst&gt;` を使用してコンテナーを開始する前に、それらを削除しておかないと、競合を招く可能性があります。

ポート 80 における静的マッピングとの競合の例を次に示します
```
PS C:\IISDemo> Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress
 172.16.0.2 -InternalPort 80 -ExternalPort 80


StaticMappingID               : 1
NatName                       : ContainerNat
Protocol                      : TCP
RemoteExternalIPAddressPrefix : 0.0.0.0/0
ExternalIPAddress             : 0.0.0.0
ExternalPort                  : 80
InternalIPAddress             : 172.16.0.2
InternalPort                  : 80
InternalRoutingDomainId       : {00000000-0000-0000-0000-000000000000}
Active                        : True



PS C:\IISDemo> docker run -it -p 80:80 microsoft/iis cmd
docker: Error response from daemon: Cannot start container 30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66: 
HCSShim::CreateComputeSystem - Win32 API call returned error r1=2147942452 err=You were not connected because a 
duplicate name exists on the network. If joining a domain, go to System in Control Panel to change the computer name
 and try again. If joining a workgroup, choose another workgroup name. 
 id=30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66 configuration= {"SystemType":"Container",
 "Name":"30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66","Owner":"docker","IsDummy":false,
 "VolumePath":"\\\\?\\Volume{4b239270-c94f-11e5-a4c6-00155d016f0a}","Devices":[{"DeviceType":"Network","Connection":
 {"NetworkName":"Virtual Switch","EnableNat":false,"Nat":{"Name":"ContainerNAT","PortBindings":[{"Protocol":"TCP",
 InternalPort":80,"ExternalPort":80}]}},"Settings":null}],"IgnoreFlushesDuringBoot":true,
 "LayerFolderPath":"C:\\ProgramData\\docker\\windowsfilter\\30b17cbe85539f08282340cc01f2797b42517924a70c8133f9d8db83707a2c66",
 "Layers":[{"ID":"4b91d267-ecbc-53fa-8392-62ac73812c7b","Path":"C:\\ProgramData\\docker\\windowsfilter\\39b8f98ccaf1ed6ae267fa3e98edcfe5e8e0d5414c306f6c6bb1740816e536fb"},
 {"ID":"ff42c322-58f2-5dbe-86a0-8104fcb55c2a",
"Path":"C:\\ProgramData\\docker\\windowsfilter\\6a182c7eba7e87f917f4806f53b2a7827d2ff0c8a22d200706cd279025f830f5"},
{"ID":"84ea5d62-64ed-574d-a0b6-2d19ec831a27",
"Path":"C:\\ProgramData\\Microsoft\\Windows\\Images\\CN=Microsoft_WindowsServerCore_10.0.10586.0"}],
"HostName":"30b17cbe8553","MappedDirectories":[],"SandboxPath":"","HvPartition":false}.
```


***軽減策***
PowerShell を使用してポート マッピングを削除すると、これが解決する場合があります。 これにより、上記の例で起こったポート 80 の競合が除去されます。
```powershell
Get-NetNatStaticMapping | ? ExternalPort -eq 80 | Remove-NetNatStaticMapping
```


### Windows のコンテナーには、ip アドレスがされません。

DHCP VM スイッチで Windows コンテナーに接続する場合、コンテナー ホストが IP を受信し、コンテナーが受信しないという状況が発生します。

コンテナーは 169.254.***.*** APIPA IP アドレスを取得します。

*** 回避策:***
これはカーネル共有の結果、副次的に発生します。 すべてのコンテナーに同じ MAC アドレスが効率よく与えられます。

コンテナー ホストで MAC アドレスのスプーフィングを有効にします。

これは PowerShell を使用して実行できます。
```
Get-VMNetworkAdapter -VMName "[YourVMNameHere]"  | Set-VMNetworkAdapter -MacAddressSpoofing On
```
### HTTPS と TLS はサポートされません。

Windows Server コンテナーおよび Hyper-V コンテナーは、HTTPS も TLS もサポートしません。 将来はこれを使用できるようにする作業を行っています。

--------------------------


## アプリケーションの互換性

Windows コンテナーで動作するアプリケーションと動作しないアプリケーションについてはさまざまな問題があります。そのため、アプリケーションの互換性情報には[専用の記事](../reference/app_compat.md)を用意しました。

最も一般的な問題の一部はここにも記載します。

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

監視された動作:
1 つのコンテナー内:
```
C:\build\vcredist_2015_x64.exe /q /norestart
C:\build>echo %errorlevel%
0
C:\build>wmic product get
No Instance(s) Available.
```

これは重複除去フィルターの相互運用の問題です。 重複除去では、名前変更ターゲットが重複除去されたファイルかどうかを確認します。 "create it issues" が失敗し、`STATUS_IO_REPARSE_TAG_NOT_HANDLED` が表示されます。これは Windows Server コンテナーのフィルターは重複除去に対応していないためです。


アプリケーションのコンテナー化に関する詳細については、[アプリケーションの互換性に関する記事](../reference/app_compat.md)を参照してください。

--------------------------



## Docker 管理

### Docker クライアントが安全ではない

このプレリリースでは、Docker 通信はパブリックとなります (探す場所がわかっている場合)。

### 一部の Docker コマンドが機能しない

* Docker exec が Hyper-V コンテナーで失敗します。
* DockerHub に関連するコマンドは現在のところ利用できません。

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

Windows コンテナーを TP4 の RDP セッションで管理/操作できません。

--------------------------


## PowerShell 管理

### 一部の *-PSSession には containerid 引数がない

これは正しいです。 将来、cimsession の完全なサポートを予定しています。

### Nano Server コンテナー ホスト内のコンテナーの終了を "exit" で実行できない

Nano Server コンテナー ホスト内のコンテナーを終了する場合、"exit" を使用すると Nano Server コンテナー ホストから切断されるだけで、コンテナーは終了しません。

**回避策:**
コンテナーを終了するには代わりに Exit-PSSession を使用します。

機能に関するご要望は[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)にお気軽にお寄せください。


--------------------------



## ユーザーとドメイン

### ローカル ユーザー

コンテナーで Windows サービスやアプリケーションを実行するためにローカル ユーザー アカウントを作成して使用できます。


### ドメインのメンバーシップ

コンテナーは、Active Directory ドメインに参加できず、ドメイン ユーザー、サービス アカウント、またはコンピューター アカウントとしてサービスまたはアプリケーションを実行することはできません。

コンテナーは、ほとんど環境に依存しない既知の一貫性のある状態ですばやく起動するように設計されています。 ドメインに参加し、ドメインのグループ ポリシー設定を適用すると、コンテナーの起動時間が長くなり、時間と共に機能が変化し、開発者間および展開間でイメージを移動または共有する機能が制限されます。

マイクロソフトは、サービスとアプリケーションが Active Directory を使用する方法、およびコンテナーへのそれらの展開の共通部分について、フィードバックを慎重に検討しています。最善な方法の詳細な例をご存知の方は是非、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)でお知らせください。

これらの種類のシナリオをサポートするソリューションを積極的に探しています。






<!--HONumber=Feb16_HO4-->


