---
number: 110
title: "[sw][epics] phoebus & epics ca tool"
state: open
url: https://github.com/konosubakonoakua/blog/issues/110
author: konosubakonoakua
createdAt: 2024-10-04T18:48:37Z
updatedAt: 2024-10-23T06:56:29Z
labels: []
---

# 110 [sw][epics] phoebus & epics ca tool

## download
- [v4.7.3](https://github.com/ControlSystemStudio/phoebus/releases/tag/v4.7.3)


---

## Comments

### konosubakonoakua · 2024-10-04T18:51:10Z

## install
### linux
```bash
sudo apt install default-jdk
```
```bash
sudo mkdir -p /epics
sudo chown rf:rf /epics
tar ~/Downloads/Phoebus-4.7.3-linux.tar.gz -C /epics
mv /epics/product-4.7.3 /epics/phoebus
chmod +x /epics/phoebus/phoebus.sh /epics/phoebus/phoebus.desktop
sed -i "s#Exec.*#Exec=/epics/phoebus/phoebus.sh#" /epics/phoebus/phoebus.desktop
ln -s /epics/phoebus/phoebus.desktop ~/.local/share/applications/phoebus.desktop
```
### windows
save as `install.ps1`, run it.
```powershell
$url = "https://epics.anl.gov/download/distributions/CA-3.15.6-windows-x64.zip"
$zipFile = "$env:TEMP\CA-3.15.6-windows-x64.zip"
$extractPath = "$env:LOCALAPPDATA\CA-3.15.6-windows-x64"
$maxRetries = 3
$retryDelay = 5

function Download-File {
    param (
        [string]$url,
        [string]$outputFile
    )
    $retryCount = 0
    while ($retryCount -lt $maxRetries) {
        try {
            Write-Host "Downloading... Retry $($retryCount + 1) / $maxRetries"
            Invoke-WebRequest -Uri $url -OutFile $outputFile
            if (Test-Path $outputFile) {
                return $true
            }
        } catch {
            Write-Host "Download failed: $_"
        }
        $retryCount++
        if ($retryCount -lt $maxRetries) {
            Write-Host "Retry after $retryDelay sec..."
            Start-Sleep -Seconds $retryDelay
        }
    }
    return $false
}

if (-Not (Download-File -url $url -outputFile $zipFile)) {
    Write-Host "Download failed. Please cheack network and URL."
    exit 1
}

Write-Host "Unzip..."
Expand-Archive -Path $zipFile -DestinationPath $extractPath -Force

if (-Not (Test-Path $extractPath)) {
    Write-Host "Unzip failed, must have Expand-Archive installed."
    exit 1
}

Write-Host "Setup PATH..."
$newPath = "$env:PATH;$extractPath\bin;$extractPath\home\phoebus\ANJ\win32\bin\"
[System.Environment]::SetEnvironmentVariable("PATH", $newPath, [System.EnvironmentVariableTarget]::User)

Write-Host "Cleaning up..."
Remove-Item $zipFile

Write-Host "Done :)"
```

### konosubakonoakua · 2024-10-04T18:51:40Z

## import config
<details>

<img src="https://github.com/user-attachments/assets/a5f86607-4fb9-44b8-b6ed-9b9e35c36331" width="60%" />

</details>
