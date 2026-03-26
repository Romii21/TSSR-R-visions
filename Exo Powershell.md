Voici 3 scripts à commenter ! Remplace chaque `# XXX` par un commentaire qui explique ce que fait le bloc en dessous.

---

**Script 1 — `GetServiceInfo.ps1`**

powershell

```powershell
Clear-Host

# XXX
$Services = Get-Service | `
    Where-Object {$_.Status -eq "Running"} | `
    Select-Object -Property Name, DisplayName, Status

# XXX
$Result = $Services | Where-Object {$_.Name -notlike "win*"}

# XXX
$Result | Format-Table -AutoSize

# XXX
$Result | Export-Csv -Path "C:\Scripts\services.csv" -Delimiter ";" -NoTypeInformation
```

---

**Script 2 — `GetUserInfo.ps1`**

powershell

```powershell
Clear-Host

# XXX
$Users = Get-LocalUser | `
    Where-Object {$_.Enabled -eq $true} | `
    Select-Object -Property Name, FullName, PasswordExpires

# XXX
$DisabledUsers = Get-LocalUser | `
    Where-Object {$_.Enabled -eq $false} | `
    Select-Object -Property Name, FullName

# XXX
Write-Host "Utilisateurs actifs :" -ForegroundColor Green
$Users

# XXX
Write-Host "Utilisateurs désactivés :" -ForegroundColor Red
$DisabledUsers
```

---

**Script 3 — `GetProcessInfo.ps1`**

powershell

```powershell
Clear-Host

# XXX
$Processes = Get-Process | `
    Select-Object -Property Name, CPU, WorkingSet | `
    Sort-Object -Property CPU -Descending

# XXX
$TopProcesses = $Processes | Select-Object -First 5

# XXX
foreach ($Process in $TopProcesses)
{
    Write-Host "Processus : $($Process.Name) - CPU : $($Process.CPU)" -ForegroundColor Yellow
}

# XXX
$TopProcesses | Export-Csv -Path "C:\Scripts\processes.csv" -Delimiter ";" -NoTypeInformation
```

---

Prends le temps de lire chaque bloc et propose tes commentaires **un script à la fois** ! On commence par lequel ? 🎯