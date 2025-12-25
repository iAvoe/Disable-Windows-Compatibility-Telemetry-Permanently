# How to permanently disable Windows Compatibility Telemetry (CompatTelRunner)

❀By December 12, 2025 (release date). This tutorial is likely to be the only working solution❀

If your Windows Compatibility Telemetry runs after every program install and uninstall (visible in Task Manager), and consumes a huge portion of CPU (20~90%), and you've tried many things without success, and if you are currently losing temper with Microsoft, then you're in the right place.

## Dead ends

Before confirming you're really out of options, here are the common solutions to try first (you've probably haven't tried all of them):

1. **Disable from Task Scheduler**
    - Task Scheduler → `Task Scheduler Library\Microsoft\Windows\Application Experience`
    - Set "Microsoft Compatibility Appraiser" to disabled
2. **Disable via Group Policy**
    - Windows Group Policy (gpedit.msc) → `Computer Configuration` → `Administrative Templates`
    - → `Windows Components` → `Data Collection and Preview Builds`
    - Set`Allow Diagnostic Data` and/or `Allow Telemetry` policy to disabled
3. **Diable via Settings**
    - Settings → Privacy → Diagnostics and feedback → Set “Feedback frequency” to Never
4. **(Confirmed useless) Delete CompatTelRunner**
    - As an administrator, take ownership of `CompatTelRunner.exe`, then delete or change its name
      - (Windows will eventually download a new copy of `CompatTelRunner.exe`)

### Why are these solutions failing

I couldn't really find the reason, maybe my Windows installation is broken. There is likely be an automatic process that "fixes" things that prevents CompatTelRunner.exe from executing.

---

## Working solution

1. Create a batch script file named "stopCompatTelRunner.bat" or anything you prefer, maybe some curse words plus Microsoft
    1. If you are viewing on GitHub, you may also download the attached batch file, but that's a harmful practice compared to just copying text
    2. Make sure you leave 2 empty lines at the beginning of this batch script (or it won't run properly)
    3. Make sure you are saving with "CRLF" line breaks, or this script cannot be read by CMD.exe
        - (Windows `Notepad.exe` is doing this correctly, but other editors may not)
    4. It will try to gain `Administrator` priviledge, take ownsership of `CompatTelRunner.exe`, and then disable `SYSTEM`'s execution permission
```



@chcp 65001
@echo off

:: BatchGotAdmin
:-------------------------------------
REM  --> Check for permissions
    IF "%PROCESSOR_ARCHITECTURE%" EQU "amd64" (
>nul 2>&1 "%SYSTEMROOT%\SysWOW64\cacls.exe" "%SYSTEMROOT%\SysWOW64\config\system"
) ELSE (
>nul 2>&1 "%SYSTEMROOT%\system32\cacls.exe" "%SYSTEMROOT%\system32\config\system"
)

REM --> If error flag set, we do not have admin.
if '%errorlevel%' NEQ '0' (
    echo Requesting administrative privileges...
    goto UACPrompt
) else ( goto gotAdmin )

:UACPrompt
    echo Set UAC = CreateObject^("Shell.Application"^) > "%temp%\getadmin.vbs"
    set params = %*:"=""
    echo UAC.ShellExecute "cmd.exe", "/c ""%~s0"" %params%", "", "runas", 1 >> "%temp%\getadmin.vbs"

    "%temp%\getadmin.vbs"
    del "%temp%\getadmin.vbs"
    exit /B

:gotAdmin
    pushd "%CD%"
    CD /D "%~dp0"

:: Keep SYSTEM off from running
takeown /f C:\Windows\System32\CompatTelRunner.exe
icacls C:\Windows\System32\CompatTelRunner.exe /deny SYSTEM:(RX)

exit
```
2. Create a Basic Task in Task Scheduler to run this batch once on every system startup
    1. Type `task scheduler` in search bar to open it
    2. Under the Actions column on the right, click `Create Basic Task` to open the wizard
    3. Set `Name` to “Stop Compatibility Telemetry” or something else you could remember, same as description
    4. Set `Trigger` to “When the computer starts”, or “When I log on” if you are the sole user on this PC
    5. Set `Action` to `Start a program`
    6. In `Start a program`, bind the program to the batch script
        - Don't move this batch script to somewhere else after making this configuration
    7. Create finish
    8. Find and open this Task
    9. Check the “Run with highest privileges” box, then Save
        - You may check with `Conditions` and `Settings` tab to configure overtime, duplicated task and execution assurance options by your need

That's it. Now SYSTEM will never be able to execute Windows Compatibility Telemetry again, since the automatic process of ‘restoring’ is less frequerent than your automatic ‘disabling’ process.
