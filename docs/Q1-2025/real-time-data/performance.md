# Performance Best Practices

## **Debounce/Throttle Messages**

WebSockets can send messages continuously, which can overload the app, especially if you're dealing with large amounts of real-time data. To prevent performance degradation:

### Debounce
Debouncing ensures that an action is only executed after a certain delay from the last time it was triggered. If the event keeps firing, the timer resets.

#### Use case
**Best for real-time updates:** Ensures data is processed at a steady interval, preventing overload while maintaining responsiveness. Ideal for live sports scores, stock market updates, GPS tracking, and multiplayer game state updates, where immediate data delivery is crucial.

```ts
import React, { useState, useEffect, useRef } from 'react';

const DebounceExample = () => {
  const [messages, setMessages] = useState<string[]>([]);
  const debounceTimeout = useRef<NodeJS.Timeout | null>(null);
  const socketRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    const socket = new WebSocket('ws://example.com/socket');

    socket.onopen = () => {
      console.log('WebSocket connected');
    };

    socket.onmessage = (event) => {
      const message = event.data;

      // Debounce logic: Reset timer and only execute after 500ms of inactivity
      if (debounceTimeout.current) {
        clearTimeout(debounceTimeout.current);
      }
      debounceTimeout.current = setTimeout(() => {
        setMessages((prevMessages) => [...prevMessages, `Debounced: ${message}`]);
      }, 500);
    };

    socketRef.current = socket;

    return () => {
      if (socketRef.current) {
        socketRef.current.close();
      }
    };
  }, []);

  return (
    <div>
      <h3>Debounced Messages:</h3>
      <ul>
        {messages.map((msg, idx) => (
          <li key={idx}>{msg}</li>
        ))}
      </ul>
    </div>
  );
};

export default DebounceExample;
```

#### Debounce scenario

```
Message 1 (0ms) - timer starts (500ms delay)
Message 2 (200ms) - timer resets (500ms delay)
Message 3 (400ms) - timer resets (500ms delay)
Message 4 (700ms) - timer resets (500ms delay)
Processed at 1200ms → "Debounced: Message 4"
```

### Throttle
Throttling ensures that an action only executes once every specified time interval, no matter how many times the event occurs.

#### Use case
**Best for delayed processing**: Waits until activity stops before executing the action, avoiding unnecessary updates. Ideal for live search, auto-save features, form validation, and filtering dashboards, where only the final input matters.

```ts
import React, { useState, useEffect, useRef } from 'react';

const ThrottleExample = () => {
  const [messages, setMessages] = useState<string[]>([]);
  const lastThrottledTime = useRef<number>(0);
  const socketRef = useRef<WebSocket | null>(null);

  useEffect(() => {
    const socket = new WebSocket('ws://example.com/socket');

    socket.onopen = () => {
      console.log('WebSocket connected');
    };

    socket.onmessage = (event) => {
      const message = event.data;
      const now = Date.now();

      // Throttle logic: Execute at most once per second
      if (now - lastThrottledTime.current >= 1000) {
        lastThrottledTime.current = now;
        setMessages((prevMessages) => [...prevMessages, `Throttled: ${message}`]);
      }
    };

    socketRef.current = socket;

    return () => {
      if (socketRef.current) {
        socketRef.current.close();
      }
    };
  }, []);

  return (
    <div>
      <h3>Throttled Messages:</h3>
      <ul>
        {messages.map((msg, idx) => (
          <li key={idx}>{msg}</li>
        ))}
      </ul>
    </div>
  );
};

export default ThrottleExample;

```

#### Throttle scenario

```
Message 1 (0ms) → "Throttled: Message 1"
Message 2 (200ms) → Ignored
Message 3 (400ms) → Ignored
Message 4 (800ms) → Ignored
Message 5 (1000ms) → "Throttled: Message 5"
```

## **Keep WebSocket Connections Efficient**

- **Reconnection Logic**: Implement automatic reconnects for when the WebSocket connection drops. Using libraries like `reconnecting-websocket` can help handle reconnections and backoff strategies.
- **Ping/Pong Mechanism**: To prevent timeouts and keep the connection alive, periodically send ping messages from the client, and handle pong responses from the server.

## **Optimize State Management**

- React’s state should be optimized to prevent unnecessary re-renders. Avoid updating the state for every single WebSocket message unless it is absolutely necessary. Use techniques like:
  - **React's `useState` and `useReducer`**: For managing and updating the state more efficiently.
  - **Memoization**: Use `React.memo()` or `useMemo()` to prevent unnecessary re-renders of components that don’t depend on the WebSocket data.
  - **Throttling State Updates**: Avoid updating the state too frequently by batching updates or using throttling techniques.

## **Efficient Data Handling**

- **Data Transformation**: If the WebSocket server is sending large objects, avoid unnecessary transformations on the client-side. Perform data transformations only when needed.
- **Data Compression**: Consider using compression (like `gzip` or `Brotli`) for the messages sent over WebSocket, especially if the data size is large.

## **Use WebSocket Subprotocols**

- WebSocket supports subprotocols that allow you to define custom protocols for your communication. This can be used to optimize data formats and ensure that only necessary data is transmitted.

## **Avoid Memory Leaks**

- Properly clean up WebSocket connections when components unmount or no longer need them:

  - Use `useEffect` to initialize and clean up WebSocket connections.
  - Close WebSocket connections in the `cleanup` function of `useEffect` to prevent memory leaks.

  Example:

  ```jsx
  useEffect(() => {
    const socket = new WebSocket("ws://your-websocket-url");

    socket.onmessage = (event) => {
      // Handle message
    };

    return () => {
      socket.close(); // Cleanup
    };
  }, []);
  ```

## **Batch Updates to the UI**

- For frequent real-time updates, batch updates together to avoid triggering too many re-renders. For instance, if WebSocket messages are related, you can batch them and update the state in a single call, reducing the number of re-renders.
- For example using useRef instead of useState prevents unnecessary re-renders by storing messages in a buffer and updating the state in batches, improving performance for high-frequency WebSocket updates.

## **Use Libraries for WebSocket Management**

- Consider using libraries like:
  - **`socket.io`**: For automatic reconnection, message buffering, and other advanced features.
  - **`reconnecting-websocket`**: For managing WebSocket connections with reconnection logic.
  - **`use-websocket`**: A React hook library for managing WebSocket connections and automatic clean-up.

## **Limit WebSocket Data Volume**

- If the WebSocket sends too much data, limit the frequency of messages and size of the payloads:
  - **On the server side**, limit the frequency of broadcasts.
  - **On the client side**, consider processing only the most relevant messages or applying filters.

## **Avoid Blocking the UI Thread**

- Keep the WebSocket-related code non-blocking. Avoid any heavy computation on the main thread. You can use `setTimeout`, `requestIdleCallback`, or workers to offload heavy tasks, ensuring the UI remains responsive.

## **Error Handling and Monitoring**

- Implement error handling for WebSocket failures and dropped connections. It's important to gracefully handle such errors to ensure a smooth user experience.
- Use `try/catch` for error management and implement logging to monitor WebSocket performance.

## **Use Web Workers for Intensive Tasks**

- JavaScript runs on a single thread, meaning heavy computations can block the UI and make the application unresponsive.
- If you're doing resource-heavy processing based on WebSocket messages, consider offloading such tasks to Web Workers to avoid blocking the UI.
- Web Workers are specifically meant for computational tasks that run in parallel with the main thread.

```js
import { useEffect, useState } from "react";

export default function App() {
  const [data, setData] = useState("");
  const [stateSocket, setStateSocket] = (useState < WebSocket) | (null > null);
  const [worker, setWorker] = (useState < Worker) | (null > null);

  useEffect(() => {
    const newWorker = new Worker(
      new URL("./websocketWorker.js", import.meta.url)
    );
    newWorker.onmessage = (event) => {
      setData(event.data); // UI updated with the Worker data
    };
    setWorker(newWorker);

    return () => {
      newWorker.terminate(); // Terminate Worker on unmount
    };
  }, []);

  const connect = () => {
    if (stateSocket) return;

    const socket = new WebSocket("ws://localhost:8080");

    socket.onopen = () => console.log("WebSocket Connected");

    socket.onmessage = (event) => {
      if (worker) worker.postMessage(event.data); // Send row data to worker
    };

    socket.onclose = () => console.log("WebSocket Disconnected");
    socket.onerror = (error) => console.error("WebSocket Error:", error);

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
  }, [worker]); // To be sure the Webworker is ready

  const handleSendMessage = () => {
    if (stateSocket) {
      stateSocket.send(
        JSON.stringify({
          type: "chat",
          payload: { message: "message", username: "user" },
        })
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
