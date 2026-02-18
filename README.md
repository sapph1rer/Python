[Uploading README.md…]()
# aes256-cli

CLI tool for encrypting/decrypting **text** and **files** using:

- **Password mode:** AES-256-GCM + **scrypt** (default)
- **PEM mode (Hybrid RSA+AES):** RSA-OAEP wraps an AES session key + AES-256-GCM for data

It supports both **Base64 token** output (easy to paste/share) and **raw binary payload** output (`--raw`, faster + smaller).

---

## ภาษาไทย

### คุณสมบัติ
- เข้ารหัส/ถอดรหัส **ข้อความ (text)** → ได้ token แบบ Base64 URL-safe
- เข้ารหัส/ถอดรหัส **ไฟล์ (file)**
- โหมดรหัสผ่าน (แนะนำสำหรับใช้กันเองแบบแชร์รหัส)
- โหมด Public/Private key ผ่านไฟล์ `.pem` (ไม่ต้องแชร์รหัสผ่าน)

### ติดตั้ง
```bash
pip install cryptography
```

ดูคำสั่งทั้งหมด:
```bash
python cli.py help
```

---

## โหมด A: Password-based (AES-256-GCM + scrypt)

### เข้ารหัสข้อความ
```bash
python cli.py encrypt-text -t "hello world" -p "your-strong-password"
python cli.py encrypt-text -t "hello world" -p password.txt
```

### ถอดรหัสข้อความ
```bash
python cli.py decrypt-text -t "<TOKEN>" -p "your-strong-password"
python cli.py decrypt-text -t "<TOKEN>" -p password.txt
```

### เข้ารหัสไฟล์ (**รับเฉพาะ .txt**)
> โหมดนี้ถูกตั้งใจให้ใช้กับข้อความ/ไฟล์ข้อความเท่านั้น จึงล็อค input เป็น `.txt`
```bash
python cli.py encrypt-file -i notes.txt -p password.txt
# default output: notes.aes  (ไฟล์ข้อความที่เก็บ token)
```

### เข้ารหัสไฟล์แบบ raw (`--raw`) → **ต้องเป็น .aesbin**
- ข้อดี: ไฟล์เล็กกว่า (ไม่บวมจาก base64) และเร็วกว่า
- กติกา: ถ้าใช้ `--raw` แล้วใส่ `-o` ต้องลงท้ายด้วย `.aesbin` เสมอ
```bash
python cli.py encrypt-file -i notes.txt -p password.txt --raw -o notes.aesbin
# ถ้าไม่ใส่ -o: default เป็น notes.aesbin
```

### ถอดรหัสไฟล์ → **บังคับ output เป็น .txt**
- ถ้าใส่ `-o` แต่ไม่ใช่ `.txt` ระบบจะปรับเป็น `.txt` ให้เอง
```bash
python cli.py decrypt-file -i notes.aes -p password.txt -o notes.dec.txt
```

### ถอดรหัสไฟล์ raw
- ใส่ `--raw` หรือให้ไฟล์ input ลงท้าย `.aesbin` (auto-detect)
```bash
python cli.py decrypt-file -i notes.aesbin -p password.txt -o notes.dec.txt
# หรือ
python cli.py decrypt-file -i notes.aesbin --raw -p password.txt -o notes.dec.txt
```

---

## โหมด B: PEM (Hybrid RSA+AES) — ไม่ต้องแชร์รหัสผ่าน

### 1) สร้าง keypair
```bash
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem
```

ตัวเลือกที่ใช้บ่อย:
```bash
python cli.py generate-keypair --bits 4096 --private-out private_key.pem --public-out public_key.pem
python cli.py generate-keypair --encrypt-private --private-out private_key.pem --public-out public_key.pem
python cli.py generate-keypair --private-pass "mypw" --private-out private_key.pem --public-out public_key.pem
python cli.py generate-keypair --private-pass pass.txt --private-out private_key.pem --public-out public_key.pem
```

> หมายเหตุ: โค้ดจะเติม timestamp ต่อท้ายชื่อไฟล์เสมอ (เช่น `_18022026_204148`)

### 2) เข้ารหัส/ถอดรหัส “ข้อความ” ด้วย .pem
```bash
python cli.py encrypt-text-pem -t "hello" --public-key friend_public.pem
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem --private-pass "mypw"
```

### 3) เข้ารหัส/ถอดรหัส “ไฟล์” ด้วย .pem (token mode)
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem
# default output: notes_pem.aes

python cli.py decrypt-file-pem -i notes_pem.aes --private-key my_private.pem -o notes.dec.txt
```

### 4) PEM file mode แบบ raw (`--raw`) → **ต้องเป็น .aesbin**
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem --raw -o notes_pem.aesbin
python cli.py decrypt-file-pem -i notes_pem.aesbin --private-key my_private.pem --raw -o notes.dec.txt
```

---

## โหมด Interactive
ถ้ารันโดยไม่ใส่อาร์กิวเมนต์ จะเข้า interactive shell:
```bash
python cli.py
```

---

## หมายเหตุด้านความปลอดภัย
- Token เป็น Base64 เพื่อความสะดวกในการส่ง ไม่ใช่ “obfuscation”
- AES-GCM มีการตรวจจับการแก้ไขข้อมูล (tamper detection)
- ถ้า decrypt แล้วขึ้นแนว ๆ “wrong password/key or corrupted data” แปลว่า **รหัส/คีย์ผิด หรือข้อมูลถูกแก้/เสียหาย**

---

## English

## Features
- Encrypt/decrypt **text** → URL-safe Base64 token
- Encrypt/decrypt **files**
- **Password mode:** AES-256-GCM + scrypt (default)
- **PEM mode:** Hybrid RSA-OAEP + AES-256-GCM (no shared password)

## Install
```bash
pip install cryptography
```

List commands:
```bash
python cli.py help
```

---

## Mode A: Password-based

### Encrypt text
```bash
python cli.py encrypt-text -t "hello world" -p "your-strong-password"
python cli.py encrypt-text -t "hello world" -p password.txt
```

### Decrypt text
```bash
python cli.py decrypt-text -t "<TOKEN>" -p "your-strong-password"
python cli.py decrypt-text -t "<TOKEN>" -p password.txt
```

### Encrypt file (**.txt only**)
```bash
python cli.py encrypt-file -i notes.txt -p password.txt
# default output: notes.aes (a text file storing the base64 token)
```

### Encrypt file as raw (`--raw`) → **output must be .aesbin**
```bash
python cli.py encrypt-file -i notes.txt -p password.txt --raw -o notes.aesbin
# if -o is omitted: default is notes.aesbin
```

### Decrypt file → **output is forced to .txt**
```bash
python cli.py decrypt-file -i notes.aes -p password.txt -o notes.dec.txt
```

### Decrypt raw file
```bash
python cli.py decrypt-file -i notes.aesbin -p password.txt -o notes.dec.txt
# or
python cli.py decrypt-file -i notes.aesbin --raw -p password.txt -o notes.dec.txt
```

---

## Mode B: PEM (Hybrid RSA+AES)

### Generate keypair
```bash
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem
```

### Encrypt/decrypt text with PEM
```bash
python cli.py encrypt-text-pem -t "hello" --public-key friend_public.pem
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem --private-pass "mypw"
```

### Encrypt/decrypt file with PEM (token mode)
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem
# default: notes_pem.aes

python cli.py decrypt-file-pem -i notes_pem.aes --private-key my_private.pem -o notes.dec.txt
```

### PEM raw (`--raw`) → **output must be .aesbin**
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem --raw -o notes_pem.aesbin
python cli.py decrypt-file-pem -i notes_pem.aesbin --private-key my_private.pem --raw -o notes.dec.txt
```

---

## Quick security notes
- Base64 tokens are for transport convenience, not obfuscation.
- AES-GCM provides confidentiality + integrity (tamper detection).
- If decryption fails, it usually means the password/key is wrong or the data is corrupted.
