# kiosk-demo-electron
[![Build status](https://ci.appveyor.com/api/projects/status/um6ul6dbwjrw913m/branch/master?svg=true)](https://ci.appveyor.com/project/syedhassaanahmed/kiosk-demo-electron/branch/master)

This Electron App demonstrates Kiosk mode and creates .msi package which executes a [PowerShell script](https://github.com/syedhassaanahmed/kiosk-demo-electron/blob/master/installer/Install-ShellLauncher.ps1) to enable [Windows 10 Shell Launcher](https://docs.microsoft.com/en-us/windows-hardware/customize/enterprise/shell-launcher) and set the app executable as custom shell.

Creating an .msi makes sure we can distribute our app to multiple kiosks via MDM e.g [Microsoft Intune](https://docs.microsoft.com/en-us/intune/apps-add).

## Create Installer
[MSI Wrapper](http://www.exemsi.com/download) must be installed on your machine in order to create the .msi, otherwise you'll get 

`Retrieving the COM class factory for component with CLSID {06983BA0-AE1E-43B4-83B6-8D6D5DFA5CEB} failed due to the following error: 80040154 Class not registered (Exception from HRESULT: 0x80040154 (REGDB_E_CLASSNOTREG)).`

**NOTE:** `electron-packager` currently has an [issue with npm v5.3.0](https://github.com/electron-userland/electron-packager/issues/686), so please use `npm install -g npm@5.2.0` until its resolved.

`npm run dist` will build everything and create the .msi in `dist` folder.

## Description
- First we package the app using [electron-packager](https://github.com/electron-userland/electron-packager)
- Then we create a [squirrel installer (.exe)](https://github.com/Squirrel/Squirrel.Windows) using [electron-winstaller](https://github.com/electron/windows-installer)
- Squirrel package is then wrapped in an .msi using [MSI Wrapper script](http://www.exemsi.com/documentation/msi-build-scripts) which makes sure we don't end up with double entries in `Add or remove programs` as well as execute the inside PowerShell script in an elevated way. 
- Script is actually executed from squirrel [post-install events](https://github.com/syedhassaanahmed/kiosk-demo-electron/blob/master/installer/setupEvents.js) via [node-powershell](https://github.com/rannn505/node-powershell).

## Telemetry
The solution uses [Application Insights](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-nodejs) to collect basic telemetry data from the Kiosk app. To enable it, create an environment variable named `APPINSIGHTS_INSTRUMENTATIONKEY` and set it to your Instrumentation Key.

## Troubleshoot
All logs (Squirrel setup, install events as well as PowerShell) will be located at `%userprofile%\AppData\Local\SquirrelTemp`

## Limitation
Currently, the user for which custom shell needs to be enabled, is hardcoded inside Squirrel events (`kioskelectron`). That's because Squirrel doesn't support [passing arguments to Setup.exe](https://github.com/Squirrel/Squirrel.Windows/issues/839) yet.