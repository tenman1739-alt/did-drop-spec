# did-drop-spec
did:drop v1.2 (Final)
1. Abstract (Updated)
The did:drop method is a Decentralized Identifier method designed for autonomous drone delivery and geospatial logistics. It binds a digital identity to a high-availability, encrypted array of physical coordinates (Landing Options) and access metadata. It utilizes the Algorand blockchain for high-speed, low-cost Encrypted State Management and is designed to interface with a Human-Readable Alias Layer (HRAL) for quick, human-friendly lookup, acting as a decentralized DNS service for landing spots. Access to the coordinates requires an explicit, cryptographic Mission Authorization Token from the controller.
2. The DID Format
The format for did:drop consists of the method name, the Algorand network environment, and the unique Application ID (AppID) of the smart contract storing the record.

did-drop = "did:drop:" network ":" app-id
network  = "mainnet" / "testnet" / "betanet"
app-id   = 1*DIGIT

Example: did:drop:mainnet:4839201

3. Operations
3.1 Create
1. The controller generates an Algorand account (the Master Key).
2. The controller deploys the DROP-Smart-Contract to the Algorand blockchain. The resulting AppID becomes the suffix of the DID.
3. The controller uses their App to encrypt the array of Landing Options (using a unique AES-256 key stored securely in the App's Secure Enclave).
4. The controller sends an ApplicationCall transaction to store the Encrypted Data Blob in Algorand Box Storage.
5. HRAL Registration: The controller registers a Human-Readable Alias (e.g., warehouse@alias) with the HRAL, mapping it to did:drop:mainnet:4839201.
3.2 Read (Resolve)
To resolve a Landing Pad:
1. HRAL Lookup: A human or service first queries the HRAL (e.g., searches for warehouse@alias) to quickly retrieve the corresponding did:drop identifier.
2. DID Resolution: The resolver queries the Algorand Indexer for the AppID.
3. It reads the Global State for controller keys and communication endpoints (serviceEndpoint).
4. It reads the Box Storage to retrieve the Encrypted Data Blob containing all Landing Options.
5. It constructs a standardized DID Document that includes the encrypted coordinates and the required communication endpoints.
The resolver DOES NOT return usable coordinates. It returns encrypted data requiring an off-chain key.
3.3 Update
1. For Landing Options Data: The controller must decrypt the existing Box data, modify the plaintext array, re-encrypt it with the original AES-256 key, and send a transaction to overwrite the Encrypted Data Blob in the Box.
2. For HRAL: The controller may send a transaction to the HRAL service to update the alias mapping (e.g., changing warehouse@alias to point to a new did:drop identifier).
3.4 Deactivate
The controller sends a DeleteApplication transaction to the Algorand network, removing the storage and effectively revoking the DID.
4. Data Model: Landing Options Array (Mandatory Encryption)
The core data is an array of Landing Option objects, where the entire array is encrypted before being stored in Algorand's Box Storage.
4.1. The Public On-Chain Data (Encrypted)
• Storage Location: Algorand Box Storage.
• Format: A single, large JSON object containing:
• Ciphertext: The AES-256-GCM Encrypted Data Blob (the array of Landing Options).
• IV: Initialization Vector.
• Tag: Authentication Tag.
• ControllerSignature: Proof of data integrity signed by the Master Key.
4.2. The Private Decrypted Schema (Array of Landing Options)
This structure is available only after decryption via the mandatory Authorization Protocol.

5. Mandatory Authorization Protocol: Key Exchange
This protocol enforces mandatory user confirmation before coordinates are revealed.
5.1. Cryptography Standards
• Key Agreement: X25519 Elliptic Curve Diffie-Hellman Key Exchange is used for securely establishing a shared secret between the Dispatcher and the User's App.
• Content Encryption: AES-256-GCM is used for authenticated symmetric encryption of the coordinates blob on-chain.
5.2. Handshake Enforcement
The AES-256 Decryption Key (the Authorization Token) is never stored on the blockchain.
• Authorization Token Flow:
1. Dispatcher sends a mission request to the User's serviceEndpoint.
2. The User's App generates a shared secret with the Dispatcher via X25519.
3. The App encrypts the AES-256 Decryption Key (stored in the Secure Enclave) using the X25519 shared secret.
4. The encrypted key is sent back as the Mission Authorization Token.
5. The Dispatcher uses the Token to decrypt the Box Data, access the coordinates, and launch.
• Mandatory Confirmation: The User's App must actively participate in the X25519 key exchange for the mission to proceed. For non-trusted parties, this must be a human-tapped approval. For trusted parties, the user can pre-configure the app to automatically execute the key exchange, maintaining security (only the authorized party gets the key) while allowing for speed.
6. Security & Privacy Considerations
6.1 PII on Public Ledgers
The complete and usable set of Landing Options, including StreetAddress and detailed Shape coordinates, is always stored as an AES-256-GCM Encrypted Data Blob. This ensures high data availability via Algorand while preserving privacy, as the decryption key remains under the exclusive control of the user's secure application.
6.2 Correlation Risks (Updated)
While the did:drop identifier is publicly resolvable on-chain, and the HRAL provides a public link between an alias (e.g., warehouse@alias) and that identifier, the actual geospatial data is encrypted. An observer can correlate the alias and the DID, but not the physical coordinates, unless they have intercepted and decrypted a valid Mission Authorization Token.














