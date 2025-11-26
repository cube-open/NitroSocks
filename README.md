# NitroSocks
A protocol

```mermaid
sequenceDiagram
  Client->>Server: TLS HandShake
  Server->>Client:
  Server->>Client:NS Challenge[Pubkey(TokenTemp)]
  Client->>Server:NS Response[TokenTemp(Token)]
  Server->>Client:NS Result[Pubkey(Token(SessionKey))]
  Client-->Server:
  Client->>Server: NS Payload[SessionKey(Payload)]
  Server->>Client:...
  Server-->Client:
  Client->>Server:NS ReleaseNotice[Empty]
  Server->>Client:NS Confirm[SessionKey(RandomString)]
  Client->>Server:NS Confirm[RandomString(SessionKey)]
  Client->>Server:TLS/TCP Release
  Server->>Client:
  ```
