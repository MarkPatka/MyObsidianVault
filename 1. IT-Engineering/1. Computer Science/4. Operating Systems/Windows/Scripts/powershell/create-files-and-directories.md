### Create an empty file

```powershell
New-Item -Path "C:\path\to\newfile.txt" -ItemType File
```

### Create a file with content

```powershell
New-Item -Path "C:\path\to\newfile.txt" -ItemType File -Value "Hello, World!"
```

Or use `Set-Content`:
```powershell
Set-Content -Path "C:\path\to\newfile.txt" -Value "Hello, World!"
```

### Create a directory

```powershell
New-Item -Path "C:\path\to\newfolder" -ItemType Directory
```

### Create nested directories

```powershell
New-Item -Path "C:\path\to\parent\child\grandchild" -ItemType Directory -Force
```
