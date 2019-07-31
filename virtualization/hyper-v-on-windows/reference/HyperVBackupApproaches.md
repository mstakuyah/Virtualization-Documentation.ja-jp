# <a name="hyper-v-backup-approaches"></a>Hyper-v バックアップの方法
Hyper-v を使用すると、仮想マシンの内部でカスタムバックアップソフトウェアを実行しなくても、ホストオペレーティングシステムから仮想マシンをバックアップすることができます。  開発者が必要に応じて利用できるいくつかの方法があります。
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS Writer
Hyper-v は、Hyper-v がサポートされているすべてのバージョンの Windows Server で VSS ライターを実装します。  この VSS ライターにより、開発者は既存の VSS インフラストラクチャを利用して仮想マシンをバックアップすることができます。  ただし、サーバー上のすべての仮想マシンが同時にバックアップされる、スモールスケールのバックアップ操作向けに設計されています。

このアーキテクチャをより的確に理解するには、次のプレゼンテーションを参照してください。https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI ベースのバックアップ
Windows Server 2016 以降、hyper-v WMI API を使用したバックアップのサポートを開始しました。  この方法では引き続き、バックアップ目的で仮想マシン内の VSS を使用しますが、ホストオペレーティングシステムでは VSS は使用されません。  代わりに、参照ポイントと回復可能な変更の追跡 (.RCT) の組み合わせを使用して、開発者がバックアップされた仮想マシンに関する情報に効率的な方法でアクセスできるようにします。  この方法は、ホストで VSS を使用する場合よりもスケーラブルですが、Windows Server 2016 以降でのみ利用できます。

このアーキテクチャをより的確に理解するには、次のプレゼンテーションを参照してください。https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

また、これらの Api を使う方法の例を次に示します。https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI ベースのバックアップからバックアップを読み取る方法
Hyper-v WMI を使用して仮想マシンのバックアップを作成する場合は、次の3つの方法でバックアップから実際のデータを読み取ることができます。  それぞれに固有の長所と短所があります。
### <a name="wmi-export"></a>WMI のエクスポート
開発者は、Hyper-v WMI インターフェイス経由でバックアップデータをエクスポートすることができます (上の例で使用されています)。  Hyper-v は、変更を仮想ハードドライブにコンパイルし、要求された場所にファイルをコピーします。  この方法は使いやすいため、すべてのシナリオで動作し、リモート処理できます。  ただし、生成された仮想ハードドライブは、多くの場合、ネットワーク経由で転送する大量のデータを作成します。
### <a name="win32-apis"></a>Win32 Api
開発者は、ここで説明しているように、仮想ハードディスク Win32 API セットの SetVirtualDiskInformation、GetVirtualDiskInformation、QueryChangesVirtualDisk Api を使うことができます。これらの Api を使うには、Hyper-v WMI を使用して参照を作成する必要があります。 https://docs.microsoft.com/windows/desktop/api/_vhd/関連付けられている仮想マシン上のポイント。  これらの Win32 Api により、バックアップされた仮想マシンのデータに効率的にアクセスできます。  Win32 Api にはいくつかの制限事項があります。
* ローカルでのみアクセスできます。
* 共有仮想ハードディスクファイルからのデータの読み取りはサポートされません
* 仮想ハードディスクの内部構造に関連するデータアドレスを返します。

### <a name="remote-shared-virtual-disk-protocol"></a>リモート共有仮想ディスクプロトコル
最後に、開発者が共有仮想ハードディスクファイルからバックアップデータ情報に効率的にアクセスする必要がある場合は、リモート共有仮想ディスクプロトコルを使用する必要があります。  このプロトコルについては、[ここ](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)に記載されています。
