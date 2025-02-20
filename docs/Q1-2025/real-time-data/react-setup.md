# React Set Up


## Coding

-  **Direct WebSocket Integration**: WebSockets enable real-time communication in React applications without requiring additional dependencies.
-  **Lifecycle Management**: WebSocket connections should be managed in sync with the React component lifecycle for optimal performance and resource efficiency.
-  **Using `useEffect` for WebSocket Handling**:
    - Open the WebSocket connection when the component mounts.
    - Close the connection and clean up event handlers when the component unmounts to prevent memory leaks.
- **Preventing Duplicate Messages**: Nullify the `onmessage` handler when unmounting to avoid sending messages after the component is destroyed.
- **Error Handling**: Implement `onerror` to detect and respond to network issues or WebSocket failures, ensuring robust application behavior.
- **Automatic Reconnection**: Implement a reconnection mechanism to maintain a stable connection, particularly in environments with unreliable network conditions.

```ts
'use client';
import { useEffect, useState } from 'react';

export default function App() {
  const [data, setData] = useState('');
  const [stateSocket, setStateSocket] = useState<WebSocket | null>(null);

  const connect = () => {
    if (stateSocket) {
      return;
    }
    const socket = new WebSocket('ws://localhost:8080');
    socket.onopen = () => {
      console.log('WebSocket Connected');
    };
    socket.onmessage = (event) => {
      const data = JSON.parse(event.data);
      if (data.type === 'chat') {
        setData(data.payload.message);
      }
    };
    socket.onclose = () => {
      console.log('WebSocket Disconnected');
    };
    socket.onerror = (error) => {
      console.error('WebSocket Error:', error);
    };
    setStateSocket(socket);
  };

  const disconnect = () => {
    if (stateSocket) {
      stateSocket.close();
      stateSocket.onmessage = null;
      setStateSocket(null);
    }
  };

  useEffect(() => {
    connect();
    return () => {
      disconnect();
    };
  }, []);

  const handleSendMessage = () => {
    if (stateSocket) {
      stateSocket.send(
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

## Testing

**Testing WebSockets in React** requires simulating real-time communication, ensuring connections, message handling, and cleanup. [`jest-websocket-mock`](https://www.npmjs.com/package/jest-websocket-mock) provides a **mock WebSocket server**, allowing controlled testing of WebSocket interactions in Jest.

### WebSocket Connection & Lifecycle Management

-   **Check initial state** before WebSocket connection is established.
-   **Verify connection establishment** after rendering the component.
-   **Ensure the WebSocket URL is correct** (`ws://localhost:8080`).
-   **Handle WebSocket closure properly** when the component unmounts.

### Handling WebSocket Messages

-   **Simulate receiving a valid message** and check if the component updates accordingly.
-   **Verify correct parsing and processing of received messages** (e.g., `type: 'chat'`).
-   **Ensure the UI reflects new messages in real-time.**
-   **Check how invalid or unexpected messages are handled** (e.g., missing required fields).

### Error Handling & Resilience

-   **Simulate a WebSocket error (`server.error()`)** and verify how the app handles it.
-   **Ensure the UI displays an appropriate error message** or reacts accordingly.
-   **Verify reconnection logic (if implemented)** when the WebSocket unexpectedly closes.

### WebSocket Disconnection Handling

-   **Simulate WebSocket closure (`server.close()`)** and verify cleanup behavior.
-   **Ensure that event listeners are properly removed** to prevent memory leaks.
-   **Confirm the UI reflects the disconnected state** and updates accordingly.

### Additional Considerations

-   **Ensure WebSocket reopens after a reconnection attempt** (if auto-reconnect is implemented).
-   **Check that multiple WebSocket instances are not created** when re-rendering the component.
-   **Test WebSocket interactions in different scenarios**, like switching between online and offline states.
-   **Verify outgoing messages** sent to the server via `WebSocket.send()`.

```ts
import React from 'react';
import {
  render,
  act,
  waitFor,
} from '@testing-library/react';
import { WebSocketComponent } from './WebSocketComponent';
import WS from 'jest-websocket-mock';

describe('WebSocket Demo App', () => {
  let server: WS;

  beforeEach(async () => {
    server = new WS('ws://localhost:8080');
  });

  afterEach(() => {
    WS.clean();
  });

  it('should connect and disconnect from the WebSocket server', async () => {
    render(<WebSocketComponent />);
    const messageSpy = jest.spyOn(server, "send");

    // add here initial expectations //

    await act(async () => {
      await server.connected;
    });

    await waitFor(() => {
      // on connect expectations //
    });

    await act(async () => {
      server.send(
        JSON.stringify({
          type: 'chat',
          payload: {}, 
        }),
      );
    });

    await waitFor(() => {
      expect(messageSpy).toHaveBeenCalledTimes(1);
      // on specific server message expectations //
    })

    act(() => {
      server.error();
    });

    await waitFor(() => {
      // on specific error expectations //
    });

    act(() => {
      server.close();
    });

    await waitFor(() => {
      // on specific close expectations //
    });
  });

});
```