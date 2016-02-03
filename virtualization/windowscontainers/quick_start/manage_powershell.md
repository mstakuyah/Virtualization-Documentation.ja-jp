# Windows コンテナー クイック スタート -PowerShell

Windows コンテナーを使用すると、1 つのコンピューター システムに多数の独立したアプリケーションを短時間でデプロイできます。 このクイック スタートでは、PowerShell を使用した Windows Server と Hyper-V 両方のコンテナーのデプロイメントと管理について説明します。 この演習では、Windows Server と Hyper-V コンテナーの両方で動作するとても単純な "hello world" アプリケーションを 1 から構築します。 この処理中、コンテナー イメージを作成し、コンテナー共有フォルダーを操作し、コンテナー ライフサイクルを管理します。 完了すると、Windows コンテナーのデプロイメントと管理の基本について理解できます。

このチュートリアルでは、Windows Server コンテナーと Hyper-V コンテナーの両方について説明します。 コンテナーの種類によって基本的な必要条件は異なります。 Windows コンテナー ドキュメントには、コンテナー ホストを簡単にデプロイする手順が記載されています。 Windows コンテナーを初めて使用するときは、これが最も簡単な方法です。 コンテナー ホストをお持ちでない場合は、「[Container Host Deployment Quick Start (コンテナー ホストの展開のクイック スタート)](./container_setup.md)」を参照してください。

各演習には、次のアイテムが必要です。

**Windows サーバー コンテナー:**

- オンプレミスまたは Azure で Windows Server 2016 Core を実行している Windows コンテナー ホスト。

**Hyper-V コンテナー:**

- 仮想化の入れ子に対応した Windows コンテナー ホスト。
- Windows Server 2016 メディア - [ダウンロード](https://aka.ms/tp4/serveriso)。

>Microsoft Azure は、Hyper-V コンテナーをサポートしていません。 Hyper-V の演習を完了するには、オンプレミスのコンテナー ホストが必要です。

## Windows Server コンテナー

Windows Server コンテナーは、アプリケーションとホスト プロセスを実行できる、独立した、ポータブルでリソースが制御された運用環境を提供します。 Windows Server コンテナーを使用すると、プロセスと名前空間を分離することで、コンテナーとホスト間、ホストで実行されているコンテナー間を分離できます。

### コンテナーの作成

TP4 の時点で、Windows Server 2016 または Windows Server 2016 Core で実行されている Windows Server コンテナーには、Windows Server 2016 Core OS イメージが必要です。

「`powershell`」と入力して PowerShell セッションを開始します。

```powershell
C:\> powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\>
```

Windows Server Core OS イメージがインストールされていることを確認するには、`Get-ContainerImage` コマンドを使用します。 複数の OS イメージが表示される場合もありますが、問題ありません。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Windows Server コンテナーを作成するには、`New-Container` コマンドを使用します。 次の例では、`WindowsServerCore` OS イメージから `TP4Demo` というコンテナーを作成し、そのコンテナーを `Virtual Switch` という VM スイッチに接続します。

```powershell
PS C:\> New-Container -Name TP4Demo -ContainerImageName WindowsServerCore -SwitchName "Virtual Switch"

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

既存のコンテナーを視覚的に確認するには、`Get-Container` コマンドを使用します。

```powershell
PS C:\> Get-Container

Name    State Uptime   ParentImageName
----    ----- ------   ---------------
TP4Demo Off   00:00:00 WindowsServerCore
```

`Start-Container` コマンドを使用してコンテナーを開始します。

```powershell
PS C:\> Start-Container -Name TP4Demo
```

`Enter-PSSession` コマンドを使用してコンテナーに接続します。 コンテナーで PowerShell セッションが作成されると、PowerShell のプロンプトが、コンテナー名を反映するように変わります。

```powershell
PS C:\> Enter-PSSession -ContainerName TP4Demo -RunAsAdministrator

[TP4Demo]: PS C:\Windows\system32>
```

### IIS イメージの作成

コンテナーを変更できるようになったら、変更をキャプチャして新しいコンテナー イメージを作成します。 この例では、IIS がインストールされています。

コンテナーに IIS の役割をインストールするには、`Install-WindowsFeature` コマンドを使用します。

```powershell
[TP4Demo]: PS C:\> Install-WindowsFeature web-server

Success Restart Needed Exit Code      Feature Result
------- -------------- ---------      --------------
True    No             Success        {Common HTTP Features, Default Document, D...
```

IIS のインストールが完了したら、「`exit`」と入力してコンテナーを終了します。 その結果、PowerShell セッションはコンテナー ホストのセッションに戻ります。

```powershell
[TP4Demo]: PS C:\> exit
PS C:\>
```

最後に、`Stop-Container` コマンドを使用してコンテナーを停止します。

```powershell
PS C:\> Stop-Container -Name TP4Demo
```

このコンテナーの状態は、新しいコンテナー イメージにキャプチャできます。 この処理には `New-ContainerImage` コマンドを使用します。

この例では、名前が `WindowsServerCoreIIS`、発行元が `Demo`、バージョンが `1.0` の新しいコンテナー イメージを作成します。

```powershell
PS C:\> New-ContainerImage -ContainerName TP4Demo -Name WindowsServerCoreIIS -Publisher Demo -Version 1.0

Name                 Publisher Version IsOSImage
----                 --------- ------- ---------
WindowsServerCoreIIS CN=Demo   1.0.0.0 False
```

これで、このコンテナーは新しいイメージにキャプチャされたため、必要なくなります。 `Remove-Container` コマンドを使用してコンテナーを削除することができます。

```powershell
PS C:\> Remove-Container -Name TP4Demo -Force
```


### IIS コンテナーの作成

今度は `WindowsServerCoreIIS` コンテナー イメージから新しいコンテナーを作成します。

```powershell
PS C:\> New-Container -Name IIS -ContainerImageName WindowsServerCoreIIS -SwitchName "Virtual Switch"

Name State Uptime   ParentImageName
---- ----- ------   ---------------
IIS  Off   00:00:00 WindowsServerCoreIIS
```
コンテナーを起動します。

```powershell
PS C:\> Start-Container -Name IIS
```

### ネットワークの構成

Windows コンテナー クイック スタートの既定のネットワーク構成では、ネットワーク アドレス変換 (NAT) が構成された仮想スイッチにコンテナーを接続します。 そのため、コンテナー内で実行されるアプリケーションに接続するには、コンテナー ホスト上のポートをコンテナーのポートにマッピングする必要があります。 コンテナーのネットワークの詳細については、「[コンテナーのネットワーク](../management/container_networking.md)」を参照してください。

この演習の Web サイトは、コンテナー内で実行されている IIS でホストされています。 ポート 80 で Web サイトにアクセスするには、コンテナー ホスト IP アドレスのポート 80 をコンテナー IP アドレスのポート 80 にマッピングします。

コンテナーの IP アドレスを返す次のコマンドを実行します。

```powershell
PS C:\> Invoke-Command -ContainerName IIS {ipconfig}

Windows IP Configuration


Ethernet adapter vEthernet (Virtual Switch-7570F6B1-E1CA-41F1-B47D-F3CA73121654-0):

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::ed23:c1c6:310a:5c10%16
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

NAT ポート マッピングを作成するには、`Add-NetNatStaticMapping` コマンドを使用します。 次の例では、既存のポート マッピング規則を確認し、存在しない場合は作成します。 `-InternalIPAddress` は、コンテナーの IP アドレスと一致する必要があります。

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```

ポート マッピングを作成した場合、構成したポートの受信ファイアウォール規則も構成する必要があります。 ポート 80 で構成する場合は、次のスクリプトを実行します。 80 以外の外部ポートの NAT 規則を作成した場合は、それに合わせてファイアウォール規則を作成する必要があります。

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

Azure で作業していて、ネットワーク セキュリティ グループをまだ作成していない場合は、新規作成する必要があります。 ネットワーク セキュリティ グループの詳細については、「[ネットワーク セキュリティ グループ (NSG) について](https://azure.microsoft.com/en-us/documentation/articles/virtual-networks-nsg/)」を参照してください。

### アプリケーションの作成

これまでの手順で、IIS イメージからコンテナーを作成し、ネットワークを構成しました。次はブラウザーを開き、コンテナー ホストの IP アドレスにアクセスします。 次のような IIS スプラッシュ画面が表示されます。

![](media/iis1.png)

IIS インスタンスが実行されていることを確認したら、"Hello World" アプリケーションを作成し、IIS インスタンスでホストできるようになります。 そのために、コンテナーで PowerShell セッションを作成します。

```powershell
PS C:\> Enter-PSSession -ContainerName IIS -RunAsAdministrator
[IIS]: PS C:\Windows\system32>
```

次のコマンドを実行して、IIS スプラッシュ画面を削除します。

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
次のコマンドを実行して、既定の IIS サイトを新しい静的サイトに置き換えます。

```powershell
[IIS]: PS C:\> "Hello World From a Windows Server Container" > C:\inetpub\wwwroot\index.html
```

再びコンテナー ホストの IP アドレスにアクセスすると、"Hellow World" アプリケーションが表示されます。 注: 更新されたアプリケーションを表示するには、既存のブラウザー接続を閉じたり、ブラウザー キャッシュをクリアしたりする必要がある場合があります。

![](media/HWWINServer.png)

リモート コンテナー セッションを終了します。

```powershell
[IIS]: PS C:\> exit
PS C:\>
```

### コンテナーの削除

コンテナーを削除する前に、コンテナーを停止しておく必要があります。

```powershell
PS C:\> Stop-Container -Name IIS
```

コンテナーを停止したら、`Remove-Container` コマンドを使用して削除することができます。

```powershell
PS C:\> Remove-Container -Name IIS -Force
```

最後に、`Remove-ContainerImage` コマンドを使用して、コンテナー イメージを削除できます。

```powershell
PS C:\> Remove-ContainerImage -Name WindowsServerCoreIIS -Force
```

## Hyper-V コンテナー

Hyper-V コンテナーは、Windows Server コンテナー上に分離したレイヤーを追加します。 各 Hyper-V コンテナーは、高度に最適化された仮想マシン内に作成されます。 Windows Server コンテナーは、コンテナー ホストや、そのホスト上で実行されているその他すべての Windows Server コンテナーとカーネルを共有していますが、Hyper-V コンテナーは他のコンテナーと完全に独立しています。 Hyper-V コンテナーは、 Windows Server コンテナーと同じ方法で作成され、管理されます。 Hyper-V コンテナーの詳細については、「[Managing Hyper-V Containers (Hyper-V コンテナーの管理)](../management/hyperv_container.md)」を参照してください。

>Microsoft Azure は、Hyper-V コンテナーをサポートしていません。 Hyper-V コンテナーの演習を完了するには、オンプレミスのコンテナー ホストが必要です。

### コンテナーの作成

TP4 の時点で、Hyper-V コンテナーは Nano Server Core OS イメージを使用する必要があります。 Nano Server OS イメージがインストールされていることを確認するには、`Get-ContainerImage` コマンドを使用します。

```powershell
PS C:\> Get-ContainerImage

Name              Publisher    Version      IsOSImage
----              ---------    -------      ---------
NanoServer        CN=Microsoft 10.0.10586.0 True
WindowsServerCore CN=Microsoft 10.0.10586.0 True
```

Hyper-V コンテナーを作成するには、Hyper-V のランタイムを指定して `New-Container` コマンドを使用します。

```powershell
PS C:\> New-Container -Name HYPV -ContainerImageName NanoServer -SwitchName "Virtual Switch" -RuntimeType HyperV

Name State Uptime   ParentImageName
---- ----- ------   ---------------
HYPV Off   00:00:00 NanoServer
```

コンテナーを作成しても**起動しないでください**。

### 共有フォルダーの作成

共有フォルダーは、コンテナー ホストからコンテナーに対して直接公開されます。 共有フォルダーを作成すると、共有フォルダーに配置したすべてのファイルはコンテナーで使用できるようになります。 この例では、共有フォルダーを使用して、Nano Server IIS パッケージをコンテナーにコピーします。 これらのパッケージを使用して、IIS をインストールします。 共有フォルダーの詳細については、「[コンテナー共有フォルダー](../management/manage_data.md)」を参照してください。

コンテナー ホストに `c:\share\en-us` というディレクトリを作成します。

```powershell
S C:\> New-Item -Type Directory c:\share\en-us

    Directory: C:\share

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

`Add-ContainerSharedFolder` コマンドを使用して、新しいコンテナーに新しい共有フォルダーを作成します。

>共有フォルダーを作成するときは、コンテナーが停止した状態である必要があります。

```powershell
PS C:\> Add-ContainerSharedFolder -ContainerName HYPV -SourcePath c:\share -DestinationPath c:\iisinstall

ContainerName SourcePath DestinationPath AccessMode
------------- ---------- --------------- ----------
HYPV          c:\share   c:\iisinstall   ReadWrite
```

共有フォルダーが作成されたら、コンテナーを起動します。

```powershell
PS C:\> Start-Container -Name HYPV
```
`Enter-PSSession` コマンドを使用して、コンテナーとの PowerShell リモート セッションを作成します。

```powershell
PS C:\> Enter-PSSession -ContainerName HYPV -RunAsAdministrator
[HYPV]: PS C:\windows\system32\config\systemprofile\Documents>cd /
```
リモート セッションの場合、共有フォルダー `c:\iisinstall\en-us` は作成されていても、その中身は空です。

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:27 PM                en-us
```

### IIS イメージの作成

コンテナーは Nano Server OS イメージを実行しているため、IIS をインストールするには Nano Server IIS パッケージが必要です。 これらのパッケージは、Windows Server 2016 TP4 インストール メディアの `NanoServer\Packages` ディレクトリにあります。

`Microsoft-NanoServer-IIS-Package.cab` を `NanoServer\Packages` からコンテナー ホストの `c:\share` にコピーします。

`NanoServer\Packages\en-us\Microsoft-NanoServer-IIS-Package.cab` をコンテナー ホストの `c:\share\en-us` にコピーします。

c:\share フォルダーに unattend.xml という名前のファイルを作成し、次のテキストを unattend.xml ファイルにコピーします。

```powershell
<?xml version="1.0" encoding="utf-8"?>
<unattend xmlns="urn:schemas-microsoft-com:unattend">
    <servicing>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="neutral" />
            <source location="c:\iisinstall\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
        <package action="install">
            <assemblyIdentity name="Microsoft-NanoServer-IIS-Package" version="10.0.10586.0" processorArchitecture="amd64" publicKeyToken="31bf3856ad364e35" language="en-US" />
            <source location="c:\iisinstall\en-us\Microsoft-NanoServer-IIS-Package.cab" />
        </package>
    </servicing>
</unattend>
```

完了すると、コンテナー ホストの `c:\share` ディレクトリが次のように構成されます。

```
c:\share
|-- en-us
|    |-- Microsoft-NanoServer-IIS-Package.cab
|
|-- Microsoft-NanoServer-IIS-Package.cab
|-- unattend.xml
```

コンテナーのリモート セッションに戻ると、c:\iisinstall ディレクトリに IIS パッケージと unattended.xml ファイルが表示されます。

```powershell
[HYPV]: PS C:\> ls c:\iisinstall

    Directory: C:\iisinstall

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----       11/18/2015   5:32 PM                en-us
-a----       10/29/2015  11:51 PM        1922047 Microsoft-NanoServer-IIS-Package.cab
-a----       11/18/2015   5:31 PM            789 unattend.xml
```

次のコマンドを実行して IIS をインストールします。

```powershell
[HYPV]: PS C:\> dism /online /apply-unattend:c:\iisinstall\unattend.xml

Deployment Image Servicing and Management tool
Version: 10.0.10586.0

Image Version: 10.0.10586.0


[                           1.0%                           ]

[=====                      10.1%                          ]

[=====                      10.3%                          ]

[===============            26.2%                          ]
```

IIS のインストールが完了したら、次のコマンドを使用して手動で IIS を起動します。

```powershell
[HYPV]: PS C:\> Net start w3svc
The World Wide Web Publishing Service service is starting.
The World Wide Web Publishing Service service was started successfully.
```

コンテナー セッションを終了します。

```powershell
[HYPV]: PS C:\> exit
```

コンテナーを停止します。

```powershell
PS C:\> Stop-Container -Name HYPV
```

このコンテナーの状態は、新しいコンテナー イメージにキャプチャできます。

この例では、名前が `NanoServerIIS`、発行元が `Demo`、バージョンが `1.0` の新しいコンテナー イメージを作成します。

```powershell
PS C:\> New-ContainerImage -ContainerName HYPV -Name NanoServerIIS -Publisher Demo -Version 1.0

Name          Publisher Version IsOSImage
----          --------- ------- ---------
NanoServerIIS CN=Demo   1.0.0.0 False
```

### IIS コンテナーの作成

`New-Container` コマンドを使用して、IIS イメージから新しい Hyper-V コンテナーを作成します。

```powershell
PS C:\> New-Container -Name IISApp -ContainerImageName NanoServerIIS -SwitchName "Virtual Switch" -RuntimeType HyperV

Name   State Uptime   ParentImageName
----   ----- ------   ---------------
IISApp Off   00:00:00 NanoServerIIS
```

コンテナーを起動します。

```powershell
PS C:\> Start-Container -Name IISApp
```

### ネットワークの構成

Windows コンテナー クイック スタートの既定のネットワーク構成では、ネットワーク アドレス変換 (NAT) が構成された仮想スイッチにコンテナーを接続します。 そのため、コンテナー内で実行されるアプリケーションに接続するには、コンテナー ホスト上のポートをコンテナーのポートにマッピングする必要があります。

この演習の Web サイトは、コンテナー内で実行されている IIS でホストされています。 ポート 80 で Web サイトにアクセスするには、コンテナー ホスト IP アドレスのポート 80 をコンテナー IP アドレスのポート 80 にマッピングします。

コンテナーの IP アドレスを返す次のコマンドを実行します。

```powershell
PS C:\> Invoke-Command -ContainerName IISApp {ipconfig}

Windows IP Configuration


Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . : DNS
   Link-local IPv6 Address . . . . . : fe80::c574:5a5e:d5f5:18a0%4
   IPv4 Address. . . . . . . . . . . : 172.16.0.2
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 172.16.0.1
```

NAT ポート マッピングを作成するには、`Add-NetNatStaticMapping` コマンドを使用します。 次の例では、既存のポート マッピング規則を確認し、存在しない場合は作成します。 `-InternalIPAddress` は、コンテナーの IP アドレスと一致する必要があります。

```powershell
if (!(Get-NetNatStaticMapping | where {$_.ExternalPort -eq 80})) {
Add-NetNatStaticMapping -NatName "ContainerNat" -Protocol TCP -ExternalIPAddress 0.0.0.0 -InternalIPAddress 172.16.0.2 -InternalPort 80 -ExternalPort 80
}
```
また、コンテナー ホストでポート 80 を開く必要もあります。 80 以外の外部ポートの NAT 規則を作成した場合は、それに合わせてファイアウォール規則を作成する必要があります。

```powershell
if (!(Get-NetFirewallRule | where {$_.Name -eq "TCP80"})) {
    New-NetFirewallRule -Name "TCP80" -DisplayName "HTTP on TCP/80" -Protocol tcp -LocalPort 80 -Action Allow -Enabled True
}
```

### アプリケーションの作成

これまでの手順で、IIS イメージからコンテナーを作成し、ネットワークを構成しました。次はブラウザーを開き、コンテナー ホストの IP アドレスにアクセスします。IIS スプラッシュ画面が表示されます。

![](media/iis1.png)

IIS インスタンスが実行されていることを確認したら、"Hello World" アプリケーションを作成し、IIS インスタンスでホストできるようになります。 そのために、コンテナーで PowerShell セッションを作成します。

```powershell
PS C:\> Enter-PSSession -ContainerName IISApp -RunAsAdministrator
[IISApp]: PS C:\windows\system32\config\systemprofile\Documents>
```

次のコマンドを実行して、IIS スプラッシュ画面を削除します。

```powershell
[IIS]: PS C:\> del C:\inetpub\wwwroot\iisstart.htm
```
次のコマンドを実行して、既定の IIS サイトを新しい静的サイトに置き換えます。

```powershell
[IISApp]: PS C:\> "Hello World From a Hyper-V Container" > C:\inetpub\wwwroot\index.html
```

再びコンテナー ホストの IP アドレスにアクセスすると、"Hellow World" アプリケーションが表示されます。 注: 更新されたアプリケーションを表示するには、既存のブラウザー接続を閉じたり、ブラウザー キャッシュをクリアしたりする必要がある場合があります。

![](media/HWWINServer.png)

リモート コンテナー セッションを終了します。

```powershell
exit
```




<!--HONumber=Jan16_HO1-->
