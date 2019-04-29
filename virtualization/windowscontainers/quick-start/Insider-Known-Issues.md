# <a name="known-issues-for-insider-builds"></a>内部ビルドの既知の問題

## <a name="build-16237"></a>ビルド 16237

- HYPER-V 分離が正しく動作していません。 ビルド 16237 で HYPER-V 分離を使用するには、この回避策が必要です。 PowerShell で、以下のコマンドを実行します。

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- 管理者権限が必要なコマンドが失敗するため、ユーザーは、Nano サーバーが実行されます。 "RUN setx /M PATH" などの行が含まれている場合は、ビルドできません。 このシナリオでは、以下のコマンドを代わりに使用できます。

```dockerfile
RUN setx PATH <path>
```
