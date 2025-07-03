# Compiled Python (The Deceptive Version) - Reverse Engineering Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Compiled Python
*   **Category:** Reverse Engineering

---

### The Challenge

This was a masterful challenge in misdirection. Just like a previous challenge, it involved a `main` executable created with PyInstaller. However, this one had multiple layers of deception designed to trick the solver.

### The Solution

The process started the same way: I used `pyinstxtractor.py` to unpack the executable and then ran `strings` on the extracted `main.pyc` file. This revealed the same red-herring password as before: `th1s_1s_n0t_th3_p4ssw0rd_I_sw3arz`.

**Deception Layer 1: The Fake Flag Generator**

When I first ran the program with a random input like "test", it printed what looked like a valid flag: `grodno{9f86d081884c7d659a2feaa0c55ad015}`. I quickly realized the program was simply hashing my input and presenting the first 32 characters as a "flag". This was the first trap.

**Deception Layer 2: The Red Herring Password**

My next step was to try the password I found in the strings: `th1s_1s_n0t_th3_p4ssw0rd_I_sw3arz`. But this was the second trap. When I used this password, the program *still* just gave me the hash of that string as a fake flag. It did not print the "You are right!" message.

**The Real Solution**

The actual solution was incredibly subtle. The real password was the string from the bytecode, but *without the final 'z'*.

Correct password: `th1s_1s_n0t_th3_p4ssw0rd_I_sw3ar`

Only when I entered this exact string did the program print "You are right!" followed by the real flag. This required careful observation and a bit of trial and error, noticing that the embedded string was *almost* right.

### The Flag

```
grodno{88ce08dee4f5c6c9a2188d49fd3e9fdd}
```

This challenge was a great lesson in not trusting what you see at face value and always verifying your results, even when they look correct.

---

*Written by Mohamed Armaoui (C4spr0x1A)*
