# Privilege Escalation

###

###

### **Enumerating Running Processes and Services**

```
# windows
# return processes that are mapped to a specific Windows service
tasklist /SVC
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows"

#linux

```

### **Enumerating Installed Applications and Patch Levels**

```
# windows
# # lists applications that are installed by the Windows Installer
wmic product get name, version, vendor 
#  list system-wide updates by querying the Win32_QuickFixEngineering (qfe) WMI class
wmic qfe get Caption, Description, HotFixID, InstalledOn

# linux
dpkg -l
```

### **Enumerating Readable/Writable Files and Directories**

```
# windows
accesschk.exe -uws "Everyone" "C:\Program Files"
Get-ChildItem "C:\Program Files" -Recurse | Get-ACL | ?{$_.AccessToString -match "Everyone\sAllow\s\sModify"}
```

Specifically, we will enumerate the Program Files directory in search of any file or directory that allows the _Everyone_ group _write_ permissions.

We will use -u to suppress errors, -w to search for write access permissions, and -s to perform a recursive search

```
# linux
find / -writable -type d 2>/dev/null
```

### **Enumerating Unmounted Disks**

```
# windows
mountvol

# linux
mount #  list all mounted filesystems
cat /etc/fstab # lists all drives that will be mounted at boot time
/bin/lsblk # view all available disks
```

### **Enumerating Device Drivers and Kernel Modules**

```
# windows
driverquery.exe /v /fo csv | ConvertFrom-CSV | Select-Object ‘Display Name’, ‘Start Mode’, Path
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion, Manufacturer | Where-Object {$_.DeviceName -like "*VMware*"}

# linux
lsmod # list loaded kernel modules
/sbin/modinfo [module] # list additional info about a module
```

### **Enumerating Binaries That AutoElevate**

We should check the status of the _AlwaysInstallElevated_ registry setting. If this key is enabled (set to _1_) in either _HKEY\_CURRENT\_USER_ or _HKEY\_LOCAL\_MACHINE_, any user can run Windows Installer packages with elevated privileges. If this setting is enabled, we could craft an _MSI_ file and run it to elevate our privileges.

```
# windows
c:\Users\student>reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1


c:\Users\student>reg query HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\Installer
    AlwaysInstallElevated    REG_DWORD    0x1
    
# linux

```









