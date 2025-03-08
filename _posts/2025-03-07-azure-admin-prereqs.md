---

layout: single
title:  "Prerequisites for Azure administrators"
categories: Cloud
tags: Azure
show_date: true
classes: wide
header:
  teaser: /assets/images/az-104.png
author:
  name     : "Microsoft"
  avatar   : "/assets/images/az-104.png"

sidebar:
  - title: "Blog"
   
    text: "Checkout other topics"
    nav: my-sidebar

---

# Prerequisites for Azure administrators

Here is the offical learning plan from Microsoft

https://learn.microsoft.com/en-us/plans/50nrtozg28d354#?sharingId=364D17115B079D2D



[Manage services with the Azure portal 						](https://learn.microsoft.com/en-us/training/modules/tour-azure-portal/)

[Introduction to Azure Cloud Shell 						](https://learn.microsoft.com/en-us/training/modules/intro-to-azure-cloud-shell/)

[Introduction to Bash ](https://learn.microsoft.com/en-us/training/modules/bash-introduction/)

[Introduction to PowerShell 						](https://learn.microsoft.com/en-us/training/modules/introduction-to-powershell/)

```sh
PS /Users/pradeep> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      7.5.0
PSEdition                      Core
GitCommitId                    7.5.0
OS                             Darwin 24.4.0 Darwin Kernel Version 24.4.0: Sat Feb 15 22:46:38 PST 2025; root:xnu-11417.100.533.501.4~3/RELEASE_X86_64
Platform                       Unix
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0

PS /Users/pradeep> 
```

```sh
PS /Users/pradeep> Get-Command

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Get-PSResource                                     1.1.0      Microsoft.PowerShell.PSResourceGet
Function        cd..                                                          
Function        cd\                                                           
Function        cd~                                                           
Function        Clear-Host                                                    
Function        Compress-Archive                                   1.2.5      Microsoft.PowerShell.Archive
Function        exec                                                          
Function        Expand-Archive                                     1.2.5      Microsoft.PowerShell.Archive
Function        Find-Command                                       2.2.5      PowerShellGet
Function        Find-DSCResource                                   2.2.5      PowerShellGet
Function        Find-Module                                        2.2.5      PowerShellGet
Function        Find-RoleCapability                                2.2.5      PowerShellGet
Function        Find-Script                                        2.2.5      PowerShellGet
Function        Get-CredsFromCredentialProvider                    2.2.5      PowerShellGet
Function        Get-InstalledModule                                2.2.5      PowerShellGet
Function        Get-InstalledScript                                2.2.5      PowerShellGet
Function        Get-PSRepository                                   2.2.5      PowerShellGet
Function        help                                                          
Function        Import-PSGetRepository                             1.1.0      Microsoft.PowerShell.PSResourceGet
Function        Install-Module                                     2.2.5      PowerShellGet
Function        Install-Script                                     2.2.5      PowerShellGet
Function        New-ScriptFileInfo                                 2.2.5      PowerShellGet
Function        oss                                                           
Function        Pause                                                         
Function        prompt                                                        
Function        PSConsoleHostReadLine                              2.3.6      PSReadLine
Function        Publish-Module                                     2.2.5      PowerShellGet
Function        Publish-Script                                     2.2.5      PowerShellGet
Function        Register-PSRepository                              2.2.5      PowerShellGet
Function        Save-Module                                        2.2.5      PowerShellGet
Function        Save-Script                                        2.2.5      PowerShellGet
Function        Set-PSRepository                                   2.2.5      PowerShellGet
Function        TabExpansion2                                                 
Function        Test-ScriptFileInfo                                2.2.5      PowerShellGet
Function        Uninstall-Module                                   2.2.5      PowerShellGet
Function        Uninstall-Script                                   2.2.5      PowerShellGet
Function        Unregister-PSRepository                            2.2.5      PowerShellGet
Function        Update-Module                                      2.2.5      PowerShellGet
Function        Update-ModuleManifest                              2.2.5      PowerShellGet
Function        Update-Script                                      2.2.5      PowerShellGet
Function        Update-ScriptFileInfo                              2.2.5      PowerShellGet
Cmdlet          Add-Content                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Add-History                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Add-Member                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Add-Type                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Clear-Content                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Clear-History                                      7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Clear-Item                                         7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Clear-ItemProperty                                 7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Clear-Variable                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Compare-Object                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Compress-PSResource                                1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Convert-Path                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          ConvertFrom-CliXml                                 7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-Csv                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-Json                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-Markdown                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertFrom-SecureString                           7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          ConvertFrom-StringData                             7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-CliXml                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-Csv                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-Html                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-Json                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          ConvertTo-SecureString                             7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          ConvertTo-Xml                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Copy-Item                                          7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Copy-ItemProperty                                  7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Debug-Job                                          7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Debug-Process                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Debug-Runspace                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Disable-ExperimentalFeature                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Disable-PSBreakpoint                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Disable-RunspaceDebug                              7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Enable-ExperimentalFeature                         7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Enable-PSBreakpoint                                7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Enable-RunspaceDebug                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Enter-PSHostProcess                                7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Enter-PSSession                                    7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Exit-PSHostProcess                                 7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Exit-PSSession                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Export-Alias                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Export-Clixml                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Export-Csv                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Export-FormatData                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Export-ModuleMember                                7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Export-PSSession                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Find-Package                                       1.4.8.1    PackageManagement
Cmdlet          Find-PackageProvider                               1.4.8.1    PackageManagement
Cmdlet          Find-PSResource                                    1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          ForEach-Object                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Format-Custom                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Format-Hex                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Format-List                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Format-Table                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Format-Wide                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Alias                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-ChildItem                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-Clipboard                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-CmsMessage                                     7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Get-Command                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-Content                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-Credential                                     7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Get-Culture                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Date                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Error                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Event                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-EventSubscriber                                7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-ExecutionPolicy                                7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Get-ExperimentalFeature                            7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-FileHash                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-FormatData                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Help                                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-History                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-Host                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-InstalledPSResource                            1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Get-Item                                           7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-ItemProperty                                   7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-ItemPropertyValue                              7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-Job                                            7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-Location                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-MarkdownOption                                 7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Member                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Module                                         7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-Package                                        1.4.8.1    PackageManagement
Cmdlet          Get-PackageProvider                                1.4.8.1    PackageManagement
Cmdlet          Get-PackageSource                                  1.4.8.1    PackageManagement
Cmdlet          Get-PfxCertificate                                 7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Get-Process                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-PSBreakpoint                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-PSCallStack                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-PSDrive                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-PSHostProcessInfo                              7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-PSProvider                                     7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-PSReadLineKeyHandler                           2.3.6      PSReadLine
Cmdlet          Get-PSReadLineOption                               2.3.6      PSReadLine
Cmdlet          Get-PSResourceRepository                           1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Get-PSScriptFileInfo                               1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Get-PSSession                                      7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Get-Random                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Runspace                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-RunspaceDebug                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-SecureRandom                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-TimeZone                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Get-TraceSource                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-TypeData                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-UICulture                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Unique                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Uptime                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Variable                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Get-Verb                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Group-Object                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-Alias                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-Clixml                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-Csv                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-LocalizedData                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-Module                                      7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Import-PackageProvider                             1.4.8.1    PackageManagement
Cmdlet          Import-PowerShellDataFile                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Import-PSSession                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Install-Package                                    1.4.8.1    PackageManagement
Cmdlet          Install-PackageProvider                            1.4.8.1    PackageManagement
Cmdlet          Install-PSResource                                 1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Invoke-Command                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Invoke-Expression                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Invoke-History                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Invoke-Item                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Invoke-RestMethod                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Invoke-WebRequest                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Join-Path                                          7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Join-String                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Measure-Command                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Measure-Object                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Move-Item                                          7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Move-ItemProperty                                  7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          New-Alias                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-Event                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-Guid                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-Item                                           7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          New-ItemProperty                                   7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          New-Module                                         7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-ModuleManifest                                 7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-Object                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-PSDrive                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          New-PSRoleCapabilityFile                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-PSScriptFileInfo                               1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          New-PSSession                                      7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-PSSessionConfigurationFile                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-PSSessionOption                                7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-PSTransportOption                              7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          New-TemporaryFile                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-TimeSpan                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          New-Variable                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Out-Default                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Out-File                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Out-Host                                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Out-Null                                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Out-String                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Pop-Location                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Protect-CmsMessage                                 7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Publish-PSResource                                 1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Push-Location                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Read-Host                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Receive-Job                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Register-ArgumentCompleter                         7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Register-EngineEvent                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Register-ObjectEvent                               7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Register-PackageSource                             1.4.8.1    PackageManagement
Cmdlet          Register-PSResourceRepository                      1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Remove-Alias                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Remove-Event                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Remove-Item                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Remove-ItemProperty                                7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Remove-Job                                         7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Remove-Module                                      7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Remove-PSBreakpoint                                7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Remove-PSDrive                                     7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Remove-PSReadLineKeyHandler                        2.3.6      PSReadLine
Cmdlet          Remove-PSSession                                   7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Remove-TypeData                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Remove-Variable                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Rename-Item                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Rename-ItemProperty                                7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Resolve-Path                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Restart-Computer                                   7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Save-Help                                          7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Save-Package                                       1.4.8.1    PackageManagement
Cmdlet          Save-PSResource                                    1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Select-Object                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Select-String                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Select-Xml                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Send-MailMessage                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-Alias                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-Clipboard                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-Content                                        7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-Date                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-ExecutionPolicy                                7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Set-Item                                           7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-ItemProperty                                   7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-Location                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Set-MarkdownOption                                 7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-PackageSource                                  1.4.8.1    PackageManagement
Cmdlet          Set-PSBreakpoint                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-PSDebug                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Set-PSReadLineKeyHandler                           2.3.6      PSReadLine
Cmdlet          Set-PSReadLineOption                               2.3.6      PSReadLine
Cmdlet          Set-PSResourceRepository                           1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Set-StrictMode                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Set-TraceSource                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Set-Variable                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Show-Markdown                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Sort-Object                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Split-Path                                         7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Start-Job                                          7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Start-Process                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Start-Sleep                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Start-ThreadJob                                    2.0.3      ThreadJob
Cmdlet          Start-Transcript                                   7.0.0.0    Microsoft.PowerShell.Host
Cmdlet          Stop-Computer                                      7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Job                                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Stop-Process                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Stop-Transcript                                    7.0.0.0    Microsoft.PowerShell.Host
Cmdlet          Switch-Process                                     7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Tee-Object                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Test-Connection                                    7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Test-Json                                          7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Test-ModuleManifest                                7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Test-Path                                          7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Test-PSScriptFileInfo                              1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Trace-Command                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Unblock-File                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Uninstall-Package                                  1.4.8.1    PackageManagement
Cmdlet          Uninstall-PSResource                               1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Unprotect-CmsMessage                               7.0.0.0    Microsoft.PowerShell.Security
Cmdlet          Unregister-Event                                   7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Unregister-PackageSource                           1.4.8.1    PackageManagement
Cmdlet          Unregister-PSResourceRepository                    1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Update-FormatData                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Update-Help                                        7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Update-List                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Update-PSModuleManifest                            1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Update-PSResource                                  1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Update-PSScriptFileInfo                            1.1.0      Microsoft.PowerShell.PSResourceGet
Cmdlet          Update-TypeData                                    7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Wait-Debugger                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Wait-Event                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Wait-Job                                           7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Wait-Process                                       7.0.0.0    Microsoft.PowerShell.Management
Cmdlet          Where-Object                                       7.5.0.500  Microsoft.PowerShell.Core
Cmdlet          Write-Debug                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Error                                        7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Host                                         7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Information                                  7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Output                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Progress                                     7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Verbose                                      7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Write-Warning                                      7.0.0.0    Microsoft.PowerShell.Utility

PS /Users/pradeep> 
```

```
PS /Users/pradeep> Get-Uptime 

Days              : 1
Hours             : 20
Minutes           : 14
Seconds           : 10
Milliseconds      : 0
Ticks             : 1592500000000
TotalDays         : 1.8431712962963
TotalHours        : 44.2361111111111
TotalMinutes      : 2654.16666666667
TotalSeconds      : 159250
TotalMilliseconds : 159250000

PS /Users/pradeep> Get-Date  

Saturday, March 8, 2025 8:50:36 AM

PS /Users/pradeep> 
```

```
PS /Users/pradeep> Get-Command -Verb Get -Noun File*

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-FileHash                                       7.0.0.0    Microsoft.PowerShell.Utility

PS /Users/pradeep> Get-Command  -Noun File*         

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Cmdlet          Get-FileHash                                       7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Out-File                                           7.0.0.0    Microsoft.PowerShell.Utility
Cmdlet          Unblock-File                                       7.0.0.0    Microsoft.PowerShell.Utility

PS /Users/pradeep> 
```



[Deploy Azure infrastructure by using JSON ARM templates 						](https://learn.microsoft.com/en-us/training/modules/create-azure-resource-manager-template-vs-code/)

