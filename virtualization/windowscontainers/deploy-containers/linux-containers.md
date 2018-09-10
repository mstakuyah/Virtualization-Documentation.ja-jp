# <a name="linux-containers"></a>Linux コンテナー

この機能では、[Hyper-V による分離](../manage-containers/hyperv-container.md)を使って、コンテナーをサポートするために十分な OS のみで Linux カーネルを実行します。 _Windows 10 Fall Creators Update_ と _Windows Server バージョン 1709_ 以降では、Windows と Hyper-V に変更が加えられて、このようなビルドに対応していますが、この機能を統合するには、Docker テクノロジの基盤であるオープン ソースの [Moby プロジェクト](https://www.github.com/moby/moby)に加え Linux カーネルでの作業も必要でした。 

[!VIDEO https://sec.ch9.ms/ch9/1e5a/08ff93f2-987e-4f8d-8036-2570dcac1e5a/LinuxContainer.mp4]

これを試みるには、次の前提条件を満たす必要があります。

- Windows 10 または Windows Server Insider Preview ビルド 16267 以降
- Moby マスター ブランチに基づく Docker デーモンのビルド (`--experimental` フラグを指定して実行)
- 互換性のある任意の Linux イメージ

このプレビューには、入門ガイドが用意されています。

- [Docker Enterprise Edition Preview](https://blog.docker.com/2017/09/docker-windows-server-1709/) には、LinuxKit システムと Linux コンテナーを実行できる Docker EE のプレビューが含まれています。 背景情報の詳細については、[LinuxKit を使って Windows 上で Linux コンテナーをプレビューする方法についての記事](https://go.microsoft.com/fwlink/?linkid=857061)を参照してください。
- [Windows 10 および Windows Server 上での Hyper-V による分離を使った Ubuntu コンテナーの実行](https://go.microsoft.com/fwlink/?linkid=857067)


## <a name="work-in-progress"></a>進行中の作業

Moby プロジェクトで進行中の作業の進捗状況は、[GitHub](https://github.com/moby/moby/issues/33850) で追跡できます。


### <a name="known-app-issues"></a>アプリの既知の問題

これらのアプリケーションはすべてボリューム マッピングを必要とするため、若干の制限があります。詳しくは、「[バインド マウント](#Bind-mounts)」をご覧ください。 以下は、開始しないか、正常に実行されません。

- MySQL
- PostgreSQL
- WordPress
- Jenkins
- MariaDB
- RabbitMQ


### <a name="bind-mounts"></a>バインド マウント

`docker run -v ...` を使ってボリュームをバインド マウントした場合、Windows NTFS ファイルシステムにファイルが格納されるため、POSIX 操作のための変換が必要になります。 しかしファイル システム操作の中には、現在部分的に、またはまったく実装されていない操作があるため、一部のアプリは互換性がありません。

以下の操作は、現在、バインド マウントされたボリュームには機能しません。

- MkNod
- XAttrWalk
- XAttrCreate
- Lock
- Getlock
- Auth
- Flush
- INotify

完全に実装されていない操作も少数ですが存在します。

- GetAttr: Nlink カウントは、常に 2 と報告されます。
- Open: ReadWrite フラグ、WriteOnly フラグ、および ReadOnly フラグのみが実装されています。
