# Compiled Python - Reverse Engineering Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Compiled Python
*   **Category:** Reverse Engineering

---

### The Challenge

This challenge provided a single Linux executable named `main`. When run, it prompted for a password. The challenge title, "Compiled Python," strongly suggested that this was a Python script that had been packaged into a standalone executable, most likely with a tool like PyInstaller.

### The Solution

My first step was to confirm that it was a PyInstaller file. Running `strings` on the binary and grepping for "python" showed many PyInstaller-related strings, confirming my suspicion.

The next step was to extract the original Python script from the executable. I used `pyinstxtractor.py`, a well-known tool for this purpose.

```bash
python3 pyinstxtractor.py main
```

This command unpacked the executable and created a `main_extracted` directory. Inside, among many other files, was `main.pyc`—the compiled Python code for the main script.

Since it was a small file, I didn't need a full decompiler. I just ran `strings` on `main.pyc` to see what text constants were embedded in the code.

```bash
strings main_extracted/main.pyc
```

The output contained several gems:
*   `sha256`
*   `Enter password:`
*   `grodno{`
*   `You are right!`
*   And most importantly: `th1s_1s_n0t_th3_p4ssw0rd_I_sw3arz`

This last string was the key. In the world of CTFs, when a string tells you it's "not the password," it's almost certainly the password.

I ran the original `main` executable and entered that string.

### The Flag

The program accepted the password and printed the flag:

```
grodno{ce305c285daab77d7ea0ca2bdc86583c}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*