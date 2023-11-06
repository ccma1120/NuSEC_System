```mermaid
sequenceDiagram
    autonumber
    participant c as Cloud Server
    participant d as Desktop PC
    participant a as Adapter (NuLink3)
    participant t as Target Board (MCU UID_A)
    a ->> a : Trigger
    a ->> t : Program KeyGen snippet code to SRAM_TEMP via debug interface 
    Note right of a: KeyGen code snippet 
    Note over t: [SRAM_TEMP] KeyGen code snippet 
    t ->> t : Run KeyGen code snippet 
    t ->> t : Generate authentication private key in keystore
    Note over t: [KEY_STORE] AUTH_PRI key
    t ->> t : Generate authentication public key using private key 
    Note over t: [SRAM_TEMP] AUTH_PUB key
    t ->> a : Send Public key (AUTH_PUB) + UID (Unique ID of MCU)
    a ->> t : Send CertificationRequestInfo    
    t ->> t : Generate Signature using private key
    t ->> a : Send Signature
    a ->> a : Generate CSR
    Note over a: Certificate Signing Request(CSR)
    a ->> c : Send CSR to REST API server, using WIFI interface and HTTPS protocol
    c ->> a : Create device certificate
    Note right of c: Device Certificate
    a ->> t : Provision device certificate into OTP memory 
    Note right of a: Device Certificate
    Note over t: [OTP_MEMORY] DEV_CERT
```
