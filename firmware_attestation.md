```mermaid
sequenceDiagram
    autonumber
    participant c as Cloud Server
    participant d as Desktop PC
    participant a as Adapter (Debugger)
    participant t as Target Board (MCU UID_A)
    t ->> a : Get (SWD)
    Note right of a: UID_A
    a ->> c : Send
    Note left of a: UID_A
    c ->> c : Generate key pair for UID_A
    Note over c : SecureBoot_PRI_A & SecureBoot_PUB_A
    c ->> c : Sign BootLoader_FW with SecureBoot_PRI_A
    Note over c : Signature_BootLoader
    c ->> a : Send
    Note right of c : Signature_BootLoader
    c ->> a : Send
    Note right of c : SecureBoot_PUB_A
    c ->> a : Send
    Note right of c : APP_FW & BootLoader_FW
    a ->> t : Program (debugger interface)
    Note right of a : BootLoader_FW (with Signature_BootLoader)
    a ->> t : Program (debugger interface)
    Note right of a : SecureBoot_PUB_A
    c ->> c : Generate key pair for UID_A
    Note over c : SecureBoot_PRI_A_APP & SecureBoot_PUB_A_APP
    c ->> c : Sign APP_FW with SecureBoot_PRI_A_APP
    Note over c : Signature_APP
    c ->> a : Send
    Note right of c : Signature_APP
    c ->> a : Send
    Note right of c : SecureBoot_PUB_A_APP
    a ->> t : Issue secure lock & chip reset (debugger interface, after lock debugger interface, communicate with serial port)
    t ->> t : Boot from SecureBoot 
    
    %% ECDH
    t ->> t: Generate private key for <br/> ECC key to KeyStore
    Note over t: private key A
    t ->> t: Generate using private key
    Note over t: public key A
    t ->> a: Send
    Note left of t: public key A
    a ->> a: Generate private key for <br/> ECC key to KeyStore
    Note over a: private key B
    a ->> a: Generate using private key
    Note over a: public key B
    a ->> t: Send
    Note right of a: public key B
    t ->> t: Generate shared AES key using ECDH
    Note over t: AES_SESSION key
    a ->> a: Generate shared AES key using ECDH
    Note over a: AES_SESSION key

    a ->> a: Attach signature_APP & SecureBoot_PUB_A_APP to APP_FW
    Note over a: APP_FW
    a ->> a: Encrypt with AES_SESSION key
    Note over a: APP_FW
    a ->> t: Send
    Note right of a: APP_FW
    t ->> t: Decrypt with AES_SESSION key
    Note over t: APP_FW
    t ->> t: Program to APP address
    Note over t: APP_FW
```
