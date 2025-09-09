---
layout: post
title: "Imaginary CTF Writeups"
subtitle: "Echoes from the Terminal: My First Log Entry"
date: 2025-04-14 06:11:00 +0530
author: s0c41dgvns 
categories: [CTF writeups]
tags: [
  CTF writeup,
  Forensic,
  Web,
  Hacking
]
---

## 🕵️ ImaginaryCTF Writeups

This post covers some of the challenges I solved during ImaginaryCTF. Each challenge demonstrates a different style of exploitation — from forensic registry artifacts to audio file analysis and subtle web authentication bypasses.

---

# 🔎 Forensics

## Obfuscated-1

**Name:** obfuscated-1
**Category:** Forensics
**Author:** Eth007
**Description:**

> I installed every old software known to man... The flag is the VNC password, wrapped in ictf{}.
> 

**Attachment:** [Users.zip](https://www.notion.so/assets/files/obfuscated-1/Users.zip)

---

### 💡 Thought Process

The mention of a *VNC password* instantly made me think of the Windows Registry, because TightVNC stores its server settings (including passwords) under the `NTUSER.DAT` hive.
My intuition was:

1. Load the registry hive.
2. Search for TightVNC keys.
3. Locate and decrypt the stored password.

This reasoning comes from past forensic challenges — legacy remote desktop tools often use weak, static encryption (like DES), making them easy targets once you know where to look.

---

### 🛠️ Steps to Solve

**Step 1: Load the Registry**

- Extracted `Users.zip`.
- Located the `NTUSER.DAT` file.
- Opened it using **Registry Explorer**.

**Step 2: Find TightVNC Settings**

- Path: `Root > Software > TightVNC > Server`
- Found a value named **Password**.

**Step 3: Decrypt the Password**

- Password was stored in DES-encrypted bytes.
- Used a custom Python script to decrypt:

```python
from Crypto.Cipher import DES

# TightVNC fixed DES key
KEY = bytes([0x17, 0x52, 0x6b, 0x20, 0x4b, 0x72, 0x41, 0x4b])

def decrypt_vnc(enc_hex: str) -> str:
    enc = bytes.fromhex(enc_hex)
    cipher = DES.new(KEY, DES.MODE_ECB)
    dec = cipher.decrypt(enc)
    return dec.decode(errors="ignore").rstrip("\\x00")

if __name__ == "__main__":
    encrypted = "CA8D9F..."  # replace with actual registry value
    print("Decrypted password:", decrypt_vnc(encrypted))
```

- Running it produced the password: `Slay4U!!`
- Wrapped in `ictf{}` → `ictf{Slay4U!!}`

---

### 🪞 Reflection

This was a classic registry-artifact challenge: knowing where software hides credentials can be more valuable than brute-force guessing. Forensics often rewards prior knowledge of *where data likes to live*.

---

## Wave

**Name:** wave

**Category:** Forensics

**Author:** Eth007

**Description:**

> not a steg challenge i promise
> 

**Attachment:** [wave.wav](https://chatgpt.com/assets/files/wave/wave.wav)

**Flag:** `ictf{obligatory_metadata_challenge}`

---

### 💡 Thought Process

When handed an audio file, my first suspicion was **steganography**. Common hiding techniques include:

- **LSBs of audio samples**
- **Spectrogram tricks** (drawings in frequency space)
- **Metadata tags** (artist/comment fields)
- **Appended raw data**

So I followed the typical forensic workflow:

1. **Check file type:**
    
    ```bash
    file wave.wav
    
    ```
    
    → Valid RIFF/WAV file.
    
2. **Strings search:**
    
    ```bash
    strings wave.wav | less
    
    ```
    
    → No obvious hits.
    
3. **Spectrogram analysis (Audacity / Sonic Visualizer):**
    - No unusual shapes or hidden messages.
4. **Check metadata:**
    - No suspicious tags.

At this point, I suspected hidden raw data.

---

### 🛠️ Solution

Dumped the hex:

```bash
xxd wave.wav | less

```

Scrolling revealed the flag plainly embedded in the data section:

```
... ictf{obligatory_metadata_challenge} ...

```

So the challenge wasn’t deep stego — just a subtle trick hidden in plain hex.

---

### ✅ Flag

```
ictf{obligatory_metadata_challenge}

```

---

# 🌐 Web

## Passwordless

**Name:** passwordless

**Category:** Web

**Author:** Ciaran

**Description:**

> Didn't have time to implement the email sending feature but that's ok, the site is 100% secure if nobody knows their password to sign in!
> 

**Challenge URL:** [http://passwordless.chal.imaginaryctf.org](http://passwordless.chal.imaginaryctf.org/)

**Attachment:** [passwordless.zip](https://chatgpt.com/assets/files/passwordless/passwordless.zip)

**Flag:** `ictf{8ee2ebc4085927c0dc85f07303354a05}`

---

### 💡 Intuition

The description screamed **auth bypass**. If users never get their password, how can they log in? That implied a logic flaw in password handling.

Looking at the source:

- Registration creates `password = email + randomHex`.
- This is hashed with **bcrypt**.
- Problem: bcrypt truncates inputs longer than **72 bytes**.
- Additionally, **normalize-email** strips dots, `+tags`, and lowercases Gmail addresses.

That gave me the idea: if the email is long enough, the random hex part would be ignored due to truncation, leaving a deterministic password based only on the email prefix.

---

### 🛠️ Steps to Solve

**Step 1: Read the code**

- Normalize email → shorter version is stored.
- Password = `email + randomHex`.
- If `len(password) > 72`, bcrypt truncates at 72.

**Step 2: Craft email**

Register with something like:

```
test+aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@gmail.com

```

- Normalizes to `test@gmail.com`.
- Password becomes long enough that random hex is truncated.

**Step 3: Login**

- Email: `test@gmail.com`
- Password: `test+aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa@gmail.com` (truncated prefix part).

This bypass worked and logged me in as the user, revealing the flag.

---

### ✅ Flag

```
ictf{8ee2ebc4085927c0dc85f07303354a05}

```

---

# 📌 Closing Thoughts

Each challenge here tested a different mindset:

- **Obfuscated-1:** Knowing where forensic artifacts hide.
- **Wave:** Systematically eliminating stego techniques until only raw hex remained.
- **Passwordless:** Recognizing a subtle bug in bcrypt + email normalization.

Together, they highlight a key lesson: in CTFs, **intuition + systematic checks** often lead straight to the flag.