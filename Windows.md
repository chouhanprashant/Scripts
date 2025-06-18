
# Define report path
$serverName = $env:COMPUTERNAME
$reportFolder = "C:\Reports"
$reportPath = "$reportFolder\Server_Report_$serverName.txt"

# Create folder if not exists
if (!(Test-Path $reportFolder)) {
    New-Item -ItemType Directory -Path $reportFolder -Force | Out-Null
}

# Clear any existing report
if (Test-Path $reportPath) {
    Remove-Item $reportPath -Force
}

# Server Name and Details
"Server Name and Details:`n======================================================================================================" | Out-File $reportPath
systeminfo | Out-String | Add-Content $reportPath

# Roles and Features
"`n--------------------------------------------`nRoles and Features:`n" | Add-Content $reportPath
Get-WindowsFeature | Where-Object {$_.Installed -eq $true} | Format-Table DisplayName, Name, FeatureType -AutoSize | Out-String | Add-Content $reportPath

# Resource Utilization
"`n--------------------------------------------`nResource Utilization(CPU, RAM, Processor):`n" | Add-Content $reportPath
"CPU:`n" | Add-Content $reportPath
Get-CimInstance Win32_Processor | Format-List Name, MaxClockSpeed, NumberOfCores, NumberOfLogicalProcessors | Out-String | Add-Content $reportPath

"Memory:`n" | Add-Content $reportPath
Get-CimInstance Win32_OperatingSystem | Select-Object TotalVisibleMemorySize, FreePhysicalMemory | ForEach-Object {
    "Total Memory: $([math]::Round($_.TotalVisibleMemorySize / 1MB, 2)) GB`nFree Memory: $([math]::Round($_.FreePhysicalMemory / 1MB, 2)) GB"
} | Out-String | Add-Content $reportPath

# Antivirus Status
"`n--------------------------------------------`nAntivirus Status:`n" | Add-Content $reportPath
Try {
    Get-CimInstance -Namespace "root\SecurityCenter2" -ClassName AntivirusProduct | Select-Object displayName, productState | Out-String | Add-Content $reportPath
} Catch {
    "Could not retrieve antivirus information." | Add-Content $reportPath
}

# Backup Status (Manual Note)
"`n--------------------------------------------`nBackup Status:`n" | Add-Content $reportPath
"Backup is scheduled on BCDR" | Add-Content $reportPath

# Services in Automatic State but Not Started
"`n--------------------------------------------`nServices in automatic state but not started:`n" | Add-Content $reportPath
Get-Service | Where-Object { $_.StartType -eq 'Automatic' -and $_.Status -ne 'Running' } | Select-Object DisplayName, Status | Format-Table -AutoSize | Out-String | Add-Content $reportPath

# Uptime
"`n--------------------------------------------`nUptime:`n" | Add-Content $reportPath
(New-TimeSpan -Start (Get-CimInstance Win32_OperatingSystem).LastBootUpTime -End (Get-Date)) | Out-String | Add-Content $reportPath

# Windows Updates
"`n--------------------------------------------`nWindows Updates:`n" | Add-Content $reportPath
$wuStatus = Get-Service -Name wuauserv
"Windows Update Service Status: $($wuStatus.Status)" | Add-Content $reportPath

# Event Logs
"`n--------------------------------------------`nEvent Logs:`n" | Add-Content $reportPath
Get-EventLog -LogName System -EntryType Error -Newest 5 | Format-Table TimeGenerated, Source, Message -AutoSize | Out-String | Add-Content $reportPath

# External IP
"`n--------------------------------------------`nExternal IP:`n" | Add-Content $reportPath
Try {
    $ip = (Invoke-RestMethod -Uri "https://api.ipify.org?format=text")
    "External IP: $ip" | Add-Content $reportPath
} Catch {
    "Could not retrieve external IP." | Add-Content $reportPath
}

# Open the report
Start-Sleep -Seconds 2
Start-Process notepad.exe $reportPath
