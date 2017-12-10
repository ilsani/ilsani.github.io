---
title:  "Does this decryptor really decrypts my files?"
date:   2017-08-13 10:30:00
preview: "A story of my last week before the summer holiday and a malware."
---

In this post I'll try to summarize a story of my last week before the summer holiday. I must avoid some details, but I hope that the story is interesting.

## The Armageddon Day

The week started as usual, but Monday morning became more interesting very quickly. One of my work contact wrote me that one of his Clients has found strange files into one of his server and some documents were corrupted.
Just after a brief description it was clear that the server has been compromised by a ransomware.<br />

I think that in this case the first questions are: **DID I BACKUP MY DATA?** and **WHERE ARE MY BACKUP?**<br />

Unfortunately those guys had not any backup. I do not know the rest of their days but I'm sure that they did not get bored.

## The Reckoning Day

After a couple of days my contact rewrote to me saying that his Client has decided to pay the ransom in order to receive the instructions to decrypt his documents. In this case anybody has the certainty to receive some answers after the payment but this is the last attempt, of course.

After the payment they received 3 files:
1. decrypt.exe
2. cfg.cfg
3. decrypt.key

Just for fun I asked to see these files... and:

* *decrypt.exe* is an EXE file and should be the tool that decrypts the encrypted documents
* *cfg.cfg* is a plain text file that contains a line: *.zip,.mdb*
* *decrypt.key* is a binary file and should be the key that *decrypt.exe* uses to decrypt the documents

After a first review nothing strange about these files except the *cfg.cfg* file. Why the decryptor needs a text file with the strings *.zip* and *.mdb* into it? Maybe are these extensions the next files types to encrypt?!

Running an executable file received by the person that has encrypted your data could not be the best thing to do :-)

I asked to VirusTotal to analyze the *decrypt.exe* file and only 3/64 engines detect this file as malicous. So, maybe this file is not a malware or has been coded in a super-ninja way. I decided to reverse engineering this file in order to understand its behavior.

## Reversing the decryptor (the funny part of this story)

* File name: decrypt.exe
* File type: PE32 executable (GUI) Intel 80386, for MS Windows
* MD5: 657e5ca06d98b09f4d9c46916a5bd312

I started my analysis dumping some general information about this file, and with the help of objdump I extracted the imported DLLs.

```
$ objdump -x decrypt.exe  |grep "DLL"

DLL Name: USER32.dll
DLL Name: KERNEL32.dll
DLL Name: ADVAPI32.dll
DLL Name: COMDLG32.dll
DLL Name: SHELL32.dll

```

This file imports a lot of functions. Such as:
* USER32.dll to import User Interface API
* KERNEL32.dll to import some string, file, memory and process manipulation API
* ADVAPI32.dll to import cryptographic API (e.g. CryptDecrypt)

`IsDebuggerPresent()` is also imported from KERNEL32.dll but looks like that the decryptor does not use it in its main code. Anyway bypassing this anti-debugger protecion should be easy, if needed.

As all Win32 executable file, *decrypt.exe* has an entry function: WinMain(). After retrieving some environmental information such as the current directory path, the `sub_406080` function is called in order to parse the command line arguments. If command line arguments are less than 2 the function sets the name of the file that contains the decryption key to *decrypt.key* and exits.

![parse main args]({{ site.url }}/assets/images/posts/does-this-decryptor-really-decrypts-my-files/parse-main-args-1.png)
![decryption key file name]({{ site.url }}/assets/images/posts/does-this-decryptor-really-decrypts-my-files/parse-main-args-2.png)

Otherwise, the function will parse command line arguments if are provided. The parser logic is a straightforward if...else, and the supported arguments are: *-f <<param1>>* and *-k <<param2>>*

![parse main args if...else]({{ site.url }}/assets/images/posts/does-this-decryptor-really-decrypts-my-files/parse-main-args-3.png)

Where *-f* is the file to decrypt and *-k* is the filename that contains the decryption key.

This decryptor has 2 main execution modes:
1. Batch mode: when above command line arguments are provided it opens a command prompt shell and starts the decryption routine
2. Interactive mode: when no command line arguments are provided it opens a Win32 window to ask which operation to execute

![win32-gui]({{ site.url }}/assets/images/posts/does-this-decryptor-really-decrypts-my-files/win32gui.png)

Both modes run the same routine to start the decryption phase: sub_405AA0.

sub_405AA0 function initializes some settings and calls sub_406E80, where the real decryption of data takes place. The decryption is performed with the CryptDecrypt() Win32 API, which takes the pointer to the encrypted data and the pointer to the decryption key.

![cryptdecrypt]({{ site.url }}/assets/images/posts/does-this-decryptor-really-decrypts-my-files/cryptdecrypt.png)

At the end of this brief analysis it seems that this decryptor really decrypts the data without performing malicious activities :-)

And the *cfg.cfg* file? At this time looks like that this file is never used inside the *decrypt.exe*.


## Conclusion

In this post I focused only to some technical points of the story. I did not deliberately deal some hot topics, such as the importance to have an efficient incident management plan, an efficient disaster recovery plan, an efficient security and vulnerability management plan.

## References
* [CryptDecrypt() Win32 API](https://msdn.microsoft.com/en-us/library/windows/desktop/aa379913(v=vs.85).aspx)