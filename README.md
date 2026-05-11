# Decrypting an AES-Encrypted ZIP Using PkCrack (Known-Plaintext Attack)

## Overview

This walkthrough covers how to decrypt a password-protected ZIP file using a known-plaintext attack with PkCrack. The scenario involves an AES-encrypted JPG file, a secret key, and an encrypted ZIP containing exfiltrated data.

**!!Other walkthroughs will have you use "cmake .." on this lab, and it does not work!!**

**Tools Used:**
- AESCrypt (console version)
- 7-Zip
- PkCrack
- FileZilla (SFTP)

---

## Environment

- **Workstation:** Windows 10 machine (172.16.30.101)
- **Attack Machine:** Kali Linux (172.16.30.6)

---

## Step 1: Decrypt the AES-Encrypted JPG

The flash drive contained the following files:
- `astral.jpg.aes` — AES encrypted JPG
- `exfiltratroll.zip` — password-protected ZIP
- `secretkey` — plaintext key file
- `tools/` — folder containing AESCrypt binaries

Open PowerShell on the Windows machine and run AESCrypt to decrypt the JPG using the key found in the `secretkey` file:

```powershell
E:\Ione\tools\AESCrypt\AESCrypt_console_v309_64\aescrypt.exe -d -p 28334F91DBD9A77D167C9D6C1457030CB0361E8EAAABF73190EC19CE328F77A9 E:\Ione\astral.jpg.aes
```

This produces `astral.jpg` in `E:\Ione\`.

> **Note:** The executable is named `aescrypt.exe` (all lowercase), located in the `AESCrypt_console_v309_64` subfolder. Use the 64-bit version on modern systems.

---

## Step 2: Transfer Files to Kali

PkCrack needs to run on Kali. Use FileZilla to transfer the files from the Windows machine to Kali:

- Open FileZilla on the **Windows machine**
- Connect to Kali:
  - **Host:** `172.16.30.6`
  - **Username:** `playerone`
  - **Password:** (your password)
  - **Port:** `22`
- Navigate to `E:\Ione\` on the left (local/Windows side)
- Transfer `astral.jpg` and `exfiltratroll.zip` to your desired folder on Kali (e.g. `~/Desktop/Ione/`)

---

## Step 3: Create a ZIP of astral.jpg Using 7-Zip on Kali

PkCrack requires a ZIP file containing the plaintext version of one of the encrypted files. The ZIP **must** be created with 7-Zip — using Windows' built-in ZIP utility will not work.

On Kali, navigate to the folder and create the ZIP:

```bash
cd ~/Desktop/Ione
7z a astral.zip astral.jpg
```

This creates `astral.zip` containing `astral.jpg`.

---

## Step 4: Install PkCrack on Kali

Clone the PkCrack repository to your home directory:

```bash
cd ~
git clone https://github.com/keyunluo/pkcrack
```

Once cloned, check the `bin` folder — the binaries are already compiled and ready:

```bash
ls /home/playerone/pkcrack/bin
```

You should see `pkcrack`, `pkcrack.exe`, `extract`, `findkey`, `makekey`, and `zipdecrypt`.

The binary will likely not have execute permissions, so fix that with:

```bash
chmod +x /home/playerone/pkcrack/bin/pkcrack
```

> **Note:** This `chmod +x` step is not covered in most walkthroughs but is required. Without it you will get a `permission denied` error when trying to run PkCrack.

---

## Step 5: Run PkCrack

With all files in place, run the known-plaintext attack:

```bash
/home/playerone/pkcrack/bin/pkcrack -C ~/Desktop/Ione/exfiltratroll.zip -c astral.jpg -P ~/Desktop/Ione/astral.zip -p astral.jpg -d ~/Desktop/Ione/cracked.zip -a
```

**Flag breakdown:**
| Flag | Description |
|------|-------------|
| `-C` | The encrypted ZIP you want to crack |
| `-c` | The filename inside the encrypted ZIP to use as the known plaintext |
| `-P` | Your ZIP containing the plaintext file |
| `-p` | The filename inside your plaintext ZIP |
| `-d` | Output path for the decrypted ZIP |
| `-a` | Search all files in the encrypted ZIP for a match |

If successful, you will see output like:

```
Decrypting 2EWX83.txt ... OK!
Decrypting astral.jpg ... OK!
Decrypting fog.jpg ... OK!
Decrypting prince.jpg ... OK!
Finished on Mon May 11 17:33:17 2026
```

---

## Step 6: Extract the Decrypted ZIP

```bash
cd ~/Desktop/Ione
unzip cracked.zip
```

The decrypted files are now accessible.

---

## Key Takeaways

- AESCrypt on Windows decrypts `.aes` files when you have the key
- PkCrack performs a **known-plaintext attack** — it needs one file from inside the encrypted ZIP in plaintext form
- The plaintext ZIP **must** be created with 7-Zip to match the compression method used by the attacker
- If PkCrack returns `permission denied`, run `chmod +x` on the binary before executing
- FileZilla is a reliable way to transfer files between Windows and Kali when SCP isn't cooperating
