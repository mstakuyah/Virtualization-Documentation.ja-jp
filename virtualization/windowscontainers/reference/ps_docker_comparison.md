# Windows コンテナーを管理するための PowerShell と Docker の比較

インボックス Windows ツール (このプレビューでは PowerShell) とオープン ソース管理ツール (Docker など) の両方を使用して Windows コンテナーを管理するには、さまざまな方法があります。  
それぞれの概要を示す次のガイドを参照してください。
* [Manage Windows Containers with Docker (Docker を使用した Windows コンテナーの管理)](../quick_start/manage_docker.md)
* [Manage Windows Containers with PowerShell (PowerShell を使用した Windows コンテナーの管理)](../quick_start/manage_powershell.md)

このページでは、Docker ツールと PowerShell 管理ツールについて、両者を比較しながら詳しく説明します。

## コンテナー用 PowerShell と Hyper-V VM

PowerShell コマンドレットを使用して、Windows コンテナーを作成、実行、および操作することができます。 作業を開始するのに必要なものはすべて、インボックスで提供されています。

Hyper-V PowerShell を使用したことがあれば、コマンドレットの設計はわかりやすいはずです。 ワークフローの多くの部分は、Hyper-V モジュールを使用して仮想マシンを管理する方法に似ています。 `New-VM`、`Get-VM`、`Start-VM`、`Stop-VM` の代わりに、`New-Container`、`Get-Container`、`Start-Container`、`Stop-Container` を使用します。 さまざまなコンテナー固有のコマンドレットおよびパラメーターがありますが、Windows コンテナーの全般的なライフ サイクルと管理は、Hyper-V VM の場合とほぼ同じです。

## PowerShell 管理を Docker の場合と比較してみましょう。

コンテナー PowerShell コマンドレットは API を公開しますが、これは Docker の API とまったく同じわけではありません。一般的に、コマンドレットの方が操作が細やかです。 次のように、一部の Docker コマンドは基本的に PowerShell に似ています。

| Docker コマンド| PowerShell コマンドレット|
|----|----|
| `docker ps -a`| `Get-Container`|
| `docker images`| `Get-ContainerImage`|
| `docker rm`| `Remove-Container`|
| `docker rmi`| `Remove-ContainerImage`|
| `docker create`| `New-Container`|
| `docker commit <コンテナー ID>`| `New-ContainerImage -Container <コンテナー>`|
| `docker load <tarball>`| `Import-ContainerImage <AppX パッケージ>`|
| `docker save`| `Export-ContainerImage`|
| `docker start`| `Start-Container`|
| `docker stop`| `Stop-Container`|

PowerShell コマンドレットは完全なパリティではなく、PowerShell による代替が提供されていないコマンドも多く存在します* (特に `docker build` や `docker cp`)。 中でも、`docker run` に対して 1 行の代替が提供されていないことは意外かもしれません。

\* 変更されることがあります。

### docker run の機能を実現するには

PowerShell を既に使いこなしているユーザー向けに、わかりやすい操作モデルを次に示します。 docker の機能に慣れている場合は、少し視点を変えてみましょう。

1.  PowerShell モデルのコンテナーのライフサイクルはわずかに異なります。 コンテナー PowerShell モジュールでは、`New-Container` (停止している新しいコンテナーを作成する) および `Start-Container` のより細やかな操作を公開します。

    コンテナーを作成してから起動するまでに、コンテナーの設定を構成することもできます。TP3 では、公開する予定の他の唯一の構成は、コンテナーに対してネットワーク接続を設定する機能です。 その場合、(Add/Remove/Connect/Disconnect/Get/Set)-ContainerNetworkAdapter コマンドレットを使用します。

2.  現在のところ、起動時にコンテナー内で実行するコマンドを渡すことはできません。ただし、`Enter-PSSession -ContainerId <実行中のコンテナーの ID>` を使用して、実行中のコンテナーに対して対話型の PowerShell セッションを取得したり、`Invoke-Command -ContainerId <コンテナー ID> -ScriptBlock {コンテナー内で実行するコード}` または `Invoke-Command -ContainerId <コンテナー ID> -FilePath <スクリプトのパス>` を使用して、実行中のコンテナー内でコマンドを実行することはできます。  
    これらのコマンドは両方とも、権限の高いアクションに対して省略可能な `-RunAsAdministrator` フラグを使用できます。


## 注意事項と既知の問題

1.  現在のところ、コンテナー コマンドレットは、Docker を使用して作成されたコンテナーやイメージについて何も認識せず、Docker は、PowerShell を使用して作成されたコンテナーやイメージについて何も認識しません。 Docker で作成した場合は Docker で管理し、PowerShell で作成した場合は PowerShell で管理します。

2.  エラー メッセージの向上、進行状況レポートの向上、無効なイベント文字列など、エンド ユーザー エクスペリエンスを向上させるために実行すべき作業はかなりあります。 より多くの情報や、より良質の情報が必要になった場合は、ご提案をフォーラムまでお送りください。

## 概要

次に、一般的なワークフローについて説明します。

ここでは、"ServerDatacenterCore" という名前の OS コンテナー イメージをインストールしており、"Virtual Switch" という名前の仮想スイッチを作成した (New-VMSwitch を使用) と仮定します。

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

### 各自のサンプルの作成

`Get-Command -Module Containers` を使用して、すべてのコンテナー コマンドレットを表示できます。 ここで説明されていないその他のコマンドレットについては、各自でご確認ください。    
**注** これにより、コア PowerShell に含まれる `Enter-PSSession` および `Invoke-Command` コマンドレットは返されません。

`Get-Help [コマンドレット名]`、または同等の `[コマンドレット名] -?` を使用して、任意のコマンドレットに関するヘルプを表示することもできます。 現時点では、ヘルプ出力は自動生成され、コマンドの構文のみが示されます。コマンドレット設計の最終処理が近づいたら、さらに詳細なドキュメントを追加する予定です。

PowerShell をあまり使用したことがない場合はご存知ないかもしれませんが、構文を検出するより良い方法として、PowerShell ISE があります。 ISE を許可する SKU 上で実行している場合は、コマンド ウィンドウを開き、「コンテナー」モジュールを選択して、ISE を起動してみてください。これにより、コマンドレットとそのパラメーター セットがグラフィック表示されます。

PS: 次に、この操作が可能であることを証明するために、既に確認したいくつかのコマンドレットから代用の `docker run` を構成する PowerShell 関数を示します (これは概念を証明するもので、アクティブに開発しているものではありません)。

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

Windows コンテナーは、Docker コマンドを使用して管理できます。 Windows コンテナーは対応する Linux コンテナーと同等であり、Docker を使用して同じ管理エクスペリエンスが得られるはずですが、Docker コマンドの中には、Windows コンテナーでは機能しないものがあります。 また、まだテストしていないものもあります (今後テスト予定です)。

Docker で利用可能な API ドキュメントが重複しないよう、その管理 API のリンクを示します。 これは非常に有用なチュートリアルです。

当社の「進行中の作業」のドキュメントでは、Docker API で機能する操作と機能しない操作を追跡しています。



