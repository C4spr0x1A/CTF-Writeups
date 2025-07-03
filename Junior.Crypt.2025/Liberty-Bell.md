# Liberty Bell - Cryptography Challenge

*   **CTF:** Junior.Crypt.2025
*   **Challenge:** Liberty Bell
*   **Category:** Cryptography

---

### The Challenge

This was a fun challenge disguised as a game. I had to play a "Liberty Bell" slot machine and turn an initial $100 into $1000. Since this was a cryptography CTF, I knew the slot machine's "random" number generator had to be flawed.

### The Solution

My first step was to play the game with small, one-nickel bets and record the results of each spin. After a few dozen spins, I noticed that the sequences of results started repeating. This was the classic sign of a weak pseudo-random number generator (PRNG) with a short cycle.

The slot machine's outcomes were not random at all; they were part of a deterministic, repeating loop.

My strategy then became:
1.  **Map the cycle:** Continue playing with small bets to map out a significant portion of the PRNG's cycle.
2.  **Identify high-payout rounds:** Find the positions in the cycle that result in big wins (like three Liberty Bells).
3.  **Bet strategically:** Wait for the cycle to approach a known high-payout round and then bet the maximum amount.

I wrote a script to automate this. It would connect to the server, play with minimum bets to gather data, find the cycle, and then switch to a high-betting strategy once it was confident in its predictions.

Here is the conceptual logic of the script:

```python
# (Conceptual script)
import socket

def play_liberty_bell():
    # Connect to the server
    # ...

    results_history = []
    bank_balance = 2000  # Start with 2000 nickels

    # --- Phase 1: Data Collection ---
    print("[*] Collecting data with minimum bets...")
    for _ in range(100): # Collect 100 results
        # Send a bet of 1 nickel
        # Receive and parse the result [reel1, reel2, reel3]
        # Add the result to results_history
        # Update bank_balance
        pass

    # --- Phase 2: Cycle Detection ---
    print("[*] Analyzing data for cycles...")
    # Analyze results_history to find the repeating cycle length
    # and the positions of high-payout combinations.
    cycle_length = find_cycle(results_history)
    jackpot_rounds = find_jackpots(results_history, cycle_length)
    
    # --- Phase 3: Strategic Betting ---
    print("[*] Entering strategic betting mode...")
    current_round = len(results_history)
    while bank_balance < 20000: # Target is 20,000 nickels ($1000)
        # Is the next round a known jackpot round?
        if (current_round + 1) % cycle_length in jackpot_rounds:
            bet_amount = min(bank_balance, 500) # Bet big
        else:
            bet_amount = 1 # Bet small
        
        # Send the bet
        # Receive result and update bank_balance
        # current_round += 1

    # Get the flag
    # ...
```

### The Flag

By waiting for the right moments and betting big, it was easy to reach the $1000 goal. The server then provided the flag:

```
grodno{ed5aa020435743294541248235734654bf8ed7}
```

---

*Written by Mohamed Armaoui (C4spr0x1A)*