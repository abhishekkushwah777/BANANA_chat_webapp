# 🍌BANANA - Real-Time Chat WebApp

A full-stack **real-time chat application** built with **React.js, Node.js, Express.js, MongoDB, and Socket.IO**.

The application supports user authentication, private conversations, persistent message storage, and real-time message delivery using WebSockets.

## 🚀 Features

* 🔐 **User Authentication**

  * User registration and login
  * JWT-based authentication
  * Protected API routes
  * Authenticated Socket.IO connections

* 💬 **Real-Time Messaging**

  * Send and receive messages instantly
  * Socket.IO-based real-time communication
  * Conversation-specific Socket.IO rooms
  * Messages are broadcast to all participants in the conversation

* 👥 **Private Conversations**

  * One-to-one conversations between users
  * Automatically find or create existing private conversations
  * Conversation access is verified on the backend

* 💾 **Persistent Messages**

  * Messages are stored in MongoDB
  * Previous messages are loaded when opening a conversation
  * Messages include sender information and timestamps

* 🔒 **Security**

  * JWT authentication for REST APIs
  * JWT authentication during Socket.IO handshake
  * Backend verifies conversation membership before allowing access
  * Environment variables for sensitive configuration

* ⚡ **React Frontend**

  * Component-based architecture
  * React Context for shared socket connection
  * Dynamic conversation loading
  * Real-time UI updates when new messages arrive

---
# Downloads

App release v1 - https://drive.google.com/drive/folders/1T00gf0qyKjLpf05698rYu_bkRyjexsDc?usp=drive_link

---
## UI Design
### Login UI

<image src = "./UI/welcome.png">
<image src = "./UI/login.png">
 
### Registration
 
<image src = "./UI/register_1.png">
<image src = "./UI/register_2.png">
 
### In app screens

<image src = "./UI/homescreen.png">
<image src = "./UI/chatscreen.png">
<image src = "./UI/notification.png">
<image src = "./UI/addfriends.png">
<image src = "./UI/myprofile.png">
 
---

## 🛠️ Tech Stack

### Frontend

* React.js
* JavaScript
* Vite
* CSS
* Socket.IO Client
* Fetch API

### Backend

* Node.js
* Express.js
* Socket.IO
* JWT
* Mongoose

### Database

* MongoDB

---

## 🏗️ Architecture

The application uses both **REST APIs** and **Socket.IO**.

```text
                    ┌──────────────────┐
                    │   React Client   │
                    └────────┬─────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
             REST API                Socket.IO
                │                         │
                ▼                         ▼
        ┌───────────────┐        ┌────────────────┐
        │ Express Server│        │ Socket.IO      │
        │               │        │ Server         │
        └───────┬───────┘        └────────┬───────┘
                │                         │
                └────────────┬────────────┘
                             ▼
                    ┌─────────────────┐
                    │     MongoDB     │
                    │                 │
                    │ Users           │
                    │ Conversations   │
                    │ Messages        │
                    └─────────────────┘
```

### How messaging works

```text
User opens conversation
        │
        ▼
REST API fetches conversation
        │
        ▼
REST API fetches previous messages
        │
        ▼
React receives conversation ID
        │
        ▼
Socket.IO connection
        │
        ▼
join_conversation
        │
        ▼
conversation:<conversationId>
        │
        ▼
User sends message
        │
        ▼
send_message
        │
        ▼
Backend validates participant
        │
        ▼
Message saved to MongoDB
        │
        ▼
receive_message
        │
        ▼
All users in conversation receive message
```

---

## 📁 Project Structure

```text
chatApp/
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   │   ├── auth.js
│   │   │   └── conversation.js
│   │   │
│   │   ├── components/
│   │   │   ├── Chat.jsx
│   │   │   ├── MessageList.jsx
│   │   │   └── ...
│   │   │
│   │   ├── context/
│   │   │   └── SocketContext.jsx
│   │   │
│   │   ├── socket/
│   │   │   └── socket.js
│   │   │
│   │   ├── styles/
│   │   ├── App.jsx
│   │   └── main.jsx
│   │
│   └── package.json
│
├── backend/
│   ├── controllers/
│   ├── middleware/
│   ├── models/
│   │   ├── User.js
│   │   ├── Conversation.js
│   │   └── Message.js
│   │
│   ├── routes/
│   │   ├── authRoutes.js
│   │   └── conversationRoutes.js
│   │
│   ├── socket/
│   │   └── socketHandler.js
│   │
│   ├── utils/
│   │   └── generateToken.js
│   │
│   ├── server.js
│   ├── .env
│   └── package.json
│
├── .gitignore
└── README.md
```

> The exact structure may vary depending on the current implementation.

---

## 🔌 Socket.IO Communication

The application uses Socket.IO rooms to isolate conversations.

When a user opens a conversation:

```javascript
socket.emit("join_conversation", {
  conversationId,
});
```

The server places the socket into:

```text
conversation:<conversationId>
```

For example:

```text
conversation:6a944fbfd0178f9282dc62aa
```

When a message is sent:

```javascript
socket.emit("send_message", {
  conversationId,
  content,
});
```

The server:

1. Verifies the JWT
2. Identifies the authenticated user
3. Checks conversation membership
4. Saves the message to MongoDB
5. Populates sender information
6. Emits the message to the conversation room

```javascript
io.to(`conversation:${conversationId}`)
  .emit("receive_message", message);
```

---

## 🔐 Authentication Flow

JWT is used for both HTTP API authentication and Socket.IO authentication.

### REST API

The frontend sends:

```http
Authorization: Bearer <JWT>
```

### Socket.IO

The JWT is sent during the Socket.IO handshake:

```javascript
const socket = io("http://localhost:5000", {
  auth: {
    token,
  },
});
```

The backend retrieves it with:

```javascript
const token = socket.handshake.auth.token;
```

and verifies it:

```javascript
const decoded = jwt.verify(
  token,
  process.env.JWT_SECRET
);
```

The authenticated user is then available through:

```javascript
socket.user
```

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone <your-repository-url>
cd chatApp
```

### 2. Install backend dependencies

```bash
cd backend
npm install
```

### 3. Install frontend dependencies

```bash
cd ../frontend
npm install
```

---

## 🔑 Environment Variables

Create a `.env` file inside the `backend` directory:

```env
PORT=5000
MONGODB_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
```

Do **not** commit your `.env` file.

Your `.gitignore` should contain:

```gitignore
.env
node_modules/
dist/
```

---

## ▶️ Running the Application

### Start the backend

```bash
cd backend
node server.js
```

The server should run on:

```text
http://localhost:5000
```

### Start the frontend

Open another terminal:

```bash
cd frontend
npm run dev
```

The Vite development server will normally run on:

```text
http://localhost:5173
```

---

## 📡 Example Socket Events

### Join conversation

Client:

```javascript
socket.emit("join_conversation", {
  conversationId,
});
```

Server:

```javascript
socket.join(
  `conversation:${conversationId}`
);
```

### Send message

Client:

```javascript
socket.emit("send_message", {
  conversationId,
  content: "Hello!",
});
```

### Receive message

Client:

```javascript
socket.on(
  "receive_message",
  (message) => {
    setMessages((previousMessages) => [
      ...previousMessages,
      message,
    ]);
  }
);
```

### Leave conversation

Client:

```javascript
socket.emit("leave_conversation", {
  conversationId,
});
```

---

## 🗄️ Database Models

### User

Stores information such as:

```text
_id
username
email
password
avatar
status
```

### Conversation

Stores:

```text
_id
type
participants
createdAt
updatedAt
```

### Message

Stores:

```text
_id
conversationId
senderId
content
createdAt
```

This allows messages to remain persistent even after a user disconnects from Socket.IO.

---

## 🧠 Key Concepts Demonstrated

This project was built to understand and implement:

* REST API development
* JWT authentication
* Protected routes
* MongoDB and Mongoose
* React state management
* React Context API
* WebSocket communication
* Socket.IO rooms
* Real-time event handling
* Persistent chat messages
* Client/server communication
* Authentication during WebSocket handshake
* Conversation-based authorization

---

## 🔮 Future Improvements

Potential features planned for future versions:

* 👤 Online/offline user status
* ✍️ Typing indicators
* ✓ Message delivery/read status
* 🗑️ Delete messages
* ✏️ Edit messages
* 📎 File and image sharing
* 🔔 Notifications
* 🔎 Message search
* 👥 Group conversations
* 🟢 Last seen
* 📱 Responsive/mobile UI
* 🔄 Message pagination
* 🚀 Production deployment
* ☁️ Cloud file storage

---

## 👨‍💻 Author

**Abhishek Kushwah**

B.Tech — Artificial Intelligence & Machine Learning

This project was developed as a practical project to understand **real-time communication, WebSockets, backend authentication, and full-stack application architecture**.

---

## 📄 License

This project is available for educational and personal use.
