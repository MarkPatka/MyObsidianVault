## Remove a single file

```powershell
Remove-Item "C:\Users\YourName\Documents\file.txt"
```

### Remove a file without confirmation

```powershell
Remove-Item "C:\path\to\file.txt" -Force
```

The `-Force` parameter bypasses any confirmation prompts and removes read-only files.

### Remove multiple files

```powershell
Remove-Item "C:\path\to\file1.txt", "C:\path\to\file2.txt"
```

### Remove all files matching a pattern

```powershell
Remove-Item "C:\path\to\*.txt"
```

This removes all `.txt` files in the specified directory.

## Useful Parameters

|Parameter|Purpose|
|---|---|
|`-Path`|Specifies the file path (can use wildcards)|
|`-Force`|Removes files without confirmation; bypasses read-only restrictions|
|`-WhatIf`|Shows what would be deleted without actually deleting it|
|`-Confirm`|Prompts for confirmation before each deletion|
|`-Recurse`|Removes files in subdirectories (useful with `-Path` wildcards)|
## Examples with Additional Options

### Preview what will be deleted

```powershell
Remove-Item "C:\path\to\*.txt" -WhatIf
```

### Remove files recursively from subdirectories

```powershell
Remove-Item "C:\path\to\*.txt" -Recurse -Force
```

### Delete with explicit confirmation

```powershell
Remove-Item "C:\path\to\file.txt" -Confirm
```

The `-WhatIf` parameter is particularly helpful when you're unsure about your deletion command—it shows you exactly what would be removed before you commit to the action.