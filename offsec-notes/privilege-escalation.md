# Privilege Escalation

### Enumerate Users

```
#Linux and Windows
whoami

#windows
net user

#linux
id
cat /etc/passwd
```

### **Enumerating the Hostname**

```
#linux/windows
hostname
```

### **Enumerating OS Version and Architecture**

```
#windows
systeminfo
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"

#linux
cat /etc/issue
cat /etc/*-release
uname -a
```

### **Enumerating Running Processes and Services**

```
# windows
# return processes that are mapped to a specific Windows service
tasklist /SVC

#linux
# list system processes (including those run by privileged users)
ps axu
```

### **Enumerating Networking Information**

```
# windows
ipconfig /all # to display the full TCP/IP configuration of all adapters
route print # To display the networking routing tables
netstat -ano # to view the active network connections

# linux
ip a #list the TCP/IP configuration of every network adapter
/sbin/route(l) # display network routing tables
ss -anp # display active network connections and listening ports 
```
