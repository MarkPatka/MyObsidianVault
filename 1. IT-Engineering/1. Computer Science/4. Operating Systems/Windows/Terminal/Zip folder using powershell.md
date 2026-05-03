
### Zip a single folder into an archive (overwrites if exists):
```powershell
Compress-Archive -Path 'C:\Path\To\Folder' -DestinationPath 'C:\Path\To\Archive.zip' -Force
```

### Zip the folder contents (not the parent folder):
```powershell
Compress-Archive -Path 'C:\Path\To\Folder\*' -DestinationPath 'C:\Path\To\Archive.zip' -Force
```

### Add files to an existing zip:
```powershell
Compress-Archive -Path 'C:\File1.txt','C:\File2.txt' -Update -DestinationPath 'C:\Path\To\Archive.zip'
```

### Zip multiple folders:
```powershell
Compress-Archive -Path 'C:\FolderA','C:\FolderB' -DestinationPath 'C:\Archives\Combined.zip' -Force
```