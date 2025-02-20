# WebSocket API
=============

What It Is
----------

-   A low-level native browser API for real-time communication.
-   Works over TCP and provides a persistent connection between client and server.
-   Available in modern browsers via the WebSocket object.

Pros
----

-   Lightweight and Efficient: No extra overhead like HTTP polling.
-   Fast: Direct connection with low latency.
-   Standardized: Fully supported in modern browsers and environments.

Cons
----

-   No Built-in Reconnection Handling: If the connection drops, you must manually reconnect.
-   No Fallbacks: If WebSockets are blocked (e.g., by firewalls or proxies), you must handle fallback strategies yourself.
-   Lacks Advanced Features: No built-in event system, rooms, or broadcasting.

When to Use
-----------

Best for low-latency, bi-directional communication like chat apps, multiplayer games, financial data streaming, and IoT.

Testing
-------

-   Use `jest.fn()` to mock `WebSocket` events.
-   Simulate messages with `onmessage`, `onopen`, and `onclose`.
-   The library `jest-websocket-mock` helps to mock events.

### Challenges:

-   Requires manual event simulation.
-   Hard to test reconnection scenarios.

Code example:
-------------

```ts
import { useEffect, useState } from 'react';

export default function App() {
  const [data, setData] = useState('');
  const [memSocket, setMemSocket] = useState<WebSocket | null>(null);

  useEffect(() => {
    const socket = new WebSocket("ws://localhost:8080");
    socket.onmessage = (event) => setData(event.data);
    setMemSocket(socket);
    return () => {
      socket.close();
      socket.onmessage = null;
      setMemSocket(null);
    };
  }, []);

  const handleSendMessage = () => {
    if (memSocket) {
      memSocket.send(
        JSON.stringify({
          type: 'chat',
          payload: { message: 'message', username: 'user' },
        }),
      );
    }
  };

  return (
    <h1>
      Data: {data}
      <button onClick={handleSendMessage}>Send</button>
    </h1>
  );
}
```

