# NitroSocks

A Layer 7 tunnel protocol designed to provide high-security, anti-Man-in-the-Middle (MITM) attacks, and end-to-end verification over untrusted TLS environments based on TCP-TLS, making interception by middlemen ineffective or extremely difficult to crack. It enforces `AES-128-GCM/AES-256-GCM` encrypted packets to provide post-quantum security within observable future.

## Protocol Flow

For explanatory purposes, we define `abc[]` as abc payload content, `abc()` as encryption using abc, and `abc<>` as decryption using `abc`. The `Man-in-the-Middle` referred to here represents a potential TLS hijacking node that may exist in practice, **NOT** a normal network role in the connection.

```mermaid
sequenceDiagram
    participant C as Client
    participant M as Man-in-the-Middle (MITM)
    participant S as Server

    Note over C,S: Phase 1: Establish TLS Connection
    C->>M: Establish TLS Connection
    M->>S: Forward TLS Connection
    
    Note over C,S: Phase 2: Server-initiated Challenge
    C-->>M: An invalid packet, optional, for protocol obfuscation
    S->>M: Server challenge packet [PublicKey('R1.TokenTemp.encryptType')]
    M->>C: Forward (unrecognized as protocol)
    
    Note over C,S: Phase 3: Client Decryption and Response
    C->>C: PrivateKey<Challenge Packet><br>Obtain R1 and TokenTemp
    C->>M: Challenge response packet [TokenTemp(Client Signature[Sha256(Sha256(Token, Using R1 as Salt) + TOTP(Token)))]] 
    M->>S: Forward (unintelligible content)
    
    Note over C,S: Phase 4: Server Verification and Session Establishment
    S->>S: TokenTemp<Challenge Response><br>Verify Token Hash and TOTP<br>Generate SessionKey
    S->>M: Session Token Distribution [TokenTemp(SessionKey)]
    M->>C: Forward (unintelligible content)
    
    Note over C,S: Phase 5: Covert Encrypted Communication
    C->>M: Encrypted Data [SessionKey(Application Data)]
    M->>S: Forward
    S->>M: Encrypted Data [SessionKey(Response Data)]
    M->>C: Forward

    Note over C,S: Phase 5.1: Periodic SessionKey Refresh
    loop: SessionKey Refresh
    S->>S: Generate New SessionKey
    S->>M: SessionKey Refresh Packet [SessionKeyOLD(NewSessionKey)]
    M->>C: Forward
    End

    Note over C,S: Phase 5.2: Disconnection Reconnection
    C->>M: Reconnection Packet [SessionKey(Client Signature)]
    M->>S: Forward
    S->>S: Verify Signature
    S->>M: Connection Recovery Packet [SessionKeyOLD(NewSessionKey)]
    M->>C: Forward
    C->>C: Restore Tunnel Connection

    Note over C,S: Phase 6: Normal Connection Release
    C->>M: Disconnect Packet [SessionKey(Empty Packet)]
    M->>S: Forward
    S->>M: Disconnect Acknowledgement Packet [SessionKey(R2)]
    M->>C: Forward
    C->>M: Response Confirmation [SessionKey(Sha256(R2))]
    M->>S: Forward
    S->>S: Disconnect
    C->>C: Disconnection Successful

    Note over C,S: Phase 6.1: Server Exception Release
    C->>M: Disconnect Notification Packet [SessionKey(Empty Packet)] (When server does not respond, notify server to release resources regardless of delivery)
    M->>S: Forward
    
    Note over C,S: Phase 6.2: Client Exception Release
    S->>M: Disconnect Notification Packet [SessionKey(Empty Packet)] (When client does not respond, notify client to release resources regardless of delivery)
    M->>C: Forward


    Note over C,S: Phase 7: TLS Connection Disconnection
    C->>M: TLS Connection Normal Disconnection
    M->>S: TLS Connection Normal Disconnection
```