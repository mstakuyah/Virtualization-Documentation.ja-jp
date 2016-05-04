



# 管理相互運用性

**この記事は暫定的な内容であり、変更される可能性があります。**

ほとんどの場合、PowerShell で作成された Windows コンテナーは、PowerShell を使用して管理する必要があり、Docker で作成された Windows コンテナーは、Docker を使用して管理する必要があります。 ただし、ホスト コンピューティング PowerShell モジュールは、コンテナーの作成方法に関係なく、**実行中の**コンテナーの検出および停止を行う機能を提供します。 このモジュールは、コンテナー ホストで実行されているコンテナーの「タスク マネージャー」のように機能します。

## すべてのコンテナーの表示

コンテナーの一覧を取得するには、`Get-ComputeProcess` コマンドを使用します。

```powershell
PS C:\> Get-ComputeProcess

Id                                                Name                                      Owner       Type
--                                                ----                                      -----       ----
2088E0FA-1F7C-44DE-A4BC-1E29445D082B              DEMO1                                     VMMS   Container
373959AC-1BFA-46E3-A472-D330F5B0446C              DEMO2                                     VMMS   Container
d273c80b6e..                                      d273c80b6e..                              docker Container
e49cd35542..                                      e49cd35542..                              docker Container
```

## コンテナーの停止

コンテナーが PowerShell または Docker のいずれを使用して作成されたかに関係なく、コンテナーを停止するには、`Stop-ComputeProcess` コマンドを使用します。

> この記事の執筆時に、`Get-Container` コマンドを使用すると、コンテナーを停止として表示するには、VMMS サービスを再起動する必要があります。

```powershell
PS C:\> Stop-ComputeProcess -Id 2088E0FA-1F7C-44DE-A4BC-1E29445D082B -Force
```






<!--HONumber=Feb16_HO3-->


