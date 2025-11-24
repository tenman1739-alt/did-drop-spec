# 🛰️ **did-drop-spec**  
### **did:drop v1.2 (Final)**  

---

## **1. Abstract (Updated)**  
> The did:drop method is a Decentralized Identifier method designed for autonomous drone delivery and geospatial logistics.  
> It binds a digital identity to a high-availability, encrypted array of physical coordinates (Landing Options) and access metadata.  
> It utilizes the **Algorand blockchain** for high-speed, low-cost Encrypted State Management and is designed to interface with a **Human-Readable Alias Layer (HRAL)** for quick, human-friendly lookup, acting as a decentralized DNS service for landing spots.  
>  
> **Access to the coordinates requires an explicit, cryptographic Mission Authorization Token from the controller.**

---

## **2. The DID Format**
did-drop = “did:drop:” network “:” app-id
network  = “mainnet” / “testnet” / “betanet”
app-id   = 1*DIGIT

**Example:**
did:drop:mainnet:4839201

---

## **3. Operations**

---

### **3.1 Create**

1. The controller generates an Algorand account (the Master Key).  
2. The controller deploys the DROP-Smart-Contract to the Algorand blockchain. The resulting AppID becomes the suffix of the DID.  
3. The controller uses their App to encrypt the array of Landing Options (using a unique AES-256 key stored securely in the App's Secure Enclave).  
4. The controller sends an ApplicationCall transaction to store the Encrypted Data Blob in Algorand Box Storage.  
5. **HRAL Registration:** The controller registers a Human-Readable Alias (e.g., `warehouse@alias`) with the HRAL, mapping it to `did:drop:mainnet:4839201`.  

---

### **3.2 Read (Resolve)**

To resolve a Landing Pad:

1. **HRAL Lookup:** Retrieve the DID via alias.  
2. **DID Resolution:** Query the Algorand Indexer for the AppID.  
3. Read Global State for controller keys and communication endpoints (`serviceEndpoint`).  
4. Read Box Storage to retrieve the Encrypted Data Blob.  
5. Construct a standardized DID Document containing encrypted coordinates and communication endpoints.  

⚠️ **Resolvers do NOT receive usable coordinates — only encrypted payloads requiring off-chain keys.**

---

### **3.3 Update**

1. **Landing Options:**  
   - Decrypt existing Box data  
   - Modify plaintext array  
   - Re-encrypt with original AES-256 key  
   - Overwrite Box Storage  

2. **HRAL:**  
   - Update alias mappings (e.g., point `warehouse@alias` to a new DID).  

---

### **3.4 Deactivate**

The controller sends a `DeleteApplication` transaction, removing DID storage and revoking the identifier.

---

## **4. Data Model: Landing Options Array (Mandatory Encryption)**

### **4.1 Public On-Chain Data (Encrypted)**

- **Storage Location:** Algorand Box Storage  
- **Format:** JSON object containing:  
  - `Ciphertext` — AES-256-GCM encrypted data  
  - `IV` — Initialization Vector  
  - `Tag` — Authentication Tag  
  - `ControllerSignature` — Signed integrity proof  

---

### **4.2 Private Decrypted Schema (Landing Options Array)**  
Accessible **only** after authorization and decryption.

---

## **5. Mandatory Authorization Protocol: Key Exchange**

> This protocol enforces **explicit user confirmation** before any coordinates are revealed.

---

### **5.1 Cryptography Standards**

- **Key Agreement:** X25519 ECDH  
- **Content Encryption:** AES-256-GCM  

---

### **5.2 Handshake Enforcement**

The AES-256 Decryption Key is **never stored on-chain**.

**Authorization Token Flow:**

1. Dispatcher sends mission request to user's `serviceEndpoint`.  
2. User's App generates shared secret via X25519.  
3. App encrypts AES-256 Decryption Key using shared secret.  
4. Encrypted key is returned as Mission Authorization Token.  
5. Dispatcher decrypts Box Data → obtains coordinates → launches mission.  

**Mandatory Confirmation:**  
- Human-tap approval for non-trusted parties.  
- Optional auto-approval for trusted parties.

---

## **6. Security & Privacy Considerations**

---

### **6.1 PII on Public Ledgers**

All Landing Options (including StreetAddress + Shape geometry) are always:  
- **AES-256-GCM encrypted**,  
- **Stored in Box Storage**,  
- **Decryption key remains off-chain & user-controlled.**

---

### **6.2 Correlation Risks (Updated)**

- The DID and HRAL alias mappings are **public**.  
- Coordinates remain **undiscoverable** without a valid Mission Authorization Token.  

---
