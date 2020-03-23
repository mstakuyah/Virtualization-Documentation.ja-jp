# <a name="hyper-v-backup-approaches"></a>Hyper-V のバックアップ方法
Hyper-V を使用すると、仮想マシン内でカスタム バックアップ ソフトウェアを実行することなく、ホスト オペレーティング システムから仮想マシンをバックアップすることができます。  開発者がニーズに応じて使用できる方法がいくつかあります。
## <a name="hyper-v-vss-writer"></a>Hyper-V の VSS ライター
Hyper-V により、Hyper-V がサポートされているすべてのバージョンの Windows Server に VSS ライターが実装されます。  この VSS ライターを使用すると、開発者は既存の VSS インフラストラクチャを利用して仮想マシンをバックアップできます。  ただしこれは、サーバー上のすべての仮想マシンが同時にバックアップされる小規模なバックアップ操作に向けて設計されています。

このアーキテクチャについての理解を深めるには、次のプレゼンテーションを参照してください: https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>Hyper-V の WMI ベースのバックアップ
Windows Server 2016 以降、Hyper-V では Hyper-V WMI API を通してバックアップのサポートが開始されました。  この方法でも、バックアップを目的として仮想マシン内で VSS を利用しますが、ホスト オペレーティング システムでは VSS が使われなくなっています。  その代わり、開発者が効率的な方法で、バックアップされた仮想マシンに関する情報にアクセスできるように、参照ポイントと回復性がある変更追跡 (RCT) の組み合わせが使用されます。  この方法は、ホストで VSS を使用するよりもスケーラブルですが、使用できるのは Windows Server 2016 以降のみです。

このアーキテクチャについての理解を深めるには、次のプレゼンテーションを参照してください: https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

これらの API を使用する方法の例は、次の場所でも提供されています: https://www.powershellgallery.com/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI ベースのバックアップからバックアップを読み取るための方法
Hyper-V WMI を使用して仮想マシンのバックアップを作成するときには、バックアップから実際のデータを読み取るための方法が 3 つあります。  それぞれに固有の長所と短所があります。
### <a name="wmi-export"></a>WMI エクスポート
開発者は (上の例で使用されているように) Hyper-V WMI インターフェイスを介して、バックアップ データをエクスポートできます。  Hyper-V では、変更が仮想ハード ドライブにコンパイルされ、要求された場所にファイルがコピーされます。  この方法は、使用が簡単ですべてのシナリオで機能し、リモート処理が可能です。  ただし、生成される仮想ハード ドライブからは、多くの場合、ネットワーク経由で転送する大量のデータが作成されます。
### <a name="win32-apis"></a>Win32 API
開発者は、次の場所に記載されているように、仮想ハード ディスクの Win32 API セットの SetVirtualDiskInformation、GetVirtualDiskInformation、QueryChangesVirtualDisk API を使用できます。 https://docs.microsoft.com/windows/desktop/api/_vhd/ これらの API を使用するには、関連付けられる仮想マシン上に参照ポイントを作成するために、Hyper-V WMI も使用する必要があることに注意してください。  これらの Win32 API によって、バックアップされた仮想マシンのデータに効率的にアクセスできます。  Win32 API にはいくつかの制限事項があります。
* これらにはローカルでのみアクセスできます
* 共有仮想ハード ディスク ファイルからのデータの読み取りはサポートされていません
* これらは、仮想ハード ディスクの内部構造に対して相対的なデータ アドレスを返します

### <a name="remote-shared-virtual-disk-protocol"></a>Remote Shared Virtual Disk Protocol
最後に、開発者が共有仮想ハード ディスク ファイルからバックアップ データ情報に効率的にアクセスする必要がある場合は、Remote Shared Virtual Disk Protocol を使用する必要があります。  このプロトコルについては、[ここ](https://docs.microsoft.com/openspecs/windows_protocols/ms-rsvd/c865c326-47d6-4a91-a62d-0e8f26007d15)で説明されています。
