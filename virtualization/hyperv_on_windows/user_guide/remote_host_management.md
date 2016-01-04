# Hyper-V マネージャーを使用したリモート Hyper-V ホストの管理

Hyper-V マネージャーは、ローカル Hyper-V ホストおよび少数のリモート ホストを診断および管理するためのインボックス ツールです。 この記事では、サポートされているすべての構成で、Hyper-V マネージャーを使用して Hyper-V ホストに接続するための構成手順について説明します。

> Hyper-V マネージャーは、[Hyper-V が含まれているすべての Windows OS](../quick_start/walkthrough_compatibility.md#OperatingSystemRequirements) 上で、**[プログラムと機能]** にある **[Hyper-V 管理ツール]** として使用できます。 リモート ホストを管理するために、Hyper-V プラットフォームを有効にする必要はありません。

Hyper-V マネージャーで Hyper-V ホストに接続するには、左側のウィンドウで **[Hyper-V マネージャー]** が選択されていることを確認してから、右側のウィンドウで **[サーバーへの接続]** を選択します。

![](media/HyperVManager-ConnectToHost.png)

## サポートされている Hyper-V ホストと Hyper-V マネージャーとの組み合わせ

Windows 10 の Hyper-V マネージャーでは、次の Hyper-V ホストを管理できます。
* Windows 10
* Windows 8.1
* Windows 8
* Windows Server 2012 R2 + Windows Server Core、Datacenter、および Hyper-V Server
* Windows 2012 + Windows Server Core、Datacenter、および Hyper-V Server

Windows 8.1 および Windows Server 2012 R2 の Hyper-V マネージャーでは、次を管理できます。
* Windows 8.1
* Windows 8
* Windows Server 2012 R2 + Windows Server Core、Datacenter、および Hyper-V Server
* Windows 2012 + Windows Server Core、Datacenter、および Hyper-V Server

Windows 8 および Windows Server 2012 の Hyper-V マネージャーでは、次を管理できます。
* Windows 8
* Windows 2012 + Windows Server Core、Datacenter、および Hyper-V Server

Windows 8 より、Windows で Hyper-V を利用できるようになりました。 Windows 8.1/Server 2012 より前には、Hyper-V マネージャーはバージョンが一致する Hyper-V のみを管理していました。

> **注:** Hyper-V マネージャーの機能は、管理しているバージョンで使用できる機能と一致します。 つまり、Server 2012R2 からリモート Server 2012 ホストを管理している場合、2012R2 の新しい Hyper-V マネージャー ツールは使用できません。

## ローカル ホストの管理

Hyper-V ホストとしてローカル ホストを Hyper-V マネージャーに追加するには、**[コンピューターの選択]** ダイアログ ボックスで **[ローカル コンピューター]** を選択します。

![](media/HyperVManager-ConnectToLocalHost.png)

接続を確立できない場合は、次の手順に従います。
*  Hyper-V プラットフォーム ロールが有効になっていることを確認します。  
    Hyper-V がサポートされているかどうかを確認するには、[互換性を確認するためのチュートリアル セクション](../quick_start/walkthrough_compatibility.md)を参照してください。
*  ユーザー アカウントが Hyper-V 管理者グループに含まれることを確認します。


## 同じドメイン内の別の Hyper-V ホストの管理

リモート Hyper-V ホストを Hyper-V マネージャーに追加するには、**[コンピューターの選択]** ダイアログ ボックスで **[別のコンピューター]** を選択し、テキスト フィールドにリモート ホストのホスト名 NetBIOS または FQDN を入力します。

![](media/HyperVManager-ConnectToRemoteHost.png)

リモート Hyper-V ホストを管理するには、ローカル コンピューターとリモート ホストの両方でリモート管理を有効にする必要があります。

この操作を行うには、`[システムのプロパティ]、[リモート管理の設定]` の順に選択するか、管理者として次の PowerShell コマンドを実行します。

``` PowerShell
winrm quickconfig
```

現在のユーザー アカウントがリモート ホスト上の Hyper-V 管理者アカウントと一致する場合は、**[OK]** をクリックして接続します。

> これは、Windows 8 または Windows 8.1 の Hyper-V マネージャーでリモート ホストを管理するためにサポートされている唯一の方法です。


Windows 10 では、リモート接続の種類の有効な組み合わせが大幅に拡張されています。  
ホスト名または IP アドレスを使用して、リモート Windows 10 以降のホストに接続できるようになりました。 Hyper-V マネージャーでは、代替ユーザー資格情報もサポートされるようになりました。


### 別のユーザーとしてリモート ホストに接続する

> この操作は、Windows 10 または Server 2016 Technical Preview 3 以降のリモート ホストに接続する場合にのみ可能です。

Windows 10 では、リモート ホスト用の正しいユーザー アカウントで実行していない場合、代替資格情報で別のユーザーとして接続できます。

リモート Hyper-V ホストの資格情報を指定するには、**[コンピューターの選択]** ダイアログ ボックスで **[別のユーザーとして接続する]** を選択し、**[ユーザーの設定]** を選択します。

![](media/HyperVManager-ConnectToRemoteHostAltCreds.png)


### IP アドレスを使用してリモート ホストに接続する

> この操作は、Windows 10 または Server 2016 Technical Preview 3 以降のリモート ホストに接続する場合にのみ可能です。

ホスト名の代わりに IP アドレスを使用して接続する方が容易な場合があります。 Windows 10 では、この方法のみが可能です。

IP アドレスを使用して接続するには、**[別のコンピューター]** テキスト フィールドに IP アドレスを入力します。


## ドメインの外部 (またはドメインなし) で Hyper-V ホストを管理する

> この操作は、Windows 10 または Server 2016 Technical Preview 3 以降のリモート ホストに接続する場合にのみ可能です。

管理対象の Hyper-V ホストで、管理者として次を実行します。

1.  [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx)
    * [Enable-PSRemoting](https://technet.microsoft.com/en-us/library/hh849694.aspx) は、*プライベート* ネットワーク ゾーンに必要なファイアウォール規則を作成します。 パブリック ゾーンでこのアクセスを許可するには、CredSSP および WinRM のルールを有効にする必要があります。
2. Set-Item WSMan:\localhost\Client\TrustedHosts -value "fqdn-of-managing-pc"
    * 代わりに、管理用にすべてのホストを信頼することを許可できます。
    * Set-Item WSMan:\localhost\Client\TrustedHosts -value * -force
3. [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer "fqdn-of-managing-pc"
    * 代わりに、管理用にすべてのホストを信頼することを許可できます。
    * [Enable-WSManCredSSP](https://technet.microsoft.com/en-us/library/hh849872.aspx) -Role client -DelegateComputer *




