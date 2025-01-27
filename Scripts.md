####	CMD 提权命令

```powershell
@echo off
%1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit cd /d "%~dp0"
```



####	CMD ltsc2024 转换为ltsc lot 2024

```powershell
@echo off
%1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit cd /d "%~dp0"
:: Win10 LTSC IOT 版本密钥
slmgr.vbs -ipk CGK42-GYN6Y-VD22B-BX98W-J8JXD
:: timeout /t 3
exit
```

