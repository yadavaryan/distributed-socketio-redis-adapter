# ⚡ Distributed Socket.IO Redis Adapter

<div align="center">

[![Node.js](https://img.shields.io/badge/Node.js-14+-green?style=for-the-badge&logo=node.js)](https://nodejs.org/)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-4.x-black?style=for-the-badge&logo=socket.io)](https://socket.io/)
[![Redis](https://img.shields.io/badge/Redis-Pub%2FSub-red?style=for-the-badge&logo=redis)](https://redis.io/)
[![TypeScript](https://img.shields.io/badge/TypeScript-Ready-blue?style=for-the-badge&logo=typescript)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

**A high-performance adapter for scaling real-time WebSockets horizontally by broadcasting packets between multiple Socket.IO servers via Redis.**

</div>

---

## 📖 Overview

The **Distributed Socket.IO Redis Adapter** is a critical infrastructure component designed to solve the challenge of scaling real-time WebSocket applications. By default, Socket.IO clients connected to different server instances cannot communicate. This project utilizes Redis Pub/Sub mechanisms to seamlessly broadcast messages across an entire cluster of Node.js servers, ensuring real-time state synchronization regardless of which node a user connects to.

This project demonstrates deep understanding of distributed systems, high-availability architecture, inter-process communication (IPC), and real-time networking.

## ✨ Features

- **🚀 Horizontal Scalability**: Run multiple Socket.IO servers concurrently while maintaining a single, unified real-time network.
- **🔄 Seamless Broadcasting**: Emit messages to rooms, namespaces, or specific clients across the entire server cluster with zero extra configuration.
- **🛠️ Redis Cluster & Sentinel Support**: Fully compatible with standalone Redis, Redis Clusters, and highly-available Sentinel setups.
- **🔀 Sharded Pub/Sub Optimization**: Integrates with Redis 7.0+ Sharded Pub/Sub for massive scale without network bottlenecks.
- **✅ Guaranteed Acknowledgements**: Supports complex cross-server message acknowledgements natively.

## 🏗️ Architecture

When a client connects to **Server A** and emits a message intended for a room containing clients on **Server B** and **Server C**:

1. The local adapter on **Server A** intercepts the emit.
2. It publishes the packet to a specific Redis Pub/Sub channel.
3. The adapters on **Server B** and **Server C** are subscribed to this channel.
4. They receive the packet and broadcast it to their locally connected WebSocket clients.

## 🚀 Quick Start

### 1. Installation

Install the adapter alongside your Socket.IO setup:

```bash
npm install @socket.io/redis-adapter redis
```

### 2. Basic Setup (Node.js)

Here is a quick implementation of how to attach the Redis adapter to a Socket.IO instance:

```javascript
import { createClient } from "redis";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-adapter";

// 1. Create Redis Pub/Sub Clients
const pubClient = createClient({ url: "redis://localhost:6379" });
const subClient = pubClient.duplicate();

await Promise.all([
  pubClient.connect(),
  subClient.connect()
]);

// 2. Initialize Socket.IO with the Adapter
const io = new Server(3000, {
  adapter: createAdapter(pubClient, subClient)
});

// 3. Broadcast across instances seamlessly
io.on("connection", (socket) => {
  console.log("Client connected");
  
  socket.on("global_event", (data) => {
    // This broadcasts to ALL clients across ALL servers
    io.emit("global_response", data); 
  });
});
```

### 3. Advanced: Sharded Pub/Sub (Redis 7.0+)

For massive scale, use the `createShardedAdapter` method to distribute network load across Redis cluster nodes.

```javascript
import { createShardedAdapter } from "@socket.io/redis-adapter";

const io = new Server(3000, {
  adapter: createShardedAdapter(pubClient, subClient)
});
```

## 🔒 Security Notice

This adapter assumes Redis is deployed within a trusted, internal Virtual Private Cloud (VPC). Messages are not encrypted at the adapter level. Ensure Redis is secured using ACLs, TLS, and strict firewall rules.

## 📜 License

This project is licensed under the MIT License.
