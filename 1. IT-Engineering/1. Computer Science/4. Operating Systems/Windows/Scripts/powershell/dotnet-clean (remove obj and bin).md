```powershell
Get-ChildItem -Path . -Directory -Recurse -Filter 'obj' | Remove-Item -Recurse -Force

Get-ChildItem -Path . -Directory -Recurse -Filter 'bin' | Remove-Item -Recurse -Force
```