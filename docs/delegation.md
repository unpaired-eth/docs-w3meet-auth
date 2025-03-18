# 🔑 Dfinity Delegation

Dfinity Delegation allows users to authenticate securely without sharing their private keys.

## 📜 What is Dfinity Delegation?
- Instead of exposing private keys, a **delegation** allows authentication with a **temporary key**.
- This delegation is signed by a trusted **Identity Provider** (in this case, the canister).

## 🛠️ How Delegation Works
1. The canister **creates a delegation** for the user’s **public key**.
2. The delegation contains:
   - **Principal**: The user's unique identifier.
   - **Delegation Metadata**: Expiry and permissions.
   - **Public Key**: Used for verification.
3. The delegation is **signed** and sent back to the mobile app.

## ✅ Benefits of Delegation
- Users don’t need to expose their **private keys**.
- Temporary authentication reduces security risks.
- Works seamlessly with **Dfinity identity** mechanisms.

## 📖 Related Topics:
- [Authentication Flow](authentication.md)
- [System Overview](index.md)