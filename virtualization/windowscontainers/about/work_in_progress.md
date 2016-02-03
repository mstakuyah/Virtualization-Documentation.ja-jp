# 進行中の作業

問題の対処方法がここにない場合、またはご質問がございましたら、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)に投稿してください。

-----------------------


## 一般的な機能

### コンテナーとホスト ビルド番号の一致

Windows コンテナーには、ビルドとパッチのレベルに関してコンテナー ホストに一致するオペレーティング システム イメージが必要です。 一致しない場合、コンテナーまたはホストが不安定になったり、予想外の動作が発生したりします。

Windows コンテナー ホストに更新プログラムをインストールする場合、コンテナー ベース OS イメージを更新し、適合する更新プログラムを提供する必要があります。


**回避策:**   
コンテナー ホストの OS バージョンとパッチ レベルに一致するコンテナー ベース イメージをダウンロードし、インストールします。

### C:/ 以外のすべてのドライブがコンテナーに表示されます。

コンテナー ホストで利用できる C:/ 以外のドライブはすべて、新しく実行される Windows コンテナーに自動的にマッピングされます。

現時点では、一時的な回避策としてドライブが自動的にマッピングされ、コンテナーにマッピングするフォルダーを選択することはできません。

**回避策: **  
現在作業中です。 将来、フォルダーが共有されます。

### 既定のファイアウォール動作

コンテナー ホストと各コンテナーから構成される環境では、コンテナー ホストにのみファイアウォールが与えられます。 コンテナー ホストで構成されるすべてのファイアウォール ルールがそのすべてのコンテナーに反映されます。

### Windows コンテナーの起動が遅い

コンテナーの起動に 30 秒以上かかる場合、ウイルス スキャンが重複して実行されている可能性があります。

Windows Defender など、多くのマルウェア対策ソフトウェアで、コンテナー イメージ内のファイルが不必要にスキャンされている可能性があります。たとえば、コンテナー OS イメージの OS のバイナリやファイルなどです。 このような状況は、新しいコンテナーが作成されたとき、マルウェア対策の観点から、すべての「コンテナーのファイル」がまだスキャンされていない新しいファイルのように見える場合にも発生します。 コンテナー内のプロセスがそのようなファイルを読み取ろうとするとき、マルウェア対策コンポーネントは最初にファイルをスキャンしてからファイルへのアクセスを許可します。 実際には、このようなファイルは、コンテナー イメージがサーバーにインポートされたときに既にスキャンされています。 将来のプレビューでは、新しいインフラストラクチャが置かれ、Windows Defender などのマルウェア対策ソフトウェアは前述の状況を認識し、適宜動作し、重複スキャンを回避します。

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

このリリースでは、1 つのコンテナーにつき 1 つのネットワーク コンパートメントを利用できます。 つまり、コンテナーにネットワーク アダプターが複数ある場合、各アダプターで同じネットワーク ポートにアクセスすることはできません (例、192.168.0.1:80 と 192.168.0.2:80 は同じコンテナーに属します)。

**回避策: **  
複数のエンドポイントをコンテナーに公開する必要がある場合、NAT ポート マッピングを使用します。

### Windows コンテナーが IP を取得しない

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
対話型 Docker セッションにコマンド ラインをコピーする場合、現在のところ、文字数が 50 に制限されています。 貼り付けられた文字列は切り詰められます。

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

マイクロソフトは、サービスとアプリケーションが Active Directory を使用する方法、およびコンテナーへのそれらの展開の共通部分について、フィードバックを慎重に検討しています。最善な方法の詳細な例をご存知の方は是非、[フォーラム](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowscontainers)でお知らせください。これらの種類のシナリオをサポートするソリューションを積極的に探しています。




<!--HONumber=Jan16_HO3-->
