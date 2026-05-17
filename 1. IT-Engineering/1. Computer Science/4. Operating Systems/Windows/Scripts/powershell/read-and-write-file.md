### Read entire file content

```powershell
Get-Content -Path "C:\path\to\file.txt"
```

### Read specific lines

```powershell
Get-Content -Path "C:\path\to\file.txt" -Head 10  # First 10 linesGet-Content -Path "C:\path\to\file.txt" -Tail 5   # Last 5 lines
```

### Write content to a file (overwrites existing)

```powershell
Set-Content -Path "C:\path\to\file.txt" -Value "New content"
```

### Append content to a file

```powershell
Add-Content -Path "C:\path\to\file.txt" -Value "Additional line"
```

### Read file line by line

```powershell
Get-Content -Path "C:\path\to\file.txt" | ForEach-Object { Write-Host $_ }
```

---