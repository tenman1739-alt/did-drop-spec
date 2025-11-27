# 🛰️ did-drop-spec

### **did:drop v2.0 (Mission Plan Architecture)**

---

## 1. Abstract (Revised)

The `did:drop` method is a Decentralized Identifier method designed for verifiable, autonomous drone delivery and geospatial logistics.  
It binds a digital identifier (`did:drop:<AppID>:<MissionID>`) to a pointer—a Content Identifier (CID)—which references a W3C DID Document stored on a decentralized storage network (IPFS/Arweave).  
It utilizes the Algorand blockchain for high-speed, low-cost verifiable state management of these CIDs. This structure establishes a verifiable link between an on-chain ledger record and a standardized off-chain document.

Access to the actionable coordinates requires an authorized party to resolve the DID and cryptographically verify the integrity of the fetched Mission Plan.

---

## 2. The DID Format (Mission Plan Identifier)

The DID format specifies a unique mission plan tied to a specific controller's application registry:

```
did-drop = “did:drop:” app-id “:” mission-id
```

- `did:` Required URI scheme identifier.
- `drop:` The method name.
- `app-id:` The Algorand Application ID of the controller's Mission Plan Registry smart contract.
- `mission-id:` A unique, application-specific identifier (e.g., a hash or UUID) used as the Box key to retrieve the CID pointer on Algorand.

**Example:**

```
did:drop:4839201:ORDER-9876-ALPHA
```

---

## 3. Operations

### 3.1 Create (DID Document Registration)

1. The controller generates the Mission Plan DID Document (W3C standard JSON format) containing the plaintext landing coordinates within the required `missionEndpoint` property.
2. The controller cryptographically signs the complete DID Document using their private key.
3. The controller uploads the signed DID Document to a decentralized storage network (e.g., IPFS/Arweave). This yields the Content Identifier (CID).
4. The controller sends an ApplicationCall transaction to their Mission Plan Registry (AppID) on Algorand to store the following data in Box Storage, using the mission-id as the Box key:
    - The calculated CID of the off-chain document.
    - Metadata (e.g., creation timestamp, controller address).
5. **HRAL Registration (Optional):** The controller may register a Human-Readable Alias (e.g., `backyard@alias`) with the HRAL, mapping it to the full `did:drop:4839201:ORDER-9876-ALPHA`.

### 3.2 Read (Resolve)

To resolve a Mission Plan DID and retrieve actionable coordinates:

1. **Parse DID:** Extract the AppID and MissionID from the `did:drop` string.
2. **On-Chain Lookup:** Query the Algorand Indexer for the corresponding Box in the AppID's Box Storage using the MissionID as the key.
3. **Retrieve Pointer:** Read the Box content to retrieve the Mission Plan CID.
4. **Off-Chain Fetch:** Use the retrieved CID to fetch the complete Mission Plan DID Document (JSON) from IPFS/Arweave.
5. **Verification:** Verify the document's signature against the controller's public key (found in the DID Document's `verificationMethod` or controller state).
6. **Extraction:** Extract the plaintext coordinates and mission parameters from the DID Document's `missionEndpoint` property.

> **Note:** The resolver receives the complete, verifiable Mission Plan Document which contains the usable coordinates.

### 3.3 Update

1. The update operation changes the CID pointer on-chain, effectively revoking the previous Mission Plan and replacing it with a new one.
2. The controller generates a new Mission Plan DID Document with updated coordinates or parameters.
3. The new document is signed and uploaded to decentralized storage, yielding a new CID-2.
4. The controller sends an ApplicationCall transaction to the Mission Plan Registry to overwrite the Box corresponding to the original MissionID with CID-2.
5. **HRAL (Optional):** Update alias mappings if the DID itself has changed (e.g., pointing an alias to a new AppID).

### 3.4 Deactivate

The controller sends a `DeleteApplication` transaction to destroy the Mission Plan Registry smart contract. Alternatively, the controller sends an ApplicationCall to the Registry to explicitly delete the Box record for the MissionID.

---

## 4. Data Model: On-Chain and Off-Chain Components

The data model is split to leverage the strengths of the ledger (trust anchor) and decentralized storage (data availability).

### 4.1 On-Chain Data (The Trust Anchor)

- **Storage Location:** Algorand Box Storage of the AppID.
- **Box Key:** The MissionID (a byte array).
- **Box Content:** A minimal structure containing:
    - `missionCID` (String/Byte Array): The Content Identifier (CID) pointing to the off-chain Mission Plan DID Document.
    - `controllerAddress` (Address): The Algorand address of the controller who registered the mission.

### 4.2 Off-Chain DID Document (The Mission Plan Payload)

- **Storage Location:** IPFS or Arweave.
- **Format:** W3C DID Document (JSON-LD).
- **Mandatory Custom Property:** `missionEndpoint` (Object): Contains the actionable delivery data.

| Property             | Type    | Description                                                                      |
|----------------------|---------|----------------------------------------------------------------------------------|
| id                   | DID     | The full `did:drop:<AppID>:<MissionID>` being resolved.                          |
| controller           | DID     | The identity of the party that signed/created the document.                      |
| verificationMethod   | Array   | Key material used for cryptographic proof.                                       |
| missionEndpoint      | Object  | The core payload. Must contain the delivery coordinates and mission parameters in plaintext.   |

---

## 5. Mandatory Authorization Protocol: Mission Authorization Token (MAT)

This protocol enforces cryptographic proof that the resolving party is authorized to consume the specific Mission Plan data.

### 5.1 Cryptography Standards

- **Signature Algorithm:** Ed25519 (for signing the Mission Plan Document).
- **Authorization Proof:** Algorand Transaction Signature (for the MAT).

### 5.2 Authorization Flow

- The Mission Authorization Token (MAT) is a time-bound cryptographic proof from the consuming party (e.g., the UAV or Dispatcher) proving it has permission to use the coordinates.
- **Resolver (UAV/Store):** Attempts to resolve `did:drop:<AppID>:<MissionID>` (Steps 1-4 of Read).
- **Document Verification:** The Resolver verifies the authenticity of the fetched Mission Plan DID Document using the controller's public key specified in the document.
- **MAT Generation:** To confirm explicit access permission, the Resolver must generate a zero-value Algorand transaction (or a unique logic signature) and sign it, using the did:drop as the note field, creating the MAT.
- **Proof of Consumption:** The Resolver attaches this MAT to its internal log when executing the mission, proving it was authorized by the Mission Plan Registry.
- **Mandatory Security:** The coordinates are actionable (plaintext) within the DID Document, so the integrity check (Step 5 of Read) and the MAT proof are the sole gates for ensuring proper usage.

---

## 6. Security & Privacy Considerations

### 6.1 PII on Public Ledgers

- All sensitive data (landing coordinates) are **NOT** stored on the Algorand ledger. The ledger only holds the unencrypted pointer (CID) to the off-chain data.
- The coordinates are therefore only discoverable to those who:
    - Know the `did:drop` identifier.
    - Can access the decentralized storage network (IPFS/Arweave).
    - Are authorized to use the data (via the MAT Protocol).

### 6.2 Correlation Risks (Revised)

- The DID (`did:drop:<AppID>:<MissionID>`) and HRAL alias mappings are public.
- The CID (the pointer) is publicly stored on the Algorand ledger, but is non-PII.
- The coordinates remain secure because their usage is strictly tied to the cryptographic integrity verification of the Mission Plan DID Document and the Mission Authorization Token before a mission is executed.

---
