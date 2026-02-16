# Setting Up a GitHub Self-Hosted Runner

If you want to run CI on your own machine instead of GitHub's hosted runners, here's how to set one up on Windows.

## What you need

- Windows 10 or 11 with PowerShell
- Docker Desktop installed and running
- Access to the repo on GitHub

## 1. Get the runner package from GitHub

Go to your repo → **Settings** → **Actions** → **Runners** → **New self-hosted runner**. Select **Windows** and **x64**.

Open PowerShell as admin:

```powershell
mkdir C:\actions-runner; cd C:\actions-runner

# Download (check GitHub for the latest version URL)
Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/v2.311.0/actions-runner-win-x64-2.311.0.zip -OutFile actions-runner.zip

# Extract
Add-Type -AssemblyName System.IO.Compression.FileSystem
[System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD\actions-runner.zip", "$PWD")
```

## 2. Configure it

```powershell
.\config.cmd --url https://github.com/confluentlab/SMT --token YOUR_TOKEN_HERE
```

GitHub gives you the token on the runner setup page. When it asks:

- **Runner group** — hit Enter for default
- **Runner name** — something like `local-windows`
- **Work folder** — hit Enter for default

## 3. Run it

You can install it as a Windows service (recommended — it starts automatically):

```powershell
.\svc.cmd install
.\svc.cmd start
```

Or just run it interactively for testing:

```powershell
.\run.cmd
```

## 4. Verify

Go to **Settings** → **Actions** → **Runners** in your repo. You should see your runner listed as "Idle".

## If something goes wrong

**Runner not connecting:**

```powershell
Get-Service actions.runner.*

# Restart it
.\svc.cmd stop
.\svc.cmd start
```

**Docker commands failing in CI:**

Make sure Docker Desktop is running and the service account the runner uses has permission to talk to Docker.

## Removing the runner

```powershell
.\svc.cmd stop
.\svc.cmd uninstall
.\config.cmd remove --token YOUR_TOKEN_HERE
```
