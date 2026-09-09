# Backend Architecture: Rate Limit and Server-Sent Events (SSE)

**Date Created:** 2026-09-04  
**Tags:** `#SystemDesign`, `#Backend`, `#Realtime`, `#API`  
**References/Sources:** [Gemini Share](https://share.gemini.google/zEAVGArmeM7e) & [https://share.gemini.google/C3HUVEIvYfGX]

---

## 📌 Summary (TL;DR)
- **Rate Limit Constraints:** Traffic control mechanisms for incoming server requests to protect resources, prevent DDoS attacks, ensure fair use, and support service tiering (monetization).
- **Server-Sent Events (SSE):** A unidirectional connection protocol (Server -> Client) over standard HTTP, optimized for real-time features that only require read streams (e.g., AI Chatbot streaming, log systems).

## 🧠 Detailed Content

### 1. Rate Limit Constraints
Rate limiting consists of routing constraints at the Server, API Gateway, or Firewall level to restrict the request frequency from an identity (IP, User ID, API Key) within a specific time window.
- **Response Mechanism:** Exceeding threshold -> `429 Too Many Requests`.
- **Core Purposes:**
  1. **Prevent Overload (Resource Protection):** Prevent server crashes caused by peak traffic.
  2. **Security (Attack Prevention):** Block spam request streams from botnets/hackers attempting to DDoS the system.
  3. **Fair Use (Fair Resource Sharing):** Prevent the "Noisy Neighbor" problem (one user consuming the entire resource pool and slowing down the system for others).
  4. **Monetization (Business Model):** Tier third-party API services (e.g., Free for 1,000 reqs/month, Pro for 10,000 reqs/month).

### 2. Server-Sent Events (SSE)
SSE is a data push technique from Server to Client over a single persistent HTTP connection. It was designed to overcome the bandwidth inefficiency of traditional Polling (where the client repeatedly pings the server asking for new data).

- **Technical Characteristics:**
  - **Unidirectional:** Data only flows from Server -> Client.
  - **HTTP Native:** Runs over standard HTTP, requires no complex handshake mechanism, and easily traverses firewalls/proxies.
  - **Auto-reconnect:** Browsers automatically attempt to reconnect to the server if the connection is dropped.
- **Typical Use Cases:**
  - Streaming LLM Chatbot outputs (e.g., ChatGPT, Gemini) to display token-by-token (typing effect) instead of waiting for the full response.
  - Live stock/crypto price tickers.
  - Notification feeds (system notifications).
  - Live system log monitoring.

#### Basic Implementation
Principle: Client opens a `GET` request, and the Server responds with specific headers while keeping the connection open.

**Backend (Node.js/Express):**
```javascript
const express = require('express');
const app = express();

app.get('/stream', (req, res) => {
    // 1. Mandatory headers to define SSE stream
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');

    // 2. Periodically push data to client
    const intervalId = setInterval(() => {
        const time = new Date().toLocaleTimeString();
        // Mandatory SSE format: data: [payload]\n\n
        res.write(`data: Current time is ${time}\n\n`); 
    }, 2000);

    // 3. Clean up resources (prevent memory leaks) when client disconnects (closes tab)
    req.on('close', () => clearInterval(intervalId));
});

app.listen(3000, () => console.log('SSE Server running on port 3000'));
```

**Frontend (HTML/JS - EventSource API):**
```html
<script>
    // Open connection
    const eventSource = new EventSource('http://localhost:3000/stream');
    
    // Listen for incoming events
    eventSource.onmessage = (event) => {
        console.log("Server pushed:", event.data);
    };
</script>
```

## 🤔 Deep Dive Topics
- [ ] **Rate Limiting Algorithms:** How do Token Bucket, Leaky Bucket, Fixed Window Counter, and Sliding Window Log work, and what are their pros and cons?
- [ ] **Practical Configuration:** How to set up Rate Limiting on Nginx, Kong Gateway, or using Redis for distributed counter storage.
- [ ] **Protocol Comparison:** When is SSE sufficient, and when is it necessary to switch to WebSockets (Bi-directional) or gRPC streaming?
- [ ] **Network Infrastructure:** How to configure Load Balancers (HAProxy, AWS ALB) to avoid timing out (prematurely closing) long-lived SSE connections?

## 📚 References & Recommended Reading
- Gemini Source: [https://share.gemini.google/zEAVGArmeM7e](https://share.gemini.google/zEAVGArmeM7e)
- Additional Reference: [https://share.gemini.google/C3HUVEIvYfGX](https://share.gemini.google/C3HUVEIvYfGX)
