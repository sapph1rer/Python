[README.md](https://github.com/user-attachments/files/25391839/README.md)
# aes256-cli (CLI Encryption Tool)

> **File:** `cli.py`  
> **What it does:** Encrypt/Decrypt **text** and **files** using AES-256-GCM (password mode) and optional RSA PEM (public/private) hybrid mode.

---

## ภาษาไทย

### ภาพรวม
`cli.py` เป็นเครื่องมือแบบ Command Line (CLI) สำหรับ **เข้ารหัส/ถอดรหัส**
- **ข้อความ (Text)**: ได้ผลลัพธ์เป็น token แบบ Base64 URL-safe (ส่งทางแชทได้)
- **ไฟล์ (File)**: เข้ารหัสแล้วเขียนเป็นไฟล์ผลลัพธ์

รองรับ 2 โหมดหลัก:
1) **Password-based (AES-256-GCM + scrypt)** — เหมาะกับการใช้กันเองแบบ “แชร์รหัสผ่านร่วมกัน”
2) **Public/Private key (RSA PEM + AES-256-GCM)** — ไม่ต้องแชร์รหัสผ่าน ใช้ public key ของผู้รับเข้ารหัส และใช้ private key ของผู้รับถอดรหัส (Hybrid encryption)

---

### ติดตั้ง
ต้องมี Python 3.x และติดตั้งไลบรารี:
```bash
pip install cryptography
```

ดูคำสั่งทั้งหมด:
```bash
python cli.py help
```

---

## โหมด A: Password-based (แชร์รหัสผ่าน)

### 1) เข้ารหัส “ข้อความ”
```bash
python cli.py encrypt-text -t "hello world" -p "your-strong-password"
```

อ่านรหัสผ่านจากไฟล์ (ไฟล์ .txt มี 1 บรรทัด):
```bash
python cli.py encrypt-text -t "hello world" -p password.txt
```

### 2) ถอดรหัส “ข้อความ”
```bash
python cli.py decrypt-text -t "<TOKEN>" -p "your-strong-password"
```

### 3) เข้ารหัส “ไฟล์”
> หมายเหตุ: ในโค้ดนี้คำสั่ง `encrypt-file` ถูกล็อคให้รับเฉพาะไฟล์ `.txt`
```bash
python cli.py encrypt-file -i notes.txt -p "your-strong-password"
# output เริ่มต้น: notes.aes (เป็นไฟล์ข้อความที่เก็บ token)
```

### 4) ถอดรหัส “ไฟล์”
```bash
python cli.py decrypt-file -i notes.aes -o notes.dec.txt -p "your-strong-password"
```

---

## โหมด B: Public/Private key (RSA PEM) — ไม่ต้องแชร์รหัสผ่าน

### 1) สร้างคู่คีย์
```bash
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem
```

ตัวเลือกสำคัญ:
```bash
# เลือกขนาดคีย์
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --bits 4096

# เข้ารหัส private key (แนะนำ)
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --encrypt-private

# หรือส่งรหัสผ่านให้ private key
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --private-pass "mypw"
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --private-pass pass.txt
```

### 2) เข้ารหัส “ข้อความ” ด้วย public key ของผู้รับ
```bash
python cli.py encrypt-text-pem -t "hello" --public-key friend_public.pem
```

### 3) ถอดรหัส “ข้อความ” ด้วย private key ของตัวเอง
```bash
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem
# ถ้า private key ถูกเข้ารหัสไว้:
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem --private-pass "mypw"
```

### 4) เข้ารหัส “ไฟล์” ด้วย public key ของผู้รับ
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem
# output เริ่มต้น: notes_pem.aes
```

### 5) ถอดรหัส “ไฟล์” ด้วย private key ของตัวเอง
```bash
python cli.py decrypt-file-pem -i notes_pem.aes --private-key my_private.pem -o notes.dec.txt
```

---

## โหมด Interactive (พิมพ์คำสั่งทีละบรรทัด)
ถ้ารันโดยไม่ใส่อาร์กิวเมนต์ จะเข้าโหมด interactive:
```bash
python cli.py
```
แล้วพิมพ์คำสั่ง เช่น:
```
encrypt-text -t "hello" -p password.txt
```

---

## English

### Overview
`cli.py` is a command-line tool to **encrypt/decrypt**
- **Text**: outputs a URL-safe Base64 token (easy to share)
- **Files**: writes the encrypted token to an output file

Two modes:
1) **Password-based (AES-256-GCM + scrypt)** — share a password
2) **Public/Private key (RSA PEM + AES-256-GCM)** — hybrid encryption using the recipient’s public key and the recipient’s private key for decryption

---

### Installation
Requires Python 3.x and:
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
```

Read password from file (single-line `.txt`):
```bash
python cli.py encrypt-text -t "hello world" -p password.txt
```

### Decrypt text
```bash
python cli.py decrypt-text -t "<TOKEN>" -p "your-strong-password"
```

### Encrypt file
> Note: `encrypt-file` is restricted to `.txt` files in this codebase.
```bash
python cli.py encrypt-file -i notes.txt -p "your-strong-password"
# default output: notes.aes
```

### Decrypt file
```bash
python cli.py decrypt-file -i notes.aes -o notes.dec.txt -p "your-strong-password"
```

---

## Mode B: RSA PEM public/private (no shared password)

### Generate keypair
```bash
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem
```

Useful options:
```bash
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --bits 4096
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --encrypt-private
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --private-pass "mypw"
python cli.py generate-keypair --private-out private_key.pem --public-out public_key.pem --private-pass pass.txt
```

### Encrypt text with recipient public key
```bash
python cli.py encrypt-text-pem -t "hello" --public-key friend_public.pem
```

### Decrypt text with your private key
```bash
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem
python cli.py decrypt-text-pem -t "<TOKEN>" --private-key my_private.pem --private-pass "mypw"
```

### Encrypt file with recipient public key
```bash
python cli.py encrypt-file-pem -i notes.txt --public-key friend_public.pem
# default output: notes_pem.aes
```

### Decrypt file with your private key
```bash
python cli.py decrypt-file-pem -i notes_pem.aes --private-key my_private.pem -o notes.dec.txt
```

---

## Interactive mode
Run without arguments:
```bash
python cli.py
```
Then type commands line by line.

---

## Security notes (quick)
- The token is Base64 for transport—it’s not “security by obfuscation”.
- AES-GCM provides confidentiality + integrity (tamper detection).
- Never share your private key. Prefer encrypting private keys (`--encrypt-private`).

