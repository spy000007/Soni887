name: Android RDP Setup

on:
  workflow_dispatch:
    inputs:
      duration:
        description: 'Session Time (minutes)'
        required: true
        default: '20'

jobs:
  setup-windows:
    runs-on: windows-latest
    
    steps:
    - name: Enable RDP
      run: |
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -name "fDenyTSConnections" -value 0
        netsh advfirewall firewall set rule group="remote desktop" new enable=Yes
        
    - name: Set Password
      run: |
        $password = "${{ secrets.RDP_PASSWORD }}"
        if ($password) {
            $securePassword = ConvertTo-SecureString $password -AsPlainText -Force
            Set-LocalUser -Name "runner" -Password $securePassword
            Set-LocalUser -Name "runner" -PasswordNeverExpires $true
            Add-LocalGroupMember -Group "Remote Desktop Users" -Member "runner"
        }
        
    - name: Setup ngrok
      run: |
        Invoke-WebRequest -Uri "https://bin.equinox.io/c/bNyj1m1r5gY/ngrok-v3-stable-windows-amd64.zip" -OutFile "ngrok.zip"
        Expand-Archive -Path "ngrok.zip" -DestinationPath "."
        .\ngrok.exe config add-authtoken ${{ secrets.NGROK_AUTH }}
        
    - name: Start Tunnel
      run: |
        Start-Process -FilePath ".\ngrok.exe" -ArgumentList "tcp 3389" -NoNewWindow
        
    - name: Show Connection
      run: |
        Start-Sleep -Seconds 15
        try {
            $ngrokApi = Invoke-RestMethod -Uri "http://localhost:4040/api/tunnels"
            $url = $ngrokApi.tunnels[0].public_url -replace "tcp://", ""
            echo "RDP_CONNECTION=$url" >> $env:GITHUB_ENV
            Write-Host "✅ CONNECT: $url"
            Write-Host "👤 User: runner"
            Write-Host "🔑 Pass: [Your Password]"
        } catch {}
        
    - name: Keep Alive
      run: |
        $time = [int]${{ github.event.inputs.duration }}
        for ($i = 1; $i -le $time; $i++) {
            Start-Sleep -Seconds 60
        }
