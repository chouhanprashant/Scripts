
### 📋 Windows Server Maintenance Checklist (All-in-One PowerShell Commands)

```powershell
# 1. Server Name and Details
systeminfo

# 2. Installed Roles and Features
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Format-Table DisplayName, Name, FeatureType -AutoSize

# 3. CPU Details
Get-CimInstance Win32_Processor | Format-List Name, MaxClockSpeed, NumberOfCores, NumberOfLogicalProcessors

# 4. Memory Usage
Get-CimInstance Win32_OperatingSystem | ForEach-Object {
    "Total Memory (GB): " + [math]::Round($_.TotalVisibleMemorySize / 1MB, 2)
    "Free Memory (GB): " + [math]::Round($_.FreePhysicalMemory / 1MB, 2)
}

# 5. Antivirus Status
Get-CimInstance -Namespace "root\SecurityCenter2" -ClassName AntivirusProduct | Select-Object displayName, productState

# 6. Services in Automatic State but Not Running
Get-Service | Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -ne 'Running' } | Select-Object DisplayName, Status

# 7. System Uptime
(New-TimeSpan -Start (Get-CimInstance Win32_OperatingSystem).LastBootUpTime -End (Get-Date))

# 8. Windows Update Service Status
Get-Service -Name wuauserv

# 9. Last 5 System Errors from Event Logs
Get-EventLog -LogName System -EntryType Error -Newest 5 | Format-Table TimeGenerated, Source, Message -AutoSize

# 10. External IP Address
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-RestMethod -Uri "https://api.ipify.org?format=text"

# 11. Upcoming Scheduled Tasks (Not Currently Running)
Get-ScheduledTask | Where-Object { $_.State -ne 'Running' -and $_.NextRunTime -ne $null } | 
Sort-Object NextRunTime | 
Select-Object TaskName, NextRunTime, State | 
Format-Table -AutoSize
```
