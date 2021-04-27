---
author: swanandx
title: "Reversing Binaries: Key generators"
date: 2021-04-27
description: Reverse engineering binaries and writing key-gen for it.
series: ["Reverse Engineering"]
tags: ["RE"]
ShowBreadCrumbs: true
---

# What is in this post?
You might have seen something like "Product Keys" for some softwares or some other applications, but how exactly do they verify if the key is correct ? They implement a function that will validate key. We will be reverse engineering a binary, which validates a key using some function, and then we will write a key generator, i.e key-gen, for it to generate valid keys.

# Key checker binary
We got a binary that validates our key :

![keygen_no.png](./images/41a1217977df4f4ca7fe5957dd27dc7a.png#center)

## `main` function
Lets decompile it using `ghidra` and look at `main` function :

![main.png](./images/da99bf0f380545cfbfeec4b94d3c7cc0.png#center)

As we can see, it scans key using `scanf` and then pass it to `validate_key` function. Before going ahead, lets see what is the first argument of `scanf` :

![scanf.png](./images/f360ab23be7b4c11bd6800905a894102.png#center)

So the binary is using `scanf("%d",&local_14)`, by this we can say that key will be integer. Now, looking at `validate_key` function.

## `validate_key` function

![validate.png](./images/90464be03f9a4241813c44a4aaf003f5.png#center)

Here, it will return true if our key is perfectly divisible by `0x4c7` which is `1223` in decimal.
Therefore, any number divisible by `1223` is a valid key. Lets verify this :



![1223.png](./images/21108b27f9e04ff5b073c9c748fed3ac.png#center)

> As you can see, `-1223` is also a valid key, as in the end remainder is `0`.

Now we know the logic, lets write a key generator for this.

# Writing key-gen

From the analysis, we know the following:
- Key is a integer.
- It should be divisible by `1223`.

Therefore, we can say, key can range from `-2147483647` to `2147483648` as per `INT_MIN` and `INT_MAX` in `C`. But as our key has to be divisible by `1223`, our range become `-2147482822` to `2147482822`,as they are smallest and largest multiple of `1223` in our range.
```py
MAX_INT = 2147482822
MIN_INT = -2147482822
```
Then, we generate a random integer in that range by using `random` 
```py
import random
random_number = random.randint(MIN_INT,MAX_INT)
```
Now, we have a random integer, but we need it to be divisible by `1223`. So, we divide the random number by `1223` and then subtract the remainder from it.
```
key = random_number - (random_number % 1223)
```
So, final script becomes,

```py
import random

MAX_INT = 2147482822
MIN_INT = -2147482822

random_number = random.randint(MIN_INT,MAX_INT)

key = random_number - (random_number % 1223)

print(key)
```

Lets execute it, and see if it generates valid keys.

![verify.png](./images/afb6a73868c44c02b6846cdfb87f8c63.png#center)

We did it!! We successfully wrote a key generator that generates valid keys!!

> **NOTE** : we didn't have any restrictions on length of key, but if there was any, like key length must be 12, then we can just prepend the generated key with `0`s till it's length is 12. For example,
![00s.png](./images/e9c19276d70e492f967084c5454feab7.png#center)

# Try it yourself!
What are you waiting for? Compile some binaries and try to write key-gen for it. This will help you in understanding and will improve scripting skills too.
