# <a name="hyper-v-backup-approaches"></a>HYPER-V バックアップ方法
HYPER-V では、仮想マシンのバックアップ、ホスト オペレーティング システムで、仮想マシン内のユーザー設定のバックアップ ソフトウェアを実行する必要はありません。 ことができます。  自分のニーズに応じてを利用する開発者向けに利用できるいくつかの方法があります。
## <a name="hyper-v-vss-writer"></a>HYPER-V VSS ライター
HYPER-V では、すべてのバージョンの Windows Server が HYPER-V がサポートされている場所に VSS ライターを実装します。  この VSS ライター仮想マシンのバックアップを既存の VSS インフラストラクチャを利用できるようにします。  ただし、小規模なバックアップをサーバー上のすべての仮想マシンが同時にバックアップして向けです。

このアーキテクチャの理解を深める – このプレゼンテーションを参照してください。https://channel9.msdn.com/Events/TechEd/NorthAmerica/2010/VIR322
## <a name="hyper-v-wmi-based-backup"></a>HYPER-V WMI ベースのバックアップ
Windows Server 2016 から始めて、HYPER-V は HYPER-V WMI API をバックアップするサポート用を開始します。  この方法では、バックアップ用には、仮想マシンの内部も VSS を利用しながら、ホスト オペレーティング システムで VSS を使用しません。  代わりに、参照の要素 (RCT) を追跡するための変更の英数字の組み合わせを使ってに関する情報にアクセスすることもできる効率的な方法での仮想マシンのバックアップします。  この方法は、Windows Server 2016 で使用できると以降ではのみホストで VSS を使用するよりもスケーラブルです。

このアーキテクチャの理解を深める – このプレゼンテーションを参照してください。https://channel9.msdn.com/Events/TechEd/Europe/2014/CDP-B318 

ここでは利用可能なこれらの Api を使用する方法については、例があります。https://msconfiggallery.cloudapp.net/packages/xHyper-VBackup
## <a name="methods-for-reading-backups-from-wmi-based-backup"></a>WMI ベースのバックアップからバックアップを閲覧する方法
HYPER-V WMI を使用して仮想マシンのバックアップを作成するには、実際のデータをバックアップから閲覧用の 3 つの方法があります。  一意の利点と欠点します。
### <a name="wmi-export"></a>WMI のエクスポート
開発者は、(上記の例で使用)、HYPER-V WMI インターフェイスを使用バックアップ データをエクスポートできます。  HYPER-V をコンパイル仮想ハード ドライブに変更し、指定した場所にファイルをコピーします。  この方法を使いやすい、すべてのシナリオに対して都合が良い、リモートします。  ただし、多くの場合に生成された仮想ハード ドライブの領域は、大量のネットワークに転送するデータを作成します。
### <a name="win32-apis"></a>Win32 Api
開発者は SetVirtualDiskInformation、GetVirtualDiskInformation を使用し、仮想ハード_ディスク WIN32 QueryChangesVirtualDisk Api 記述されているをここでは設定:https://docs.microsoft.com/en-us/windows/desktop/api/_vhd/メモこれらの Api を使用するには、HYPER-V WMI もする必要がある参照を作成するために使用します。関連付けられている仮想マシンでポイントします。  バックアップの仮想マシンのデータへのアクセスを効率的に使用し、これらの Win32 Api を許可します。  Win32 Api は、いくつかの制限には。
*   ローカルでアクセスすることができますのみ
*   しますかからデータを読み取るサポートされていません仮想ハード_ディスク上のファイルを共有します。
*   仮想ハード_ディスク上の内部構造は、データのアドレスを返す

### <a name="remote-shared-virtual-disk-protocol"></a>リモートの共有仮想ディスク プロトコル
最後に、開発者は、効率的に – 仮想ハード_ディスク上の共有ファイルからデータをバックアップ情報にアクセスする必要がある場合は、リモートの共有の仮想ディスク プロトコルを使用して、必要があります。  このプロトコルは、ここに記載されています。https://msdn.microsoft.com/en-us/library/dn393384.aspx
