---
redirect_url: https://msdn.microsoft.com/virtualization/hyperv_on_windows/windows_welcome
---

## ネットワーク アダプターとメモリのホット アドとホット リムーブ

仮想マシンが稼働中でも、ダウンタイムなしで、ネットワーク アダプターを追加したり、削除したりできるようになりました。 これは Windows と Linux のオペレーティング システムの両方を実行している第 2 世代仮想マシンで機能します。

動的メモリを有効にしていなくても、仮想マシンに割り当てられているメモリの量を稼働中に調整することもできます。 これは第 1 世代と第 2 世代の両方の仮想マシンで機能します。

## 運用チェックポイント

運用チェックポイントを利用すると、仮想マシンの「ある時点」のイメージを簡単に作成できます。このイメージを後に、あらゆる運用ワークロードで完全に復元できます。 これは、保存された状態を利用する代わりに、ゲスト内のバックアップ技術を利用してチェックポイントを作成することで行います。 運用チェックポイントでは、ボリューム スナップショット サービス (VSS) が Windows 仮想マシン内で使用されます。 Linux 仮想マシンはシステム バッファーを消去し、ファイル システムで一貫性のあるチェックポイントを作成します。 「保存された状態」技術でチェックポイントを作成する場合、仮想マシンに標準のチェックポイントを使用するように選択できます。


> <g id="1" ctype="x-strong">重要:</g> 新しい仮想マシンの既定では、標準のチェックポイントにフォールバックするように運用チェックポイントが作成されます。


## Hyper-V マネージャーの機能強化

- <g id="1" ctype="x-strong">代替の資格情報のサポート</g> – 別の Windows 10 Technical Preview リモート ホストに接続するとき、Hyper-V マネージャーで別セットの資格情報を使用できるようになりました。 後で簡単にログオンできるようにこれらの資格情報を保存することもできます。

- <g id="1" ctype="x-strong">ダウンレベルの管理</g> - Hyper-V マネージャーを利用し、さらに多くのバージョンの Hyper-V を管理できるようになりました。 Windows 10 Technical Preview で Hyper-V マネージャーを利用すると、Windows Server 2012、Windows 8、Windows Server 2012 R2、Windows 8.1 で Hyper-V を実行しているコンピューターを管理できます。

- <g id="1" ctype="x-strong">管理プロトコルの更新</g> - Hyper-V マネージャーが更新され、CredSSP、Kerberos、NTLM 認証を許可する WS-MAN プロトコルを利用してリモート Hyper-V ホストと通信できるようになりました。 CredSSP を利用してリモート Hyper-V ホストに接続するとき、Active Directory で最初に制約付き委任を有効にしなくてもライブ マイグレーションを実行できます。 また、WS MAN ベースのインフラストラクチャでは、リモート管理でホストを有効にするために必要な構成が簡単です。 WS-MAN はポート 80 で接続します。このポートは既定で開かれています。


## コネクト スタンバイの互換性

Always On/Always Connected (AOAC) 電源モデルを使用するコンピューターで Hyper-V が有効になっているとき、コネクト スタンバイ電源状態が利用できるようになりました。

Windows 8 と 8.1 で Hyper-V を使用すると、Always On/Always Connected (AOAC) 電源モデル (別名 InstantON) を使用するコンピューターがスリープ状態になりませんでした。 詳細については、[サポート技術情報の記事](
https://support.microsoft.com/en-us/kb/2973536) を参照してください。


## Linux セキュア ブート

第 2 世代仮想マシンで実行される Linux オペレーティング システムで、セキュア ブート オプションを有効にして起動できる Linux オペレーティング システムの数が増えました。 Ubuntu 14.04 以降と SUSE Linux Enterprise Server 12 が Technical Preview を実行するホストでセキュア ブートできます。 初めて仮想マシンを起動する前に、仮想マシンで Microsoft UEFI 証明機関を使用するように指定する必要があります。 管理者特権で Windows PowerShell ウィンドウを開き、次のように入力します。

    Set-VMFirmware [-VMName] <VMName> [-SecureBootTemplate] <MicrosoftUEFICertificateAuthority>

Hyper-V で実行する Linux 仮想マシンの詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Hyper-V の Linux と FreeBSD 仮想マシン</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。


## 仮想マシン構成バージョン

Windows 8.1 を実行するホストから Windows 10 で Hyper-V を実行するホストに仮想マシンを移動またはインポートするとき、仮想マシンの構成ファイルは自動的にアップグレードされません。 このため、Windows 8.1 を実行するホストに仮想マシンを戻すことができます。 仮想マシンの構成バージョンを手動で更新するまで、仮想マシンで新しい Hyper-V 機能を使用することはできません。

仮想マシンの構成バージョンは、仮想マシンの構成、保存された状態、スナップショット ファイルとの間で互換性がある Hyper-V のバージョンを示しています。 構成バージョン 5 の仮想マシンは Windows 8.1 との間に互換性があり、Windows 8.1 と Windows 10 の両方で実行できます。 構成バージョン 6 の仮想マシンは Windows 10 との間に互換性があり、Windows 8.1 では実行できません。

### 構成バージョンの確認

管理者特権のコマンド プロンプトで、次のコマンドを実行します。

``` PowerShell
Get-VM * | Format-Table Name, Version
```

### 構成バージョンのアップグレード

管理者特権の Windows PowerShell プロンプトで、次のコマンドのいずれかを実行します。

``` 
Update-VmConfigurationVersion <VMName>
```

または

``` 
Update-VmConfigurationVersion <VMObject>
```

> <g id="1" ctype="x-strong">重要:</g>

- 仮想マシンの構成バージョンをアップグレードした後、Windows 8.1 を実行するホストに仮想マシンを移動することはできません。
- バージョン 6 からバージョン 5 に仮想マシンの構成バージョンをダウングレードすることはできません。
- 仮想マシンの構成をアップグレードするには、仮想マシンをオフにする必要があります。
- アップグレード後、仮想マシンは新しい構成ファイル形式を使用します。 詳細については、「<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">構成ファイルの形式</g><g id="2CapsExtId3" ctype="x-title"></g></g>」を参照してください。


## <g id="1" ctype="x-html"></g><g id="2" ctype="x-html"></g>構成ファイルの形式

仮想マシンの構成ファイル形式が新しくなりました。仮想マシン構成データを読み書きするときの効率を上げるように設計されています。 ストレージが故障した場合にデータが壊れる可能性を減らすようにも設計されています。 新しい構成ファイルでは、仮想マシン構成データに .VMCX 拡張子が、ランタイム状態データに .VMRS 拡張子が使用されます。

> <g id="1" ctype="x-strong">重要:</g> .VMCX ファイルはバイナリ形式です。 .VMCX ファイルと .VMRS ファイルは直接編集できません。





<!--HONumber=May16_HO1-->


