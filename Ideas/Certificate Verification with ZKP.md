
**Thesis Outline: ZK-Certs - Privacy-Preserving On-Chain Verification of IPFS-Linked Academic Credentials**

**Core Objective:** To design and demonstrate a system where anyone can verify specific claims about a university certificate (stored on IPFS) using Zero-Knowledge Proofs, without revealing the entire certificate content, thus preserving student privacy.

**Entities:**

*   **University (Issuer):** Generates certificates, signs metadata, and registers records on-chain.
*   **Student (Prover/Holder):** Owns the certificate, generates ZKPs for specific claims.
*   **Blockchain (Smart Contracts):** Stores certificate commitments, issuer public keys, and ZK Verifier contracts.
*   **IPFS:** Stores the actual certificate PDF document.
*   **Verifier (e.g., Employer):** Requests proof of specific claims and verifies the ZKP.

---

**Step 1: University Issues Certificate and Registers On-Chain Commitment**

*   **Action by University:**
    1.  **Certificate Generation:**
        *   The University (e.g., "BRAC") generates an academic certificate for a student (e.g., "Abdul Ahad") in PDF format.
        *   **Certificate Content (Example):** Student Name ("Abdul Ahad"), Student ID ("S12345"), Degree ("Bachelor of Science in Computer Science"), GPA (3.8), Issue Date ("2025-05-28"), University Name ("BRAC").
        *   *Data:* `abdul_ahad_brac_bscs_cert.pdf`
    2.  **Metadata Preparation & Hashing:**
        *   The University extracts key metadata from the certificate into a structured format (e.g., JSON).
            ```json
            certificate_metadata = {
              "student_name": "Abdul Ahad",
              "student_id": "S12345",
              "degree": "Bachelor of Science in Computer Science",
              "gpa": 3.8,
              "issue_date": "2025-05-28",
              "university_name": "BRAC"
            }
            ```
        *   This `certificate_metadata` is hashed (e.g., SHA-256) to create a unique commitment.
            *   `metadata_hash = SHA256(canonical_json_string(certificate_metadata))`
            *   *Example:* `metadata_hash = 0xabc123def456...`
    3.  **Digital Signature by University:**
        *   The University signs this `metadata_hash` using its private key.
            *   `issuer_signature = Sign(metadata_hash, university_private_key)`
            *   *Example:* `issuer_signature = 0x1f2e3d4c...`
        *   *Note:* The signature could also be over the full metadata itself, rather than just its hash, depending on ZKP circuit design. Signing the hash is common for brevity.
    4.  **Upload Certificate PDF to IPFS:**
        *   The `abdul_ahad_brac_bscs_cert.pdf` is uploaded to IPFS.
        *   IPFS returns a unique Content Identifier (CID).
            *   *Example:* `ipfs_cid = QmZkCert123abc...`
    5.  **On-Chain Registration (via Smart Contract):**
        *   The University (or an authorized administrator) interacts with a "CertificateRegistry" smart contract on the blockchain.
        *   The University's public key (or an identifier for it) must be registered/known by the smart contract system.
        *   The following information is submitted to the smart contract transaction:
            *   `student_address` (Optional, depends on system design; could be student's public key or DID if using SSI principles). For this ZKP model, it might not be directly stored if privacy is paramount, or it could be an anonymized identifier. Let's assume it's not stored directly on this record for now to maximize privacy for the ZKP use case.
            *   `issuer_address` (The University's blockchain address, linked to its public key).
            *   `ipfs_cid` (The CID of the certificate PDF on IPFS).
            *   `metadata_hash` (The hash of the certificate metadata).
            *   `issuer_signature` (The university's signature over the `metadata_hash`).
            *   `revocation_status` (Initially "false" or "valid").
        *   The smart contract stores this record, making `metadata_hash`, `ipfs_cid`, `issuer_signature`, and `issuer_address` publicly queryable.

*   **Rationale:** This step establishes an immutable, verifiable link between the issuer, the certificate's content (via its hash and IPFS CID), and the issuer's endorsement. The `metadata_hash` acts as the public commitment against which ZKPs will be verified.

---

**Step 2: Student Generates Zero-Knowledge Proof for Specific Claims**

*   **Scenario:** Abdul Ahad wants to prove to an Employer that he has a "Bachelor of Science in Computer Science" degree from "BRAC" with a GPA > 3.5, *without revealing his name, student ID, or exact GPA (only that it's > 3.5).*

*   **Action by Student (Prover) using a ZK-Proving Application (off-chain):**
    1.  **Input Gathering:**
        *   **Private Inputs (known only to the student):**
            *   The full `certificate_metadata` (name, ID, degree, GPA, date, university).
            *   The `issuer_signature` (provided by the university, likely retrieved from the on-chain record or directly from the university during issuance).
            *   The University's public key (needed to verify the signature inside the ZKP circuit).
        *   **Public Inputs (known to student and verifier, will be part of the proof and sent to the on-chain verifier contract):**
            *   The `metadata_hash` (as stored on the blockchain).
            *   The `ipfs_cid` (as stored on the blockchain – optional as a public input to the ZK verifier contract, but good for context).
            *   The `issuer_address` (University's address, to fetch its public key or confirm identity).
            *   The specific claim values, e.g.:
                *   `target_degree = "Bachelor of Science in Computer Science"`
                *   `target_university = "BRAC"`
                *   `gpa_threshold = 3.5`
    2.  **ZK Circuit Execution (e.g., using Circom/SnarkJS):**
        *   The student's application feeds these inputs into a pre-defined ZK-SNARK circuit. This circuit is designed to perform the following checks internally without revealing the private inputs:
            *   **Metadata Integrity:** Re-calculates the hash of the provided private `certificate_metadata` and asserts that it matches the public `metadata_hash`. (Ensures the student is using the correct, unaltered metadata corresponding to the on-chain commitment).
            *   **Issuer Signature Verification:** Verifies the `issuer_signature` against the re-calculated hash (or the full metadata) using the University's public key (derived from `issuer_address` or provided as a public input). (Ensures the metadata was indeed signed by the claimed university).
            *   **Specific Claim Verification:**
                *   Asserts that the `degree` field in the private `certificate_metadata` equals `target_degree`.
                *   Asserts that the `university_name` field in the private `certificate_metadata` equals `target_university`.
                *   Asserts that the `gpa` field in the private `certificate_metadata` is greater than `gpa_threshold`.
        *   If all assertions pass, the ZK proving system generates:
            *   A **Zero-Knowledge Proof** (`proof.json`).
            *   The **Public Inputs** used for this proof generation (`public.json`, which includes `metadata_hash`, `target_degree`, `target_university`, `gpa_threshold`, `issuer_address`).
    3.  **Proof Delivery:** The student provides the `proof.json` and `public.json` to the Employer (Verifier).

*   **Rationale:** The student can generate cryptographic proof for specific claims without disclosing any other information from their certificate. The ZKP guarantees that the claims are true for a certificate that is genuinely issued by the university and corresponds to an on-chain record.

---

**Step 3: Employer (Verifier) Verifies the Zero-Knowledge Proof On-Chain**

*   **Action by Employer (Verifier) using a DApp or directly interacting with a smart contract:**
    1.  **Receive Proof from Student:** The Employer receives the ZK `proof` and the `public_inputs` (which include `metadata_hash`, `issuer_address`, `target_degree`, `target_university`, `gpa_threshold`) from Abdul Ahad.
    2.  **Query On-Chain Certificate Record (Optional but Recommended Context Check):**
        *   The Employer can use the `metadata_hash` (or `ipfs_cid` if also provided as a distinct public input for lookup) from the `public_inputs` to query the "CertificateRegistry" smart contract.
        *   This allows the Employer to confirm:
            *   That a record with this `metadata_hash` actually exists on-chain.
            *   That it is associated with the claimed `issuer_address` (BRAC).
            *   That the `revocation_status` is not "revoked."
            *   The `ipfs_cid` associated with this record (allowing them to later fetch the PDF if they wish, *after* successful ZKP verification and if the student consents or the policy allows).
    3.  **Submit Proof to On-Chain ZK Verifier Contract:**
        *   The Employer (or their DApp) calls a specific function on a pre-deployed "ZKVerifier" smart contract (generated from the ZK circuit, e.g., by SnarkJS `exportSolidityVerifier`).
        *   The inputs to this smart contract function are the ZK `proof` and the `public_inputs` received from the student.
    4.  **Smart Contract Verification Logic:**
        *   The "ZKVerifier" smart contract executes its verification algorithm using the provided `proof` and `public_inputs`.
        *   This on-chain verification is computationally efficient (a key property of zk-SNARKs – verification is much faster than proof generation).
        *   The contract returns `true` if the proof is valid for the given public inputs, `false` otherwise.

*   **Rationale:** The verification is trustless. The Employer doesn't need to trust the student or even have direct access to the certificate PDF initially. They trust the mathematics of the ZKP and the immutability of the ZK Verifier smart contract on the blockchain.

---

**Step 4: Employer Makes a Decision**

*   **Action by Employer:**
    1.  **If On-Chain Verification Returns `true`:**
        *   The Employer is cryptographically assured that:
            *   A certificate exists whose metadata hashes to the `metadata_hash` on-chain.
            *   This metadata was signed by the claimed `issuer_address` (BRAC)            *   The specific claims (degree is "Bachelor of Science in Computer Science," university is "BRAC," GPA > 3.5) are true for that certificate.
        *   The Employer can accept this as valid proof for these specific facts.
        *   They *still do not know* Abdul Ahad's name, student ID, or exact GPA from this ZKP interaction.
    2.  **If On-Chain Verification Returns `false`:**
        *   The proof is invalid, or the claims are not true for the committed certificate. The Employer rejects the claim.
    3.  **(Optional) Fetching the Full Certificate from IPFS:**
        *   If the ZKP is valid and the Employer has a legitimate need (and potentially student consent for this further step), they can use the `ipfs_cid` (obtained from the public inputs of the ZKP or by querying the on-chain record using the `metadata_hash`) to download the actual certificate PDF from IPFS for visual inspection or record-keeping. This step is separate from the ZKP verification of specific facts.

*   **Rationale:** The system allows for layered disclosure. ZKPs verify specific facts privately. Access to the full document on IPFS can be a subsequent, distinct step if needed and permitted.

---

**Summary of Benefits Demonstrated:**

*   **Privacy Preservation:** Students can prove specific attributes of their certificates without revealing all details.
*   **Data Minimization for Verifier:** Verifiers only get confirmation of the exact facts they need.
*   **Trustless Verification:** Relies on cryptography and decentralized smart contracts, not on trusting the presenter directly.
*   **Issuer Control:** Universities maintain control over issuance and the original authoritative record on-chain.
*   **Data Integrity:** IPFS CIDs and on-chain hashes ensure the underlying certificate data is not tampered with.
*   **Decentralization:** Leverages blockchain and IPFS for resilience and reduced reliance on central points of failure for verification.
