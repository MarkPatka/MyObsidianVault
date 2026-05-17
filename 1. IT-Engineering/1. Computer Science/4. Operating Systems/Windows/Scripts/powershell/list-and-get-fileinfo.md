### List files in a directory

```powershell
Get-ChildItem -Path "C:\path\to\folder"
```

### List all files recursively

```powershell
Get-ChildItem -Path "C:\path\to\folder" -Recurse
```

### List only files (exclude directories)

```powershell
Get-ChildItem -Path "C:\path\to\folder" -File
```

### List only directories

```powershell
Get-ChildItem -Path "C:\path\to\folder" -Directory
```

### Get file properties

```powershell
Get-Item -Path "C:\path\to\file.txt" | Select-Object Name, FullName, Length, LastWriteTime, CreationTime
```

### Find files by name pattern

```powershell
Get-ChildItem -Path "C:\path\to\folder" -Filter "*.txt" -Recurse
```

### Get file size

```powershell

(Get-Item -Path "C:\path\to\file.txt").Length
```

---