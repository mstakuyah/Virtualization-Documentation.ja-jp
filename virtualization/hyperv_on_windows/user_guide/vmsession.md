# PowerShell ダイレクトでの Windows の管理

PowerShell ダイレクトを使用して、Windows 10 または Windows Server Technical Preview 仮想マシンを、Windows 10 または Windows Server Technical Preview Hyper-V ホストからリモートで管理できます。 PowerShell ダイレクトでは、ネットワーク構成に関係なく仮想マシン内の PowerShell 管理や、Hyper-V ホストまたは仮想マシン全体のリモート管理設定が可能です。 これを使うと、Hyper-V 管理者が仮想マシンの管理や設定を容易に自動化したり、スクリプトを作成したりすることができます。

PowerShell ダイレクトを実行するには、次の 2 つの方法があります。
* 対話型セッションとして -- [このセクションに移動して](vmsession.md#create-and-exit-an-interactive-powershell-session)、PSSession コマンドレットを使用し、PowerShell Direct セッションを作成および終了する
* 一連のコマンドまたはスクリプトを実行するには -- [このセクションに移動して](vmsession.md#run-a-script-or-command-with-invoke-command)、Invoke-Command コマンドレットを使用し、スクリプトまたはコマンドを実行する


## 要件

**オペレーティング システムの要件:**
* ホスト オペレーティング システムでは、Windows 10、Windows Server Technical Preview、または新しいバージョンを実行する必要があります。
* 仮想マシンでは、Windows 10、Windows Server Technical Preview、または新しいバージョンを実行する必要があります。

以前の仮想マシンを管理している場合は、仮想マシン接続 (VMConnect) を使用するか、[仮想マシン用の仮想ネットワークを構成します](http://technet.microsoft.com/library/cc816585.aspx)。

仮想マシンで PowerShell ダイレクト セッションを作成するには
* 仮想マシンはホストのローカルで実行されており、起動する必要があります。
* Hyper-V 管理者としてホスト コンピューターにログインする必要があります。
* 仮想マシンの有効なユーザー資格情報を指定する必要があります。

## 対話型 PowerShell セッションの作成と終了

1. Hyper-V ホストで、管理者として Windows PowerShell を開きます。

3. 仮想マシン名または GUID を使用してセッションを作成するには、次のコマンドのいずれかを実行します。
``` PowerShell
Enter-PSSession -VMName <VMName>
Enter-PSSession -VMGUID <VMGUID>
```

4. 必要なコマンドを実行します。 これらのコマンドは、セッションを作成した仮想マシン上で実行されます。
5. 終了したら、次のコマンドを実行してセッションを終了します。
``` PowerShell
Exit-PSSession 
```

> 注: セッションが接続されない場合は、接続している仮想マシンの資格情報を使用していることを確認してください (Hyper-V ではありません)。

これらのコマンドレットの詳細については、「[Enter-PSSession](http://technet.microsoft.com/library/hh849707.aspx)」と「[Exit-PSSession](http://technet.microsoft.com/library/hh849743.aspx)」を参照してください。

## Invoke-Command でのスクリプトまたはコマンドの実行

[Invoke-Command](http://technet.microsoft.com/library/hh849719.aspx) コマンドレットを使用して、仮想マシン上であらかじめ設定された一連のコマンドを実行できます。 仮想マシン名が PSTest であり、実行するスクリプト (foo.ps1) が C:/ ドライブ上のスクリプト フォルダー内にある場合に、Invoke-Command コマンドレットを使用する方法の例を以下に示します。

 ``` PowerShell
 Invoke-Command -VMName PSTest -FilePath C:\script\foo.ps1 
 ```

1 つのコマンドを使用するには、**-ScriptBlock** パラメーターを使用します。

 ``` PowerShell
 Invoke-Command -VMName PSTest -ScriptBlock { cmdlet } 
 ```

## トラブルシューティング

PowerShell ダイレクトを通じて表示される一般的なエラー メッセージが少しあります。 いくつかの最も一般的な原因、および問題を診断するためのツールは、次のとおりです。

### エラー: リモート セッションが終了した可能性がある

エラー メッセージ:
```
An error has occured which Windows PowerShell cannot handle.  A remote session might have ended. 
```

考えられる原因:
* VM が実行されていない
* ゲスト OS で PowerShell Direct をサポートしていない (「[要件](#Requirements)」を参照してください)
* PowerShell がまだゲストで使用できない
    * オペレーティング システムが起動を完了していない
    * オペレーティング システムが正しく起動できない
    * ユーザー入力が必要な起動時のイベントがある
* ゲストの資格情報を検証できない

[Get-VM](http://technet.microsoft.com/library/hh848479.aspx) コマンドレットを使用して、Hyper-V 管理者の役割を使用している資格情報を確認し、ホスト上でローカルで実行されている、起動した VM を確認します。

## サンプル

[GitHub](https://github.com/Microsoft/Virtualization-Documentation/search?l=powershell&q=-VMName+OR+-VMGuid&type=Code&utf8=%E2%9C%93) のサンプルを確認します。

使用している環境で PowerShell Direct を使用する方法についてのさまざまな例、および PowerShell で Hyper-V スクリプトを記述するためのヒントとコツについては、「[PowerShell Direct snippets](../develop/powershell_snippets.md)」を参照してください。



