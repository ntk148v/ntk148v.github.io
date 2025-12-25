---
title: "Server-sent Events (SSE)"
date: "2025-06-10T13:34:12+07:00"
tags: ["tech", "sse"]
draft: true
comment: true
---

# The Introvert's Guide to Real-time Web Updates 🙈

Ever felt like WebSocket is that extroverted friend who's always trying to start a conversation? "Hey server, how are you?" "I'm fine client, how are you?" "I'm good too!" 🙄

Well, meet Server-sent Events (SSE) - the introvert's dream come true in the world of real-time web communications. It's like having a friend who's totally cool with you just listening while they do all the talking. Perfect, right?

## What's SSE Anyway? 🤔

Imagine you're at a soccer match, but instead of watching it live, you're getting updates from your chatty friend who's there:

```
Friend: "Kick-off!"
You: *nods silently*

Friend: "GOOOAL! Manchester United 1 - 0 Liverpool!"
You: *continues nodding*

Friend: "Half-time! Still 1-0"
You: *thumbs up*
```

That's basically SSE! The server (your friend) sends updates whenever something happens, and the client (you) just... listens. No need to ask "what's happening?" every 5 seconds like with traditional polling, or maintain a complex two-way conversation like with WebSockets.

### The Technical Bits 🔧

Server-Sent Events (SSE) is a web standard that enables servers to push data to web clients via HTTP. Here's what makes it special:

1. **The Connection**

   ```http
   GET /livescore HTTP/1.1
   Accept: text/event-stream
   ```

   The client makes a single HTTP request with `Accept: text/event-stream`, and the connection stays open. It's like subscribing to your friend's match updates on WhatsApp.

2. **The Message Format**

   ```
   event: goal  # Optional event type
   data: {"score": {"home": 1, "away": 0}}

   data: {"status": "Game in progress"}
   ```

   Messages are sent as UTF-8 text, with each message separated by double newlines. Like your friend sending multiple texts, but way more organized!

3. **Event Types**
   - `message` - Default event
   - Custom events (like 'goal', 'halftime', 'fulltime')
   - Each type can have its own event handler on the client

4. **Built-in Goodies**
   - Automatic reconnection (configurable with `retry: 3000`)
   - Event IDs for tracking last received message
   - Cross-origin support with standard CORS

5. **The Not-So-Fun Parts** 😅
   - **Browser Connection Limits**: Most browsers limit the number of open SSE connections per domain (typically 6). It's like your friend can only update 6 people at once!
   - **One-Way Street**: No built-in way to send data back to the server. Need to chat back? You'll need a separate HTTP request.
   - **Plain Text Only**: Unlike WebSocket, SSE only supports UTF-8 text data. Binary data? Sorry, you'll need to encode it first.
   - **Proxy Issues**: Some proxy servers don't play nice with long-lived connections. They might think your friend fell asleep and cut the connection!
   - **Header Size Limits**: Some servers have a maximum header size limit, which can affect reconnection with large `Last-Event-ID` headers.

## Show Me The Code! 💻

We've built a simple soccer livescore system using FastAPI and SSE: <https://github.com/ntk148v/sse-fastapi>

Let's break down how it works:

### The Backend (The Chatty Friend)

```python
# 1. First, we set up our FastAPI app with CORS support
app = FastAPI(title="Soccer Livescore SSE API")
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # In production, specify your domains!
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# 2. Our event generator - the heart of SSE
async def livescore_generator():
    """Our enthusiastic commentator"""
    data = load_mock_data()
    for update in data:
        # Format: "data: {json_data}\n\n"
        yield f"data: {json.dumps(update)}\n\n"
        await asyncio.sleep(2)  # Taking a breath between updates

# 3. The SSE endpoint
@app.get("/livescore")
async def livescore():
    return StreamingResponse(
        livescore_generator(),
        media_type="text/event-stream",  # Magic happens here!
        headers={
            "Cache-Control": "no-cache",
            "Connection": "keep-alive",
        }
    )
```

Let's break down what's happening:

1. The `CORSMiddleware` allows our frontend to connect from any origin
2. `livescore_generator()` is an async generator that:
   - Loads match data (in real life, this would be live data)
   - Yields properly formatted SSE messages
   - Uses `asyncio.sleep()` to simulate real-time updates
3. `StreamingResponse` keeps the connection open and streams data

### The Frontend (The Patient Listener)

```javascript
// 1. Create the connection
const eventSource = new EventSource("http://localhost:8000/livescore");

// 2. Set up our event handlers
eventSource.onopen = () => {
  console.log("Connection established!");
};

eventSource.onmessage = (event) => {
  // Parse the JSON data from the event
  const data = JSON.parse(event.data);
  // Update our UI with the new match data
  updateMatch(data);
};

eventSource.onerror = (error) => {
  console.error("Connection lost! Attempting to reconnect...");
  // The browser will automatically try to reconnect
};

// 3. Update the UI with the match data
function updateMatch(data) {
  // Update score
  document.querySelector(".score").textContent =
    `${data.score.home} - ${data.score.away}`;
  // Update match status
  document.querySelector(".match-status").textContent = data.status;
  // ... more UI updates
}
```

## Why SSE? 🌟

1. **It's HTTP**: No special protocols, no weird handshakes, just good old HTTP. Your server can finally stop pretending it knows WebSocket dance moves.

2. **Auto-reconnect**: Like a clingy friend, it'll keep trying to reconnect if the connection drops. No need to implement retry logic!

3. **One-way communication**: Perfect for when your client is in "introvert mode" and just wants to receive updates without the pressure of responding.

4. **Built-in event types**: Want to categorize your updates? SSE has built-in event types! It's like having different chat groups for different topics.

## Try It Yourself! 🚀

1. Clone this repo:

   ```bash
   git clone https://github.com/ntk148v/sse-fastapi.git
   cd sse-fastapi
   ```

2. Install dependencies (we use `uv` because we're cool):

   ```bash
   uv venv .venv
   . .venv/bin/activate
   uv pip install -e .
   ```

3. Run the server:

   ```bash
   python main.py
   ```

4. Open http://localhost:8000 in your browser and watch the soccer match unfold!

![](https://raw.githubusercontent.com/ntk148v/sse-fastapi/refs/heads/master/sse-sample.gif?token=GHSAT0AAAAAADCSDBD3ECSYB2HNRIMMT4ZI2CH2L7A)

## When to Use SSE? 🤓

Use SSE when:

- You need real-time updates
- The data flow is mostly server → client
- You want to keep things simple
- Your server is feeling particularly chatty

Don't use SSE when:

- You need two-way communication (use WebSocket)
- Your server has social anxiety
- You're building a multiplayer game (unless it's a very slow chess match)

## Real-World Use Cases & Best Practices 🎯

### Perfect Fits for SSE 🎯

1. **Live Sports Updates**
   - Real-time scores (like our example!)
   - Play-by-play commentary
   - Team statistics

2. **Financial Applications**
   - Stock price updates
   - Currency exchange rates
   - Trading notifications

3. **Social Media Features**
   - News feeds
   - Notification systems
   - Like/comment counters

4. **System Monitoring**
   - Server health metrics
   - Log streaming
   - Resource usage stats

5. **Content Management**
   - Content update notifications
   - Publishing status
   - Collaborative editing notifications

### Best Practices 🏆

1. **Connection Management**

   ```javascript
   // Always handle reconnection gracefully
   const connect = () => {
     const eventSource = new EventSource("/events");
     eventSource.onerror = (error) => {
       eventSource.close();
       setTimeout(connect, 5000); // Custom reconnect logic
     };
     return eventSource;
   };
   ```

2. **Event ID Tracking**

   ```python
   # Server-side
   async def generator():
       for event in events:
           yield f"id: {event.id}\ndata: {json.dumps(event.data)}\n\n"

   # Client-side
   eventSource.addEventListener('message', (e) => {
       localStorage.setItem('lastEventId', e.lastEventId);
   });
   ```

3. **Resource Management**
   - Keep payload sizes small (< 10KB recommended)
   - Batch updates when possible
   - Use compression for large datasets

   ```python
   # Batch updates example
   async def batch_generator():
       updates = []
       for event in events:
           updates.append(event)
           if len(updates) >= 5:
               yield f"data: {json.dumps(updates)}\n\n"
               updates = []
   ```

4. **Error Handling**
   - Always include error events
   - Implement custom retry logic when needed
   - Monitor connection health

   ```python
   # Server-side error handling
   @app.get("/stream")
   async def stream():
       try:
           return StreamingResponse(generator())
       except Exception as e:
           yield f"event: error\ndata: {str(e)}\n\n"
   ```

5. **Security Considerations**
   - Implement proper authentication
   - Use HTTPS in production
   - Rate limit connections per client

   ```python
   # Basic rate limiting example
   from fastapi import HTTPException

   CLIENTS = {}
   MAX_CONNECTIONS = 3

   @app.get("/stream")
   async def stream(request):
       client_ip = request.client.host
       if CLIENTS.get(client_ip, 0) >= MAX_CONNECTIONS:
           raise HTTPException(429, "Too many connections")
       CLIENTS[client_ip] = CLIENTS.get(client_ip, 0) + 1
   ```

6. **Production Tips**
   - Use a load balancer that supports long polling
   - Implement heartbeat messages
   - Monitor server resources
   ```python
   # Heartbeat example
   async def generator_with_heartbeat():
       while True:
           yield ":\n\n"  # Heartbeat
           await asyncio.sleep(30)
   ```

## The End 🎬

So there you have it! SSE is like having a friend who's happy to keep you updated without expecting you to respond. It's perfect for live scores, news feeds, status updates, or any situation where you just want to sit back and let the data flow.

Remember: In a world of chatty WebSockets, sometimes it's okay to be a quiet SSE. 🤫

## Contributing 🤝

Found a bug? Want to add features? Feel free to create an issue or submit a PR. Just remember to be nice - this is a judgment-free zone for both extroverted WebSockets and introverted SSEs!
