# <a name="hyper-v-backup-approaches"></a>Hyper-v のバックアップ方法
Hyper-v を使用すると、仮想マシン内でカスタムバックアップソフトウェアを実行することなく、ホストオペレーティングシステムから仮想マシンをバックアップすることができます。  開発者がニーズに応じて利用できる方法はいくつかあります。
## <a name="hyper-v-vss-writer"></a>Hyper-v VSS ライター
Hyper-v は、Hyper-v がサポートされているすべてのバージョンの Windows Server 上に VSS ライターを実装します。  この VSS ライターを使用すると、開発者は既存の VSS インフラストラクチャを利用して仮想マシンをバックアップできます。  ただし、サーバー上のすべての仮想マシンが同時にバックアップされる、小規模なバックアップ操作向けに設計されています。

このアーキテクチャについて理解を深めるには、次のプレゼンテーションを参照してください: https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-v WMI ベースのバックアップ
Windows Server 2016 以降、hyper-v は Hyper-v WMI API を使用してバックアップのサポートを開始しました。  このアプローチでも、バックアップのために仮想マシン内で VSS を利用しますが、ホストオペレーティングシステムでは VSS を使用しなくなりました。  代わりに、参照ポイントと回復性の高い変更追跡 (.RCT) の組み合わせを使用して、開発者がバックアップされた仮想マシンに関する情報に効率的な方法でアクセスできるようにします。  この方法は、ホストで VSS を使用するよりもスケーラブルですが、Windows Server 2016 以降でのみ使用できます。

このアーキテクチャについて理解を深めるには、次のプレゼンテーションを参照してください: https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

これらの Api を使用する方法については、次の例も参照してください。 https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI ベースのバックアップからバックアップを読み取る方法
Hyper-v WMI を使用して仮想マシンのバックアップを作成する場合は、次の3つの方法でバックアップから実際のデータを読み取ることができます。  それぞれには、それぞれ固有の長所と短所があります。
### <a name="wmi-export"></a>WMI エクスポート
開発者は、(上の例で使用しているように) Hyper-v WMI インターフェイスを使用してバックアップデータをエクスポートできます。  Hyper-v は、仮想ハードドライブに変更をコンパイルし、要求された場所にファイルをコピーします。  この方法は簡単に使用でき、すべてのシナリオで機能し、リモート処理が可能です。  ただし、生成される仮想ハードドライブは、多くの場合、ネットワーク経由で大量のデータを転送するように作成されます。
### <a name="win32-apis"></a>Win32 API
開発者は、ここに記載されているように、仮想ハード Win32 API ディスクで SetVirtualDiskInformation、GetVirtualDiskInformation、および QueryChangesVirtualDisk Api を使用でき https://docs.microsoft.com/windows/desktop/api/_vhd/ ます。これらの Api を使用するには、Hyper-v WMI を使用して関連する仮想マシン上に参照ポイントを作成する必要があることに注意してください。  これらの Win32 Api を使用すると、バックアップされた仮想マシンのデータに効率的にアクセスできます。  Win32 Api には、いくつかの制限があります。
* ローカルでのみアクセスできます
* は、共有仮想ハードディスクファイルからのデータの読み取りをサポートしていません。
* これらのユーザーは、仮想ハードディスクの内部構造に関連するデータアドレスを返します。

### <a name="remote-shared-virtual-disk-protocol"></a>リモート共有仮想ディスクプロトコル
最後に、開発者が共有仮想ハードディスクファイルからバックアップデータ情報に効率的にアクセスする必要がある場合は、リモート共有仮想ディスクプロトコルを使用する必要があります。  このプロトコルについては、[こちら](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)を参照してください。
