# Powershell

Shows the windows version

```powershell
winver
```

Get-windowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

Start-Process powershell -verb runas #Run powershell as admin case insensitive

Start-Process wt -verb runas #Run windows terminal as admin case insensitive


Self elevating Powershell script

```powershell
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) { Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`"" -Verb RunAs; exit }
```



```powershell
if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")) { Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`" `"$args`"" -Verb RunAs; exit }
```
