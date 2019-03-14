---
layout: post
title: Hack The Box
tags: [Defense Against the Dark Arts]
---

In this post, I will walkthrough some challenges from [Hack The Box](https://www.hackthebox.eu/login).

# Find The Easy Pass
### Tools that I used:
- PE Explorer
- Immunity Debugger 

### Step:
1. First I used PE Explorer, to analyze `EasyPass.exe`. I checked the string dump for any suspicious activity. To do this, click on the disassemblr icon, this will convert the machine code of the executable into readable assembly language.

2. You will notice some information is given on the right most column. I looked for strings that would give us a hint on what the password is. As I was searching for suspicious strings, I found this:
![img](https://drive.google.com/uc?export=view&id=1kWxOt-CqiouhTy2xVpx8BNBBhKEx1AMy)

3. Looking at the figure above, the two strings `..._Good_Job__Congratulations` and `..._Wrong_Password_` looks like it could be a hint on where we can start analyzing. With these two strings, my guess is that, this is the location where it checks if the user has entered the right password.

4. Let's use a debugger to get a better understanding on what's happening. I'm using Immunity Debugger but you can also use other debuggers as well (I have used x32dbg which also works).

5. To find the same strings we found in PE explorer, we need to analyze the `EasyPass.exe` module. To do this,
`Right Click > View > Module 'EasyPass.exe'`

6. Now, you can find the same location you saw the strings in PE Explorer.
![img](https://drive.google.com/uc?export=view&id=1bMTKVfEjbxIvyn-4co9FWg2sxCWEp0Bl)

7. I noticed that the line command above `Good_Job__Congratulations` is a `jnz` instruction. This instruction is short for `jump if not zero` which means there has to be some conditional statement going on before that line. I set a break point at line `call	EasyPass.00404628`, to see what will happen at this point when I run the program.

8. Let's run the debugger on the executable.

9. As it hits the breakpoint, the values in the registers changed. One of the strings `hello world!` is the password I entered, and another register contains the string `fortran!`. That seems interesting.

![img](https://drive.google.com/uc?export=view&id=1cZJULbuqYI5bQFh4CqBm_huotwhmm0CF)

10. Click `Step into` to go to the next instruction.

11. As you can see, the following instructions will show you some `mov` instructions but there is one significant instruction taking place: 
```
0040462F  |. 39D0           CMP EAX,EDX
```
![img](https://drive.google.com/uc?export=view&id=1nKVFgfJuogvkuVWLXdwlz37vTDtuYA9j)

12. The register `EAX` contain `hello world!` and register `EDX` contain `fortran!`. This means that it is comparing the password that I entered with `fortran!`. Thus, this could mean that `fortran!` could be the possible password to the program.

13. As I enter `fortran!`, it prompts me `Good Job. Congratulations`
