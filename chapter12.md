# Chapter 12: Designing a Chat System

## Key Terms & Definitions

- **Chat App**: A software application for online communication between users in real-time.
- **Daily Active Users (DAU)**: The number of unique users who engage with a service or app in a given day.
- **HTTP**: Hypertext Transfer Protocol, the foundation of data communication on the world wide web.
- **WebSocket**: A computer communications protocol, providing full-duplex communication channels over a single TCP connection.

## Clarifying Questions & High-Level Design

1. **Type of chat app**: The system will support both 1 on 1 and group chat.
2. **Platforms**: The system should be accessible as a mobile app and web app.
3. **Scale**: The system needs to support 50 million daily active users.
4. **Group size**: For group chat, the maximum member limit is 100.
5. **Features**: The system will support text-based 1 on 1 chat, group chat, and online indicators. The system will not support attachments.
6. **Message size**: Text length should be less than 100,000 characters long.
7. **Encryption**: End-to-end encryption is not required at this point.
8. **Message retention**: The system should store chat history indefinitely.

The high-level design features a chat service that receives messages from clients, finds the right recipients for each message, and relays the message to them. If a recipient is offline, the chat service holds the messages for that recipient on the server until they come online.

## Deep Dive into Design Aspects

- **Network Protocols**: The choice of network protocol is a significant aspect of the chat service's design.
  - **HTTP**: On the sender side, a time-tested protocol, HTTP, is used. The sender opens an HTTP connection with the chat service and sends the message. The HTTP Keep-Alive header allows a client to maintain a persistent connection with the chat service, reducing the number of TCP handshakes.
  - **WebSocket**: A bidirectional, persistent connection initiated by the client. This protocol starts as an HTTP connection but can be upgraded via a defined handshake to a WebSocket connection. It enables the server to send updates to the client and can work even if a firewall is in place. WebSocket is typically used for both sending and receiving, simplifying the design and making implementation more straightforward on both the client and server sides.

## Comparison of Different Techniques

1. **Polling**: The client periodically asks the server if there are messages available. It can be costly and inefficient due to high polling frequency.
2. **Long Polling**: The client holds the connection open until there are new messages available or a timeout threshold is reached. While this is more efficient than polling, it has drawbacks, such as complications with connections if sender and receiver do not connect to the same chat server, inefficient connections for less active users, and the server's inability to know if a client has disconnected.
3. **WebSocket**: The most common solution for sending asynchronous updates from server to client. This technique has a persistent connection that is efficient and reliable, but the management of WebSocket connections is crucial on the server-side.

# Chapter 12 - System Design Interview

## Scalability

- All services could theoretically fit on a single server, but it's not a good practice due to single-point-of-failure considerations.
- Starting with a single-server design is acceptable as long as it's understood that it's a starting point.
- Various servers are used in the system:
  - **Chat servers**: Facilitate real-time messaging.
  - **Presence servers**: Manage online/offline status.
  - **API servers**: Handle login, signup, profile changes, etc.
  - **Notification servers**: Send push notifications.
  - **Key-value store**: Store chat history.

## Storage

- Two types of data exist in a typical chat system:
  - **Generic data** (user profiles, settings, friends list): Stored in robust and reliable relational databases. Replication and sharding are used for availability and scalability.
  - **Chat history data**: The read/write pattern is informed by these data.
    - The volume of data is enormous.
    - Only recent chats are accessed frequently.
    - Users might require random access for features like search, view mentions, jump to specific messages, etc.
    - The read to write ratio is about 1:1 for one-on-one chat apps.
- Key-value stores are recommended for the following reasons:
  - They allow easy horizontal scaling.
  - They provide very low latency to access data.
  - Relational databases struggle with long tail data. When the indexes grow large, random access becomes expensive.

## Data Models

- For message data, a key-value store is recommended.
- There are separate message tables for one-on-one chats and group chats. The primary key differs for each; for one-on-one chats, it's `message_id`, for group chats, it's a composite of `channel_id` and `message_id`.
- `message_id` ensures message order. Two requirements of `message_id` are:
  - IDs must be unique.
  - IDs should be sortable by time.
- Generating `message_id` can use either a global 64-bit sequence number generator or a local sequence number generator. Local IDs only need to be unique within a group, and they are easier to implement.

## Step 3 - Design Deep Dive

Some components require a deeper exploration, such as service discovery, messaging flows, and online/offline indicators.

### Service Discovery

- This recommends the best chat server for a client based on criteria like geographical location, server capacity, etc.
- Apache Zookeeper is a popular open-source solution for service discovery.

### Message Flows

- Understanding the end-to-end flow of a chat system includes examining one-on-one chat flow, message synchronization across multiple devices, and group chat flow.
- Different processes take place depending on whether the user is online or offline. If the recipient is online, the message is forwarded to their chat server; if they're offline, a push notification is sent.
- Messages are synchronized across multiple devices with the help of `cur_max_message_id`, which tracks the latest message ID on the device.

### Small Group Chat Flow

- For small groups, each message is copied to each group member’s message sync queue.
- For larger groups, storing a message copy for each member is not feasible due to storage concerns.

## Online Presence

An online presence indicator is an essential feature of many chat applications. Presence servers manage online status and communicate with clients via WebSocket. Here are a few flows that trigger online status changes:

### User Login

- After a WebSocket connection is established between the client and the real-time service, the user's online status and `last_active_at` timestamp are saved in the KV store.

### User Logout

- When a user logs out, the online status is changed to offline in the KV store.

### User Disconnection

- Internet connections can be inconsistent, thus user disconnection must be addressed.
- A naive way to handle disconnection is to mark the user as offline and change the status to online when reconnected. But this results in poor user experience due to frequent status changes.
- Solution: A heartbeat mechanism. Periodically, an online client sends a heartbeat event to presence servers. If a heartbeat event is received within a certain time (x seconds), a user is considered as online. Otherwise, offline.

### Online Status Fanout

- How do friends know about the status changes? This uses a publish-subscribe model. Each friend pair maintains a channel.
- When a user's online status changes, the event is published to all relevant channels, which are subscribed by friends. Hence, they get online status updates.
- This design is effective for a small user group. For larger groups, updating all members about online status is expensive and time-consuming. The solution could be to fetch online status only when a user enters a group or manually refreshes the friend list.

## Step 4 - Wrap Up

The chapter presented a chat system architecture supporting both 1-to-1 chat and small group chat. Additional discussion points could include:

### Extend the Chat App

- Support media files, which are significantly larger than text. Interesting topics include compression, cloud storage, and thumbnails.

### End-to-end Encryption

- Only the sender and recipient can read messages, like in WhatsApp.

### Caching Messages

- Caching messages on the client-side can effectively reduce data transfer between the client and server.

### Improve Load Time

- For better load time, a geographically distributed network could be used to cache user data and channels, as Slack does.

### Error Handling

- **Chat Server Error**: If a chat server goes offline, the service discovery will provide a new chat server for clients to establish new connections.
- **Message Resent Mechanism**: Retry and queuing are common techniques for resending messages.
