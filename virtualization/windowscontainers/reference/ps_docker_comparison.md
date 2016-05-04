



# Windows コンテナーを管理するための PowerShell と Docker の比較

インボックス Windows ツール (このプレビューでは PowerShell) とオープン ソース管理ツール (Docker など) の両方を使用して Windows コンテナーを管理するには、さまざまな方法があります。  
ここではアウトラインの両方を個別に使用可能なガイド:
* [Docker を使用した Windows コンテナーの管理](../quick_start/manage_docker.md)
* [PowerShell を使用した Windows コンテナーの管理](../quick_start/manage_powershell.md)

このページでは、Docker ツールと PowerShell 管理ツールの比較の深さの参照になります。

## コンテナー用 PowerShell と Hyper-V VM

PowerShell コマンドレットを使用して、Windows コンテナーを作成、実行、および操作することができます。 作業を開始するのに必要なものはすべて、インボックスで提供されています。

PowerShell の HYPER-V を使用した場合、コマンドレットの設計がよく知っている場合があります。 ワークフローの多くは、HYPER-V モジュールを使用して、仮想マシンを管理する方法に似ています。 `New-VM`、`Get-VM`、`Start-VM`、`Stop-VM` の代わりに、`New-Container`、`Get-Container`、`Start-Container`、`Stop-Container` を使用します。 多くのコンテナーに固有のコマンドレットとパラメーターが、全般的なライフ サイクルがあるし、Windows コンテナーの管理は、HYPER-V の仮想マシンの場合と同様にほぼ探します。

## PowerShell 管理は Docker にどのような比較をするでしょうか。

Docker の; とまったく同じではない API を公開する、コンテナーの PowerShell コマンドレット一般的な規則として、コマンドレットは、さらに詳細に操作します。 Docker のいくつかのコマンドでは、PowerShell で緯線の非常に簡単ですがあります。

| Docker コマンド| PowerShell コマンドレット|
|----|----|
| `docker ps -a`| `Get-Container`|
| `docker images`| `Get-ContainerImage`|
| `docker rm`| `Remove-Container`|
| `docker rmi`| `Remove-ContainerImage`|
| `docker create`| `New-Container`|
| `docker commit <コンテナー ID>`| `New-ContainerImage -Container <コンテナー>`|
| `docker load &lt;tarball&gt;`| `Import-ContainerImage <AppX パッケージ>`|
| `docker save`| `Export-ContainerImage`|
| `docker start`| `Start-Container`|
| `docker stop`| `Stop-Container`|

PowerShell コマンドレットは完全なパリティではなく、PowerShell による代替が提供されていないコマンドも多く存在します* (特に `docker build` や `docker cp`)。 中でも、`docker run` に対して 1 行の代替が提供されていないことは意外かもしれません。

\ * 変更される可能性があります。

### Docker 必要がありますが、実行します。何が起こっているでしょうか。

PowerShell には、その方法を既に知っているユーザーに若干の他の一般的な対話モデルを提供する次の点をここで行ってきました。 もちろん、docker の動作に慣れて、これは少し観念の移行のなります。

1.  PowerShell のモデルのコンテナーのライフ サイクルは若干異なります。 コンテナー PowerShell モジュールでは、`New-Container` (停止している新しいコンテナーを作成する) および `Start-Container` のより細やかな操作を公開します。

  作成して、コンテナーの開始、間に、コンテナーの設定を構成することもTP3、コンテナーのネットワーク接続を設定する機能は、その他の構成のみを公開することを計画してです。 (追加/削除/接続/切断/取得/設定) を使用して-ContainerNetworkAdapter コマンドレットです。

2.  現在、コンテナー内の起動時に実行するコマンドを渡すことはできません。ただし、`Enter-PSSession -ContainerId <実行中のコンテナーの ID>` を使用して、実行中のコンテナーに対して対話型の PowerShell セッションを取得したり、`Invoke-Command -ContainerId <コンテナー ID> -ScriptBlock {コンテナー内で実行するコード}` または `Invoke-Command -ContainerId <コンテナー ID> -FilePath <スクリプトのパス>` を使用して、実行中のコンテナー内でコマンドを実行することはできます。  
これらのコマンドは両方とも、権限の高いアクションに対して省略可能な `-RunAsAdministrator` フラグを使用できます。


## 注意事項と既知の問題

1.  現在、コンテナーのコマンドレットを使用しないため、コンテナーまたは Docker、によって作成されたイメージに関するに関する知識を持って Docker が認識していないすべてのコンテナーとイメージ、PowerShell を使用して作成します。 Docker 内で作成した場合は Docker; で管理します。powershell で作成した場合は、PowerShell を使って管理します。

2.  多くの作業をエラー メッセージの向上より優れた進行状況レポート、無効なイベント文字列、およびなど--、エンド ユーザー エクスペリエンスを向上させるために必要があります。 状況の詳細を受け取っている場合に実行した場合や、情報の向上、ご自由にフォーラムをご提案を送信します。

## クイック runthrough

ここでは、いくつかの一般的なワークフローの説明です。

これには、"ServerDatacenterCore"という名前の OS コンテナー イメージをインストールし、「仮想スイッチ」(New-vmswitch を使用) という名前の仮想スイッチを作成したと仮定しています。

``` PowerShell
### 1. Enumerating images
# At this point, you can enumerate the images on the system:
Get-ContainerImage

# Get-ContainerImage also accepts filters.
# For example, this will return all container images whose Name starts with S (case-insensitive):
Get-ContainerImage -Name S*

# You can save the results of this to a variable.
# (If you're not familiar with PowerShell, the "$" denotes a variable.)
$baseImage = Get-ContainerImage -Name ServerDatacenterCore
$baseImage

### 2. Creating and enumerating containers
# Now, we can create a new container using this image:
New-Container -Name "MyContainer" -ContainerImage $baseImage -SwitchName "Virtual Switch"

# Now we can enumerate all containers.
Get-Container

# Similarly, we can save this container to a variable:
$container1 = Get-Container -Name "MyContainer"

### 3. Starting containers, interacting with running containers, and stopping containers
# Now let's go ahead and start the container.
Start-Container -Name "MyContainer"

# (We could've also started this container using "Start-Container -Container $container1".)

# With the container now running, let's go ahead and enter an interactive PowerShell session:
Enter-PSSession -ContainerId $container1.Id

# This should eventually bring up a PowerShell prompt from inside the container.
# You can try all the things that you did in the interactive cmd prompt given by "docker run -it".
# For now, just to prove we've been here, we can create a new file:
cd \
mkdir Test
cd Test
echo "hello world" > hello.txt
exit

# Now we should be back in the outside world. Even though we've exited the PowerShell session,
# the container itself is still running, as you can see by printing out the container again:
$container1

# Before we can commit this container to a new image, we need to stop the container.
# Let's do that now.
Stop-Container -Container $container1

### 4. Creating a new container image
# And now let's commit it to a new image.
$image1 = New-ContainerImage -Container $container1 -Publisher Test -Name Image1 -Version 1.0

# Enumerate all the images again, for sanity's sake:
Get-ContainerImage

# Rinse and repeat! Make another container based on the new image.
$container2 = New-Container -Name "MySecondContainer" -ContainerImage $image1 -SwitchName "Virtual Switch"

# (If you like, you can start the second container and verify that the new file
# "\Test\hello.txt" is there as expected.)

### 5. Removing a container
# The first container we created is now stopped. Let's get rid of it:
Remove-Container -Container $container1

# And confirm that it's gone:
Get-Container

### 6. Exporting, removing, and importing images
# For images that aren't the base OS image, we can export them into an .appx package file.
Export-ContainerImage -Image $image1 -Path "C:\exports"
# This should create a .appx file in the C:\exports folder.
# If you've given your image the same publisher, name, and version we used earlier,
# you'd expect the resulting .appx to be named "CN=Test_Image1_1.0.0.0.appx".

# Before we can try importing the image again, we need to remove the image.
# (If you have any running containers that depend on this image, you'll want to stop them first.)
Remove-ContainerImage -Image $image1

# Now let's import the image again:
Import-ContainerImage -Path C:\exports\CN=Test_Image1_1.0.0.0.appx

# We'd previously created a container dependent on this image. You should be able to start it:
Start-Container -Container $container2 
```

### 独自のサンプルをビルドします。

`Get-Command -Module Containers` を使用して、すべてのコンテナー コマンドレットを表示できます。 これは、いくつか他のコマンドレットは、ここで説明されていない、自分でについて学習するためのままにします。    
**注** これにより、コア PowerShell に含まれる `Enter-PSSession` および `Invoke-Command` コマンドレットは返されません。

`Get-Help [コマンドレット名]`、または同等の `[コマンドレット名] -?` を使用して、任意のコマンドレットに関するヘルプを表示することもできます。 今日では、ヘルプの出力は自動生成され、コマンドの構文を知らせるだけ追加される予定さらにドキュメント、コマンドレットのデザインを最終処理中に近づくにつれて、します。

構文を検出するより良い方法は、可能性がありますいないを調べる前に PowerShell は非常によく使用することがない場合を PowerShell ISE にです。 これを可能にするための SKU で実行している場合は、ISE で、コマンド ウィンドウを開くと、コマンドレットとそのパラメーターのセットがグラフィック表示を表示する「コンテナー」モジュールを選択するを起動してみてください。

PS: 次に、この操作が可能であることを証明するために、既に確認したいくつかのコマンドレットから代用の `docker run` を構成する PowerShell 関数を示します (確かに、これは、アクティブな開発ではなく、概念実証) です。

``` PowerShell
function Run-Container ([string]$ContainerImageName, [string]$Name="fancy_name", [switch]$Remove, [switch]$Interactive, [scriptblock]$Command) {
    $image = Get-ContainerImage -Name $ContainerImageName
    $container = New-Container -Name $Name -ContainerImage $image
    Start-Container $container

    if ($Interactive) {
         Start-Process powershell ("-NoExit", "-c", "Enter-PSSession -ContainerId $($container.Id)") -Wait
    } else {
        Invoke-Command -ContainerId $container.Id -ScriptBlock $Command
    }

    Stop-Container $container

    if ($Remove) {
        Remove-Container $container -Force
    }
} 
```

## Docker

Windows コンテナーは、Docker コマンドを使用して管理できます。 Windows コンテナーは対応する Linux コンテナーと同等であり、Docker を使用して同じ管理エクスペリエンスが得られるはずですが、Docker コマンドの中には、Windows コンテナーでは機能しないものがあります。 他のユーザーだけではまだされてテスト (取得があります)。

いない Docker で利用可能な API のドキュメントを複製するために、次に、管理 Api へのリンクを示します。 そのチュートリアルではすばらしいです。

いるものと、進行中の作業の文書の Docker Api では機能しない管理するいるとします。





<!--HONumber=Feb16_HO3-->


