# <a name="known-issues-for-insider-builds"></a>Insider ビルドの既知の問題

## <a name="build-16237"></a>ビルド 16237

- Hyper-v 分離が正常に動作していません。 この回避策は、ビルド16237で Hyper-v 分離を使用するために必要です。 PowerShell で、以下のコマンドを実行します。

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server はユーザーとして実行されるようになったため、管理者特権が必要なコマンドは失敗します。 "RUN setx /M PATH" などの行が含まれている場合は、ビルドできません。 このシナリオでは、以下のコマンドを代わりに使用できます。

```dockerfile
RUN setx PATH <path>
```
