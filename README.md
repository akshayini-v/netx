# netx
NetX is a secure file-transfer web app built with Flask and React. Files are encrypted using AES-256-GCM, the session key is wrapped with RSA-OAEP, integrity is verified via SHA-256, and senders are authenticated with Ed25519 signatures. Private keys are encrypted at rest with scrypt, and every action is recorded in a tamper-evident audit trail.
