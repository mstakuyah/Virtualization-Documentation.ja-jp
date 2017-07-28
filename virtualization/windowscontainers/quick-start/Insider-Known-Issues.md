# Insider ビルドに関する既知の問題

## ビルド 16237

- Hyper-V コンテナーが正しく機能しません。 ビルド 16237 で Hyper-V コンテナーを使用するには、この回避策を実行する必要があります。 PowerShell で、以下のコマンドを実行します。

```PowerShell
Get-ComputeProcess | ? IsTemplate -eq $true | Stop-ComputeProcess -Force
Set-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Virtualization\Containers\' -Name TemplateVmCount -Type dword -Value 0 -Force
```

- Nano Server がユーザーとして実行されるようになったため、管理者特権を必要とするコマンドが失敗します。 "RUN setx /M PATH" などの行が含まれている場合は、ビルドできません。 このシナリオでは、以下のコマンドを代わりに使用できます。

```dockerfile
RUN setx PATH <path>
```
