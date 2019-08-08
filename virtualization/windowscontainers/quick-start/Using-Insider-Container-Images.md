
# <a name="using-insider-container-images"></a>Insider コンテナー イメージの使用

この演習では、Windows Insider Preview プログラムで提供されている最新の insider ビルドの Windows Server に、Windows コンテナー機能を展開して使用する手順を説明します。 この演習では、コンテナーの役割をインストールし、基本 OS イメージのプレビュー版を展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイック スタートは Windows Server Insider Preview プログラムの Windows Server コンテナーのみに適用されます。 このクイック スタートに進む前に、このプログラムに慣れておいてください。

## <a name="prerequisites"></a>前提条件: 

- [Windows Insider Program](https://insider.windows.com/GettingStarted) に参加し、使用条件を確認していること。
- Windows Insider Program から提供された Windows Server の最新ビルドや、Windows Insider Program から提供された Windows 10 の最新ビルドが実行されている 1 台のコンピューター システム (物理マシンまたは仮想マシン)。

> [!IMPORTANT]
> Windows server Insider Preview プログラムまたは windows 10 のビルドから windows Server のビルドを使用する必要があります。 windows Insider Preview プログラムでは、以下に示す基本イメージを使用します。 これらのビルドを使用せずに、対象の基本イメージを使用した場合、コンテナーを開始できません。

## <a name="install-docker-enterprise-edition-ee"></a>Docker Enterprise Edition (EE) のインストール

Windows コンテナーを使用するには Docker EE が必要です。 Docker EE は、Docker エンジンと Docker クライアントで構成されます。

Docker EE をインストールするには、OneGet プロバイダー PowerShell モジュールを使用します。 このプロバイダーは、コンピューターでコンテナー機能を有効にして、Docker EE をインストールします。これには、再起動が必要です。 管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

> [!NOTE]
> Windows Server Insider ビルドでの Docker EE のインストールには、Insider 以外のビルドに使用されるものとは異なる OneGet provider が必要です。 Docker EE と DockerMsftProvider OneGet プロバイダーが既にインストールされている場合は、続行する前に削除します。

```powershell
Stop-Service docker
Uninstall-Package docker
Uninstall-Module DockerMsftProvider
```

Windows Insider ビルドで使用するための OneGet PowerShell モジュールをインストールします。

```powershell
Install-Module -Name DockerProvider -Repository PSGallery -Force
```

OneGet を使用して最新バージョンの Docker EE Preview をインストールします。

```powershell
Install-Package -Name docker -ProviderName DockerProvider -RequiredVersion Preview
```

インストールが完了したら、コンピューターを再起動します。

```powershell
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>コンテナーの基本イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 Windows Insider Program に参加すると、これらの基本イメージの最新ビルドをテストすることもできます。 Insider の基本イメージでは、現在、Windows Server に基づく 4 つの基本イメージが用意されています。 以下の表に、それぞれの基本イメージの使用目的を示します。

| 基本 OS イメージ                       | 使い方                      |
|-------------------------------------|----------------------------|
| mcr.microsoft.com/windows/servercore         | 実稼働と開発 |
| mcr.microsoft.com/windows/nanoserver              | 実稼働と開発 |
| mcr.microsoft.com/windows/servercore/insider | 開発専用           |
| mcr.microsoft.com/windows/nanoserver/insider        | 開発専用           |

Nano Server Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/nanoserver/insider
```

Windows Server Core Insider 基本イメージをプルするには、次のコマンドを実行します。

```console
docker pull mcr.microsoft.com/windows/servercore/insider
```

> [!IMPORTANT]
> Windows コンテナ OS イメージ[EULA](../EULA.md )と windows Insider プログラム[利用規約](https://www.microsoft.com/software-download/windowsinsiderpreviewserver)をお読みください。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [サンプルアプリケーションをビルドして実行する](./Nano-RS3-.NET-Core-and-PS.md)
