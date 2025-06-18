# 📋 Windows Server Maintenance Checklist

---

## 🖥️ 1. Server Name and Details  
📋 **Command:**
```
systeminfo
```

---

## 🧰 2. Installed Roles and Features  
📋 **Command:**
```
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Format-Table DisplayName, Name, FeatureType -AutoSize
```

---

## 🧠 3. CPU Details  
📋 **Command:**
```
Get-CimInstance Win32_Processor | Format-List Name, MaxClockSpeed, NumberOfCores, NumberOfLogicalProcessors
```

---

## 💾 4. Memory Usage  
📋 **Command:**
```
Get-CimInstance Win32_OperatingSystem | ForEach-Object {
    "Total Memory (GB): " + [math]::Round($_.TotalVisibleMemorySize / 1MB, 2)
    "Free Memory (GB): " + [math]::Round($_.FreePhysicalMemory / 1MB, 2)
}
```

---

## 🛡️ 5. Antivirus Status  
📋 **Command:**
```
Get-CimInstance -Namespace "root\\SecurityCenter2" -ClassName AntivirusProduct | Select-Object displayName, productState
```

---

## ⚙️ 6. Services in Automatic State but Not Running  
📋 **Command:**
```
Get-Service | Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -ne 'Running' } | Select-Object DisplayName, Status
```

---

## ⏱️ 7. System Uptime  
📋 **Command:**
```
(New-TimeSpan -Start (Get-CimInstance Win32_OperatingSystem).LastBootUpTime -End (Get-Date))
```

---

## 🔄 8. Windows Update Service Status  
📋 **Command:**
```
Get-Service -Name wuauserv
```

---

## 🛑 9. Check for Any Critical System Events  
📋 **Command:**
```
$criticalEvents = Get-WinEvent -LogName System | Where-Object { $_.LevelDisplayName -eq "Critical" }
if ($criticalEvents) {
    $criticalEvents | Format-Table TimeCreated, ProviderName, Message -AutoSize
} else {
    Write-Output "No critical events found in the System log."
}
```

---

## 🌐 10. External IP Address  
📋 **Command:**
```
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Invoke-RestMethod -Uri "https://api.ipify.org?format=text"
```

---

## 📅 11. Upcoming Scheduled Tasks  
📋 **Command:**
```
Get-ScheduledTask | ForEach-Object {
    try {
        $info = Get-ScheduledTaskInfo -TaskName $_.TaskName -TaskPath $_.TaskPath
        [PSCustomObject]@{
            TaskName     = $_.TaskName
            TaskPath     = $_.TaskPath
            NextRunTime  = if ($info.NextRunTime) { $info.NextRunTime } else { "N/A" }
        }
    } catch {
        [PSCustomObject]@{
            TaskName     = $_.TaskName
            TaskPath     = $_.TaskPath
            NextRunTime  = "Access Denied or Unavailable"
        }
    }
} | Sort-Object NextRunTime | Format-Table -AutoSize
```

---

✅ Let me know if you'd like this exported to PDF or a `.ps1` script file!
