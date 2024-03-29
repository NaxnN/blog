## autologon via editing registry editor
In HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon, set the following item:
```powershell
"AutoAdminLogon"="1"
"DefaultUserName"="username"
"DefaultPassword"="password"
"DefaultDomainName"="domain.local"
```

## autologon vis sysinternal's autologon.exe
* domain name and username location: `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon`
* encrypted password location: `HKEY_LOCAL_MACHINE/ Security/ Policy/ Secrets/DefaultPassword` (`psexec.exe -i -s regedit`)

## Reference
* https://keithga.wordpress.com/2013/12/19/sysinternals-autologon-and-securely-encrypting-passwords/
* https://www.thewindowsclub.com/decrypt-the-defaultpassword-value-saved-in-registry-for-autologon
