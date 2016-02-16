# コンテナー共有フォルダー

**この記事は暫定的な内容であり、変更される可能性があります。**

共有フォルダーにより、コンテナー ホストとコンテナー間でデータを共有できます。 共有フォルダーが作成されていると、コンテナー内で共有フォルダーを使用できるようになります。 ホストから共有フォルダーに配置されたすべてのデータを、コンテナー内で使用できます。 コンテナー内から共有フォルダーに配置されたすべてのデータを、ホストで使用できまます。 ホスト上の単一のフォルダーを多くのコンテナーで共有でき、この構成では、実行中のコンテナー間でデータを共有できます。

## データの管理 - PowerShell

### 共有フォルダーの作成

共有フォルダーを作成するには、`Add-ContainerSharedFolder` コマンドを使用します。 下の例では、コンテナーにディレクトリ `c:\shared_data` を作成し、それがホスト上のディレクトリ `c:\data_source` にマップされます。

> 共有フォルダーを追加する場合は、コンテナーが停止状態である必要があります。

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\data_source -DestinationPath c:\shared_data

ContainerName SourcePath       DestinationPath AccessMode
------------- ----------       --------------- ----------
DEMO          c:\data_source   c:\shared_data  ReadWrite
```

### 読み取り専用共有フォルダー

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName DEMO -SourcePath c:\sf1 -DestinationPath c:\sf2 -AccessMode ReadOnly

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\sf1     c:\sf2          ReadOnly
```

### 共有フォルダーの一覧表示

特定のコンテナーの共有フォルダーの一覧を表示するには、`Get-ContainerSharedFolder` コマンドを使用します。

```powershell
PS C:\> Get-ContainerSharedFolder -ContainerName DEMO2

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
DEMO         c:\source  c:\source       ReadWrite
```

### 共有フォルダーの変更

既存の共有フォルダーの構成を変更するには、`Set-ContainerSharedFolder` コマンドを使用します。

```powershell
PS C:\> Set-ContainerSharedFolder -ContainerName SFRO -SourcePath c:\sf1 -DestinationPath c:\sf1
```

### 共有フォルダーの削除

共有フォルダーを削除するには、`Remove-ContainerSharedFolder` コマンドを使用します。

> 共有フォルダーを作成する場合、コンテナーは停止状態である必要があります。

```powershell
PS C:\> Remove-ContainerSharedFolder -ContainerName DEMO2 -SourcePath c:\source -DestinationPath c:\source
```
## データの管理 - Docker

### ボリュームのマウント

Docker を使用して Windows コンテナーを管理する場合、`-v` オプションを使用して、ボリュームをマウントできます。

下の例では、マウント元のフォルダーは c:\source で、マウント先のフォルダーは c:\destination です。

```powershell
PS C:\> docker run -it -v c:\source:c:\destination 1f62aaf73140 cmd
```

Docker によるコンテナー内のデータの管理の詳細については、[Docker.com の Docker ボリューム](https://docs.docker.com/userguide/dockervolumes/)に関するドキュメントを参照してください。

## ビデオ チュートリアル

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-3-Shared-Folders/player" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>



<!--HONumber=Feb16_HO1-->
