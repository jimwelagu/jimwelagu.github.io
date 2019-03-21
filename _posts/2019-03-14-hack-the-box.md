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

### Steps:
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

# Snake
### Tools that I used:
- Python 2.7
- Atom (Text Editor)

### Steps:
1. The first thing you will notice when you open the `Snake.zip` file, is a python script called `snake`.

2. If you run the script, you will see something like this
![img](https://drive.google.com/uc?export=view&id=1U2AFT4p5G77jw0gdng4Yn6P9QiNpKKYN)

3. The script asks for a username. This can easily be found in the python code.

4. Open the python file in a text editor (e.g. Atom), and look for the following code block:
```python
user_input = raw_input('Enter your username\n')
if user_input == slither:
    pass
else:
    print 'Wrong username try harder'
    exit()
```

5. It is obvious that the username is stored in a variable called slither. So, let's print the value of slither before the code block.
```python
print("Username: {}".format(slither))
```
6. When you run the script, the value of slither is `anaconda`.
![img](https://drive.google.com/uc?export=view&id=1m5HfI4QopLYVeU2O0ruYUdoY1PGbRKT5)

7. Once you type `anaconda` as the username, it will then ask you for a password.

8. We will do the something similar to find the password. Instead, look for the following code block:
```python
pass_input = raw_input('Enter your password\n')
for passes in pass_input:
    for char in chars:
        if passes == str(chr(char)):
            print 'Good Job'
            break
        else:
            print 'Wrong password try harder'
            exit(0)
    break
```

9. It is obvious that the password is stored in a list valled `chars`. So, let's print the values in list `chars`
```python
for char in chars:
    print(str(chr(char))),
print("\n")
```

10. When you run the script, you will find the password printed on the screen.
![img](https://drive.google.com/uc?export=view&id=1r4v1SyC30n_N5L46pRX1n2qBp-eVvr7a)

11. Type the result without spaces as the password, and you should get a message `Good Job`.

12. However, there's a catch. The values printed includes the "key" and a "chain". I'm assuming that this chain is used to strengthen the encrpytion of the password.
```python
for key in keys:
    keys_encrypt = lock ^ key
    chars.append(keys_encrypt)
for chain in chains:
    chains_encrypt = chain + 0xA
    chars.append(chains_encrypt)
```

13. Since you know that the key gets appended to the `chars` list first, everything else following the key will be the chain. We just want to focus on the key.

14. Since the key is 10 characters long based on this line of code, we need the first 10 characters in chars.
```python
keys = [0x70, 0x61, 0x73, 0x73, 0x77, 0x6f, 0x72, 0x64, 0x21, 0x21]
```

15. The first 10 characters in `chars` is `udvvrjwa$$`.

16. Thus, the flag will be `HTB:{anaconda:udvvrjwa$$}`
