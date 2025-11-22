# did-drop-spec
DID Method Specification: did:drop
Status: Draft
Target Network: Algorand (Layer 1) & FIO Protocol (Naming Layer)
1. Abstract
The did:drop method is a Decentralized Identifier method designed for autonomous drone delivery and geospatial logistics. It binds a digital identity to precise physical coordinates (landing pads) and access metadata. It utilizes the Algorand blockchain for high-speed, low-cost state management of coordinate data and the FIO Protocol for human-readable aliasing.
2. The DID Format
The format for did:drop consists of the method name, the Algorand network environment, and the unique Application ID (AppID) of the smart contract storing the record.



did-drop = "did:drop:" network ":" app-id
network  = "mainnet" / "testnet" / "betanet"
app-id   = 1*DIGIT

Example: did:drop:mainnet:4839201
3. Operations
3.1 Create
To create a did:drop:
1. The controller generates an Algorand account (the Master Key).
2. The controller deploys the DROP-Smart-Contract to the Algorand blockchain.
3. The resulting AppID (e.g., 555123) becomes the suffix of the DID.
4. Optional: The controller registers a FIO Handle (e.g., warehouse@drone) and maps it to did:drop:mainnet:555123.
3.2 Read (Resolve)
To resolve did:drop:mainnet:555123:
1. The resolver queries the Algorand Indexer for the AppID 555123.
2. It reads the Global State (for controller keys) and Box Storage (for landing pad data).
3. It constructs a standardized DID Document (JSON-LD).
3.3 Update
To update (e.g., adding a new landing pad or updating coordinates):
1. The controller sends an ApplicationCall transaction to the specific AppID.
2. Arguments include the operation (e.g., upsert_pad) and the data (e.g., 5-point coordinates).
3. Authorized Surveyors may sign Verifiable Credentials allowing the controller to ingest verified coordinates.
3.4 Deactivate
The controller sends a DeleteApplication transaction to the Algorand network, removing the storage and effectively revoking the DID.
4. Data Model: Drone Landing Pads
The did:drop method introduces a specific service type: DroneLandingPad.
Structure of a Service Endpoint:
• Type: DroneLandingPad
• Shape: Polygon (5 points required: 4 corners + 1 center) or Point.
• Metadata: Max altitude, approach vector, private/public access.
5. Security Considerations
• Authorization: Only the holder of the Algorand private key defined in the Smart Contract's Global State can mutate the DID Document.
• Surveyor Integrity: Coordinate updates can be gated by requiring a signature from a recognized Surveyor DID, preventing user-input errors.


## 6. Privacy Considerations

### 6.1 PII on Public Ledgers
The `did:drop` method stores data on the Algorand public blockchain. As such, no Personally Identifiable Information (PII) should be stored in the DID Document or the associated smart contract state.

* **Coordinates:** GPS coordinates are considered public infrastructure data (landing pads) and not private user data. Users should not register coordinates for sensitive locations (e.g., "Bedroom Window") that they do not wish to be publicly resolvable.
* **Labels:** Users are advised to use generic labels (e.g., "Drop Point A") rather than personal ones (e.g., "John's House") to minimize correlation risks.

### 6.2 Correlation Risks
Since `did:drop` is often paired with FIO handles, an observer could correlate a wallet address with a specific landing pad location. Users should be aware that the link between their FIO handle and their `did:drop` identifier is publicly visible on-chain.










