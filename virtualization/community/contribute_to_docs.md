---
title: &681911817 コミュニティ リソース
description: コミュニティ リソース
keywords: windows 10, hyper-v, container, docker
author: scooley
manager: timlt
ms.date: 05/02/2016
ms.topic: article
ms.prod: &641512809 virtualization
ms.service: virtualization
ms.assetid: 731ed95a-ce13-4c6e-a450-49563bdc498c
---

# ドキュメントに投稿する

> <g id="1" ctype="x-strong">注:</g> ドキュメントに投稿するには、<g id="3CapsExtId1" ctype="x-link"><g id="3CapsExtId2" ctype="x-linkText">GitHub</g><g id="3CapsExtId3" ctype="x-title"></g></g> アカウントを所有している必要があります。

## 既存のドキュメントの編集

1. 編集するドキュメントを探します。

2. <g id="2" ctype="x-strong">[このトピックに投稿する]</g> を選択します。  
  <g id="1" ctype="x-linkText"></g>

  すると、そのファイルに関連付けられた GitHub のマークダウン ファイルへと自動的にリダイレクトされます。

  GitHub にサインインしていることを確認します。 サインインしていなければ、サインインするか、GitHub アカウントを作成します。

  <g id="1" ctype="x-linkText"></g>

3. 編集アイコンを選択し、ブラウザー内のエディターで編集します。

  <g id="1" ctype="x-linkText"></g>

4. インラインで変更を加えます。

  実行できる操作は次のとおりです。
  1. ファイルの編集
  2. 変更のプレビュー
  3. ファイル名の変更 (この操作が必要になることはまずありません)

  <g id="1" ctype="x-linkText"></g>

5. 変更内容をプル要求として提案します。

  <g id="1" ctype="x-linkText"></g>

6. 変更内容を確認します。

  <g id="1" ctype="x-strong">プル要求においてマイクロソフトが重視する要素</g>
  * 変更内容が正しいこと (テクノロジが正確に表現されていること)
  * スペルや文法が正しいこと
  * ドキュメント内の論理的な場所

  <g id="1" ctype="x-linkText"></g>

7. <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">プル要求</g><g id="2CapsExtId3" ctype="x-title"></g></g>の作成

## プル要求

ほとんどの変更はプル要求を介してコミットされます。 プル要求は、複数のレビューアーで変更セットを審査したり、変更を加えたり、現在のコンテンツについてコメントしたりする手段です。


## リポジトリの分岐とローカルでの編集

ドキュメントで長時間作業する場合、リポジトリをローカルにクローンし、自分のコンピューター上で作業することができます。

次に、私 (Sarah Cooley) のセットアップをエミュレートする方法を示します。 同様に正常に動作するセットアップが他にも多数あります。

> <g id="1" ctype="x-strong">注:</g> これらすべてのドキュメント ツールは、Linux または OSX でも同様に機能します。 他にガイドが必要な場合には、お問い合わせください。

これは、3 つのセクションに分かれています。
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">Git のセットアップ</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * Git のインストール
  * 最初のセットアップ
  * ドキュメント リポジトリの分岐
  * ローカル マシンへのコピーのクローン
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">資格情報の最初の管理方法</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * 資格情報の格納と資格情報ヘルパーに関する情報。
* <g id="1CapsExtId1" ctype="x-link"><g id="1CapsExtId2" ctype="x-linkText">ドキュメントの環境のセットアップ</g><g id="1CapsExtId3" ctype="x-title"></g></g>
  * VSCode のインストール
  * Git 用の VSCode。便利ないくつかの機能を説明します。
  * 最初のコミットの実行。

### Git のセットアップ

1. (Windows では) Git は<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ここ</g><g id="2CapsExtId3" ctype="x-title">からインストールします</g></g>

  インストールで変更する必要のある値は次のみです。

  <g id="1" ctype="x-strong">PATH の環境変数の調整</g>
  Windows のコマンド プロンプトから Git を使用する

  <g id="1" ctype="x-linkText"></g>

  これにより、任意の Windows コンソールの PowerShell コンソールで Git コマンドを使用できます。

2. Git ID の構成

  PowerShell ウィンドウを開き、次を実行します。

  ``` PowerShell
  git config --global user.name "User Name"
  git config --global user.email username@microsoft.com
  ```

  Git では、これらの値をコミットのラベル付けに使用します。

> 次のエラーが表示される場合、Git は正常にインストールされていないか、PowerShell を再起動する必要があります。
>    ``` PowerShell
>    git : 用語 'git' は、コマンドレット、関数、スクリプト ファイル、または操作可能なプログラムの名前として認識されません。 名前が正しく記述されていることを確認し、パスが含まれている場合はそのパスが正しいことを確認してから、再試行してください。
>    ```

3. Git 環境の構成

   (このマシンに少なくとも) 一度しかユーザー名とパスワードを入力しなくて済むように、資格情報ヘルパーをセットアップします。
   私はこのベーシックな <g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">Windows 資格情報ヘルパー</g><g id="2CapsExtId3" ctype="x-title">を使用しています</g></g>

   これをインストールしたら、次を実行して資格情報ヘルパーを有効にしてプッシュの動作を設定します。
   ```
   git config --global credential.helper manager
   git config --global push.default simple
   ```

   GitHub に対する最初の認証時には、ユーザー名と、有効にしている場合には 2 要素認証コードを求められます。
   例:
   ```
   C:\Users\plang\Source\Repos\Virtualization-Documentation [master]> git pull
   Please enter your GitHub credentials for https://github.com/
   username: plang@microsoft.com
   password:
   authcode (app): 562689
   ```
   これにより、GitHub 上の正しい資格情報が使用され、自動的に<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">個人用アクセス トークン</g><g id="2CapsExtId3" ctype="x-title"></g></g>にアクセスされます。
   次いで、ローカル マシンにそのトークンが安全に格納されます。 今後それが求められることはありません。

4. リポジトリの分岐

5. リポジトリのクローン

  Git クローンでは、同じリポジトリの他のクローンと同期するために正しいフックを使用して Git リポジトリをローカルにクローンします。

  既定では、クローンにより現在のディレクトリのリポジトリと同じ名前でフォルダーが作成されます。 私はすべての Git リポジトリを自分のユーザー ディレクトリに保持しています。 Git クローンの詳細については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">ここ</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。

  ``` PowerShell
  cd ~
  git clone https://github.com/Microsoft/Virtualization-Documentation.git
  ```

  成功した場合、<g id="2" ctype="x-code">Virtualization-Documentation</g> フォルダーが作成されているはずです。

  ``` PowerShell
  cd Virtualization-Documentation
  ```

5. [省略可能] Posh-Git のセットアップ

  Posh-Git は、PowerShell で Git を若干使いやすくする、コミュニティで作成された PowerShell モジュールです。 これは、PowerShell の Git にタブ補完を追加し、分岐やファイルの状態の迅速な表示も有益にします。 詳細については、<g id="2CapsExtId1" ctype="x-link"><g id="2CapsExtId2" ctype="x-linkText">こちら</g><g id="2CapsExtId3" ctype="x-title"></g></g>を参照してください。 管理者の PowerShell コンソールで以下を実行すると、Posh-Git をインストールできます。

  ``` PowerShell
  Install-Module -Name posh-git
  ```

  PowerShell を起動するたびに Posh-Git が自動的に動作をするようにするには、次のコードを PowerShell のプロファイルに追加します (例: <g id="2" ctype="x-code">%UserProfile%\My Documents\WindowsPowerShell\profile.ps1</g>)

  ``` PowerShell
  Import-Module posh-git

  function global:prompt {
    $realLASTEXITCODE = $LASTEXITCODE

    Write-Host($pwd.ProviderPath) -nonewline

    Write-VcsStatus

    $global:LASTEXITCODE = $realLASTEXITCODE
    return "> "
  }
  ```

### 資格情報の検証と格納

  リポジトリが正しくセットアップされたことの検証には、新しいコンテンツの取得を試行します。

  ``` PowerShell
  git pull
  ```


### マークダウン編集環境のセットアップ

1. VSCode のダウンロード

6. テスト コミットの実行。 資格情報が正しく格納されている場合、正しく動作するはずです。









<!--HONumber=May16_HO1-->


