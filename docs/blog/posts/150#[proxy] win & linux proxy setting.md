---
number: 150
title: "[proxy] win & linux proxy setting"
state: open
url: https://github.com/konosubakonoakua/blog/issues/150
author: konosubakonoakua
createdAt: 2025-02-26T01:47:06Z
updatedAt: 2025-12-15T05:22:44Z
labels: []
---

# 150 [proxy] win & linux proxy setting

```powershell
function proxy {
    param(
        [string]$Action = "toggle",
        [string]$IP = "127.0.0.1",
        [string]$Port = "7897"
    )

    $ProxyServer = "http://$IP`:$Port"
    
    # Colors for better UX
    $Green = "Green"
    $Red = "Red" 
    $Yellow = "Yellow"
    $Cyan = "Cyan"

    # Check current status
    $CurrentProxy = $env:HTTP_PROXY ?? $env:http_proxy
    $IsEnabled = $CurrentProxy -eq $ProxyServer

    switch ($Action.ToLower()) {
        "on" { 
            if ($IsEnabled) { 
                Write-Host "Proxy already enabled: $ProxyServer" -ForegroundColor $Green
                return 
            }
            $env:HTTP_PROXY = $ProxyServer
            $env:HTTPS_PROXY = $ProxyServer
            $env:http_proxy = $ProxyServer
            $env:https_proxy = $ProxyServer
            Write-Host "Proxy enabled: $ProxyServer" -ForegroundColor $Green
        }
        
        "off" { 
            if (-not $IsEnabled) { 
                Write-Host "Proxy already disabled" -ForegroundColor $Green
                return 
            }
            Remove-Item Env:HTTP_PROXY -ErrorAction SilentlyContinue
            Remove-Item Env:HTTPS_PROXY -ErrorAction SilentlyContinue
            Remove-Item Env:http_proxy -ErrorAction SilentlyContinue
            Remove-Item Env:https_proxy -ErrorAction SilentlyContinue
            Write-Host "Proxy disabled" -ForegroundColor $Red
        }
        
        "status" {
            Write-Host "Proxy Status:" -ForegroundColor $Cyan
            Write-Host "  HTTP_PROXY:  $($env:HTTP_PROXY ?? 'Not set')"
            Write-Host "  HTTPS_PROXY: $($env:HTTPS_PROXY ?? 'Not set')"
            
            # Test connectivity if proxy is enabled
            if ($IsEnabled) {
                Write-Host "  Testing connection to Google..." -ForegroundColor $Yellow
                try {
                    $test = Invoke-WebRequest -Uri "https://www.google.com" -TimeoutSec 5 -UseBasicParsing
                    Write-Host "  Connection test: SUCCESS" -ForegroundColor $Green
                } catch {
                    Write-Host "  Connection test: FAILED" -ForegroundColor $Red
                }
            } else {
                Write-Host "  Status: DISABLED" -ForegroundColor $Red
            }
        }
        
        "toggle" {
            if ($IsEnabled) {
                proxy -Action off
            } else {
                proxy -Action on -IP $IP -Port $Port
            }
        }
        
        default {
            Write-Host "Unknown action: $Action" -ForegroundColor $Red
            Write-Host "Valid actions: on, off, status, toggle" -ForegroundColor $Yellow
        }
    }
}

# Create alias for convenience
Set-Alias -Name pxy -Value proxy
```


---

## Comments

### konosubakonoakua · 2025-12-15T04:32:19Z

```bash
proxy() {
  local action="${1:-status}"
  local ip="${2:-127.0.0.1}"
  local port="${3:-7897}"

  local proxy_url="http://$ip:$port"

  case "$action" in
  on | enable)
    echo "Enabling proxy: $proxy_url"
    export http_proxy="$proxy_url"
    export https_proxy="$proxy_url"
    export HTTP_PROXY="$proxy_url"
    export HTTPS_PROXY="$proxy_url"
    echo "✅ Proxy enabled: $proxy_url"
    ;;

  off | disable)
    echo "Disabling proxy..."
    unset http_proxy https_proxy HTTP_PROXY HTTPS_PROXY
    echo "✅ Proxy disabled"
    ;;

  toggle)
    if [ -n "$http_proxy" ]; then
      proxy off
    else
      proxy on "$ip" "$port"
    fi
    ;;

  status)
    echo "=== Proxy Environment Variables ==="
    if [ -n "$http_proxy" ]; then
      echo "Current Proxy: $http_proxy"
    else
      echo "Current Proxy: Not set"
    fi

    echo -e "\n=== Testing Google Connectivity ==="
    local timeout=2

    echo -n "Testing https://www.google.com ... "
    if curl -s --connect-timeout "$timeout" "https://www.google.com" >/dev/null 2>&1; then
      echo "✅ SUCCESS"
      echo "Network connectivity is working correctly!"
    else
      echo "❌ FAILED"
      if [ -n "$http_proxy" ]; then
        echo "Proxy may not be working properly."
      else
        echo "No proxy is set or there's a connection issue."
      fi
    fi
    ;;

  *)
    echo "Usage: proxy {on|off|toggle|status} [ip] [port]"
    echo "Examples:"
    echo "  proxy on                        # Enable with default 127.0.0.1:7897"
    echo "  proxy on 192.168.138.254 7897   # Enable with custom IP:port"
    echo "  proxy off                       # Disable proxy"
    echo "  proxy toggle                    # Toggle proxy state"
    echo "  proxy status                    # Show status and test connectivity"
    return 1
    ;;
  esac
}

```
