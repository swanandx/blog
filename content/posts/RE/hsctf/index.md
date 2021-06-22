---
author: swanandx
title: "Reverse Engineering : Java code"
date: 2021-06-22
description: Reverse Engineering Java source code to get the flag.
series: ["Reverse Engineering"]
tags: ["RE","CTF"]
categories: ["CTF"]
ShowBreadCrumbs: true
---

# Challenge

This was a easy challenge from HSCTF 8. We were given source code of a program which we had to reverse engineer and find the flag.

```java
import java.util.Scanner;

public class WarmupRev {
  
    public static String cold(String t) {
        return t.substring(17) + t.substring(0, 17);
    }
    
    public static String cool(String t) {
        String s = "";
        for (int i = 0; i < t.length(); i++)
            if (i % 2 == 0)
                s += (char) (t.charAt(i) + 3 * (i / 2));
            else
                s += t.charAt(i);
        return s;
    }
        
    public static String warm(String t) {
        String a = t.substring(0, t.indexOf("l") + 1);
        String t1 = t.substring(t.indexOf("l") + 1);
        String b = t1.substring(0, t1.indexOf("l") + 1);
        String c = t1.substring(t1.indexOf("l") + 1);
        return c + b + a;
    }
    
    public static String hot(String t) {
        int[] adj = {-72, 7, -58, 2, -33, 1, -102, 65, 13, -64, 
                21, 14, -45, -11, -48, -7, -1, 3, 47, -65, 3, -18, 
                -73, 40, -27, -73, -13, 0, 0, -68, 10, 45, 13};
        String s = "";
        for (int i = 0; i < t.length(); i++)
            s += (char) (t.charAt(i) + adj[i]);
        return s;
    }

    public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.print("Let's get warmed up! Please enter the flag: ");
        String flag = in.nextLine();
        String match = "4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy";
        if (flag.length() == 33 && hot(warm(cool(cold(flag)))).equals(match))
            System.out.println("You got it!");
        else
            System.out.println("That's not correct, please try again!");
        in.close();
    }
  
}
```

I never had used Java before, so lets see how I approached this challenge and got flag.

![Java](./img/java.gif#center)

***

## `main` function

```java
public static void main(String[] args) {
        Scanner in = new Scanner(System.in);
        System.out.print("Let's get warmed up! Please enter the flag: ");
        String flag = in.nextLine();
        String match = "4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy";
        if (flag.length() == 33 && hot(warm(cool(cold(flag)))).equals(match))
            System.out.println("You got it!");
        else
            System.out.println("That's not correct, please try again!");
        in.close();
    }
```

As we can see, the main logic is :

`
input(of length 33) -> cold -> cool -> warm -> hot -> match with "4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy"
`

So, in order to get the flag, lets reverse the logic :

`
"4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy" -> hot -> warm -> cool -> cold -> Flag!!!
`

Now we know the logic, lets start coding our script. I will be using python.

***

## `hot` function

```java
    public static String hot(String t) {
        int[] adj = {-72, 7, -58, 2, -33, 1, -102, 65, 13, -64, 
                21, 14, -45, -11, -48, -7, -1, 3, 47, -65, 3, -18, 
                -73, 40, -27, -73, -13, 0, 0, -68, 10, 45, 13};
        String s = "";
        for (int i = 0; i < t.length(); i++)
            s += (char) (t.charAt(i) + adj[i]);
        return s;
    }
```

After looking at the function, we can see that it is doing the following :

`
for every character in string -> add the number at the same index in adj
`

so, in order to reverse it, we need to subtract the number at same index.

```py
adj = [-72, 7, -58, 2, -33, 1, -102, 65, 13, -64, 21, 14, -45, -11, -48, -7, -1, 3, 47, -65, 3, -18, -73, 40, -27, -73, -13, 0, 0, -68, 10, 45, 13]
our_string = "4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy"
output_string = ""

for i in range(len(our_string)):
	output_string += chr(ord(our_string[i]) - adj[i])

print(output_string)

# Output is |g1c3[s]_^nyyk0u_GyJ}~l3nwh:l

```

***

## `warm` function

```java
    public static String warm(String t) {
        String a = t.substring(0, t.indexOf("l") + 1);
        String t1 = t.substring(t.indexOf("l") + 1);
        String b = t1.substring(0, t1.indexOf("l") + 1);
        String c = t1.substring(t1.indexOf("l") + 1);
        return c + b + a;
    }

```

I didn't know how substring works, so I did a quick search on Google and found out what it is doing :

`
a -> From start to l
`
`
b -> After l to next l
`
`
c -> remaining part
`

The string we have is in form of `c + b + a`, to reverse it, we need, `a + b + c`.

So `b` ends with `l`, and `a` ends with `l`, and `a` is at end, so from this we can say that `a` is `3nwh:l` this part.
But we can't tell which part is `c`, will see it later.

***

## `cool` function

```java
    public static String cool(String t) {
        String s = "";
        for (int i = 0; i < t.length(); i++)
            if (i % 2 == 0)
                s += (char) (t.charAt(i) + 3 * (i / 2));
            else
                s += t.charAt(i);
        return s;
    }

```

We can see the function is adding `3 * (i/2)` to the character if `i % 2 == 0`.
So, we will subtract `3 * (i/2)` from the character if `i % 2 == 0`.

```py
new_string = ""
for i in range(len(output_string)):
    if i % 2 == 0:
        new_string += chr(ord(output_string[i]) - int(3*(i/2)))
    else:
        new_string += output_string[i]
print(new_string)

```

After playing around with it for a bit, I figured out correct `b` and `c`, so it became

```py
output_string = output_string[-6:] + output_string[15:-6] + output_string[:15]
new_string = ""
for i in range(len(output_string)):
    if i % 2 == 0:
        new_string += chr(ord(output_string[i]) - int(3*(i/2)))
    else:
        new_string += output_string[i]
print(new_string)

# Prints 3nth4lpy_0f_5y5}flag{1ncr34s3_1n_
```

***

## `cold` function

```java
public static String cold(String t) {
        return t.substring(17) + t.substring(0, 17);
    }
```

By this time, we can already see the flag, all this function do is

![swap](./img/swap.gif#center)

Just swap it again which gives us the flag!!

Final script:

```py
adj = [-72, 7, -58, 2, -33, 1, -102, 65, 13, -64, 21, 14, -45, -11, -48, -7, -1, 3, 47, -65, 3, -18, -73, 40, -27, -73, -13, 0, 0, -68, 10, 45, 13]
our_string = "4n_3nd0th3rm1c_rxn_4b50rb5_3n3rgy"
output_string = ""

for i in range(len(our_string)):
    output_string += chr(ord(our_string[i]) - adj[i])

output_string = output_string[-6:] + output_string[15:-6] + output_string[:15]
new_string = ""
for i in range(len(output_string)):
    if i % 2 == 0:
        new_string += chr(ord(output_string[i]) - int(3*(i/2)))
    else:
        new_string += output_string[i]

flag = new_string[16:] + new_string[:16]
print(flag)
```

> flag{1ncr34s3_1n_3nth4lpy_0f_5y5}

***

# Summary

Don't be afraid of programming languages that you don't know, all we care about is how it is working. Syntax or working of a particular inbuilt function can be easily found by a quick google search. Just start!!

![got this](./img/dude.gif#center)
