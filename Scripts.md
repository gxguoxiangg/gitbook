####	CMD 提权命令

```powershell
@echo off
%1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit cd /d "%~dp0"
```

新式提权命令：

```bash
@echo off
setlocal EnableDelayedExpansion

:: ============ 定义内部变量 ============
:: set "_elev=1"
set "psc=powershell -Command"
set "eline=echo."
set "nul1=>nul 2>&1"

:: ============ 检查管理员权限 ============
%nul1% fltmc || (
if not defined _elev %psc% "start cmd.exe -verb runas" && exit /b
%eline%
echo This script needs admin rights.
echo Right click on this script and select 'Run as administrator'.
goto dk_done
)

:: ============ 你的管理员操作写在下面 ============

timeout /t 2
echo operation done.
pause

:: ============ 提权失败 ==============
:dk_done
endlocal
pause
exit /b
```

简短的新式提权命令：

```bash
@echo off
fltmc >nul 2>&1 || (powershell -Command "Start-Process '%~f0' -ArgumentList '\"%*\"' -Verb RunAs" && exit /b)
# 在脚本开头加入管理员权限检测，只有在非管理员时才执行提权逻辑，
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

