# コンテナー用 PowerShell

## Install-ContainerOSImage

**名前**      
Install-ContainerOSImage

**概要**  
Windows Server または Hyper-V コンテナーで使用するコンテナー OS イメージとして、特定の WIM をインストールします。


**構文**
``` PowerShell  
Install-ContainerOSImage [-WimPath] <String> [-Force] [< CommonParameters >]
```

**説明**  
WIM ファイルから、Windows Server および Hyper-V コンテナー機能の共有の中央イメージ ストアに基本イメージをインストールします。

**パラメーター**
``` PowerShell
    -WimPath <String>
        A path to the WIM file that will be installed.

        Required?                    true
        Position?                    1
        Default value
        Accept pipeline input?       false
        Accept wildcard characters?  false

    -Force [<SwitchParameter>]

        Required?                    false
        Position?                    named
        Default value                False
        Accept pipeline input?       false
        Accept wildcard characters?  false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**      
None

**出力**  
None

**エイリアス**  
None

-------------------------- 例 1 --------------------------

``` PowerShell
PS C:\>Install-ContainerOSImage c:\baseimage.wim
```

## Uninstall-ContainerOSImage

**名前**  
Uninstall-ContainerOSImage

**概要**  
以前にインストールされたコンテナー OS イメージを削除します。

**構文**
```PowerShell
Uninstall-ContainerOSImage [-ImageName] <string> [-Force]  [< CommonParameters >]

Uninstall-ContainerOSImage [-ContainerImage] <Object> [-Force]  [< CommonParameters >]
```

**パラメーター**
``` PowerShell
    -ContainerImage <Object>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByContainerImage
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ImageName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           ByName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
None


**出力**  
System.Object

**エイリアス**  
None

## Add-ContainerNetworkAdapter

**名前**  
Add-ContainerNetworkAdapter

**概要**  
新しいネットワーク アダプターを既存のコンテナーに追加します。

**構文**
``` PowerShell  
Add-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-SwitchName <string>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Add-ContainerNetworkAdapter [-Container] <Container[]> [-SwitchName <string>] [-Name <string>]
    [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```


**入力**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Connect-ContainerNetworkAdapter

**名前**  
Connect-ContainerNetworkAdapter

**概要**  
コンテナーのネットワーク アダプターを仮想スイッチに接続します。

**構文**
``` PowerShell
    Connect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-SwitchName] <string>
    [-Passthru] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    Connect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-SwitchName] <string> [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Object_SwitchName
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Object_SwitchName, Name_SwitchName
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Disconnect-ContainerNetworkAdapter

**名前**  
Disconnect-ContainerNetworkAdapter

**概要**  
仮想スイッチからコンテナーのネットワーク アダプターを切断します。

**構文**
``` PowerShell
    Disconnect-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string[]>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Disconnect-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Export-ContainerImage

**名前**  
Export-ContainerImage

**概要**  
ローカル ストアからコンテナー イメージをコピーします。

**構文**
``` PowerShell
    Export-ContainerImage [[-Name] <string>] [-Path] <string> [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Export-ContainerImage [-Image] <ContainerImage[]> [-Path] <string> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**エイリアス**  
None

## Get-Container

**名前**  
Get-Container

**概要**  
現在のシステム上のコンテナーを列挙します。

**構文**
``` PowerShell
    Get-Container [[-Name] <string[]>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]

    Get-Container [[-Id] <guid>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Id, Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name, Id
        Aliases                      None
        Dynamic?                     false

    -Id <guid>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue, ByPropertyName)
        Parameter set name           Id
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
System.String\[\]  
System.Guid


**出力**  
Microsoft.Containers.PowerShell.Objects.Container


**エイリアス**  
None

## Get-ContainerHost

**名前**  
Get-ContainerHost

**概要**  
コンテナー ホストのホスト オブジェクトを取得します。

**構文**
``` PowerShell
    Get-ContainerHost [[-ComputerName] <string[]>] [[-Credential] <pscredential[]>]  [<CommonParameters>]

    Get-ContainerHost [-CimSession] <CimSession[]>  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByPropertyName)
        Parameter set name           CimSession
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    1
        Accept pipeline input?       true (ByValue)
        Parameter set name           ComputerName
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Management.Infrastructure.CimSession\[\]  
System.String\[\]  
System.Management.Automation.PSCredential\[]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerHost


**エイリアス**  
None

## Get-ContainerImage

**名前**  
Get-ContainerImage

**概要**  
コンテナー ホスト上のコンテナー イメージを一覧表示します。

**構文**
``` PowerShell
Get-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-ChildOf <ContainerImage>]
[-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -ChildOf <ContainerImage>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
None


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**エイリアス**  
None

## Get-ContainerNetworkAdapter

**名前**  
Get-ContainerNetworkAdapter

**概要**  
コンテナーに関連付けられているネットワーク アダプターを一覧表示します。

**構文**
``` PowerShell
    Get-ContainerNetworkAdapter [-ContainerName] <string[]> [[-Name] <string>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>]  [<CommonParameters>]

    Get-ContainerNetworkAdapter [-Container] <Container[]> [[-Name] <string>]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Import-ContainerImage

**名前**  
Import-ContainerImage

**概要**  
別のコンピューターからエクスポートされたコンテナー イメージをインポートします。

**構文**
``` PowerShell
    Import-ContainerImage [-Path] <string> [-AsJob] [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
System.String


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**エイリアス**  
None

## Move-ContainerImageRepository

**名前**  
Move-ContainerImageRepository

**概要**  
コンテナー イメージの格納場所を変更します。 ローカル ディスク上の場所である必要があります。 システムにイメージが存在しない場合にのみ変更できます。

**構文**
``` PowerShell
    Move-ContainerImageRepository [-Path] <string> [-AsJob] [-Passthru] [-CimSession <CimSession[]>] [-ComputerName
    <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
None


**出力**  
Microsoft.HyperV.PowerShell.VMHost


**エイリアス**
None

## New-Container

**名前**  
New-Container

**概要**  
新しいグループを作成します。

**構文**
``` PowerShell
    New-Container [[-Name] <string>] -ContainerImageName <string> [-ContainerImagePublisher <string>]
    [-ContainerImageVersion <version>] [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-MemoryStartupBytes <long>] [-SwitchName <string>] [-Path <string>] [-AsJob] [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-Container [[-Name] <string>] -ContainerImage <ContainerImage> [-MemoryStartupBytes <long>] [-SwitchName
    <string>] [-Path <string>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -ContainerImage <ContainerImage>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -ContainerImageName <string>

        Required?                    true
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImagePublisher <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -ContainerImageVersion <version>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers
        Aliases                      None
        Dynamic?                     false

    -MemoryStartupBytes <long>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Image Identifiers, Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Path <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -SwitchName <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**出力**  
Microsoft.Containers.PowerShell.Objects.Container


**エイリアス**  
None

## New-ContainerImage

**名前**  
New-ContainerImage

**概要**  
既存のコンテナーから新しいコンテナー イメージを作成します。

**構文**
``` PowerShell
    New-ContainerImage [-ContainerName] <string> [-Name] <string> [-Publisher] <string> [-Version] <version>
    [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    New-ContainerImage [-Container] <Container> [-Name] <string> [-Publisher] <string> [-Version] <version> [-WhatIf]
    [-Confirm]  [<CommonParameters>]

    New-ContainerImage [-ContainerId] <guid> [-Name] <string> [-Publisher] <string> [-Version] <version> [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Id, Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerId <guid>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Id
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Id
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    true
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    true
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    true
        Position?                    3
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.Container


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**エイリアス**  
None

## Remove-Container

**名前**  
Remove-Container

**概要**  
システムから既存のコンテナーを削除します。

**構文**
``` PowerShell
    Remove-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-Container [-Container] <Container[]> [-Force] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.Container


**エイリアス**  
None

## Remove-ContainerImage

**名前**  
Remove-ContainerImage

**概要**  
コンテナー ホストからコンテナー イメージを削除します。

**構文**
``` PowerShell
    Remove-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession
    <CimSession[]>] [-ComputerName <string[]>] [-Credential <pscredential[]>] [-Force] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerImage [-Image] <ContainerImage> [-Force] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Force

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**出力**  
System.Object

**エイリアス**  
None

## Remove-ContainerNetworkAdapter

**名前**  
Remove-ContainerNetworkAdapter

**概要**  
コンテナーからネットワーク アダプターを削除します。

**構文**
``` PowerShell
    Remove-ContainerNetworkAdapter [-ContainerName] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter[]> [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]

    Remove-ContainerNetworkAdapter [-Container] <Container[]> [-Name <string>] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name, Container Object
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter\[\]  
System.String\[\]  
Microsoft.Containers.PowerShell.Objects.Container\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Set-ContainerNetworkAdapter

**名前**  
Set-ContainerNetworkAdapter

**概要**  
コンテナー内のネットワーク アダプターで MAC アドレスを設定します。

**構文**
``` PowerShell
    Set-ContainerNetworkAdapter [-ContainerName] <string> [-CimSession <CimSession[]>] [-ComputerName <string[]>]
    [-Credential <pscredential[]>] [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress <string>] [-Passthru]
    [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-NetworkAdapter] <ContainerNetworkAdapter> [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Set-ContainerNetworkAdapter [-Container] <Container> [-Name <string>] [-DynamicMacAddress] [-StaticMacAddress
    <string>] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -ContainerName <string>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Name
        Aliases                      None
        Dynamic?                     false

    -DynamicMacAddress

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -NetworkAdapter <ContainerNetworkAdapter>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           ResourceObject
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -StaticMacAddress <string>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           ResourceObject, Container Object, Container Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
System.String  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter  
Microsoft.Containers.PowerShell.Objects.Container


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerNetworkAdapter


**エイリアス**  
None

## Start-Container

**名前**  
Start-Container

**概要**  
コンテナーを開始します。

**構文**
``` PowerShell
    Start-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Start-Container [-Container] <Container[]> [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.Container


**エイリアス**  
None

## Stop-Container

**名前**  
Stop-Container

**概要**  
コンテナーを停止します。

**構文**
``` PowerShell
    Stop-Container [-Name] <string[]> [-CimSession <CimSession[]>] [-ComputerName <string[]>] [-Credential
    <pscredential[]>] [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Stop-Container [-Container] <Container[]> [-TurnOff] [-AsJob] [-Passthru] [-WhatIf] [-Confirm]
    [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Container <Container[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Object
        Aliases                      None
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Name <string[]>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Passthru

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -TurnOff

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.Container\[\]  
System.String\[\]


**出力**  
Microsoft.Containers.PowerShell.Objects.Container


**エイリアス**  
None

## Test-ContainerImage

**名前**  
Test-ContainerImage

**概要**  
コンテナー ホスト システム上のコンテナー イメージを検証します。

**構文**
``` PowerShell
    Test-ContainerImage [[-Name] <string>] [[-Publisher] <string>] [[-Version] <version>] [-CimSession <CimSession[]>]
    [-ComputerName <string[]>] [-Credential <pscredential[]>] [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]

    Test-ContainerImage [-Image] <ContainerImage> [-AsJob] [-WhatIf] [-Confirm]  [<CommonParameters>]
```

**パラメーター**
``` PowerShell
    -AsJob

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      None
        Dynamic?                     false

    -CimSession <CimSession[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -ComputerName <string[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Confirm

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      cf
        Dynamic?                     false

    -Credential <pscredential[]>

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Image <ContainerImage>

        Required?                    true
        Position?                    0
        Accept pipeline input?       true (ByValue)
        Parameter set name           Container Image Object
        Aliases                      None
        Dynamic?                     false

    -Name <string>

        Required?                    false
        Position?                    0
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Publisher <string>

        Required?                    false
        Position?                    1
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -Version <version>

        Required?                    false
        Position?                    2
        Accept pipeline input?       false
        Parameter set name           Name
        Aliases                      None
        Dynamic?                     false

    -WhatIf

        Required?                    false
        Position?                    Named
        Accept pipeline input?       false
        Parameter set name           (All)
        Aliases                      wi
        Dynamic?                     false

    <CommonParameters>
        This cmdlet supports the common parameters: Verbose, Debug,
        ErrorAction, ErrorVariable, WarningAction, WarningVariable,
        OutBuffer, PipelineVariable, and OutVariable. For more information, see
        about_CommonParameters (http://go.microsoft.com/fwlink/?LinkID=113216).
```

**入力**  
Microsoft.Containers.PowerShell.Objects.ContainerImage


**出力**  
Microsoft.Containers.PowerShell.Objects.ContainerImageReport


**エイリアス**  
None



