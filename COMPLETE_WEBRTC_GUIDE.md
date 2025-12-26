# Complete WebRTC Video Call Guide

## Table of Contents
1. [What is WebRTC?](#what-is-webrtc)
2. [Architecture Overview](#architecture-overview)
3. [Backend Logic Explained](#backend-logic-explained)
4. [Frontend Logic Explained](#frontend-logic-explained)
5. [Complete Call Flow](#complete-call-flow)
6. [Code Breakdown](#code-breakdown)
7. [Text-Based Diagrams](#text-based-diagrams)

---

## What is WebRTC?

**WebRTC (Web Real-Time Communication)** is a technology that allows:
- **Peer-to-Peer (P2P) Communication**: Direct audio/video/data transfer between browsers
- **No Plugins Required**: Built into modern browsers
- **Real-Time**: Low latency communication

### Key Concepts:

1. **Peer Connection**: Direct connection between two browsers
2. **Signaling**: Exchange of connection information (NOT part of WebRTC itself)
3. **ICE Candidates**: Network routes to reach each peer
4. **SDP (Session Description Protocol)**: Describes media capabilities (codecs, formats)

---

## Architecture Overview

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│   Browser A     │         │  NestJS Server  │         │   Browser B     │
│   (Frontend)    │◄───────►│   (Signaling)   │◄───────►│   (Frontend)    │
│                 │         │                 │         │                 │
│  - React App    │         │  - WebSocket    │         │  - React App    │
│  - WebRTC API   │         │  - Socket.IO    │         │  - WebRTC API   │
└─────────────────┘         └─────────────────┘         └─────────────────┘
         │                                                       │
         │                                                       │
         └───────────────────────────────────────────────────────┘
                    Direct P2P Connection (Audio/Video)
                         (After Signaling)
```

### Why Do We Need a Server?

**WebRTC is P2P, but we need a server for:**
1. **Signaling**: Exchange connection info (offer/answer/ICE candidates)
2. **Room Management**: Know who's in which room
3. **Discovery**: Help peers find each other

**The server does NOT handle media** - that goes directly peer-to-peer!

---

## Backend Logic Explained

### File: `backend/src/webrtc.gateway.ts`

```typescript
@WebSocketGateway({ cors: true })
export class WebRtcGateway {
  @WebSocketServer()
  server: Server;
```

**What this does:**
- `@WebSocketGateway`: Creates a WebSocket server using Socket.IO
- `cors: true`: Allows connections from different origins (frontend on port 5173)
- `server: Server`: Reference to the Socket.IO server instance

---

### 1. Join Room Handler

```typescript
@SubscribeMessage('join-room')
handleJoin(
  @MessageBody() roomId: string,
  @ConnectedSocket() client: Socket,
) {
  console.log(`[WebRTC] Client ${client.id} joining room: ${roomId}`);
  client.join(roomId);
  
  const room = this.server.sockets.adapter.rooms.get(roomId);
  const numClients = room ? room.size : 0;
  console.log(`[WebRTC] Room ${roomId} now has ${numClients} client(s)`);
  
  client.to(roomId).emit('user-joined');
  console.log(`[WebRTC] Notified other clients in room ${roomId} about new user`);
}
```

**Step-by-step:**
1. User sends `join-room` event with a room ID
2. Server adds the user's socket to that room
3. Server counts how many users are in the room
4. Server notifies OTHER users in the room that someone joined
5. The new user does NOT receive `user-joined` (only existing users do)

**Why?**
- First user joins → No one to notify → No `user-joined` event
- Second user joins → First user gets `user-joined` → First user becomes initiator

---

### 2. Offer Handler

```typescript
@SubscribeMessage('offer')
handleOffer(
  @MessageBody() data: { roomId: string; offer: any },
  @ConnectedSocket() client: Socket,
) {
  console.log(`[WebRTC] Forwarding offer in room ${data.roomId}`);
  client.to(data.roomId).emit('offer', data.offer);
}
```

**What this does:**
- Receives an offer from User A
- Forwards it to all OTHER users in the same room (User B)
- The offer contains SDP (media capabilities)

---

### 3. Answer Handler

```typescript
@SubscribeMessage('answer')
handleAnswer(
  @MessageBody() data: { roomId: string; answer: any },
  @ConnectedSocket() client: Socket,
) {
  console.log(`[WebRTC] Forwarding answer in room ${data.roomId}`);
  client.to(data.roomId).emit('answer', data.answer);
}
```

**What this does:**
- Receives an answer from User B
- Forwards it back to User A
- The answer also contains SDP

---

### 4. ICE Candidate Handler

```typescript
@SubscribeMessage('ice-candidate')
handleIce(
  @MessageBody() data: { roomId: string; candidate: any },
  @ConnectedSocket() client: Socket,
) {
  console.log(`[WebRTC] Forwarding ICE candidate in room ${data.roomId}`);
  client.to(data.roomId).emit('ice-candidate', data.candidate);
}
```

**What this does:**
- Receives ICE candidates (network routes) from one user
- Forwards them to the other user
- Both users exchange multiple ICE candidates

**ICE Candidates contain:**
- IP addresses
- Ports
- Network protocols (UDP/TCP)
- Priority information

---

## Frontend Logic Explained

### File: `frontend/src/VideoCall.tsx`

### 1. State Management

```typescript
const [isMicOn, setIsMicOn] = useState(true);
const [isCamOn, setIsCamOn] = useState(true);
const [isInitiator, setIsInitiator] = useState(false);
const [callStarted, setCallStarted] = useState(false);
const [hasIncomingOffer, setHasIncomingOffer] = useState(false);
const [pendingOffer, setPendingOffer] = useState<RTCSessionDescriptionInit | null>(null);
```

**State Variables:**
- `isMicOn/isCamOn`: Track mic/camera status
- `isInitiator`: Is this user the first one (creates offer)?
- `callStarted`: Has the P2P connection been established?
- `hasIncomingOffer`: Did we receive an offer?
- `pendingOffer`: Store the offer until user clicks "Accept"

---

### 2. Initialize Media

```typescript
async function initializeMedia() {
  // 1. Get camera + microphone
  const stream = await navigator.mediaDevices.getUserMedia({
    audio: true,
    video: true,
  });

  localStreamRef.current = stream;
  if (localVideoRef.current) {
    localVideoRef.current.srcObject = stream;
  }

  // 2. Create RTCPeerConnection
  const pc = new RTCPeerConnection({
    iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
  });

  pcRef.current = pc;

  // 3. Add local tracks to PeerConnection
  stream.getTracks().forEach((track) => {
    pc.addTrack(track, stream);
  });
```

**Step-by-step:**
1. **getUserMedia**: Ask browser for camera/mic access
2. **Display local video**: Show your own video
3. **Create PeerConnection**: Create WebRTC connection object
4. **Add tracks**: Add your audio/video to the connection

**STUN Server:**
- `stun:stun.l.google.com:19302`: Google's public STUN server
- STUN = Session Traversal Utilities for NAT
- Helps discover your public IP address for P2P connection

---

### 3. Handle Remote Tracks

```typescript
pc.ontrack = (event: RTCTrackEvent) => {
  console.log("Received remote track:", event.track.kind);
  if (remoteVideoRef.current) {
    remoteVideoRef.current.srcObject = event.streams[0];
  }
};
```

**What this does:**
- When the other user's audio/video arrives
- Display it in the remote video element
- `event.streams[0]`: The remote user's media stream

---

### 4. Handle ICE Candidates

```typescript
pc.onicecandidate = (event: RTCPeerConnectionIceEvent) => {
  if (event.candidate) {
    console.log("Sending ICE candidate");
    socket.emit("ice-candidate", {
      roomId,
      candidate: event.candidate,
    });
  }
};
```

**What this does:**
- Browser discovers network routes (ICE candidates)
- Send each candidate to the other user via signaling server
- Other user adds these candidates to their connection

---

### 5. Socket Event Listeners

```typescript
socket.on("user-joined", async () => {
  console.log("Another user joined the room. You can now start the call.");
  setIsInitiator(true);
});

socket.on("offer", async (offer: RTCSessionDescriptionInit) => {
  console.log("Received offer:", offer);
  setPendingOffer(offer);
  setHasIncomingOffer(true);
});

socket.on("answer", async (answer: RTCSessionDescriptionInit) => {
  console.log("Received answer:", answer);
  await handleAnswer(answer);
});

socket.on("ice-candidate", async (candidate: RTCIceCandidateInit) => {
  console.log("Received ICE candidate:", candidate);
  await handleIceCandidate(candidate);
});
```

**Event Flow:**
1. `user-joined`: Someone joined → You become initiator
2. `offer`: Received offer → Show "Accept Call" button
3. `answer`: Received answer → Complete connection
4. `ice-candidate`: Received network route → Add to connection

---

### 6. Create Offer (Initiator)

```typescript
async function createOffer() {
  if (!pcRef.current) {
    console.error("PeerConnection not initialized");
    return;
  }

  try {
    console.log("Creating offer...");
    const offer = await pcRef.current.createOffer();
    await pcRef.current.setLocalDescription(offer);
    console.log("Sending offer:", offer);
    socket.emit("offer", { roomId, offer });
  } catch (error) {
    console.error("Error creating offer:", error);
  }
}
```

**What this does:**
1. Create an SDP offer (describes your media capabilities)
2. Set it as your local description
3. Send it to the other user via signaling server

**SDP Offer contains:**
- Audio/video codecs you support
- Media formats
- Network information

---

### 7. Handle Offer (Receiver)

```typescript
async function handleOffer(offer: RTCSessionDescriptionInit) {
  if (!pcRef.current) {
    console.error("PeerConnection not initialized");
    return;
  }

  try {
    await pcRef.current.setRemoteDescription(new RTCSessionDescription(offer));
    console.log("Creating answer...");
    const answer = await pcRef.current.createAnswer();
    await pcRef.current.setLocalDescription(answer);
    console.log("Sending answer:", answer);
    socket.emit("answer", { roomId, answer });
  } catch (error) {
    console.error("Error handling offer:", error);
  }
}
```

**What this does:**
1. Set the received offer as remote description
2. Create an answer (your media capabilities)
3. Set answer as your local description
4. Send answer back to initiator

---

### 8. Handle Answer (Initiator)

```typescript
async function handleAnswer(answer: RTCSessionDescriptionInit) {
  if (!pcRef.current) {
    console.error("PeerConnection not initialized");
    return;
  }

  try {
    await pcRef.current.setRemoteDescription(new RTCSessionDescription(answer));
    console.log("Answer set successfully");
  } catch (error) {
    console.error("Error handling answer:", error);
  }
}
```

**What this does:**
- Set the received answer as remote description
- Connection is now established!
- ICE candidates will continue to be exchanged

---

## Complete Call Flow

### Text-Based Sequence Diagram

```
User A (First)          Signaling Server          User B (Second)
     |                         |                         |
     |--[join-room: 1234]----->|                         |
     |                         |                         |
     |<--[joined room 1234]----| (No user-joined event)  |
     |                         |                         |
     | (Waiting...)            |                         |
     |                         |                         |
     |                         |<--[join-room: 1234]-----|
     |                         |                         |
     |<--[user-joined]---------|---[joined room 1234]--->|
     |                         |                         |
     | (Becomes Initiator)     |                         |
     | (Shows "Start Call")    |    (Shows "Waiting")    |
     |                         |                         |
     |                         |                         |
[User A clicks "Start Call"]   |                         |
     |                         |                         |
     |--[offer: SDP]---------->|                         |
     |                         |                         |
     |                         |--[offer: SDP]---------->|
     |                         |                         |
     |                         |    (Shows "Accept Call")|
     |                         |                         |
     |                         |   [User B clicks Accept]|
     |                         |                         |
     |                         |<--[answer: SDP]---------|
     |                         |                         |
     |<--[answer: SDP]---------|                         |
     |                         |                         |
     |                         |                         |
     |--[ICE candidate]------->|                         |
     |                         |--[ICE candidate]------->|
     |                         |                         |
     |                         |<--[ICE candidate]-------|
     |<--[ICE candidate]-------|                         |
     |                         |                         |
     | (Multiple ICE candidates exchanged)               |
     |<===================================================>|
     |                                                    |
     |          Direct P2P Connection Established!        |
     |          (Audio/Video flows directly)              |
     |                                                    |
```

---

## Detailed Step-by-Step Flow

### Phase 1: Room Setup

```
Step 1: User A opens app
  ├─ Enters room ID: "1234"
  ├─ Clicks "Join Room"
  └─ Browser requests camera/mic permission

Step 2: User A's browser
  ├─ Gets media stream (camera + mic)
  ├─ Creates RTCPeerConnection
  ├─ Adds local tracks to peer connection
  ├─ Displays local video
  └─ Sends "join-room: 1234" to server

Step 3: Server
  ├─ Adds User A's socket to room "1234"
  ├─ Counts clients in room: 1
  ├─ No one else in room → No "user-joined" event
  └─ User A waits...

Step 4: User B opens app
  ├─ Enters same room ID: "1234"
  ├─ Clicks "Join Room"
  ├─ Gets media stream
  ├─ Creates RTCPeerConnection
  └─ Sends "join-room: 1234" to server

Step 5: Server
  ├─ Adds User B's socket to room "1234"
  ├─ Counts clients in room: 2
  ├─ Sends "user-joined" to User A (NOT to User B)
  └─ User A receives "user-joined"

Step 6: User A
  ├─ Receives "user-joined" event
  ├─ Sets isInitiator = true
  └─ Shows "Start Call" button
```

### Phase 2: Signaling (Offer/Answer Exchange)

```
Step 7: User A clicks "Start Call"
  ├─ Calls createOffer()
  ├─ pc.createOffer() → Generates SDP offer
  ├─ pc.setLocalDescription(offer)
  └─ Sends offer to server via socket

Step 8: Server
  ├─ Receives offer from User A
  └─ Forwards offer to User B

Step 9: User B
  ├─ Receives offer
  ├─ Stores in pendingOffer
  ├─ Shows "Accept Call" button
  └─ Waits for user action

Step 10: User B clicks "Accept Call"
  ├─ Calls handleOffer(pendingOffer)
  ├─ pc.setRemoteDescription(offer) ← Sets User A's SDP
  ├─ pc.createAnswer() → Generates SDP answer
  ├─ pc.setLocalDescription(answer)
  └─ Sends answer to server via socket

Step 11: Server
  ├─ Receives answer from User B
  └─ Forwards answer to User A

Step 12: User A
  ├─ Receives answer
  ├─ pc.setRemoteDescription(answer) ← Sets User B's SDP
  └─ Connection negotiation complete!
```

### Phase 3: ICE Candidate Exchange

```
Step 13: Both browsers (parallel process)
  ├─ Browser discovers network routes (ICE candidates)
  ├─ onicecandidate event fires multiple times
  ├─ Each candidate sent to server
  └─ Server forwards to other peer

Step 14: User A
  ├─ Discovers ICE candidate #1 → Send to User B
  ├─ Discovers ICE candidate #2 → Send to User B
  └─ Receives User B's candidates → Add to connection

Step 15: User B
  ├─ Discovers ICE candidate #1 → Send to User A
  ├─ Discovers ICE candidate #2 → Send to User A
  └─ Receives User A's candidates → Add to connection

Step 16: Connection Established
  ├─ Browsers test all candidate pairs
  ├─ Find best route (lowest latency)
  ├─ Connection state: "connected"
  └─ Media flows directly P2P!
```

### Phase 4: Media Streaming

```
Step 17: Direct P2P Connection
  ├─ Audio/video flows directly between browsers
  ├─ Server is NO LONGER involved
  ├─ ontrack event fires on both sides
  └─ Remote video displayed

Step 18: User A sees
  ├─ Local video (their own camera)
  └─ Remote video (User B's camera)

Step 19: User B sees
  ├─ Local video (their own camera)
  └─ Remote video (User A's camera)
```

---

## Code Breakdown by Feature

### 1. Room Management

**Backend:**
```typescript
client.join(roomId);  // Add socket to room
client.to(roomId).emit('event', data);  // Send to others in room
```

**Frontend:**
```typescript
socket.emit("join-room", roomId);  // Join a room
```

---

### 2. Media Capture

```typescript
const stream = await navigator.mediaDevices.getUserMedia({
  audio: true,  // Request microphone
  video: true,  // Request camera
});
```

**What happens:**
1. Browser shows permission prompt
2. User allows/denies
3. If allowed, returns MediaStream object
4. MediaStream contains audio and video tracks

---

### 3. Peer Connection Setup

```typescript
const pc = new RTCPeerConnection({
  iceServers: [{ urls: "stun:stun.l.google.com:19302" }],
});
```

**ICE Servers:**
- **STUN**: Discover your public IP
- **TURN**: Relay media if P2P fails (not used here)

---

### 4. Adding Tracks

```typescript
stream.getTracks().forEach((track) => {
  pc.addTrack(track, stream);
});
```

**What this does:**
- Get all tracks from stream (audio + video)
- Add each track to peer connection
- These tracks will be sent to the other peer

---

### 5. Receiving Tracks

```typescript
pc.ontrack = (event: RTCTrackEvent) => {
  remoteVideoRef.current.srcObject = event.streams[0];
};
```

**What this does:**
- Fires when remote track arrives
- Attach remote stream to video element
- Video automatically plays

---

### 6. Toggle Mic/Camera

```typescript
function toggleMic() {
  localStreamRef.current.getAudioTracks().forEach((track) => {
    track.enabled = !track.enabled;  // Mute/unmute
    setIsMicOn(track.enabled);
  });
}

function toggleCamera() {
  localStreamRef.current.getVideoTracks().forEach((track) => {
    track.enabled = !track.enabled;  // On/off
    setIsCamOn(track.enabled);
  });
}
```

**How it works:**
- `track.enabled = false`: Stops sending media (mute/hide)
- `track.enabled = true`: Resumes sending media
- Track is NOT stopped, just disabled

---

## Key Concepts Explained

### 1. SDP (Session Description Protocol)

**What is it?**
A text format describing media capabilities.

**Example SDP Offer:**
```
v=0
o=- 123456789 2 IN IP4 127.0.0.1
s=-
t=0 0
m=audio 9 UDP/TLS/RTP/SAVPF 111 103
a=rtpmap:111 opus/48000/2
m=video 9 UDP/TLS/RTP/SAVPF 96 97
a=rtpmap:96 VP8/90000
```

**Contains:**
- Media types (audio/video)
- Codecs (Opus, VP8, H.264)
- Network info
- Encryption keys

---

### 2. ICE (Interactive Connectivity Establishment)

**What is it?**
A framework to find the best network path between peers.

**ICE Candidate Types:**
1. **Host**: Local network address (192.168.x.x)
2. **Server Reflexive**: Public IP (discovered via STUN)
3. **Relay**: TURN server address (fallback)

**Example ICE Candidate:**
```json
{
  "candidate": "candidate:1 1 UDP 2130706431 192.168.1.100 54321 typ host",
  "sdpMid": "0",
  "sdpMLineIndex": 0
}
```

---

### 3. Signaling

**What is it?**
The process of exchanging connection info.

**What needs to be signaled:**
1. Offer (SDP)
2. Answer (SDP)
3. ICE Candidates

**Signaling is NOT standardized** - we use Socket.IO, but you could use:
- WebSockets
- HTTP polling
- Firebase
- Any messaging system

---

## Common Issues & Solutions

### Issue 1: "Failed to load resource: 404 (Not Found)"

**Cause:** Socket.IO packages not installed

**Solution:**
```bash
npm install @nestjs/platform-socket.io @nestjs/websockets socket.io
```

---

### Issue 2: No video/audio

**Possible causes:**
1. Camera/mic permissions denied
2. Offer/answer not exchanged
3. ICE candidates not exchanged
4. Firewall blocking connection

**Debug steps:**
1. Check browser console for errors
2. Verify "Connection state: connected"
3. Check "ICE connection state: connected"
4. Verify tracks are added: `pc.getSenders()`

---

### Issue 3: Connection stuck in "checking"

**Cause:** ICE candidates not reaching peer

**Solution:**
1. Check signaling server is running
2. Verify ICE candidates are being sent
3. Add TURN server for NAT traversal

---

## Advanced Topics

### Adding TURN Server

```typescript
const pc = new RTCPeerConnection({
  iceServers: [
    { urls: "stun:stun.l.google.com:19302" },
    {
      urls: "turn:your-turn-server.com:3478",
      username: "user",
      credential: "pass"
    }
  ],
});
```

**When needed:**
- Symmetric NAT
- Corporate firewalls
- Mobile networks

---

### Adding Text Chat

**Backend:**
```typescript
@SubscribeMessage('chat-message')
handleChat(
  @MessageBody() data: { roomId: string; message: string },
  @ConnectedSocket() client: Socket,
) {
  client.to(data.roomId).emit('chat-message', data.message);
}
```

**Frontend:**
```typescript
socket.on('chat-message', (message) => {
  setMessages(prev => [...prev, message]);
});
```

---

### Screen Sharing

```typescript
async function startScreenShare() {
  const screenStream = await navigator.mediaDevices.getDisplayMedia({
    video: true
  });
  
  const videoTrack = screenStream.getVideoTracks()[0];
  const sender = pcRef.current.getSenders()
    .find(s => s.track?.kind === 'video');
  
  sender.replaceTrack(videoTrack);
}
```

---

## Summary

### What We Built:

1. **Backend (NestJS)**
   - WebSocket server using Socket.IO
   - Room management
   - Signaling relay (offer/answer/ICE)

2. **Frontend (React)**
   - Media capture (camera/mic)
   - WebRTC peer connection
   - UI for call controls
   - Socket.IO client

### Key Takeaways:

✅ **WebRTC is P2P** - Media flows directly between browsers
✅ **Signaling is required** - To exchange connection info
✅ **Server doesn't handle media** - Only coordinates connection
✅ **ICE candidates** - Find best network route
✅ **SDP** - Describes media capabilities
✅ **STUN** - Discover public IP
✅ **TURN** - Relay media if P2P fails (not implemented here)

### Flow Summary:

```
1. Join Room → 2. Exchange Offer/Answer → 3. Exchange ICE → 4. Connect P2P → 5. Stream Media
```

---

## Testing Your App

### Step 1: Start Backend
```bash
cd backend
npm run start:dev
```

### Step 2: Start Frontend
```bash
cd frontend
npm run dev
```

### Step 3: Open Two Browsers
1. Open `http://localhost:5173` in two tabs
2. Enter same room ID (e.g., "1234") in both
3. First tab: Click "Start Call"
4. Second tab: Click "Accept Call"
5. Both should see each other's video!

---

## Next Steps

1. **Add TURN server** for better connectivity
2. **Add reconnection logic** if connection drops
3. **Add screen sharing** feature
4. **Add text chat** alongside video
5. **Add recording** capability
6. **Support group calls** (3+ users)
7. **Add call quality indicators**
8. **Add noise cancellation**

---

## Resources

- [WebRTC Official Docs](https://webrtc.org/)
- [MDN WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [Socket.IO Docs](https://socket.io/docs/)
- [NestJS WebSockets](https://docs.nestjs.com/websockets/gateways)

---

**Happy Coding! 🚀**
