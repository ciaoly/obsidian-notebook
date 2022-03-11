
## 回收站名称：
### powershell命令:
```powershell
(Get-ItemProperty -Path registry::HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CLSID\`{645FF040-5081-101B-9F08-00AA002F954E`}).'(default)'
```

### cmd命令:
```cmd
reg query HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\CLSID\{645FF040-5081-101B-9F08-00AA002F954E} /ve
```