# 新しい管理サービスの作成

Windows 10 以降、Hyper-V により、ネットワーク接続に依存せずに、Hyper-V ゲストとホスト間で登録済みソケット接続が可能です。 Hyper-V ソケットを使用することで、サービスはネットワーク スタックと関係なく実行でき、すべてのデータが同じ物理メモリ上に格納されます。

このドキュメントでは、Hyper-V ソケット上に構築された簡単なアプリケーションの作成とそれらの使用を開始する方法の手順について説明します。

[PowerShell Direct](../user_guide/vmsession.md) は Hyper-V ソケットを使用して通信するアプリケーション (この例では、インボックス Windows サービス) の例です。

**サポートされるホスト OS**
* Windows 10
* Windows Server Technical Preview 3
* 今後のリリース (Server 2016 以上)

**サポートされるゲスト OS**
* Windows 10
* Windows Server Technical Preview 3
* 今後のリリース (Server 2016 以上)

**機能と制限事項**
* カーネル モードまたはユーザー モード操作をサポート
* データ ストリームのみ
* ブロック メモリなし (バックアップ/ビデオに最適でない)

--------------


## はじめに

現在、Hyper-V ソケットは、ネイティブ コード (C++) で使用できます。

簡単なアプリケーションを作成するには、次が必要です。
* C コンパイラ。 ない場合は、[Visual Studio Code](https://aka.ms/vs) を確認してください。
* Hyper-V と仮想マシンを実行しているコンピューター。
* ホストおよびゲスト (VM) OS は、Windows 10、Windows Server Technical Preview 3 以降である必要があります。
* Windows SDK - ここに、`hvsocket.h` を含む [Win10 SDK](https://dev.windows.com/en-us/downloads/windows-10-sdk) のリンクを示します。

## 新しいアプリケーションの登録

Hyper-V ソケットを使用するには、アプリケーションを Hyper-V ホストのレジストリに登録する必要があります。

レジストリにサービスを登録することで、次が得られます。
*  使用可能なサービスを有効化、無効化、および一覧表示するための WMI 管理
*  仮想マシンと直接通信するアクセス許可

次の PowerShell は、"HV Socket Demo" という名前の新しいアプリケーションを登録しています。 これは管理者として実行する必要があります。 手動の手順は次のとおりです。

``` PowerShell
$friendlyName = "HV Socket Demo"

# Create a new random GUID and add it to the services list then add the name as a value

$service = New-Item -Path "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices" -Name ([System.Guid]::NewGuid().ToString())

$service.SetValue("ElementName", $friendlyName)

# Copy GUID to clipboard for later use
$service.PSChildName | clip.exe
```

** レジストリの場所と情報 **

``` 
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
```
このレジストリの場所には、いくつかの GUID が表示されます。 それらはインボックス サービスです。

サービスごとのレジストリ内の情報:
* `サービス GUID`
* `ElementName (REG_SZ)` - これは、サービスのフレンドリ名です。

独自のサービスを登録するには、独自の GUID とフレンドリ名を使用して新しいレジストリ キーを作成します。

フレンドリ名は、新しいアプリケーションに関連付けられます。 これは、GUID が適切でないパフォーマンス カウンターやその他の場所に表示されます。

レジストリ エントリは、次のようになります。
```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\GuestCommunicationServices\
    999E53D4-3D5C-4C3E-8779-BED06EC056E1\
        ElementName REG_SZ  VM Session Service
    YourGUID\
        ElementName REG_SZ  Your Service Friendly Name
```

>** ヒント:** PowerShell で GUID を生成し、それをクリップボードにコピーするには、次を実行します。
``` PowerShell
[System.Guid]::NewGuid().ToString() | clip.exe
```

## Hyper-V ソケットの作成

最も基本的な例で、ソケットの定義には、アドレス ファミリ、接続の種類、およびプロトコルが必要です。

これは、簡単な[ソケット定義](
https://msdn.microsoft.com/en-us/library/windows/desktop/ms740506(v=vs.85).aspx
)です。

``` C
SOCKET WSAAPI socket(
  _In_ int af,
  _In_ int type,
  _In_ int protocol
);
```

Hyper-V ソケットの場合:
* アドレス ファミリ - `AF_HYPERV`
* 種類 - `SOCK_STREAM`、`SOCK_DGRAM`、または `SOCK_RAW`
* プロトコル – `HV_PROTOCOL_RAW`


これは宣言/インスタンスの例です。
``` C
SOCKET sock = socket(AF_HYPERV, SOCK_STREAM, HV_PROTOCOL_RAW);
```


## Hyper-V ソケットへのバインド

バインドは、ソケットと接続情報を関連付けます。

便宜上、下に関数定義をコピーしていますが、バインドの詳細については[ここ](https://msdn.microsoft.com/en-us/library/windows/desktop/ms737550.aspx)を参照してください。

``` C
int bind(
  _In_ SOCKET                s,
  _In_ const struct sockaddr *name,
  _In_ int                   namelen
);
```

ホスト マシンの IP アドレスとそのホスト上のポート番号から構成される標準インターネット プロトコル アドレス ファミリ (`AF_INET`) のソケット アドレス (sockaddr) と対照的に、`AF_HYPERV` のソケット アドレスでは、仮想マシンの ID と上に定義したアプリケーション ID を使用して、接続を確立します。

Hyper-V ソケットは、ネットワーク スタック、TCP/IP、DNS などに依存しないため、ソケット エンドポイントには、引き続きあいまいに接続を記述する、ホスト名ではない非 IP 形式が必要です。

Hyper-V ソケットのソケット アドレスの定義を次に示します。

``` C
struct SOCKADDR_HV
{
     ADDRESS_FAMILY Family;
     USHORT Reserved;
     GUID VmId;
     GUID ServiceId;
};
```

IP またはホスト名の代わりに、AF_HYPERV エンドポイントは 2 つの GUID に大きく依存します。
* VM ID – これは、VM ごとに割り当てられる一意の ID です。 VM の ID は、次の PowerShell スニペットを使用して見つけることができます。
  ```PowerShell
  (Get-VM -Name $VMName).Id
  ```
* サービス ID - アプリケーションを Hyper-V ホスト レジストリに登録するための[上記](#RegisterANewApplication)の GUID。

接続が特定の仮想マシンとの接続でない場合は、一連の VMID ワイルドカードを使用することもできます。

### VMID ワイルドカード

| 名前| GUID| 説明|
|:-----|:-----|:-----|
| HV_GUID_ZERO| 00000000-0000-0000-0000-000000000000| リスナーは、すべてのパーティションからの接続を受け入れるために、この VmId にバインドする必要があります。|
| HV_GUID_WILDCARD| 00000000-0000-0000-0000-000000000000| リスナーは、すべてのパーティションからの接続を受け入れるために、この VmId にバインドする必要があります。|
| HV_GUID_BROADCAST| FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF| |
| HV_GUID_CHILDREN| 90db8b89-0d35-4f79-8ce9-49ea0ac8b7cd| 子のワイルドカード アドレス。リスナーは、その子からの接続を受け入れるために、この VmId にバインドする必要があります。|
| HV_GUID_LOOPBACK| e0e16197-dd56-4a10-9195-5ee7a155a838| ループバック アドレス。この VmId を使用して、コネクタと同じパーティションに接続します。|
| HV_GUID_PARENT| a42e7cda-d03f-480c-9cc2-a4de20abb878| 親アドレス。この VmId を使用して、コネクタの親パーティションに接続します。*|


***HV_GUID_PARENT**
仮想マシンの親は、そのホストです。 コンテナーの親は、コンテナーのホストです。
仮想マシンで実行しているコンテナーからの接続は、コンテナーをホストしている仮想マシンに接続します。
この VmId でリッスンし、次からの接続を受け入れます。
(コンテナー内): コンテナー ホスト。
(VM 内: コンテナー ホスト/コンテナーなし): VM ホスト。
(VM 内でない: コンテナー ホスト/コンテナーなし): サポートされていません。

## サポートされているソケット コマンド

Socket()
Bind()
Connect()
Send()
Listen()
Accept()

[完全な WinSock API](https://msdn.microsoft.com/en-us/library/windows/desktop/ms741394.aspx)

## 進行中の作業

正常な切断
選択



<!--HONumber=Dec15_HO1-->
