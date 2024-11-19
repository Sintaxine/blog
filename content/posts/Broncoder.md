+++ 
draft = false
date = 2024-11-19T12:37:33+01:00
title = "Broncoder.wsf Malware Analysis: USB Worm"
description = "Today we'll be analysing a WSF worm that spreads through USB."
slug = ""
authors = ["miku"]
tags = ["Cybersecurity", "Malware Analysis", "Reverse Engineering"]
categories = ["Cybersecurity"]
externalLink = ""
series = []
+++


# Introduction

I found this file called **Broncoder.wsf** on the USB I gave my mom in order for her to put on a movie for her middle school students.
When asked where the USB has been, she said an old Windows 7 Laptop.
So I decided to analyse it.


## Analysis Goals:
- Trolling
- Fun 
- Knowledge

### WARNING 
**DO NOT EXECUTE ANY CODE PROVIDED, THIS WAS ANALYSED IN A LINUX MACHINE. I AM NOT RESPONSIBLE FOR ANY DAMAGE DONE TO YOUR SYSTEM!**
# Initial Analysis

I started by opening the Broncoder.wsf in **Neovim**, the text editor of all time.
I found the following code:

```vb
<?XML version="1.0"?><job>
<script language="VBScript">
<![CDATA[
'Coded By BronCoder { BronCoder@Gmail.com }
Function opqj(opqjp)
  For mehokg = 1 To Len(opqjp)
    mehokgp = mehokgp  + (Mid(opqjp, Len(opqjp) - mehokg + (11 - 10), (152 - 151)) * ((121 - 119) ^ (mehokg - (5 - 4))))
  Next
  opqj = mehokgp
End Function
mehokgpd = 'ALOT of binary seperated by "#"'
mehokgpdp = ""
For Each mehokgpdpa In Split(mehokgpd, "#")
  mehokgpdp = mehokgpdp & chr(opqj(mehokgpdpa))
next
Execute mehokgpdp
]]>
</script>
</job>
```

This guy's obsfucation technique is really bad. Here are a couple rookie mistakes he made:

1. Left his name, and email address in  MALWARE code.
2. Left the entire algorithm that deobfuscates his code in the main code that can be seen by anyone. 
3. Poor obfuscation. This type of basic obfuscation can easily be decompiled or debugged.
4. Relying on such simple methods to hide malware is ineffective against modern detection tools.


# Analysing the WSF script

## Initial Observations:
- This script builds a payload (mehokgpdp) dynamically and executes it with the **Execute** statement.
- This is an obvious red flag, that is often flagged by antivirus software. 
**This poorly written code makes several rookie mistakes, making it really easy to analyze.**

## Understanding the Script:
The function "opqj" takes a parameter "oqpjp", then for every character in that string, it uses powers and other mathematical operations.
**This is clearly a decryption function.**




After the binary string is intialized in the variable "mehokgpd", the script is intiliazing an empty string "mehokgpdp". **This variable is going to store the decrypted script.**

The binary string is seperated by #'s.  A For loop grabs the binary data and converts it to characters. The output is stored in the mehokgpdp variable.
The script then executes the data.

**Now let's get to the root.**
# Extraction of the real malicious code.

I started by simply writing his decryption function in python. Here's the code:
```python

def opqj(opqjp):
    mehokgp = 0
    for mehokg in range(1, len(opqjp) + 1):
        mehokgp += int(opqjp[len(opqjp) - mehokg]) * (2 ** (mehokg - 1))
    return mehokgp

mehokgpd = #binary data
decoded_payload = ""

for mehokgpdpa in mehokgpd.split("#"):
    decoded_payload += chr(opqj(mehokgpdpa))

print(decoded_payload)


```


Just like that, I echoed the output to **output.wsf**, and we now have the real payload.
Shows you how bad this guy is.




# The Payload 

* The worm installs itself in the "%temp%" directory and creates a startup entry ensuring running at system boot.
* It uses registry keys to maintain persistence:

```VBScript
shellobj.regdelete "HKEY_CURRENT_USER\software\microsoft\windows\currentversion\run\" & split (installname,".")(0)
```

* It communicates with a remote server (serviceupdating.ddns.net) on port 82. It receives commands like uninstalling, updating itself or executing code.
* Responds with system info or results of those commands.

## The USB Propagation:
* The worm spreads by copying itself to ALL connected USB drives.
* It hides its payload file by setting file attributes like Hidden and System.
* It creates .lnk (shortcut) files that look like legit files / folders, which will execute the payload if opened.
* It can delete files, create shortcuts and manipulate file attributes.


# Weaknesses:
* Hard coded server address
* The techniques are too simple. The .lnk file method is easily detectable by any antivirus.
* Despite the author's pitiful attempt at obfuscation, the VBScript payload is really ease to reverse engineer.



# Summary:

Really bad. This is really bad. 
Thank you for reading.
