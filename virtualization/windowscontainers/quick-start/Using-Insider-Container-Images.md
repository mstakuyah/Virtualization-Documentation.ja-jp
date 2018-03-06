
# <a name="using-insider-container-images"></a>Insider コンテナー イメージの使用

この演習では、Windows Insider Preview プログラムで提供されている最新の insider ビルドの Windows Server に、Windows コンテナー機能を展開して使用する手順を説明します。 この演習では、コンテナーの役割をインストールし、基本 OS イメージのプレビュー版を展開します。 コンテナーの概要については、[コンテナーについてのページ](../about/index.md)」をご覧ください。

このクイック スタートは Windows Server Insider Preview プログラムの Windows Server コンテナーのみに適用されます。 このクイック スタートに進む前に、このプログラムに慣れておいてください。

**前提条件: **

- [Windows Insider Program](https://insider.windows.com/GettingStarted) に参加し、使用条件を確認していること。
- Windows Insider Program から提供された Windows Server の最新ビルドや、Windows Insider Program から提供された Windows 10 の最新ビルドが実行されている 1 台のコンピューター システム (物理マシンまたは仮想マシン)。

>以下で説明する基本イメージを使用するには、Windows Server Insider Preview プログラムから提供された Windows Server のビルド、または Windows Insider Preview プログラムから提供された Windows 10 のビルドを使用する必要があります。 これらのビルドを使用せずに、対象の基本イメージを使用した場合、コンテナーを開始できません。

## <a name="install-docker-enterprise-edition-ee"></a>Docker Enterprise Edition (EE) のインストール
Windows コンテナーを使用するには Docker EE が必要です。 Docker EE は、Docker エンジンと Docker クライアントで構成されます。 

Docker EE をインストールするには、OneGet プロバイダー PowerShell モジュールを使用します。 このプロバイダーは、コンピューターでコンテナー機能を有効にして、Docker EE をインストールします。これには、再起動が必要です。 管理者特権の PowerShell セッションを開き、次のコマンドを実行します。

>注: Windows Server の Insider ビルドを使用して Docker EE をインストールする場合、Insider ビルド以外に使用するものとは異なる OneGet プロバイダーが必要です。 Docker EE と DockerMsftProvider OneGet プロバイダーが既にインストールされている場合は、続行する前に削除します。
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
```
Restart-Computer -Force
```

## <a name="install-base-container-image"></a>コンテナーの基本イメージのインストール

Windows コンテナーを使用する前に、基本イメージをインストールする必要があります。 Windows Insider Program に参加すると、これらの基本イメージの最新ビルドをテストすることもできます。 Insider の基本イメージでは、現在、Windows Server に基づく 4 つの基本イメージが用意されています。 以下の表に、それぞれの基本イメージの使用目的を示します。

| 基本 OS イメージ                       | 使い方                      |
|-------------------------------------|----------------------------|
| microsoft/windowsservercore         | 実稼働と開発 |
| microsoft/nanoserver                | 実稼働と開発 |
| microsoft/windowsservercore-insider | 開発専用           |
| microsoft/nanoserver-insider        | 開発専用           |

Nano Server Insider 基本イメージをプルするには、次のコマンドを実行します。

```
docker pull microsoft/nanoserver-insider
```

Windows Server Core Insider 基本イメージをプルするには、次のコマンドを実行します。

```
docker pull microsoft/windowsservercore-insider
```

Windows コンテナーの OS イメージの使用許諾契約書 ([ここをクリック](../EULA.md )) と Windows Insider Program の使用条件 ([ここをクリック](https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewserver)) を確認してください。

## <a name="next-steps"></a>次の手順

[.NET Core 2.0 または PowerShell Core 6 を使用した、または使用しないアプリケーションの構築と実行](./Nano-RS3-.NET-Core-and-PS.md)
