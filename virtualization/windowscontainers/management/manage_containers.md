# Windows Server コンテナー管理

**この記事は暫定的な内容であり、変更される可能性があります。**

コンテナーのライフ サイクルには、コンテナーの開始、停止、削除などの操作が含まれます。 これらの操作を実行する際に、コンテナー イメージの一覧を取得し、コンテナーのネットワークを管理して、コンテナー リソースを制限する必要がある場合もあります。 このドキュメントでは、PowerShell を使用した基本的なコンテナーの管理タスクについて説明します。

Docker による Windows コンテナーの管理に関するドキュメントについては、Docker の[コンテナーの操作](https://docs.docker.com/userguide/usingdocker/)に関するドキュメントを参照してください。

## PowerShell

### コンテナーの作成

新しいコンテナーを作成する場合、コンテナー ベースとして使用するコンテナー イメージの名前が必要です。 これは、`Get-ContainerImage` コマンドを使用して見つけることができます。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version         IsOSImage
----              ---------    -------         ---------
NanoServer        CN=Microsoft 10.0.10584.1000 True
WindowsServerCore CN=Microsoft 10.0.10584.1000 True
```

新しいコンテナーを作成するには、`New-Container` コマンドを使用します。 `- ContainerComputerName` パラメーターを使用して、コンテナーに NetBIOS 名を指定することもできます。

```powershell
PS C:\> New-Container -ContainerImageName WindowsServerCore -Name demo -ContainerComputerName demo

Name State Uptime   ParentImageName
---- ----- ------   ---------------
demo  Off   00:00:00 WindowsServerCore
```

コンテナーが作成されたら、コンテナーにネットワーク アダプターを追加します。

```powershell
PS C:\> Add-ContainerNetworkAdapter -ContainerName TST
```

コンテナーのネットワーク アダプターを仮想スイッチに接続するには、スイッチ名が必要です。 `Get-VMSwitch` を使用すると、仮想スイッチの一覧が返されます。

```powershell
PS C:\> Get-VMSwitch

Name SwitchType NetAdapterInterfaceDescription
---- ---------- ------------------------------
DHCP External   Microsoft Hyper-V Network Adapter
NAT  NAT
```

`Connect-ContainerNetworkAdapter` を使用して、ネットワーク アダプターを仮想スイッチに接続します。 **注:** この操作は、コンテナーの作成時に、–SwitchName パラメーターを使用して実行することもできます。

```powershell
PS C:\> Connect-ContainerNetworkAdapter -ContainerName TST -SwitchName NAT
```

### コンテナーの開始

コンテナーを開始するには、そのコンテナーを表す PowerShell オブジェクトを列挙します。 これを行うには、`Get-Container` の出力を PowerShell 変数に配置します。

```powershell
PS C:\> $container = Get-Container -Name TST
```

このデータは、コンテナーを開始する `Start-Container` コマンドで使用できます。

```powershell
PS C:\> Start-Container $container
```

次のスクリプトは、ホスト上のすべてのコンテナーを開始します。

```powershell
PS C:\> Get-Container | Start-Container
```

### コンテナーとの接続

PowerShell Direct を使用して、コンテナーに接続できます。 これは、ソフトウェアのインストール、プロセスの起動、コンテナーのトラブルシューティングなどのタスクを手動で実行する必要がある場合に役立つことがあります。 PowerShell Direct が使用されるため、ネットワーク構成に関係なく、コンテナーと PowerShell セッションを作成できます。 PowerShell Direct の詳細については、[PowerShell Direct のブログ](http://blogs.technet.com/b/virtualization/archive/2015/05/14/powershell-direct-running-powershell-inside-a-virtual-machine-from-the-hyper-v-host.aspx)を参照してください。

コンテナーとの対話型セッションを作成するには、`Enter-PSSession` コマンドを使用します。

 ```powershell
PS C:\> Enter-PSSession -ContainerName TST –RunAsAdministrator
 ```

リモート PowerShell セッションが作成されると、シェル プロンプトがコンテナー名を反映するように変更されます。

```powershell
[TST]: PS C:\>
```

永続的な PowerShell セッションを作成せずに、コンテナーに対してコマンドを実行することもできます。 それには、`Invoke-Command` を実行します。

次の例では、コンテナー内に 'Application' という名前のフォルダーを作成します。

```powershell

PS C:\> Invoke-Command -ContainerName TST -ScriptBlock {New-Item -ItemType Directory -Path c:\application }

Directory: C:\
Mode                LastWriteTime         Length Name                                                 PSComputerName
----                -------------         ------ ----                                                 --------------
d-----       10/28/2015   3:31 PM                application                                          TST
```

### コンテナーの停止

コンテナーを停止するには、そのコンテナーを表す PowerShell オブジェクトが必要です。 これを行うには、`Get-Container` の出力を PowerShell 変数に配置します。

```powershell
PS C:\> $container = Get-Container -Name TST
```

これは、コンテナーを停止する `Stop-Container` コマンドで使用できます。

```powershell
PS C:\> Stop-Container $container
```

次は、ホスト上のすべてのコンテナーを停止します。

```powershell
PS C:\> Get-Container | Stop-Container
```

### コンテナーの削除

コンテナーが不要になった場合は、削除できます。 コンテナーを削除するには、停止状態にする必要があり、コンテナーを表す PowerShell オブジェクトを作成する必要かあります。

```powershell
PS C:\> $container = Get-Container -Name TST
```

コンテナーを削除するには、`Remove-Container` コマンドを使用します。

```powershell
PS C:\> Remove-Container $container -Force
```

次は、ホスト上のすべてのコンテナーを削除します。

```powershell
PS C:\> Get-Container | Remove-Container -Force
```

## Docker

### コンテナーの作成

Docker によってコンテナーを作成するには、`docker run` を使用します。

```powershell
PS C:\> docker run -p 80:80 windowsservercoreiis
```

Docker run コマンドの詳細については、Docker run リファレンス (https://docs.docker.com/engine/reference/run/) を参照してください。

### コンテナーの停止

Docker によってコンテナーを停止するには、`docker stop` コマンドを使用します。

```powershell
PS C:\> docker stop tender_panini

tender_panini
```

この例では、Docker によって実行中のすべてのコンテナーを停止します。

```powershell
PS C:\> docker stop $(docker ps -q)

fd9a978faac8
b51e4be8132e
```

### コンテナーの削除

Docker によってコンテナーを削除するには、`docker rm` コマンドを使用します。

```powershell
PS C:\> docker rm prickly_pike

prickly_pike
```

Docker によってすべてのコンテナーを削除するには、次を実行します。

```powershell
PS C:\> docker rm $(docker ps -a -q)

dc3e282c064d
2230b0433370
```

Docker rm コマンドの詳細については、[Docker rm リファレンス](https://docs.docker.com/engine/reference/commandline/rm/)を参照してください。




<!--HONumber=Feb16_HO1-->
