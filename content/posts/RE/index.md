---
author: swanandx
title: "Reversing Binaries: CTF Edition"
date: 2021-04-18
description: Methodology for reverse engineering ctf challenges
series: Reverse Engineering
tags: ["RE","CTF"]
categories: ["CTF"]
ShowBreadCrumbs: true
---

# What is this about?
I saw a lot of people struggling to solve easy reverse engineering (RE) challenges (me too when I was just starting), well, I am not saying I am great at it now, but have really improved my skills so that now I can solve many RE challenges in ctfs. That is why, I am writing this blog, hoping it will help at least a few people.

# But what is in RE challenges?

Basically, in RE challenges, you will be getting an application or executable, of which, we have to find the correct input which will lead to desired output by doing reverse engineering, end goal is to get the flag. I mostly saw the following types of executables:

- **Password checker** - This will take password as input, and will print flag if password is correct.
- **Flag checker** - This will take the flag as input and check if it is correct.

# So I got binary in the challenge, now what?
We have to basically find the _flag_ , which can be in different formats like THM{}, ictf{}, etc. or just digits, really depending on the challenge.

# I get it, but how?
Here is my simple methodology, or path, for solving it.

![path](./images/path.png#center)

> You can use radare2 / gdb instead of Ghidra for static analysis too if you are familiar and comfortable with assembly.

I will elaborate why I choose to do it like that later in the blog.

## strings
`strings` is very useful yet so simple command. 

![0a6a3b10bb3aec43d5b9beb96e80b044.png](./images/005bd2b6f25443479718e8af4f83718e.png#center)

As per manual, it prints the printable characters in a file. So why it is helpful?
Well, if the flag is present as printable characters, we can find it straight away.
Here is example, suppose I have a binary `rev_me`

![a75a930bff416e16dab39a06e9a5d502.png](./images/3490a905f88048eaa4e935800b8e418d.png#center)

I ran `strings rev_me` and found the password and flag :)

![b39d978974ce39958bd39613eda91592.png](./images/fd929c91cbd44b1fb1426ac913723769.png#center)

 Here is proof,
 
 ![5df05378c7e2e7795ac90e7c10dd6af2.png](./images/968170f05c7647389dba1c0c7ab43d05.png#center).
 
 - why start with `strings` ? --> You will find a lot of challenges that can be solved using this single command. If you just run it and found the flag, it will save a lot of your precious time.

> **NOTE**: `strings` command is also very helpful in malware analysis, check out [John Hammond](https://www.youtube.com/c/JohnHammond010/videos) malware analysis videos on YouTube or [MAL:strings](https://tryhackme.com/room/malstrings) room on TryHackMe for that.

## ltrace
Now what if flag was not in strings? Next place is to trace library calls such as `strcmp()` if they are being used, we do it by `ltrace`.

![d103acfbe36cf136388425dd84aa61f9.png](./images/69acf362f0e94ed085452ea973f3198e.png#center)

Syntax is `ltrace /path/to/binary`
It will show the library calls being used like:
```
...
strcmp('idk','p@ssw0rd')
...
```
> **IMP** : I did not show example as now-a-days, `ltrace` is not able to trace calls due to the way in which gcc compiles program. But it still can be useful in ctfs.
It only takes 2 sec. to do it, so, always give it a try, we never know.

***
Now lets begin some real cool stuff. Let me first elaborate why I divided challenges in two categories in beginning.

***
## Password checker
This binaries have work flow like:
```
1. Take input -> 2. check if input matches the password -> 3. print flag if it matches.
```
Our main goal is to get flag, so instead of figuring out password, we directly jump to code where it prints the flag.

For example, I have a binary which prints flag if password is correct.
Let us use `gdb`

![52bc4fb12d3beebd382466f34d77be32.png](./images/b241c1e579924d59aca5046d18b50c90.png#center)

As you can see, I disassembled the main function. In the main function

![cac26373a57c04f6f29094bc18bb2a9d.png](./images/fef434cc8c7d46eea01600ed0499df9c.png#center)

we are looking for `cmp` here. we can see it at `<+103>` . It compares 2 values and if they are equal, it jumps to `<main+129>`. We can tell that the code at `<main+129>` prints the flag as it is called if compare is successful, so let's set a breakpoint just after it takes input, i.e. `<main+68>` and then run it.

![208f810cfb5561dfea4cbf156dbfb3d6.png](./images/508fc5d2f0024fce83566c9329e09d63.png#center)

we supply it with any random password, as we are just going to skip the check anyway.
Now, lets jump to `<main+129>` and see what happens.

![c2ae8affab305a8a3f67c6b7f2055814.png](./images/156dc22fb8c540ffb60ebfa64337a5e4.png#center)

We successfully got the flag!!

> **IMP**: same can be done using radare2. instead of `jump`, use `dr rip={address to jump}` and then continue it, rest procedure is similar.

## Flag checker
Now what if the flag itself is the password? work flow is like
```
Take flag as input -> check if it is correct.
```
Now, no use of bypassing the check, as our goal is to get the flag. So how to do it?
Let us say we have binary called `rev` like:

![9781fa4d82a217e25625c5d7879068af.png](./images/d6536584c29e4894917528d710859213.png#center)

So, in order to find flag, I used Ghidra. Open ghidra, create new project and then import the binary.

> You can learn about ghidra more from [CC:Ghidra](https://tryhackme.com/room/ccghidra) room on TryHackMe or there are many tutorials available for it.

After analyzing binary, I checked the _pseudo code_ generated by ghidra.

In `main` function, you can see it is xoring every character with `5` and then adding `5` to it , then it is comparing result with `flag`.


![main.png](./images/09370b9f8848484dba3ead4cafe53634.png#center)


Let's check what is in `flag` and take note of those values.



![flag.png](./images/224739b99dd74f37a2a7a2555005a17f.png#center)


Now, we know what the result is, let's find what leads to it, for that, write a script that will take character from result then subtract `5` and then xor against `5`. We are doing the operations it did to our input in reverse order.

> NOTE: If you wonder why it looks so complicated, well , that is just how ghidra works. Try compiling own binaries and then reversing it to see how ghidra is generating the code. 

```py
flag_bytes = [0x48, 0x4e, 0x49, 0x47, 0x83, 0x82, 0x3a, 0x7c, 0x5f, 0x82, 0x6f, 0x7c, 0x5f, 0x82, 0x3a, 0x7c, 0x7d]

flag = ""
for x in flag_bytes:
	flag += chr((x - 5)^5)

print(flag)
```
Executing the python script gives the flag!


![Screenshot_2021-04-17_10-42-55.png](./images/565b5989acb144beb1a02bcc1ec98ca9.png#center)

Let's verify the flag
![93c9847c8ff54c6d9a4c481b61d2861d.png](./images/3d6cd59f934f43b8baa4d6eab8dabf88.png#center)

> **IMP :** You could have solved the other challenges using ghidra, as it gives the _pseudo code_ which is very helpful. It might be wrong sometimes though.
I preferred to use Ghidra in end as it is resource heavy, but if you have resources, you can run it right away.

## What if challenge is not linux executable?
- For `.NET assembly` use `ilspy` or some other `.NET decompiler`
- For `.apk`, There are a few steps here with different tools, but I just search for `online apk decompiler` on Google and use online tools.
- You can also decompile `.exe` binaries using ghidra.


# Wicked cool tricks

This are some tricks that I developed through practice
- If you see hex numbers getting assigned to variables in ghidra or something, always look their ASCII value. That might be the flag or password just in front of you.
- A lot of `.java` files after decompiling `.apk`? Suppose flag format is `flag{}`, just go in directory where there are all `.java` files and run `cat *.java | grep flag{` or you can also try something like `grep -iRl flag{`.
- For first time, loops in pseudo code generated by ghidra might look complicated as there are pointers `**(int)((long)local28 + 4)`, don't worry about that much, just focus on our goal and understanding the flow.
- If there is no `main` function, look `entry` function, usually first function called there is `__libc_start_main` and the first argument to it is usually the `main` function.
- If binary prints like `wrong`, `give flag` or something, look at which function is responsible for it, that is most probably the `main` function.
I suggest compile some binaries on own and look at how ghidra is generating pseudo code. This will be very helpful in understanding.
- You can use `angr`. It is a python framework for analyzing binaries. Check out videos and tutorials for it. I am sure it will worth it.

# What next?

TL;DR - **practice!!**

We just barely scratched surface of reverse engineering. This is just like a methodology, you need to practice a lot if you want to master it. Different ways of solving challenge, looking at assembly and pseudo code, where to look for flags and what might be rabbit hole etc.

Do not worry if you think like you are not getting what application is doing or not able to solve challenges, it all comes with practice.
There are many awesome resources and many lists of them you can find online, use them to learn more and keep practicing.