---
title: "Windows heap inspection for beginners with Frida"
date: 2018-07-30 20:00:00
excerpt: ""
---

After I got OSCE my interest about memory exploitation growing. Like every baby when move his first steps, I am really excited when I learn and experimenting new stuff about this topic. A couple of days ago I read a [great article](http://deniable.org/reversing/binary-instrumentation) about binary instrumentation using multiple tools (i.e. Pin, DynamoRIO, Frida). That article introduces interesting concepts proposing code samples using Pin and DynamoRIO, and left the Frida implementation to the reader. So, I decided to try it myself.

The article writer says that he was playing with Heap Spraying and he felt the need of logging all the memory allocations his code was performing, so he wrote a Pintool to trace these allocations and detect basic memory leaks, double frees and invalid frees.

# Setup

I use `Microsoft Windows 7 Ultimate x86 6.1.7601 SP1` and `Python 3.6.6`. I tried to install Frida from remote pypi.org with command below but I had a lot of trouble:

```
C:\Python36-32\Scripts\>pip install frida
```

Instead, I downloaded directly the package `frida-12.0.7-py3.6-win32.egg` from `https://pypi.org/project/frida/`.

# ...

```python
import sys
import os
import frida

def get_script_path():

    return os.path.dirname(os.path.realpath(__file__)) + os.sep

def main():

    if len(sys.argv) != 2:
       print("%s <pid>" %(sys.argv[0]))
       sys.exit(-1)

    pid = int(sys.argv[1])
    print("[i] attaching to process: %d" %(pid))

    session = frida.attach(pid)

    js_script = open(get_script_path() + 'hello-frida.js').read()
    script = session.create_script(js_script)
    script.load()

    sys.stdin.read()
		 
if __name__ == "__main__":
   main()
```

# References

- [Frida](https://www.frida.re/docs/building/)
- [Dynamic Binary Instrumentation Primer](http://deniable.org/reversing/binary-instrumentation)