# Conduct Unbecoming

| | |
|---|---|
| **Event** | DEADFACE CTF 2026 |
| **Category** | Steganography |
| **Points** | 75 |
| **Author** | @G2Gh0st |
| **Solved by** | Cookie Vampire (NightHAX) |
| **Flag** | `deadface{NVU-PRINT-17}` |

## Challenge

> DEADFACE recently breached Night Veil University and exfiltrated a number of internal documents. We were able to recover one of those documents for analysis. We need your help to identify the asset-id that was used to create the document so we can have our investigators analyze it for more clues on how DEADFACE gained access.
>
> Submit the flag as `deadface{flag-text}`.

**File:** `night_veil.jpg` (SHA-256 `6e7809f4a7893b0a8348195d0009181376ef635d465a2ee4279e78115e44f1ba`)

![Conduct Unbecoming challenge card](images/04-card.png)

## Initial thoughts

Before running any tools, I looked at what the challenge was really asking. It wants the **asset-id used to create the document**, which means the device, not the person. The challenge card art is a **printer**, so my working theory was that the answer identifies a specific printer.

That gave me two ideas to test:

1. **Metadata.** Scanners and printers sometimes write device names or serial numbers into a file's metadata. This is the quickest thing to check, so it went first.
2. **Printer tracking dots.** Many color laser printers secretly print a faint pattern of tiny yellow dots on every page, encoding the printer's serial number and a timestamp. This is a real forensic technique (sometimes called a Machine Identification Code) that has been used to trace leaked documents. If the metadata came up empty, this was my main lead.

## Solution walkthrough

### 1. Verify the file

```bash
sha256sum night_veil.jpg
```

**Why:** the challenge provided a hash, and checking it first confirms the download is complete and untampered. Otherwise, any odd results later could be blamed on a bad file. It matched.

### 2. Check the metadata

```bash
exiftool night_veil.jpg
```

**Why:** metadata is the fastest place a device ID could hide, and it was idea #1.

No asset ID or serial number appeared, but three details shaped what I did next:

- **Title:** "NVU Conduct Notice" (NVU = Night Veil University), created in Affinity Photo 2.
- **Size:** 2550 × 3300 in the EXIF data, which is exactly 8.5 × 11 inches at 300 DPI. This image was made to look like a printed letter-size page, which supported the printer theory.
- **YCbCr 4:4:4 chroma subsampling.** Most JPEGs use 4:2:0, which stores color at a lower resolution than brightness and smears tiny color details. Saving at 4:4:4 keeps full color detail. That's an unusual choice, and it's exactly what you'd need to preserve tiny yellow dots through JPEG compression. This made me more confident in idea #2.

### 3. Rule out hidden files

```bash
strings -n 6 night_veil.jpg | grep -i -E "asset|serial|deadface"
binwalk night_veil.jpg
steghide extract -sf night_veil.jpg    # pressed Enter for an empty passphrase
```

**Why:** since this is a steganography challenge, I wanted to rule out the common hiding spots before committing to the tracking-dot theory. `strings` checks for readable text, `binwalk` looks for files embedded inside, and `steghide` checks for data hidden with a password (an empty one is common in easier challenges).

All three came up empty: no matching text, nothing embedded beyond the normal JPEG structure, and steghide couldn't extract anything.

### 4. Look at the document itself

**Why:** if the answer is printed on the page as tracking dots, no tool will report it. I needed to actually look.

The image is a one-page **Student Conduct Notice** from Night Veil University's Office of Student Conduct, about a student disciplined for flying a drone in the library and putting about 2,300 googly eyes on university property. The document number at the bottom is NVU-OSC-2026-1187, but that identifies the *document*, not the device that created it.

![The recovered conduct notice](images/04-document.jpg)

The important detail was in the white margin at the **bottom-right corner, near "Page 1 of 1": a faint grid of tiny dots.** That lined up with the tracking-dot theory.

### 5. Try Aperi'Solve on the blue channel (dead end)

**Why the blue channel:** yellow ink reflects red and green light but absorbs blue. So in the blue channel, yellow dots that are nearly invisible on white paper turn **dark** against a bright background. I extracted it in Kali:

```bash
convert night_veil.jpg -channel B -separate blue.png
```

**Why Aperi'Solve:** it runs a whole batch of tools at once (exiftool, binwalk, zsteg, pngcheck, and more) and shows every color channel and bit plane as separate images. It was a fast way to check for anything else hidden in the blue channel while also getting a better view of the dots.

It didn't turn up anything useful:

- **exiftool / pngcheck:** the same metadata as the original, carried over during conversion.
- **binwalk / foremost:** only the normal PNG image data.
- **zsteg:** only metadata chunks and a few random "file type" matches (like "MPEG ADTS audio"), which are typical false positives on noisy image data.

Looking back, this makes sense. The dots aren't hidden in the file's bits; they're **printed visibly on the page**, just very faint. Bit-level tools are built for the wrong kind of hiding. Also, `blue.png` was a converted copy, so any hidden-bit results from it wouldn't reflect the original JPEG anyway.

### 6. Move back to Kali and zoom in on the dots

**Why:** the dots were in one small corner of a large 2566 × 3314 image, so they were too small to read at full-page view. Instead of analyzing the whole file, I focused on just that region and made it as readable as possible:

```bash
convert night_veil.jpg -crop 300x300+2200+2950 -channel B -separate -normalize -scale 400% dots.png
```

What each part does, and why:

- `-crop 300x300+2200+2950`: cuts out a 300 × 300 pixel square starting at x=2200, y=2950, which is the bottom-right corner near "Page 1 of 1." Working from the original JPEG, not the converted copy, avoids losing detail.
- `-channel B -separate`: keeps only the blue channel, so the yellow dots turn dark.
- `-normalize`: stretches the contrast so the faint dots become black and the paper becomes white.
- `-scale 400%`: enlarges the crop 4× so individual dots are easy to see.

This revealed a clean, regular grid of dots, **8 columns wide and 13 rows tall**, right next to "Page 1 of 1."

![Enhanced tracking dot grid next to "Page 1 of 1"](images/04-dots.png)

### 7. Decode the grid

**Why read it as binary:** a regular grid of dots and blanks is a natural way to store bits (dot = 1, blank = 0). With exactly 8 columns, each row could be one byte, the standard size of an ASCII character.

Counting 96 dots by eye is error-prone, so each grid position was sampled with a short Python script (written and run with Claude's help on my zoomed screenshot) that checked whether each spot was dark:

```
########   marker row (all dots)
01001110   N
01010110   V
01010101   U
00101101   -
01010000   P
01010010   R
01001001   I
01001110   N
01010100   T
00101101   -
00110001   1
00110111   7
```

Two things confirmed the binary theory:

- The **top row is all dots**, which works as an alignment marker showing where the data starts. Real printer tracking patterns use similar marker rows.
- The **first bit of every data row is 0**. Standard ASCII characters only use values 0 to 127, so their first bit is always 0. That's exactly what you'd expect if each row is one character.

Reading each row as an ASCII character, top to bottom, spells **NVU-PRINT-17**, which looks exactly like a university printer asset tag. One dot (in the "I" row) was faint in the zoomed image. Reading it as blank gave an uppercase `I`, which fit the all-caps pattern of the rest of the tag.

## Flag

```
deadface{NVU-PRINT-17}
```

Accepted on the first try.

---


