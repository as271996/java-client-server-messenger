# Java Client-Server Messenger

A desktop-based multi-user messaging application built in Java using TCP sockets, multithreading, and Java Swing.

The project follows a client-server architecture where multiple users connect to a central server and communicate through a desktop client.

The application is divided into two components:

- **jServer** — accepts client connections, authenticates users, tracks connected users, routes messages, and coordinates file-transfer requests
- **jMessenger** — provides the desktop interface for login, signup, messaging, file sharing, online-user selection, and chat history

The project demonstrates core networking concepts such as socket communication, concurrent client handling, message routing, and client-server coordination.

## Features

- User signup and login
- Multi-user client-server messaging
- Private messaging between connected users
- Broadcast messaging to all connected users
- Online user list and user selection
- File-transfer support between clients
- Local chat-history storage
- XML-based user and history data handling
- Java Swing desktop interface
- Multithreaded server handling for concurrent client connections

## Client-Server Architecture

The application uses a centralized client-server model.

```text
                ┌─────────────────┐
                │     jServer     │
                │                 │
                │ Authentication  │
                │ User Tracking   │
                │ Message Routing │
                │ File Transfer   │
                └────────┬────────┘
                         │
              TCP Socket Connections
                         │
          ┌──────────────┼──────────────┐
          │              │              │
          ▼              ▼              ▼
    ┌──────────┐    ┌──────────┐    ┌──────────┐
    │ Client A │    │ Client B │    │ Client C │
    │jMessenger│    │jMessenger│    │jMessenger│
    └──────────┘    └──────────┘    └──────────┘
```

### Server Responsibilities

The `jServer` application:

- Accepts incoming client connections
- Authenticates users
- Maintains the list of connected users
- Routes private and broadcast messages
- Coordinates file-transfer requests
- Handles multiple connected clients concurrently

### Client Responsibilities

The `jMessenger` application:

- Provides login and signup interfaces
- Connects to the central server using TCP sockets
- Sends and receives messages
- Displays connected users
- Supports private and broadcast communication
- Initiates and receives file-transfer requests
- Stores local chat history

## Tech Stack

**Language:** Java  
**Desktop UI:** Java Swing  
**Networking:** TCP Sockets / Java Socket Programming  
**Concurrency:** Java Multithreading  
**Data Storage:** XML-based user data and chat history  
**File Transfer:** Socket-based file transfer  
**Build Tool / IDE:** Ant, NetBeans

## Project Structure

```text
java-client-server-messenger/
├── jMessenger/
│   └── jMessenger/
│       ├── src/
│       │   └── com/
│       │       ├── socket/
│       │       │   ├── Download.java
│       │       │   ├── History.java
│       │       │   ├── Message.java
│       │       │   ├── SocketClient.java
│       │       │   └── Upload.java
│       │       │
│       │       └── ui/
│       │           ├── ChatFrame.java
│       │           └── HistoryFrame.java
│       │
│       ├── build.xml
│       ├── manifest.mf
│       └── nbproject/
│
└── jServer/
    └── jServer/
        ├── src/
        │   └── com/
        │       └── socket/
        │           ├── Database.java
        │           ├── Message.java
        │           ├── ServerFrame.java
        │           └── SocketServer.java
        │
        ├── build.xml
        ├── manifest.mf
        └── nbproject/
```

## Main Modules

### Client Application

The `jMessenger` module contains the desktop client used by end users.

- `SocketClient.java` — manages communication with the central server
- `Message.java` — represents messages exchanged between client and server
- `Upload.java` / `Download.java` — handle file-transfer operations
- `History.java` — manages local chat-history data
- `ChatFrame.java` — main Java Swing messaging interface
- `HistoryFrame.java` — displays stored conversation history

### Server Application

The `jServer` module provides the central server for connected clients.

- `SocketServer.java` — accepts and manages client socket connections
- `Database.java` — manages user information and authentication data
- `Message.java` — represents messages exchanged through the server
- `ServerFrame.java` — provides the Java Swing interface for running and monitoring the server

Together, the two modules implement the complete client-server messaging workflow.

## Message and File Transfer Flow

### Private Messaging

When a client sends a private message:

1. The client creates a `Message` object containing:
   - message type
   - sender
   - content
   - recipient
2. The message is sent to `jServer` through an `ObjectOutputStream`.
3. The server identifies the recipient from the connected-user list.
4. The server forwards the message to the recipient.
5. The server also sends a copy back to the sender so both sides can update their UI consistently.
6. The client stores sent and received messages in the local XML-based chat history.

```text
Client A
   │
   │ Message("message", sender, content, recipient)
   ▼
jServer
   │
   ├──────────────► Client B
   │
   └──────────────► Client A
                    (confirmation / local display)
```

### Broadcast Messaging

If the recipient is set to `All`, the server broadcasts the message to every connected client.

```text
                 ┌──────────► Client A
                 │
Client ──► jServer ─────────► Client B
                 │
                 └──────────► Client C
```

### File Transfer

File transfers use the server for coordination, while the actual file data is transferred directly between clients.

1. The sender sends an `upload_req` message to the server.
2. The server forwards the request to the intended recipient.
3. The recipient chooses whether to accept or reject the file.
4. If accepted, the recipient starts a temporary `ServerSocket` on an available port.
5. The recipient sends the selected port back through the server using an `upload_res` message.
6. The sender establishes a direct socket connection to the recipient.
7. The file is streamed in chunks directly from sender to receiver.

```text
             Transfer Request
Sender ───────────► jServer ───────────► Receiver

             Accept + Port
Sender ◄─────────── jServer ◄─────────── Receiver

             Direct File Transfer
Sender ═════════════════════════════════► Receiver
```

This design keeps message routing centralized while allowing file contents to be transferred directly between clients.

## Authentication and User Management

The messenger includes a simple user-registration and authentication flow backed by XML-based storage.

### Signup

1. A new user enters registration details in the client application.
2. The client sends a signup request to the server.
3. The server checks the stored user data for an existing username.
4. If the username is available, the new user is added to the XML-based user store.
5. The server returns the signup result to the client.

### Login

1. The client sends the entered username and password to the server.
2. The server validates the credentials against the stored user data.
3. On successful authentication, the server registers the client as an active connection.
4. The connected-user list is updated and shared with clients.
5. The authenticated user can then send private messages, broadcast messages, and file-transfer requests.

```text
Client
   │
   │ Login / Signup Request
   ▼
jServer
   │
   ▼
XML User Store
   │
   ├── Validate Existing User
   └── Register New User
   │
   ▼
Authentication Result
   │
   ▼
Client
```

### Connected User Tracking

The server maintains the set of currently connected clients so that:

- private messages can be routed to the correct recipient
- broadcast messages can be sent to all active users
- the client interface can display the online-user list
- disconnected users can be removed from the active-user set

## Running the Application

The project contains two separate NetBeans/Ant applications:

- `jServer` — the central messaging server
- `jMessenger` — the desktop client

### Prerequisites

Make sure the following are installed:

- Java 6 or later
- NetBeans IDE or another Java IDE with Ant support
- Ant

### 1. Clone the Repository

```bash
git clone https://github.com/as271996/java-client-server-messenger.git
cd java-client-server-messenger
```

### 2. Start the Server

Open the server project:

```text
jServer/jServer
```

The configured main class is:

```text
com.socket.ServerFrame
```

Run the project from NetBeans, or build it using Ant:

```bash
cd jServer/jServer
ant clean
ant jar
```

After a successful build, run:

```bash
java -jar dist/jServer.jar
```

### 3. Start the Client

Open the client project:

```text
jMessenger/jMessenger
```

The configured main class is:

```text
com.ui.ChatFrame
```

Build it using NetBeans or Ant:

```bash
cd jMessenger/jMessenger
ant clean
ant jar
```

Then run:

```bash
java -jar dist/jMessenger.jar
```

### 4. Run Multiple Clients

Start more than one instance of `jMessenger` to simulate multiple connected users.

Each client connects to the central `jServer`, after which users can:

- sign up or log in
- view connected users
- send private messages
- send broadcast messages
- transfer files
- view local chat history

The server should be started before launching the clients.

## Limitations and Future Improvements

This project was built as an academic networking application and can be improved further in several areas:

- Replace XML-based user storage with a database
- Hash and securely store user passwords
- Add stronger authentication and authorization
- Encrypt client-server communication
- Add delivery acknowledgements and message-status tracking
- Improve file-transfer validation, retry handling, and progress reporting
- Add better handling for disconnected clients and network failures
- Improve synchronization around shared server-side client state
- Add automated unit and integration tests
- Replace the NetBeans/Ant setup with Maven or Gradle
- Modernize the Java Swing user interface
- Add support for persistent server-side chat history
- Add group chats and richer conversation management
- Improve configuration for server host, port, and runtime settings

## Author

**Amit Singh**

Java networking project focused on client-server communication, socket programming, multithreading, desktop messaging, and file transfer.

- GitHub: [github.com/as271996](https://github.com/as271996)
- LinkedIn: [linkedin.com/in/amit-singh-sp27](https://www.linkedin.com/in/amit-singh-sp27)

## Project Context

This project was built to explore core networking concepts through a multi-user desktop messenger application.

It demonstrates:

- TCP socket communication between multiple clients and a central server
- concurrent client handling using multithreading
- private and broadcast message routing
- user authentication and connected-user tracking
- direct client-to-client file transfer coordinated by the server
- XML-based user data and chat-history storage
- Java Swing desktop UI development
