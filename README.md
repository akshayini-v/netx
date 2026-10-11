# NetX — Secure Cryptographic File Transfer System

**NetX** is a modern, high-fidelity dark-mode web application and cryptographic transfer engine built for the college **Cryptography and Network Security (CNS)** project.

It provides authenticated, tamper-evident, and end-to-end protected file transfers between verified senders and receivers using authenticated symmetric encryption, public-key cryptography envelopes, cryptographic hashing, digital signatures, receiver-side private key decryption, and immutable security audit logs.

---

## 1. Cryptographic Primitives & Specifications

| Security Property | Algorithm / Standard | Implementation Details |
| :--- | :--- | :--- |
| **Confidentiality & AEAD** | **AES-256-GCM** (NIST SP 800-38D) | 256-bit dynamic key, 96-bit unique nonce per transfer, 128-bit authentication tag. Protects against ciphertext tampering. |
| **Key Encapsulation** | **RSA-OAEP** (2048-bit) | Symmetric AES key is wrapped with receiver's RSA public key using OAEP padding with SHA-256 and MGF1. |
| **Integrity Verification** | **SHA-256** (FIPS 180-4) | Computes pre-encryption digest of plaintext and matches against post-decryption output. |
| **Origin Authentication** | **Ed25519** (RFC 8032) | High-speed elliptic curve digital signature computed over file fingerprint using sender's private key. |
| **Key Derivation (KDF)** | **scrypt** (RFC 7914) | Memory-hard key derivation to encrypt users' private keys with their master password. |
| **Receiver Private Key Decryption** | **RSA-OAEP Decapsulation** | Receiver decrypts AES session key using their dedicated private key (password unlock or direct PEM paste) to decrypt and verify the file. |
| **Audit Ledger** | **Immutable SQLite Trail** | Zero-leak audit logging of all security events without recording secret keys or passwords. |

---

## 2. Pre-Seeded Demonstration Profiles

For live evaluation and testing, the database is pre-seeded with three accounts:

| Role | Username | Password | Email | Purpose |
| :--- | :--- | :--- | :--- | :--- |
| **Sender** | `sender_alice` | `AlicePass123!` | `alice@securetransfer.internal` | Uploads and encrypts files for receiver |
| **Receiver** | `receiver_bob` | `BobPass123!` | `bob@securetransfer.internal` | Decrypts files with RSA private key & verifies tags |
| **Administrator** | `admin_sec` | `AdminPass123!` | `admin@securetransfer.internal` | Access governance, RBAC, system metrics |

> **Note:** You can also register custom accounts with any username, email, and role via the UI, or switch between profiles in 1 click using the **Demo Profiles** button.

---

## 3. Project Directory Structure

```
securetransfer/
├── backend/
│   ├── app.py                     # Main Flask server entrypoint (port 5005) & demo seeding
│   ├── routes/
│   │   ├── auth.py                # Registration, login, logout, me
│   │   ├── transfers.py           # File upload, sent/received lists, decryption
│   │   ├── users.py               # Receiver directory & public keys
│   │   ├── audit.py               # Audit log querying & CSV export
│   │   └── admin.py               # Admin metrics and RBAC
│   ├── services/
│   │   ├── transfer_service.py    # End-to-end cryptographic transfer engine
│   │   ├── user_service.py        # Key generation and user management
│   │   └── audit_service.py       # Tamper-evident event recording
│   ├── crypto/
│   │   ├── aes_gcm.py             # AES-256-GCM AEAD encryption/decryption
│   │   ├── asymmetric.py          # RSA-OAEP key wrapping & scrypt key encryption
│   │   ├── signatures.py          # Ed25519 digital signatures
│   │   └── hashing.py             # SHA-256 fingerprinting & password hashing
│   ├── security/
│   │   ├── auth_middleware.py     # Session tokens & RBAC decorators
│   │   └── rate_limiter.py        # Sliding window brute-force protection
│   ├── models/
│   │   └── database.py            # SQLite schema initialization
│   └── tests/
│       ├── test_crypto.py         # Cryptographic unit test suite
│       └── test_api.py            # API integration & security test suite
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── BackgroundVideo.jsx # Ambient background visual component
│   │   │   ├── Navbar.jsx          # Top navigation bar with NetX branding & demo profiles
│   │   │   └── AuthModal.jsx       # Authentication modal with demo presets
│   │   ├── pages/
│   │   │   ├── LandingPage.jsx     # Midnight editorial hero, interactive cryptographic pipeline
│   │   │   ├── SenderDashboard.jsx # File drag-and-drop & AES-GCM encryption
│   │   │   ├── ReceiverDashboard.jsx # Receiver private-key decryption & tamper attack simulation
│   │   │   ├── AuditDashboard.jsx  # Security audit trail, filter, CSV export
│   │   │   └── AdminDashboard.jsx  # Metrics & user access governance
│   │   ├── context/
│   │   │   └── AuthContext.jsx     # Session provider
│   │   ├── services/
│   │   │   └── api.js              # Centralized API fetch client
│   │   ├── App.jsx                 # React Router routing
│   │   └── main.jsx
│   ├── vite.config.js
│   └── package.json
├── storage/
│   └── encrypted/                 # Encrypted ciphertext storage directory
└── README.md
```

---

## 4. How to Run Locally

### Step 1: Start the Flask Backend Server

In a terminal window:

```bash
cd "securetransfer"
pip install -r requirements.txt   # Flask, Flask-Cors, cryptography
cd backend
python app.py
```

The Flask backend will start on **`http://127.0.0.1:5005`** and automatically initialize `securetransfer.db` with seeded demo accounts.

---

### Step 2: Start the React + Vite Frontend

In a second terminal window:

```bash
cd "securetransfer/frontend"
npm run dev
```

The frontend will start on **`http://localhost:5173`** (or `http://localhost:5174`) with an automatic proxy to the Flask backend on port 5005.

---

## 5. Running Cryptographic & API Test Suites

Run the test suite to verify encryption, decryption, wrong-key failure, tampering rejection, and digital signatures:

```bash
cd "securetransfer/backend"

# Run Cryptographic Unit Tests
python -m unittest tests/test_crypto.py

# Run API Integration & Security Tests
python -m unittest tests/test_api.py
```

All tests will execute and output `OK`.

---

## 6. Live Demonstration Workflow for CNS Examiners

1. **Open the Web Application** at `http://localhost:5173` (or `http://localhost:5174`).
2. Click **"Demo Profiles" -> "Alice (Sender)"** in the top navigation bar.
3. In the **Sender Dashboard**, drag and drop a test file (e.g. a document, PDF, or text file).
4. Select **`Bob (receiver_bob)`** as the recipient.
5. Click **"Encrypt & Transmit File"**. Observe real-time hashing, AES-256-GCM encryption, and RSA-OAEP envelope wrapping.
6. Switch user profile to **"Bob (Receiver)"**.
7. In the **Receiver Dashboard**, see the incoming payload from Alice.
8. Click **"Decrypt & Verify"**:
   - **Normal Decryption**: Enter Bob's password (`BobPass123!`) or use Bob's private key, click decrypt. The AES key is unwrapped via RSA-OAEP, authentication tag is verified, SHA-256 matched, and the recovered file is downloaded.
   - **Tampering Attack Simulation**: Enable the *"Simulate Ciphertext Tampering Attack"* toggle and click execute. Invert bits in the ciphertext and witness AES-256-GCM AEAD immediately detect the alteration, reject the decrypted output, and log a `TAMPERING_DETECTED` security incident.
9. Switch user profile to **"Sec Admin"**.
10. Open **"Audit Trail"** (`/audit`) to view the immutable ledger of transfers, SHA-256 verification results, and attack alerts. Download the audit CSV report.
11. Open **"Admin Control Panel"** (`/admin`) to inspect overall system metrics and manage user account activations and roles.
