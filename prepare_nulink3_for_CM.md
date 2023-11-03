![image](https://github.com/ccma1120/NuSEC_System/assets/25842853/cb31bb72-5725-43ca-a61e-5e10d980890f)### Prepare NuLink3 programmer for CM (by OEM)

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
    o ->> n: Send RootCA Certificate
    n ->> p: Program to flash (SWD)
    Note over p: [OTP memory] RootCA Certificate
    n ->> p: Program to SRAM (SWD)
    Note over p: [SRAM] DevAuth.bin
    p ->> p: Run DevAuth.bin

    Rect rgb(210, 233, 233)
    p ->> n: Talk with host
    p ->> p: Generate private key for <br/>device authentication to KeyStore
    Note over p: AUTHKEY_PRI
    p ->> p: Generate using AUTHKEY_PRI
    Note over p: AUTHKEY_PUB
    p ->> n: Send
    Note left of p: AUTHKEY_PUB
    n ->> p: Send
    Note right of n: CertificationRequestInfo
    p ->> p: Generate using AUTHKEY_PRI
    Note over p: Signature
    p ->> n: Send
    Note left of p: Signature
    n ->> n: Generate
    Note over n: Certificate Signing Request(CSR)
    n ->> o: Request
    Note left of n: Certificate Signing Request(CSR)
    o ->> n: Create
    Note right of o: NuLink3 Certificate
    n ->> p: Program
    Note right of n: NuLink3 Certificate
    end

    Rect rgb(227, 244, 244)
    n ->> o: Ask REST API server to generate<br/>private key for Programmer CM
    Note over o: SECURE_BOOT_PRI_NL3
    o ->> o: Generate using SECURE_BOOT_PRI_NL3
    Note over o: ROTPK
    o ->> n: Send
    Note right of o: ROTPK
    n ->> p: Burn
    Note right of n: ROTPK
    n ->> o: Send
    Note left of n: NuLink3 firmware
    o ->> o: Sign it with SECURE_BOOT_PRI_NL3
    Note over o: NuLink3 firmware (signed)
    o ->> n: Send
    Note right of o: NuLink3 firmware (signed)
    n ->> p: Burn
    Note right of n: NuLink3 firmware (signed)
    end

    p ->> p: Enable secure lock
    
    %%ECDH
    rect rgb(196, 223, 223)
    p ->> p: Generate private key for <br/> AES key to KeyStore
    Note over p: private key A
    p ->> p: Generate using private key
    Note over p: public key A
        p ->> o: Send
    Note left of p: public key A
    o ->> o: Generate private key for <br/> AES key to HSM
    Note over o: private key B
    o ->> o: Generate using private key
    Note over o: public key B
    o ->> p: Send
    Note right of o: public key B
    p ->> p: Generate shared AES key using ECDH
    Note over p: AES_NuLink3
    o ->> o: Generate shared AES key using ECDH
    Note over o: AES_NuLink3
    end

```
