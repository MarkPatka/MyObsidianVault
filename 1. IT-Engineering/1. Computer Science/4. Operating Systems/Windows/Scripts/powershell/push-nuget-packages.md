```powershell
<#
.SYNOPSIS
    Pushes local NuGet packages to a corporate feed, skipping already pushed ones.
.DESCRIPTION
    Prompts for the API key, reads the list of previously pushed packages from a log file,
    then pushes every .nupkg in the current folder that is not yet in that log.
    Successfully pushed packages are appended to the log.
    Errors are displayed and also saved to push_errors.log.
#>

param(
    # NuGet server push URL (must end with /nuget for NuGet.Server)
    [string]$Source = "https://nuget.voxys.ru/nuget",

    # File that records the names (including version) of successfully pushed packages
    [string]$LogFile = "pushed_packages.log"
)

# ---------- Ask for API key ----------
$apiKey = Read-Host "Enter your NuGet API key"
if ([string]::IsNullOrWhiteSpace($apiKey)) {
    Write-Error "API key cannot be empty. Aborting."
    exit 1
}

# ---------- Prepare the log of already pushed packages ----------
$alreadyPushed = @{}
if (Test-Path $LogFile) {
    Get-Content $LogFile | ForEach-Object {
        $clean = $_.Trim()
        if ($clean) { $alreadyPushed[$clean] = $true }
    }
}
else {
    # Create an empty log file if it does not exist
    New-Item -Path $LogFile -ItemType File -Force | Out-Null
}

# ---------- Find new packages ----------
$allNupkg = Get-ChildItem -Path . -Recurse -Filter *.nupkg -File
$newPackages = $allNupkg | Where-Object { -not $alreadyPushed.ContainsKey($_.Name) }

if ($newPackages.Count -eq 0) {
    Write-Host "All packages have already been pushed. Nothing to do." -ForegroundColor Green
    exit 0
}

Write-Host "Found $($newPackages.Count) new package(s) to push." -ForegroundColor Cyan

# ---------- Push each new package ----------
foreach ($pkg in $newPackages) {
    Write-Host "Pushing $($pkg.Name) ..."
    
    # Build and execute the push command.
    # nuget.exe push <package> "<apiKey>" -Source <source>
    # The API key is automatically quoted by PowerShell because of the double quotes around $apiKey.
    $output = & .\nuget.exe push $pkg.FullName "$apiKey" -Source $Source 2>&1
    $exitCode = $LASTEXITCODE

    if ($exitCode -eq 0) {
        Write-Host "  Success" -ForegroundColor Green
        # Append the package file name (which includes version) to the log
        Add-Content -Path $LogFile -Value $pkg.Name
    }
    else {
        Write-Host "  FAILED (exit code $exitCode)" -ForegroundColor Red
        Write-Host "  Error output: $output" -ForegroundColor Yellow
        # Write the error to a separate error log for later inspection
        $timestamp = Get-Date -Format "yyyy-MM-dd HH:mm:ss"
        Add-Content -Path "push_errors.log" -Value "$timestamp - Failed to push $($pkg.Name): $output"
    }
}

Write-Host "Done." -ForegroundColor Cyan
```