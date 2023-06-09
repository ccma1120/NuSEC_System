### Prepare NuLink3 programmer for CM (by OEM)

```mermaid
sequenceDiagram
    autonumber
	participant o as Cloud Server (OEM)
	participant n as Desktop PC (NuSec.py)
	participant t as Adapter (Programmer Temp)
	participant p as Target Board (Programmer CM)
    o ->> o: Generate private key for <br/>RootCA, ICA and Webserver in HSM
    Note over o: Private key in HSM
    o ->> o: Generate certificate for <br/> RootCA, ICA and Webserver
    Note over o: Certificate of RootCA, ICA and Webserver
    o ->> n: Send
    Note right of o: WebServer Certificate
    n ->> p: Program to flash (SWD)
    Note right of t: WebServer Certificate
    n ->> p: Program to SRAM (SWD)
    Note right of t: DevAuth.bin
    p ->> p: Run
    Note over p: DevAuth.bin
    p ->> n: Talk with host
    p ->> p: Generate private key for <br/>device authentication to KeyStore
    Note over p: AUTHKEY_PRI
    p ->> p: Generate using AUTHKEY_PRI
    Note over p: AUTHKEY_PUB
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
    n ->> o: Ask EJBCA server to generate<br/>private key for Programmer CM
    Note left of n: SECURE_BOOT_PRI_NL3
    o ->> o: Generate using SECURE_BOOT_PRI_NL3
    Note over o: ROTPK
    o ->> n: Get
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
    p ->> p: Enable secure lock
    p ->> p: Generate private key for <br/> AES key to KeyStore
    Note over p: private key
    p ->> p: Generate using private key
    Note over p: public key
    o ->> o: Generate private key for <br/> AES key to HSM
    Note over o: private key
    o ->> o: Generate using private key
    Note over o: public key
    o ->> p: Send
    Note right of o: public key
    p ->> o: Send
    Note left of p: public key
    p ->> p: Generate shared AES key using ECDH
    Note over p: AES_NuLink3
    o ->> o: Generate shared AES key using ECDH
    Note over o: AES_NuLink3
```