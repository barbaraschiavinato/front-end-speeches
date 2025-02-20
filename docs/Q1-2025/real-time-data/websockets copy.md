# WebSockets

## What they are and how they differ from HTTP

WebSockets provide a full-duplex communication channel that operates over a single, long-lived connection between a client and a server. 
Unlike HTTP, which is a request/response protocol where the client must initiate all requests, WebSockets allow for two-way communication that can be initiated by either the client or the server. 
This enables data to be sent to a client without the client having to request it, effectively reducing latency and the overhead associated with traditional HTTP requests that open and close connections repeatedly.

### Chat example using Websocket:
``` mermaid
sequenceDiagram
    participant UI as React Component
    participant WS as WebSocket Server
    participant Backend as Backend Service (Optional)

    UI->>WS: Open WebSocket Connection (ws://server)
    WS-->>UI: Connection Established

    UI->>WS: Send Chat Message (JSON Data)
    WS-->>Backend: Forward Message to Backend (if needed)
    Backend-->>WS: Process Message & Store (if needed)
    WS-->>UI: Distribute Message to Other Users

    UI->>UI: Display New Message in Chat Window

    UI->>WS: Close WebSocket (On Unmount)
    WS-->>UI: Acknowledgement (Connection Closed)
```

### Chat example using HTTP:
``` mermaid
sequenceDiagram
    participant UI as React Component
    participant Server as HTTP Server
    participant Backend as Backend Service (Optional)

    UI->>Server: Send HTTP POST Request (Send Message)
    Server-->>Backend: Process Message
    Backend-->>Server: Store Message (if needed)
    Server-->>UI: HTTP Response (Message Sent Confirmation)

    UI->>Server: HTTP GET Request (Poll for New Messages)
    Server->>Backend: Check for New Messages
    Backend-->>Server: Retrieve New Messages
    Server-->>UI: HTTP Response (New Messages)

    UI->>UI: Update Chat Window with New Messages

    note over UI,Server: Polling occurs at regular intervals
```


## Benefits of using webSockets

The use of WebSockets offers significant advantages for real-time applications:

- **Lower Latency**: WebSockets reduce the time it takes to communicate between the client and server, as the need for establishing a connection on each interaction is eliminated.
- **Reduced Bandwidth Usage**: By keeping the connection open, WebSockets eliminate the HTTP headers that need to be sent with each request, which significantly reduces the amount of data transferred.
- **Real-Time Interaction**: WebSockets are ideal for scenarios where quick interaction and data updates are crucial, such as in gaming, live sports updates, or financial trading applications.
- **Bi-directional Communication**: Both the server and the client can send data at any time, which is essential for dynamic and interactive web applications.

## Example scenarios where webSockets are preferable

1. **Online Games**: Multiplayer online games benefit greatly from WebSockets for real-time player interaction and game state updates.
2. **Chat Applications**: Instant messaging apps use WebSockets to deliver messages immediately without any noticeable delay.
3. **Financial Trading Platforms**: Stock or forex trading platforms use WebSockets to stream market data in real time, enabling traders to see price updates immediately and execute trades without delay.
4. **Live Notifications**: Web applications that require sending real-time alerts or notifications to users (like social media platforms) rely on WebSockets to push updates efficiently as soon as new content is available.

