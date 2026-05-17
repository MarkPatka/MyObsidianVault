## Copy Files and Directories

### Copy a single file

```powershell
Copy-Item -Path "C:\source\file.txt" -Destination "C:\destination\file.txt"
```

### Copy a file and overwrite if exists

```powershell
Copy-Item -Path "C:\source\file.txt" -Destination "C:\destination\file.txt" -Force
```

### Copy a directory and all contents

```powershell
Copy-Item -Path "C:\source\folder" -Destination "C:\destination\folder" -Recurse
```

### Copy multiple files

```powershell
Copy-Item -Path "C:\source\file1.txt", "C:\source\file2.txt" -Destination "C:\destination\"
```

### Copy all files of a specific type

```powershell
Copy-Item -Path "C:\source\*.jpg" -Destination "C:\destination\"
```

---