### Find files modified in the last 7 days

```powershell
Get-ChildItem -Path "C:\path\to\folder" -File -Recurse | Where-Object { $_.LastWriteTime -gt (Get-Date).AddDays(-7) }
```

### Find large files (over 100 MB)

```powershell
Get-ChildItem -Path "C:\path\to\folder" -File -Recurse | Where-Object { $_.Length -gt 100MB }
```

### Find empty files

```powershell
Get-ChildItem -Path "C:\path\to\folder" -File -Recurse | Where-Object { $_.Length -eq 0 }
```

### Search for files by name pattern

```powershell
Get-ChildItem -Path "C:\path\to\folder" -Filter "*keyword*" -Recurse
```

---