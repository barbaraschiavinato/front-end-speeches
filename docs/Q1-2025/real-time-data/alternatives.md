# WebSocket Alternatives

## Socket.IO

### What It Is
-   A library built on top of WebSockets that provides extra features.
-   Uses fallback mechanisms (e.g., HTTP polling) when WebSockets are unavailable.
-   Works over TCP but starts with HTTP before upgrading.

### Pros
-   Automatic Reconnection: Handles dropped connections seamlessly.
-   Fallback Support: Uses HTTP long polling if WebSockets are blocked.
-   Management: Clients can join "rooms" for targeted message broadcasting.

### Cons
-   More Overhead: Additional layers make it slightly heavier than raw WebSockets.
-   Requires a Library: Both client and server need the Socket.IO package.
-   Not a WebSocket Standard Implementation: Socket.IO is not just WebSockets---it adds its own protocol.

### When to Use
Ideal for real-time apps needing automatic reconnection, rooms, and fallbacks, such as collaborative tools, notifications, and live dashboards.

### Testing
-   Use `socket.io-mock` for client-side testing.
-   Mock server behavior using `socket.io-client` with a fake server.

#### Testing Challenges

-   Requires handling full-duplex communication.
-   Room-based messaging is more complex to test.
  
### Code example:

```ts
import { useEffect, useState } from 'react';
import { io, Socket } from 'socket.io-client';

export default function App() {
  const [data, setData] = useState({});
  const [memSocket, setMemSocket] = useState<Socket | null>(null);

  useEffect(() => {
    const socket = io('http://localhost:8082', {
      transports: ['websocket'],
    });

    socket.on('chat', (data) => {
      setData(data);
    });

    setMemSocket(socket);
    return () => {
      socket.on('chat', () => {});
      socket.disconnect();
      setMemSocket(null);
    };
  }, []);

  const handleSendMessage = () => {
    if (memSocket) {
      memSocket.emit('chat', { username: 'user', message: 'message' });
    }
  };

  return (
    <h1>
      Data: {JSON.stringify(data)}
      <button onClick={handleSendMessage}>Send</button>
    </h1>
  );
}
```

## SSE EventSource

### What It Is
-   A unidirectional protocol that allows a server to push updates to a client over HTTP (using long-lived connections).
-   Uses the EventSource API in browsers to receive messages.
-   Works over HTTP/1.1 or HTTP/2.

### Pros

-   Lightweight and efficient.
-   Supports persistent connections.
-   Works over Firewalls & Proxies.

### Cons

-   Unidirectional (server → client only).
-   Limited browser support.
-   Less control over reconnection.

### When to Use

Suitable for one-way, server-to-client updates like live logs, notifications, and real-time analytics.

### Testing

-   Mock `EventSource` manually since Jest lacks built-in support.
-   Simulate events using a custom mock.

#### Testing challenges

-   Auto-reconnect behavior is tricky to test.
-   Requires manual event triggering.


### Code example: 

```ts
import { useEffect, useState } from "react";

export default function App() {
  const [data, setData] = useState("");

  useEffect(() => {
    const eventSource = new EventSource("http://localhost:8081/events");
    eventSource.onmessage = (event) => setData(event.data);

    return () => {
      eventSource.onmessage = null;
      eventSource.close();
    }
  }, []);

  return <h1>Data: {data}</h1>;
}
```

## GraphQL Subscription

### What It Is

-   A real-time mechanism within GraphQL, allowing clients to subscribe to data changes.
-   Uses WebSockets (or other transport mechanisms like MQTT) for full bi-directional communication.
-   Requires a GraphQL server supporting subscriptions (e.g., Apollo Server, Hasura).

### Pros

-   Bidirectional Communication: Unlike SSE, clients can send data too.
-   Structured Queries: Uses GraphQL queries for real-time updates.
-   Efficient with WebSockets: Reduces overhead compared to HTTP polling.

### Cons

-   Requires a GraphQL Server: Needs Apollo, Hasura, or a custom implementation.
-   More Complexity: Setting up resolvers, WebSockets, and schema requires extra effort.
-   WebSocket Dependency: If WebSockets are blocked, you need fallbacks.

### When to Use

Best for structured, real-time GraphQL queries in apps like real-time feeds, chat apps, and collaborative editing.

### Testing

-   Use `MockedProvider` from `@apollo/client/testing`.
-   Simulate subscription events using `mockSubscriptionLink`.

#### Testing challenges

-   WebSocket-based GraphQL testing requires Apollo Server mocks.
-   Ensuring real-time data flow correctness needs extra setup.

### Code example:


Setup the client: @services/index.js
```ts 
import { ApolloClient, InMemoryCache, split } from '@apollo/client';
import { HttpLink } from '@apollo/client/link/http';
import { getMainDefinition } from '@apollo/client/utilities';

export const wsEvents = new EventEmitter();

export const wsLink = new GraphQLWsLink(
  createClient({
    url: 'ws://localhost:4000/graphql',
    // optional to check the websocket status //
    on: {
      connected: () => {
        wsEvents.emit('status', { message: 'connected', ... });
      },
      closed: () => {
        wsEvents.emit('status', { message: 'closed', ... });
      },
      error: () => {
        wsEvents.emit('status', { message: 'error', ... });
      },
    },
  }),
);

const httpLink = new HttpLink({
  uri: 'http://localhost:4000/graphql',
});

const splitLink = split(
  ({ query }) => {
    const def = getMainDefinition(query);
    return (
      def.kind === 'OperationDefinition' && def.operation === 'subscription'
    );
  },
  wsLink,
  httpLink,
);

export const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache(),
});

```

Component:
```ts
import React, {useEffect} from 'react';
import { useSubscription, gql, useMutation } from '@apollo/client';
import { ApolloProvider } from '@apollo/client';
import { client, wsEvents } from '@services/index';

export const CHAT_DATA_SUBSCRIPTION = gql`
  subscription OnChatMessageReceived {
    messageReceived {
      message
      username
      time
    }
  }
`;

export const SEND_MESSAGE = gql`
  mutation SendMessage($username: String!, $message: String!) {
    sendMessage(username: $username, message: $message) {
      username
      message
      time
    }
  }
`;

function App() {
  const [sendMessage] = useMutation(SEND_MESSAGE);
  const { data } = useSubscription(CHAT_DATA_SUBSCRIPTION);

  const handleSendMessage = () => {
    sendMessage({
      variables: {
        username: 'user',
        message: 'message',
      },
    });
  };

 useEffect(() => {

    // optional to check the websocket status
    function handleStatus(eventData: any) {
      console.log(eventData)
    }

    wsEvents.on('status', handleStatus);
    return () => {
      wsEvents.off('status', handleStatus);
    };
  }, []);

  return (
    <h1>
      Data: {JSON.stringify(data)}{' '}
      <button onClick={handleSendMessage}>Send</button>
    </h1>
  );
}

export default function Provider() {
  return (
    <ApolloProvider client={client}>
      <App />
    </ApolloProvider>
  );
}

```