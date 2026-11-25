+++
date = '2026-11-17T22:03:21+01:00'
draft = false
title = 'Classic Passwd THM Writeup'
summary = 'Writeup of Classic Passwd from TryHackMe'
tags = ["THM", "Blue-Team", "Reversing", "Hardcoded-Credentials", "Writeup"]
+++

![Classic Passwd logo](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Classic_Passwd_logo.jpeg)

# Introduction

- **Platform:** TryHackMe
- **Room:** Classic Passwd
- **Difficulty:** Medium
- **Category:** Blue Team (Reversing)

## Challenge description

I forgot my password, can you give me access to the program?

## Tools

- `file`: to find out what type of file it is
- `strings`: to extract the text strings the binary contains
- `objdump`: to disassemble the executable and read its code

## Initial analysis

The first thing I'm going to do is download the file the machine gives us, by clicking the blue button in the description:

![Button to download the file](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Boton_Descarga_Classic_Passwd-THM.png)

And we can see the file has been downloaded:

![File already downloaded](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Archivo_Descargado-Classic-Psswd-THM.png)

Now that I have it, I'm going to check what type of file it is:

```bash
file Challenge_1609966715991.Challenge
```

![file output](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Tipo_Archivo-Classic-Passwd-THM.png)

It's a 64-bit ELF executable, so I give it execution permissions and run it:

```bash
chmod +x Challenge_1609966715991.Challenge
./Challenge_1609966715991.Challenge
```

![Setting permissions and running it for the first time](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Permisos_ejecucion_+_Ejecutado-Classic-Psswd-THM.png)

Good, the program starts and asks for a username. I try the usual ones (`admin`, `root`, `guest`...) but all of them return `Authentication Error`, so we're not getting anywhere this way.

![Username attempts](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Intentos_username-Classic-Passwd-THM.png)

## String extraction

Since guessing the username doesn't work, I'm going to see what the program contains. For that I use `strings`, which extracts all the readable text from the binary:

```bash
strings Challenge_1609966715991.Challenge
```

![strings output](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/string-Classic-Passwd-THM.png)

Among everything that shows up, a few things catch my attention:

- `Made by 4non`
- A GitHub URL
- `AGB6js5d`
- `9dkGf`
- `THM{%d%d}`

The most interesting one is the last: `THM{%d%d}` looks like the flag format, and the two `%d` tell us it's built from two numbers. Also, the flag isn't written as is, the program generates it. Good to know.

## Disassembly

With `strings` I can't see the full username anywhere (`AGB6js5d` and `9dkGf` show up separately, and that final `f` is a byte of the next instruction that `strings` mistakes for text), so it's time to look at the code. For that I use `objdump`:

```bash
objdump -d -M intel Challenge_1609966715991.Challenge
```

![Disassembly output](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/objdump-Classic-Psswd-THM.png)

In short, this is what it does:

- `-d`: disassembles the binary and shows me the instructions
- `-M intel`: shows them in Intel syntax, which is easier to read

In the output I look for the `vuln` function, and there I see several `mov` instructions that store hexadecimal values in memory. Those values are ASCII characters, but they're stored in little endian, meaning each block has to be read backwards.

For example, the value `0x6e6f6e34` is stored as the bytes `34 6e 6f 6e`, which in ASCII is `4non`. You can check it from the terminal:

```bash
echo 34 6e 6f 6e | xxd -r -p
```

![Translation and reversal of the values](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/traducción_valores_invertidos-Classic-Passwd-THM.png)

Converting and joining them all gives me the username, which is exactly what the program later compares with `strcmp()` to check whether what we typed is correct:

Username: `AGB6js5d9dkG7`

So now that we have the username, let's run the program again and enter it:

![Logging in with the username](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Logramos_entrar_+_Flag-Classic-Passwd-THM.png)

Perfect, when we log in the `gfl` code block runs and prints the flag to the terminal.

## Vulnerability

The vulnerability in this program is **hardcoded credentials** (CWE-798): the correct username is written inside the executable itself. The fact that it's split across several `mov` instructions instead of appearing as a whole string isn't real protection, just minimal obfuscation: anyone with `objdump` can rebuild it, as we just did.

## Second way to solve it

So far we've obtained the flag by running the program with the correct username. But it can also be deduced without running it, by reading the code of the `gfl` function directly. Both ways lead to the same flag.

### Analysis of the gfl function

Once we enter the right username, the program runs another function called `gfl`.

![gfl function](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/función_gfl-Classic-Passwd-THM.png)

There are two loops with comparison conditions, and by looking at the `cmp`, `add` and conditional jump instructions I can see which values lead to the `printf` call.

The program doesn't calculate anything: each loop adds 1 to its counter and, when the first reaches `0x638a78` and the second reaches `0x2130`, it runs `printf` with those two values. That's why the flag is simply those two numbers.

- 0x638a78 in hexadecimal = 6,523,512 in decimal
- 0x2130 in hexadecimal = 8,496 in decimal

![Identifying the loop values](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/identificacion_bucle-Classic-Passwd-THM.png)

And as we saw earlier with `strings`, the format is `THM{%d%d}`, so the flag is made of two integers in decimal:

- First number: *6523512*
- Second number: *8496*

And with this we can build the flag:

**THM{value1+value2}**

As we can see, the result matches the flag the program returned when we logged in.

## Getting the flag

With those two numbers I have the full flag and I submit it in the room:

![Censored flag](/images/writeups/TryHackMe_imágenes/Classic_Passwd_imágenes/Flag_en-THM.png)

## Conclusion

This challenge shows why credentials shouldn't be stored inside the binary: `strings` and `objdump` are enough to recover them. We also saw that the flag can be obtained in two ways: running the program with the correct username or reading the `gfl` code.

## Thanks for reading