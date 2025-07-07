# CTF Writeup: Blitz Traffic

**Author:** Mohamed Armaoui - C4spr0x1A

## Challenge Details
| Category       | Forensics |
|----------------|-----------|
| Challenge Name | Blitz Traffic |
| CTF Event      | Blitz CTF 2025 |
| Flag           | `Blitz{H3r3_1S_th3_flAG_G00d_B03}` |

## Solution

This writeup details my approach to solving the "Blitz Traffic" CTF challenge.

### Tools Used

*   `unzip`
*   `tshark`
*   `hexdump`
*   `file`
*   `xxd`
*   `mv`

### Solution Steps

#### 1. Initial Setup and File Extraction

First, I created a directory named `Blitz Traffic` to organize the challenge files. I downloaded the `protected.zip` file and then moved it into this directory.

The `protected.zip` file was extracted using the provided password:

`unzip -P iloveblitzhack protected.zip`

This extracted `blitzhack_traffic.pcap`, a packet capture file.

#### 2. Initial PCAP Analysis

Initial inspection of the `blitzhack_traffic.pcap` file using `tshark` revealed several protocols, with a large number of TCP packets and a few HTTP packets, including one noted as `_ws.malformed`.

`tshark -r blitzhack_traffic.pcap -q -z "io,phs"`

This showed:

*   ip: 4539 frames
*   tcp: 4519 frames
*   http: 6 frames (including 1 malformed `_ws.malformed` frame)
*   icmp: 20 frames

I also attempted to export HTTP objects, but the resulting files were unreadable binary data.

#### 3. Investigating TCP Conversations

Given the large number of TCP packets and the challenge title "Blitz Traffic," my focus shifted to analyzing TCP conversations for hidden data. Listing TCP conversations showed a pattern of many 540-byte packets (500 bytes of payload + TCP/IP headers) between `192.168.64.5` and `192.168.1.100`.

`tshark -r blitzhack_traffic.pcap -q -z "conv,tcp"`

The output showed numerous entries like:

`192.168.64.5:xxxxx <-> 192.168.1.100:http 0 0 bytes 1 540 bytes`

This indicated many TCP SYN packets with a payload length of 500 bytes. This uniform and unusual size for connection establishment packets was highly suspicious, suggesting data exfiltration or hidden information.

#### 4. Extracting Raw TCP Payloads

The primary hypothesis became that the flag or a file containing the flag was hidden within these repetitive 500-byte TCP payloads. After some trial and error with `tshark` field extractions, the correct command to extract only the raw TCP payload data for packets with a `tcp.len` of 500 was found:

`tshark -r blitzhack_traffic.pcap -Y "tcp.len == 500" -T fields -e tcp.payload | xxd -r -p > extracted_500_byte_payloads.bin`

This command does the following:

*   `tshark -r blitzhack_traffic.pcap`: Reads the pcap file.
*   `-Y "tcp.len == 500"`: Filters for TCP packets where the payload length is exactly 500 bytes.
*   `-T fields -e tcp.payload`: Extracts the raw TCP payload data for the filtered packets.
*   `xxd -r -p`: Converts the hexadecimal representation of the payload (output by tshark) back into raw binary data.
*   `> extracted_500_byte_payloads.bin`: Redirects the raw binary output to a new file.

#### 5. Identifying and Viewing the Extracted File

After extracting the data, the `file` command was used to determine the type of the `extracted_500_byte_payloads.bin` file:

`file extracted_500_byte_payloads.bin`

The output was:

`extracted_500_byte_payloads.bin: PNG image data, 1587 x 2245, 8-bit/color RGBA, non-interlaced`

This confirmed that the extracted data was a PNG image. The file was then renamed to `flag.png`:

`mv extracted_500_byte_payloads.bin flag.png`

Viewing `flag.png` revealed the flag.

## Flag

The flag found in the `flag.png` image is:

`Blitz{H3r3_1S_th3_flAG_G00d_B03}`
