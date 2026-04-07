# Ambidextre

**Student Project - For Educational Purposes Only**

> A browser-based encryption and steganography learning tool built as a university coursework project to explore modern cryptographic primitives and information hiding techniques.

**[Live Demo](https://debrouillez-vous.github.io/Ambidextre)** &nbsp;·&nbsp; Single HTML file &nbsp;·&nbsp; AES-256-GCM &nbsp;·&nbsp; Argon2id · 64 MiB

---

## About This Project

Ambidextre was developed as an academic exercise to study and demonstrate core concepts in applied cryptography and steganography. The goal was to build a practical, self-contained tool using only browser-native APIs, specifically the [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API), to better understand how real-world encryption pipelines work.

This project is intended solely for learning. It is **not** designed for protecting sensitive data in production environments.

---

## Learning Objectives

This project was built to explore the following topics:

- **Symmetric encryption** - understanding AES-256-GCM, authenticated encryption, and why nonce reuse is dangerous
- **Key derivation functions** - comparing PBKDF2 vs Argon2id and understanding memory-hard KDFs as a defence against GPU-based brute-force attacks
- **Classical ciphers** - implementing Caesar shift as a historical reference point and understanding why substitution ciphers alone are insecure
- **LSB steganography** - learning how data can be hidden in the least significant bits of image pixels, and the limitations of this approach
- **Client-side security** - building a zero-server-communication tool where all processing stays in the browser

---

## Features Demonstrated

### Text Encryption
Encrypts plaintext into a portable base64 token.

- Caesar cipher pre-shuffle with configurable shift key (1–25)
- AES-256-GCM encryption with a master password
- Argon2id key derivation (64 MiB · 3 passes · parallelism 4)
- Self-contained token output

### File Encryption
Encrypts any file into an `.arc` encrypted container.

- Same AES-256-GCM pipeline as text mode
- Original filename and content are fully restored on decryption

### Steganography - Inject & Extract
Hides encrypted data inside an ordinary PNG image.

- **Text injection** - embeds an encrypted text payload in the LSB of pixel RGB channels
- **File injection** - compresses, encrypts, then hides a file inside a carrier image
- Output is a standard PNG visually indistinguishable from the original
- JPEG carriers are automatically converted to lossless PNG
- Extract mode recovers and decrypts hidden content using the correct password

---

## How It Works

### Encryption Pipeline

```
Text mode:
  plaintext
    → Caesar shift
    → string reverse
    → AES-256-GCM  (Argon2id key, 32-byte salt, 12-byte IV)
    → base64 token

File mode:
  file bytes
    → byte-level shift
    → byte-level reverse
    → AES-256-GCM  (Argon2id key, 32-byte salt, 12-byte IV)
    → .arc container
```

### Steganography Pipeline

```
File injection:
  secret file
    → deflate-raw compression
    → byte-level shift + reverse
    → AES-256-GCM encryption
    → LSB embed into carrier PNG (1 bit per R/G/B channel)
    → stego PNG download

Extraction:
  stego PNG
    → LSB read → header parse (AMBI magic + mode + filename + length)
    → AES-256-GCM decryption
    → decompress
    → original file restored
```

### Crypto Parameters

| Parameter | Value |
|---|---|
| Cipher | AES-256-GCM |
| Key derivation | Argon2id |
| Memory | 64 MiB (65536 KiB) |
| Iterations | 3 passes |
| Parallelism | 4 |
| Salt | 32 bytes (random per operation) |
| IV / Nonce | 12 bytes (random per operation) |
| Auth tag | 16 bytes |
| Stego method | LSB - 1 bit per RGB channel |

---

## Steganography Capacity

Image capacity depends on dimensions:

```
max_bytes = floor(width × height × 3 / 8)
```

With deflate compression, effective capacity for compressible files (text, code, documents) is roughly doubled. Already-compressed formats (ZIP, JPEG, MP4) see little reduction.

| Secret file size | Minimum carrier image |
|---|---|
| 10 KB | ~170 × 170 px |
| 100 KB | ~520 × 520 px |
| 500 KB | ~1160 × 1160 px |
| 1 MB | ~1680 × 1680 px |

---

## Usage

### Local (offline)
```
Download index.html → open in any modern browser
```
No build step. No npm. No server. Works fully offline.

### Hosted
Visit **[https://debrouillez-vous.github.io/Ambidextre](https://debrouillez-vous.github.io/Ambidextre)**

All processing happens client-side. The page never transmits any data.

---

## Browser Compatibility

Requires a browser supporting the Web Crypto API and `CompressionStream`.

| Browser | Support |
|---|---|
| Chrome / Edge 80+ | Full |
| Firefox 113+ | Full |
| Safari 16.4+ | Full |

---

## Key Takeaways (What I Learned)

- **Authenticated encryption matters.** AES-GCM provides both confidentiality and integrity - without authentication, ciphertext can be tampered with silently.
- **KDF choice is critical.** Argon2id's memory-hardness makes brute-force significantly more expensive than older KDFs like PBKDF2 or bcrypt.
- **Classical ciphers are not security.** The Caesar shift and byte-level reverse are included as educational layers to illustrate how easily substitution ciphers are broken. All actual security comes from AES-256-GCM.
- **Steganography complements but does not replace encryption.** Hiding data without encrypting it first provides no real security - Ambidextre always encrypts before embedding.
- **Lossless formats are mandatory for stego.** JPEG recompression destroys LSB data. Any steganographic embedding must use a lossless format like PNG.
- **Client-side crypto is viable.** The Web Crypto API provides hardware-accelerated AES and secure random number generation, making browser-based tools practical for educational demonstrations.

---

## Disclaimer

This project was created strictly for educational and academic purposes as part of my studies in cybersecurity. It is not intended to facilitate any illegal activity. Users are solely responsible for ensuring their use of this tool complies with all applicable laws and regulations.

---

## License

MIT - see [LICENSE](LICENSE)
