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



####	网络小工具 ntool

同时存在有线内网和外网WIFI的情况下，实现同时连接两个网络，访问内网资源时走有线，访问外网资源时走外网。

```powershell
@echo off
chcp 65001 >nul
:: 提权
%1 mshta vbscript:CreateObject("Shell.Application").ShellExecute("cmd.exe","/c %~s0 ::","","runas",1)(window.close)&&exit cd /d "%~dp0"
setlocal enabledelayedexpansion
mode con cols=100 lines=25


:: 定义颜色（ANSI转义码）
set "green=[32m"
set "red=[31m"
set "yellow=[33m"
set "reset=[0m"


:menu
title MainMenu
echo:
echo:
echo:      ______________________________________________________________
echo:
echo:               Network Tool by guoxiang
echo:
echo:           [1] 两路访问           - 内网走有线, 外网走WIFI 
echo:           [2] 帮助			
echo:           [3] 未完待续
echo:           __________________________________________________ 
echo:
echo:           [0] Exit
echo:
echo:      ______________________________________________________________		
echo:

rem set /p sel= Choose a menu option using your keyboard [1,2,3....]

rem echo    Choose a menu option using your keyboard [1,2,3...E,H,0] :

call :colorEcho green " Choose a menu option using your keyboard [1,2,3...E,H,0] :"

choice /C:120 /N
set sel=%errorlevel%

if "%sel%"=="3" exit /b
if "%sel%"=="2" call :PrintInfo & cls & goto :menu
if "%sel%"=="1" call :flow2way & cls & goto :menu


goto menu

:flow2way
cls
route -p ADD 10.0.0.0 MASK 255.0.0.0 10.34.28.1 || call :colorEcho red "添加永久路由失败!" && pause && goto :menu
netsh interface ipv4 show interfaces | find /i "WLAN" >nul
set has_wlan=%errorlevel%
netsh interface ipv4 show interfaces | find /i "以太网" >nul
set has_ethernet=%errorlevel%
set /a logical=%has_wlan%+%has_ethernet%

if %logical% equ 0 (
	echo 正在更改以太网接口的度量值为20....
	netsh interface ipv4 set interface "以太网" metric=20
	echo 正在更改WLAN接口的度量值为10....
	netsh interface ipv4 set interface "WLAN" metric=10
) else (
	echo 需要手动指定网络接口名称^!
	netsh interface ipv4 show interfaces
	echo 请输入有线连接接口名称:
	set /p lowwer_int=
	echo 请输入无线连接接口名称:
	set /p upper_int=
	echo 正在更改%lowwer_int%接口的度量值为20....
	netsh interface ipv4 set interface "%lowwer_int%" metric=20
	echo 正在更改%upper_int%接口的度量值为10....	
	netsh interface ipv4 set interface "%upper_int%" metric=10
)
call :colorEcho green "任务完成!"
echo.

call :colorEcho yellow "Press any key to Go back..."
pause >nul
exit /b



:: 定义函数
:PrintInfo
cls
echo:
echo: ===================================================================================
echo:
echo: [1] 添加永久静态路由:
echo:    route -p ADD 10.0.0.0 MASK 255.0.0.0 10.34.28.1
echo:

echo: [2] 提高无线连接的优先级 ^(设置无线连接的跃点数为10, 有线连接为20^):
echo    netsh interface ipv4 set interface "以太网" metric=20
echo    netsh interface ipv4 set interface "WLAN" metric=10
echo:

echo: [3] 更改无线连接的DNS地址, 与有线连接相同 ^(有弊端, 如果有线断开则无法正常访问网站^):
echo:   DNS1: 10.2.1.72
echo:   DNS2: 10.2.1.175
echo:
echo: ===================================================================================

pause
cls

echo:
echo: ===================================================================================
echo:
echo: 1.显示网络接口信息:
echo:  netsh interface show interface 
echo:

echo: 2.显示网络接口的详细信息:
echo:   netsh interface ipv4 show interfaces
echo:

echo: 3.配置静态 IP 地址:
echo:   netsh interface ip set address name="连接名" static "<ip address> <mask> <gateway>"
echo:

echo: 4.配置动态 IP 地址:
echo:   netsh interface ip set address name="连接名" source=dhcp
echo:

echo: 5.配置静态 DNS 服务器:
echo:   netsh interface ip set dns name="连接名" static "x.x.x.x"
echo:

echo: 6.配置动态 DNS 服务器:
echo:   netsh interface ip set dns name="连接名" source=dhcp
echo:


echo: 7.查看路由表:
echo:    netsh interface ip show route
echo:
echo: ===================================================================================
pause

:colorEcho
rem %1=颜色参数, %2=消息
set "color=!%~1!"
echo %color%%~2%reset%
goto :eof

```



