# コンテナーのリソース管理

**この記事は暫定的な内容であり、変更される可能性があります。**

Windows コンテナーには、コンテナーが消費できる CPU 使用量、ディスク IO、ネットワーク リソース、メモリ リソースを管理する機能があります。 コンテナー リソース消費を制限することで、ホスト リソースを効率的に使用し、過度な消費を防ぐことができます。 このドキュメントでは、PowerShell と Docker の両方を使用してコンテナー リソースを管理する方法について説明します。

## PowerShell を使用したリソース管理

### メモリ

コンテナー メモリの制限は、`New-Container` コマンドの `-MaximumMemoryBytes` パラメーターを使用してコンテナーを作成するときに設定できます。 この例では、256 MB の最大メモリを設定します。

```powershell
PS C:\> New-Container –Name TestContainer –MaximumMemoryBytes 256MB -ContainerimageName WindowsServerCore
```
`Set-ContainerMemory` コマンドレットを使用して、既存のコンテナーのメモリ制限を設定することもできます。

```powershell
PS C:\> Set-ContainerMemory -ContainerName TestContainer -MaximumBytes 256mb
```

### ネットワーク帯域幅

ネットワーク帯域幅の制限は、既存のコンテナーで設定できます。 この場合、`Get-ContainerNetworkAdapter` コマンドを使用して、コンテナーにネットワーク アダプターがあることを確認します。 ネットワーク アダプターが存在しない場合は、`Add-ContainerNetworkAdapter` コマンドを使用して作成します。 最後に、`Set-ContainerNetworkAdapter` コマンドを使用して、コンテナーの最大送信ネットワーク帯域幅を制限します。

次の例では、最大帯域幅を 100 Mbps に設定しています。

```powershell
PS C:\> Set-ContainerNetworkAdapter –ContainerName TestContainer –MaximumBandwidth 100000000
```

### CPU

コンテナーが使用できるコンピューティング量を制限するには、CPU の最大割合 (%) を設定するか、コンテナーの相対的な重みを設定します。 これら 2 つの CPU 管理スキームを混在させることもできますが、推奨されません。 既定では、すべてのコンテナーはすべてのプロセッサを利用し (100% の最大値)、相対的な重みは 100 です。

次の例では、コンテナーの相対的な重みを 1000 に設定します。 コンテナーの既定の重みは 100 なので、既定に設定されているコンテナーの 10 倍の優先度になります。 最大値は、10000 です。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 –RelativeWeight 10000
```

また、コンテナーが使用できる CPU 量のハードの制限を、CPU 時間の割合単位で設定することもできます。 既定では、コンテナーは 100% の CPU を使用できます。 次の例では、コンテナーが使用できる CPU の最大割合を 30% に設定しています。 –Maximum フラグを使用すると、自動的に RelativeWeight は 100 に設定されます。

```powershell
PS C:\> Set-ContainerProcessor -ContainerName Container1 -Maximum 30
```

### 記憶域 IO

コンテナーが使用できる IO 量を、帯域幅単位 (バイト/秒) または 8k で正規化された IOPS 単位で制限できます。 これら 2 つのパラメーターは同時に設定できます。 最初の制限に達すると、スロットルが発生します。

```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumBandwidth 1000000
```
```powershell
PS C:\> Set-ContainerStorage -ContainerName Container1 -MaximumIOPS 32
```

## Docker を使用したリソース管理

Docker でコンテナー リソースの一部を管理する機能があります。 具体的には、ユーザーは複数のコンテナーで CPU を共有する方法を指定できます。

### CPU

複数のコンテナー間の CPU 共有は、ランタイムで --cpu-shares フラグを指定して管理できます。 既定では、すべてのコンテナーが同割合の CPU 時間を利用します。 コンテナーが使用する CPU の相対的な共有量を変更するには、--cpu-shares フラグを 1 から 10000 の値を指定して実行します。 既定では、すべてのコンテナーに 5000 の重みが与えられます。 CPU 共有の制約に関する詳細については、[Docker Run のリファレンス](https://docs.docker.com/engine/reference/run/#cpu-share-constraint)を参照してください。

```powershell 
C:\> docker run –it --cpu-shares 2 --name dockerdemo windowsservercore cmd
```

## の既知の問題

- 現在、CPU と IO リソース制御は、Hyper-V コンテナーでサポートされていません。
- 現在、IO リソース制御は、コンテナー共有フォルダーでサポートされていません。

## ビデオ チュートリアル

<iframe src="https://channel9.msdn.com/Blogs/containers/Container-Fundamentals--Part-4-Resource-Management/player" width="800" height="450"  allowFullScreen="true" frameBorder="0" scrolling="no"></iframe>





<!--HONumber=Feb16_HO1-->
