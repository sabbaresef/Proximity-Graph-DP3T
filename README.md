# Proximity Graph Project

## Overview
This project extends the DP-3T protocol to build a **proximity graph** of COVID-19 positive users and their verified contacts. The graph is intended as a privacy-preserving epidemiological tool, enabling researchers to study virus spread without compromising user anonymity.

**IT IS A UNIVERSITY PROJECT DEVELOPED IN COLLABORATION WITH: Rispoli Mario, Ruggiero Valeria, Russo Gabriele Luigi, Silla Marta**

---

## Key Entities
- **Government Server**: Coordinates data collection, notifies users, and maintains the proximity graph.
- **Analysis Laboratory**: Issues positive test confirmations to users.
- **Epidemiologists**: Analyze the proximity graph for spread patterns.
- **Infected User (A)**: Receives a one-time authorization code upon positive test.
- **At-Risk User (B)**: Voluntarily proves contact with an infected user.

---

## Glossary
- **Proximity Graph**: Nodes represent users; edges denote verified contact events.
- **False Contact**: Reported contact that did not actually occur.
- **ephID**: Ephemeral identifier derived from a daily seed (`seed_i`).
- **Zero-Knowledge Proof (ZKP)**: Schnorr non-interactive protocol used to authenticate contacts.

---

## Project Objectives
1. Specify desired **completeness**, **security**, and **privacy** properties.
2. Design a protocol to construct a correct proximity graph based on DP-3T.
3. Heuristically validate the design against security and privacy goals.
4. Partially implement the protocol.

---

## Protocol Design

1. **DP-3T Foundations**  
   - Users broadcast `ephID_i || i` via Bluetooth Low Energy (BLE).  
   - Upon a positive test, user A uploads selected `seed_i || i` to the server.  
   - The server broadcasts a Cuckoo filter of hashed identifiers to all clients.

2. **Contact Verification**  
   - User B detects matches locally and requests to join the graph by submitting hashed `ephID_i || i` values.
   - **Authentication Code**: A one-time code from the health authority confirms A’s infection.
   - **ZKP Protocol**: Uses Schnorr non-interactive ZKP `(G, p, q, g, x, y)` where:
     - `y = g^x mod p`, `x` is the secret shared by A in each epoch.
     - B proves knowledge of `x` corresponding to `y`, without revealing `x`.
   - The server adds edge A→B only if ZKP verifies.

---

## Communication Packets

- **A → Server**  
  ```json
  {
    "node_ID": "<random 32-byte>",
    "seed_i || i || y_i": "<public data>",
    "auth_code": "<authority code>"
  }
  ```

- **B → Server**
  ```json
  {
    "node_ID": "<random 32-byte>",
    "contacted_ephIDs": ["H(ephID_i||i)", ...],
    "ZKP_proofs": [ {"a":..., "z":...}, ... ]
  }
  ```

---

## Use Case: Building the Proximity Graph

1. **Step 1**: A is marked infected; edges added to B, C, D upon verified contact.
2. **Step 2**: B becomes infected; edges from B propagate to its contacts.
3. **Step 3 & 4**: New contacts among mapped users are added over time.

---

## Edge Labels

- **Contact Duration**: Estimated from number of epochs matched.
- **Risk Factor**: Local smartphone computation (distance, duration).
- **Contact Date**: Timestamp when BLE encounter occurred.

---

## Security & Privacy Analysis

- **Confidentiality**: User identities remain hidden; graph edges do not prove real-world meetings.
- **Integrity**: ZKP prevents false contact claims and limits gossip/fake-infection attacks.
- **Mitigations**:
  - **Gossip Attack**: ZKP reduces fake endorsements.
  - **Fake Infection Spread**: Unique secrets per contact epoch.
  - **Relay Attack**: Possible GPS encryption and MAC in BLE messages (future work).

---

## Implementation Highlights

- **Client-Server Interaction**: A Python script simulates the client role, exchanging HTTPS packets to execute the Schnorr non-interactive ZKP protocol.
- **Backend Server**: Implemented in Python using the Django framework and served via Apache2 with mod_wsgi to handle secure HTTPS requests.
- **Group Utilities Library**: group_utilities.py provides an object-oriented interface for both Prover and Verifier roles, simplifying ZKP integration.
- **HTTPS Configuration**: A custom Certificate Authority (CA) and server certificate were generated and installed to establish trusted TLS connections between clients and the server.
- **Cryptographic Parameters**: The protocol runs over the 1024-bit MODP Group with 160-bit Prime Order Subgroup as defined in RFC5114.
- **Technologies**: Python, Django, Apache2 with WSGI, TLS certificates.
- **ZKP Module**: Implements Schnorr non-interactive proofs using 1024-bit MODP (RFC5114 group).
- **Repositories**:
  - `zero_proof_knowledge/`: Django app for ZKP logic.
  - `client_scripts`: Simulate prover (client) and verifier (server).

---

## Conclusion

This proximity graph enhances DP-3T by adding verifiable contact authentication, enabling accurate, privacy-preserving epidemiological studies without exposing user identities.
