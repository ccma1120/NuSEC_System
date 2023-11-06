### Prepare NuLink3 programmer for CM (by OEM)

```mermaid
sequenceDiagram
    autonumber
	participant o as Cloud Server (OEM)
	participant n as Desktop PC (NuSec.py)
	participant t as Adapter (Programmer Temp)
	participant p as Target Board (Programmer CM)
    o ->> o: Generate private key for <br/>RootCA, ICA in HSM
    Note over o: [HSM] ROOTCA_PRI, ICA_PRI
    o ->> o: Generate certificate for <br/> RootCA, ICA 
    Note over o: [HSM] RootCA_CERT, ICA_CERT
    o ->> n: Send RootCA/ICA Certificate
    n ->> p: Program RootCA/ICA Certificate to flash (SWD)
    Note over p: [OTP_MEMORY] RootCA/ICA Certificate
    n ->> p: Program KeyGen snippet code to SRAM (SWD)
    Note over p: [SRAM] KeyGen snippet code

    Rect rgb(220, 0, 0)
    p ->> p: Run KeyGen snippet code
    p ->> p: Generate private key for <br/>NuLink3 authentication to KeyStore
    Note over p: [KEY_STORE] AUTH_PRI_NuLink3
    p ->> p: Generate public key pair
    Note over p: [KEY_STORE] AUTH_PUB_NuLink3
    p ->> n: Send AUTH_PUB_NuLink3
    n ->> p: Send CertificationRequestInfo
    p ->> p: Sign CertificationRequestInfo by AUTH_PRI_NuLink3
    p ->> n: Send Signature
    n ->> n: Generate Certificate Signing Request(CSR) <br/>by AUTH_PUB_NuLink3, Signature, UID
    n ->> o: Request certificate of NuLink3 by CSR
    o ->> n: Create NuLink3 Certificate
    n ->> p: Program NuLink3 Certificate
    Note over p: [OTP_MEMORY] NuLink3 Certificate
    end

    Rect rgb(220, 0, 0)
    n ->> o: Ask REST API server to generate<br/>secure boot key pair for NuLink3 <br/>(Secure programmer provided by CM) 
    o ->> o: Generate secure boot key pair for NuLink3
    Note over o: [HSM] SecureBoot_PRI_NuLink3 (ROTPK)
    o ->> n: Send SecureBoot_PUB_NuLink3
    n ->> p: Burn SecureBoot_PUB_NuLink3
    Note over p: [OTP_MEMORY] SecureBoot_PUB_NuLink3
    n ->> o: Send NuLink3 firmware
    o ->> o: Sign it with SecureBoot_PRI_NuLink3
    o ->> n: Send NuLink3 firmware (signed)
    n ->> p: Burn NuLink3 firmware (signed)
    p ->> p: Enable secure lock of NuLink3 (Restricting debugger access)
    Note over p: [Flash (protected)] NuLink3 firmware (signed)
    end
    
    %%ECDH
    Rect rgb(220, 0, 0)
    p ->> p: Generate ephemeral key pair for <br/> ECDH to get an symmetric AES key
    Note over p: [KEY_STORE SRAM] private/public key pair
    p ->> o: Send public key
    o ->> o: Generate ephemeral key pair for <br/> ECDH to get an symmetric AES key
    Note over o: [HSM TEMP] private/public key pair
    o ->> p: Send public key
    p ->> p: Generate shared AES key using ECDH
    Note over p: [KEY_STORE] AES_NuLink3
    o ->> o: Generate shared AES key using ECDH
    Note over o: [HSM] AES_NuLink3
    end

```
