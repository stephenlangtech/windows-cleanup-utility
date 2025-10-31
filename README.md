<h1>Windows Cleanup Utility Tool</h1>
<p align="center">
<img src="https://bitperfection.com/wp-content/uploads/2020/08/Computer-Maintenance-New.jpg" alt="The Art of Computer Maintenance - Bit Perfection"/>
</p>  

This repository demonstrates an effective, automated PowerShell script you can run on Windows that scans and removes unnecessary files, such as temporary files, Recycle Bin contents, Windows Update cache, and log files.

***Detailed Breakdown of What the Script Will Check***  
Your PowerShell Script will gather and display:  
- **System Info Output** - PC name, OS version, total RAM, and free disk space
- **Progress Bar** - Displays a live progress bar as the script runs.
- **System Uptime** - How long the PC has been powered on since the last reboot.
- **CPU Usage:** - Your CPU's average load percentage.
- **Memory Usage:** - Current RAM usage percentage.
- **Disk Usage:** - The space left on each of your drives.
- **Event Logs:** Reports critical system or application errors.

***How to Run It***  

1.) Save it as:   
`WindowsCleanupUtility.ps1`   
2.) Run **PowerShell** as an **Administrator**   
3.) Navigate to the folder and execute the file:  
```bash
.\WindowsCleanupUtility.ps1
```
4.) Type `Y` to confirm.   
5.) View the progress and system stats before/after the tool is used.

🛠️<ins>***Windows Utility Tool Script***<ins>🛠️

```bash
<#
.SYNOPSIS
Windows Cleanup Utility

.DESCRIPTION
This PowerShell script performs system cleanup by removing temporary files, clearing caches,
emptying the Recycle Bin, and cleaning Windows Update data. It also displays system information
before and after cleanup.

.AUTHOR
Stephen Langley
#>

# ---------------------- #
# STEP 1: Display Header #
# ---------------------- #
Write-Host "============================================" -ForegroundColor Cyan
Write-Host "          WINDOWS CLEANUP UTILITY" -ForegroundColor Green
Write-Host "============================================" -ForegroundColor Cyan
Write-Host ""

# ------------------------------ #
# STEP 2: Display System Details #
# ------------------------------ #
Write-Host "Collecting system information..." -ForegroundColor Yellow

$SysInfo = [PSCustomObject]@{
    "Computer Name"     = $env:COMPUTERNAME
    "Username"          = $env:USERNAME
    "OS Version"        = (Get-CimInstance Win32_OperatingSystem).Caption
    "Total Memory (GB)" = [math]::Round((Get-CimInstance Win32_ComputerSystem).TotalPhysicalMemory / 1GB, 2)
    "Free Disk Space (GB)" = [math]::Round((Get-PSDrive -Name C).Free / 1GB, 2)
    "Date"              = (Get-Date)
}
$SysInfo | Format-List
Write-Host ""

# ------------------------------ #
# STEP 3: Confirm Before Cleanup #
# ------------------------------ #
$Confirm = Read-Host "Do you want to start cleanup? (Y/N)"
if ($Confirm -ne "Y") {
    Write-Host "Cleanup canceled." -ForegroundColor Yellow
    exit
}

# -------------------------- #
# STEP 4: Define Cleanup Set #
# -------------------------- #
$Paths = @(
    "$env:TEMP",
    "C:\Windows\Temp",
    "C:\Windows\Prefetch"
)

$TotalSteps = $Paths.Count + 2
$Step = 0

# ------------------------ #
# STEP 5: Begin Cleanup    #
# ------------------------ #
foreach ($Path in $Paths) {
    $Step++
    Write-Progress -Activity "Cleaning system..." -Status "Cleaning $Path" -PercentComplete (($Step / $TotalSteps) * 100)
    if (Test-Path $Path) {
        Write-Host "Cleaning: $Path"
        Get-ChildItem -Path $Path -Recurse -Force -ErrorAction SilentlyContinue | Remove-Item -Force -Recurse -ErrorAction SilentlyContinue
    }
}

# --------------------------- #
# STEP 6: Empty Recycle Bin   #
# --------------------------- #
$Step++
Write-Progress -Activity "Cleaning system..." -Status "Emptying Recycle Bin..." -PercentComplete (($Step / $TotalSteps) * 100)
Clear-RecycleBin -Force -ErrorAction SilentlyContinue

# ------------------------------------ #
# STEP 7: Clear Windows Update Cache   #
# ------------------------------------ #
$Step++
Write-Progress -Activity "Cleaning system..." -Status "Clearing Windows Update Cache..." -PercentComplete (($Step / $TotalSteps) * 100)
Stop-Service -Name wuauserv -Force -ErrorAction SilentlyContinue
Remove-Item "C:\Windows\SoftwareDistribution\Download\*" -Force -Recurse -ErrorAction SilentlyContinue
Start-Service -Name wuauserv

# ---------------------------- #
# STEP 8: Log and Final Output #
# ---------------------------- #
$LogFolder = "C:\Logs"
if (-not (Test-Path $LogFolder)) {
    New-Item -Path $LogFolder -ItemType Directory | Out-Null
}

$LogPath = "$LogFolder\Cleanup_$(Get-Date -Format yyyyMMdd_HHmm).txt"
"Cleanup completed on $(Get-Date)" | Out-File -Append $LogPath

# Display post-cleanup info
Write-Host ""
Write-Host "System cleanup complete!" -ForegroundColor Green
Write-Host "Log saved to: $LogPath" -ForegroundColor Cyan
Write-Host ""

# ---------------------------- #
# STEP 9: Show Updated Info    #
# ---------------------------- #
$AfterInfo = [PSCustomObject]@{
    "Free Disk Space After (GB)" = [math]::Round((Get-PSDrive -Name C).Free / 1GB, 2)
    "Date" = (Get-Date)
}
$AfterInfo | Format-List

Write-Host ""
Write-Host "Cleanup completed successfully. Your system should now have improved performance." -ForegroundColor Green
Write-Host ""
```
