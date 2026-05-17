### Move a file

```powershell
Move-Item -Path "C:\source\file.txt" -Destination "C:\destination\file.txt"
```

### Move a file and overwrite if exists

```powershell

Move-Item -Path "C:\source\file.txt" -Destination "C:\destination\file.txt" -Force
```

### Rename a file

```powershell

Rename-Item -Path "C:\path\to\oldname.txt" -NewName "newname.txt"
```

### Move a directory

```powershell

Move-Item -Path "C:\source\folder" -Destination "C:\destination\folder"
```

### Move multiple files

```powershell

Move-Item -Path "C:\source\file1.txt", "C:\source\file2.txt" -Destination "C:\destination\"
```

---