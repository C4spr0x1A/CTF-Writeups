# CTF Writeup: The Warrior's Legacy #1

**Author:** Mohamed Armaoui - C4spr0x1A

## Challenge Details
| Category       | Forensics |
|----------------|-----------|
| Challenge Name | The Warrior's Legacy #1 |
| CTF Event      | Blitz CTF 2025 |
| Flag           | `Blitz{NBT_Is_Not_Just_For_Blocks}` |

## Solution

This writeup details my approach to solving the "The Warrior's Legacy #1" CTF challenge.

### Challenge Description

A legendary warrior's equipment set has been recovered from an ancient dungeon. The complete set includes enchanted weapons, armor, and various tools that served the warrior well in countless battles.

Local legends speak of a hidden message that the warrior split across their most prized possessions before disappearing into the depths. The equipment appears to be standard enchanted gear, but those who know where to look claim there's more than meets the eye.

My task was to examine the warrior's complete equipment set and piece together the secret they left behind.

### Solution Steps

1.  Environment Setup and File Download:
    *   I created a new directory `The_Warriors_Legacy_1` for the challenge.
    *   I downloaded the `warriors_legacy.zip` file using `wget`.
    *   I unzipped the file, which revealed a Minecraft world save in the `blitzctf` directory.

2.  Initial Investigation of Minecraft World Data:
    *   I listed the contents of the `blitzctf` directory to understand the file structure. It contained `level.dat`, `playerdata`, `region`, and other typical Minecraft world files.
    *   I focused on the `playerdata` directory, as it likely holds player inventories and related data. I found a file named `1bd0e62f-584f-4c8b-b5a1-d097ed12d059.dat`.

3.  NBT Data Parsing Attempts and Tool Selection:
    *   Initially, I attempted to use `nbt-dump` to inspect the `.dat` file, but it was not installed and subsequent `pip install` attempts failed due to Kali Linux's Python environment management.
    *   I searched for a reliable command-line NBT parser. I identified `nbt2json` (a Go-based tool) as a suitable option.
    *   I installed `nbt2json` using `go install github.com/midnightfreddie/nbt2json/cmd/nbt2json@latest`.

4.  Extracting and Analyzing Player Data:
    *   I converted the player data file to JSON format using `nbt2json`:
        ````bash
        $HOME/go/bin/nbt2json --java -i blitzctf/playerdata/1bd0e62f-584f-4c8b-b5a1-d097ed12d059.dat -o playerdata.json
        ````
    *   I read the `playerdata.json` file. It was quite large, so I focused on searching for keywords.

5.  Locating Flag Segments:
    *   I performed a `grep` search for common flag indicators like `CTF`, `flag`, `blitz`, `display`, and `Enchantments`.
    *   The search revealed a `custom_data` field within an item's NBT structure that contained a `flag_part` named `QmxpdHp7TkJUXw==`.
    *   I decoded the first part:
        ````bash
        echo "QmxpdHp7TkJUXw==" | base64 -d
        # Output: Blitz{NBT_
        ````
    *   I continued searching for other `custom_data` fields within the inventory.
    *   I found a second `custom_data` field in an `enchanted_book` with `secret_code: Rm9yX0Jsb2Nrc30=`.
    *   I decoded the second part:
        ````bash
        echo "Rm9yX0Jsb2Nrc30=" | base64 -d
        # Output: For_Blocks}
        ````
    *   I found a third `custom_data` field in a `diamond_chestplate` with `hidden_data: SXNfTm90X0p1c3Rf`.
    *   I decoded the third part:
        ````bash
        echo "SXNfTm90X0p1c3Rf" | base64 -d
        # Output: Is_Not_Just_
        ````

6.  Assembling the Flag:
    Combining the decoded parts in order: `Blitz{NBT_` + `Is_Not_Just_` + `For_Blocks}`.

## Flag

`Blitz{NBT_Is_Not_Just_For_Blocks}`
