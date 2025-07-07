# CTF Writeup: Incantation

**Author:** Mohamed Armaoui - C4spr0x1A

## Challenge Details
| Category       | Cryptography |
|----------------|--------------|
| Challenge Name | Incantation  |
| CTF Event      | CubeCTF 2025 |
| Flag           | `cube{4br4c4d4br4_sh3z4m_pr3st0_98f814ff}` |

## Solution

This write-up details my approach to solving the "Incantation" CTF challenge.

### Initial Analysis

I was given a remote service and a set of files. The `run.sh` script showed that the `incantation` binary is executed with the flag as its first command-line argument.

````bash
#!/bin/sh
timeout 30 /opt/incantation $FLAG
````

Running `file` and `checksec` on the initial binary revealed it's a 64-bit statically linked executable that is packed with UPX.

````bash
$ file incantation
incantation: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), statically linked, no section header

$ checksec incantation
[*] '/path/to/incantation'
    Arch:       amd64-64-little
    RELRO:      No RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Packer:     Packed with UPX
````

### Step 1: Unpacking the Binary

Since the binary is packed with UPX, my first step was to unpack it to make static analysis possible.

````bash
upx -d incantation

                       Ultimate Packer for eXecutables
                         Copyright (C) 1996 - 2024
UPX 4.2.4       Markus Oberhumer, Laszlo Molnar & John Reiser    May 9th 2024

        File size         Ratio      Format      Name
   --------------------   ------   -----------   -----------
     24567 <-      6800   27.68%   linux/amd64   incantation

Unpacked 1 file.
````

After unpacking, I ran `file` and `checksec` again. It was now a dynamically linked, non-stripped binary with all modern protections enabled.

````bash
$ file incantation
incantation: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, ... not stripped

$ checksec incantation
[*] '/path/to/incantation'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    ...
    Stripped:   No
````

The "not stripped" property was very helpful, as it meant function names and symbols were preserved.

### Step 2: Analyzing the Program's Behavior

Connecting to the remote service gave a stream of rapidly changing strings.

````bash
$ nc incantation.chal.cubectf.com 5757
VEII_ONwSauy3XdK}yCDHme6CWK58yY1aCfcyTyn
````

The string on the line kept getting overwritten. This is a common CTF trick where lines are terminated with a carriage return (`\r`) instead of a newline (`\n`). I used `tr` to translate these carriage returns into newlines to see the full output stream.

````bash
timeout 20s nc incantation.chal.cubectf.com 5757 | tr '\r' '\n' > large_output.txt
````

This command captured 20 seconds of output into `large_output.txt`, with each string on its own line. I now had a large dataset to analyze.

### Step 3: Reverse Engineering the Binary

With the unpacked binary, I analyzed its logic.

#### Using `strings`

I first looked for any hardcoded strings.

`strings incantation`

This revealed several crucial pieces of information:

*   `Usage: %s <flag>`: Confirms the program expects the flag as an argument.
*   `time`, `srand`, `rand`: Standard functions for a pseudo-random number generator (PRNG).
*   `_abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ{}`: This is the character set used by the program. It contains 65 characters.

#### Disassembly with `objdump`

Since the binary was not stripped, I could look directly at the `main` function's disassembly.

`objdump -d -M intel incantation`

By analyzing the `<main>` section, I could reverse-engineer the program's logic into C-like pseudocode:

```c
// base_alphabet = "_abcdefghijklmnopqrstuvwxyz0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ{}"
char base_alphabet[66] = "_abcdef..."; // The alphabet from strings
char modified_alphabet[66];

int main(int argc, char **argv) {
    if (argc != 2) {
        // Print usage and exit
    }
    
    char* flag = argv[1];
    int flag_len = strlen(flag);

    // Seed the PRNG with the current time. This is a critical detail.
    // The sequence of random numbers will be the same for any run within the same second.
    srand(time(0)); 
    
    // Main loop, runs forever
    while (1) {
        printf("\r"); // Print carriage return

        // Iterate through each character of the flag
        for (int i = 0; i < flag_len; i++) {
            
            // Create a temporary copy of the base alphabet
            strcpy(modified_alphabet, base_alphabet);
            
            // Replace the first character '_' with the current flag character
            modified_alphabet[0] = flag[i]; 
            
            // Get a random index from 0 to 64
            int rand_idx = rand() % 65; 

            // Print the character from the modified alphabet at the random index
            putchar(modified_alphabet[rand_idx]);
        }
        
        fflush(stdout);
        usleep(10000); // Wait 10ms
    }
}
```

### Step 4: Identifying the Vulnerability

The vulnerability was a statistical bias introduced by the "encryption" algorithm.

For any given position `i` in the output stream (which corresponds to `flag[i]`), let's consider the character `flag[i]`. This character exists at some position in the `base_alphabet`. For example, if `flag[i]` is 'c', it exists at index 3.

When the program generates the `i`-th character of the output string:

*   The `modified_alphabet` is created. `modified_alphabet[0]` is now 'c'.
*   The original 'c' is still at `modified_alphabet[3]`.
*   A random index `rand_idx` from 0-64 is chosen.
*   The output character will be 'c' if `rand_idx` is 0 OR if `rand_idx` is 3. For any other character in the alphabet (e.g., 'a'), it will only be chosen if `rand_idx` is its original index (1).

This meant the correct flag character for any position had roughly double the probability of appearing in the output stream compared to any other character.

### Step 5: The Exploit

The attack was a simple frequency analysis on each column of the captured output. For each position in the flag (0 to 40), I counted the occurrences of every character in that column across all the lines in `large_output.txt`. The most frequent character in that column was the flag character for that position.

I did this with a short Python script:

```python
import sys
from collections import Counter

# The solver script
try:
    with open('large_output.txt', 'r') as f:
        # Read all non-empty lines from the output file
        lines = [line.strip() for line in f if line.strip()]
except FileNotFoundError:
    print('Error: large_output.txt not found.')
    sys.exit(1)

# Ensure I had data
if not lines:
    print('Error: No data in large_output.txt.')
    sys.exit(1)

# Get length of the flag/output strings
num_cols = len(lines[0])
flag = ''

# Iterate through each column
for i in range(num_cols):
    # Create a list of all characters in the current column
    column_chars = [line[i] for line in lines if i < len(line)]
    if not column_chars:
        continue
        
    # Count the frequency of each character
    freq = Counter(column_chars)
    
    # The most common character is my flag character
    most_common_char = freq.most_common(1)[0][0]
    flag += most_common_char

print(f"Flag: {flag}")
```

Executing this logic revealed the flag.

````bash
$ python3 solve.py
Flag: cube{4br4c4d4br4_sh3z4m_pr3st0_98f814ff}
````

