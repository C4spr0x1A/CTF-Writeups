# Recovery Password - Forensics Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Recovery password
*   **Category:** Forensics

---

### The Challenge

In this challenge, I was given a KeePass password database (`Database.kdbx`) and a memory dump (`KeePass.DMP`). The goal was to find the master password for the database within the memory dump, a classic memory forensics task.

### The Solution

My first step was to use a specialized tool, `keepass-dump-masterkey`, which is designed to find KeePass passwords in memory. KeePass has some memory protections that try to obscure the password, so a generic tool might not work.

Running this tool on the `.DMP` file gave me a long list of potential password fragments. It successfully recovered most of the password but, due to the memory protection, the first couple of characters were garbled. The tool gave me many variations like:

*   `St_f0r_z3r0-d4y_HunT1Ng`
*   `2t_f0r_z3r0-d4y_HunT1Ng`

Knowing the general pattern of the password, I then turned to a simpler tool: `strings`. I ran `strings` on the memory dump and piped it to `grep` to search for a unique part of the password string I had already recovered.

```bash
strings KeePass.DMP | grep "d4y_HunT1Ng"
```

This was the breakthrough. The `strings` command found a complete, unobscured version of the password in the dump:

`z3St_f0r_z3r0-d4y_HunT1Ng`

With the full password, I could now open the KeePass database using `kpcli` (a command-line KeePass client).

```bash
echo "z3St_f0r_z3r0-d4y_HunT1Ng" | kpcli --kdb=Database.kdbx --readonly
```

Inside the database, there was an entry named "Junior Crypto 2025" which contained the flag.

### The Flag

```
grodno{T001_uP_0r_Dr00L_D0wn}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*

