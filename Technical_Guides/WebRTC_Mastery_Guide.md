# WebRTC Mastery Guide: Zero to Hero (Implementation Handbook)

> **Goal**: Build a production-ready Audio/Video calling app (1-on-1 & Group).
> **Stack**: React (Frontend) + NestJS (Signaling Backend).

---

## Part 1: The Missing Piece — "Signaling" Implementation

The theory tells you *what* detailed architecture looks like. This section tells you *how* to code it.

### 1.1 NestJS Signaling Gateway (`events.gateway.ts`)

You need a WebSocket gateway to exchange the SDP (session descriptions) and ICE candidates.

```typescript
// backend/src/call/call.gateway.ts
import {
  WebSocketGateway,
  WebSocketServer,
  SubscribeMessage,
  OnGatewayDisconnect,
  ConnectedSocket,
  MessageBody,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({ cors: { origin: '*' } })
export class CallGateway implements OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  // Map to track active sockets in rooms: roomId -> string[] (socketIds)
  private roomUsers: Map<string, string[]> = new Map();

  // 1. Join a Room
  @SubscribeMessage('join-room')
  handleJoinRoom(
    @ConnectedSocket() client: Socket,
    @MessageBody() payload: { roomId: string; userId: string },
  ) {
    const { roomId } = payload;
    client.join(roomId);

    // Track user in room
    if (!this.roomUsers.has(roomId)) {
      this.roomUsers.set(roomId, []);
    }
    const users = this.roomUsers.get(roomId);
    users.push(client.id);

    // Notify others in room that a new user joined
    // They will initiate the offer to this new user (Mesh topology)
    client.to(roomId).emit('user-connected', { userId: client.id });

    // Send list of existing users to the new joiner (so they know who to expect offers from)
    const existingUsers = users.filter((id) => id !== client.id);
    client.emit('all-users', existingUsers);
    
    console.log(`User ${client.id} joined room ${roomId}`);
  }

  // 2. Relay Offer (SDP)
  @SubscribeMessage('offer')
  handleOffer(@ConnectedSocket() client: Socket, @MessageBody() payload: any) {
    // payload: { sdp, targetSocketId }
    // We send the offer ONLY to the specific target user, not the whole room
    this.server.to(payload.targetSocketId).emit('offer', {
      sdp: payload.sdp,
      callerId: client.id,
    });
  }

  // 3. Relay Answer (SDP)
  @SubscribeMessage('answer')
  handleAnswer(@ConnectedSocket() client: Socket, @MessageBody() payload: any) {
    // payload: { sdp, targetSocketId }
    this.server.to(payload.targetSocketId).emit('answer', {
      sdp: payload.sdp,
      responderId: client.id,
    });
  }

  // 4. Relay ICE Candidate
  @SubscribeMessage('ice-candidate')
  handleIceCandidate(@ConnectedSocket() client: Socket, @MessageBody() payload: any) {
    // payload: { candidate, targetSocketId }
    this.server.to(payload.targetSocketId).emit('ice-candidate', {
      candidate: payload.candidate,
      senderId: client.id, // Knowing who sent the candidate is crucial for mesh
    });
  }

  handleDisconnect(client: Socket) {
    // Cleanup logic: remove user from rooms and notify others
    this.roomUsers.forEach((users, roomId) => {
      if (users.includes(client.id)) {
        const updatedUsers = users.filter((id) => id !== client.id);
        this.roomUsers.set(roomId, updatedUsers);
        this.server.to(roomId).emit('user-disconnected', { userId: client.id });
      }
    });
  }
}
```

---

## Part 2: React Frontend — 1-to-1 Implementation

For 1-to-1, you only need **one** `RTCPeerConnection`.

### 2.1 The Hook (`useWebRTC`)

```typescript
// frontend/src/hooks/useWebRTC.ts
import { useEffect, useRef, useState } from 'react';
import io, { Socket } from 'socket.io-client';

const STUN_SERVERS = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' }, // Free Google STUN
    // { urls: 'turn:your-turn-server.com', username: '...', credential: '...' } // REQUIRED for production
  ],
};

export const useWebRTC = (roomId: string, userId: string) => {
  const [localStream, setLocalStream] = useState<MediaStream | null>(null);
  const [remoteStream, setRemoteStream] = useState<MediaStream | null>(null);
  const pc = useRef<RTCPeerConnection | null>(null);
  const socket = useRef<Socket | null>(null);

  useEffect(() => {
    // 1. Initialize Socket
    socket.current = io('http://localhost:3000'); // Your backend URL

    // 2. Get User Media
    navigator.mediaDevices.getUserMedia({ video: true, audio: true })
      .then((stream) => {
        setLocalStream(stream);
        
        // 3. Create Peer Connection once we have the stream
        pc.current = new RTCPeerConnection(STUN_SERVERS);

        // Add local tracks to PC
        stream.getTracks().forEach((track) => {
          pc.current?.addTrack(track, stream);
        });

        // Handle remote tracks
        pc.current.ontrack = (event) => {
          setRemoteStream(event.streams[0]);
        };

        // Handle ICE candidates
        pc.current.onicecandidate = (event) => {
          if (event.candidate) {
            socket.current?.emit('ice-candidate', {
              candidate: event.candidate,
              targetSocketId: otherUserId, // You need to store the other user's ID
              roomId,
            });
          }
        };

        // Join the room
        socket.current.emit('join-room', { roomId, userId });
      });

    // 4. Socket Events
    socket.current.on('user-connected', async ({ userId: newUserId }) => {
       // YOU are the caller
       const offer = await pc.current?.createOffer();
       await pc.current?.setLocalDescription(offer);
       socket.current?.emit('offer', { sdp: offer, targetSocketId: newUserId });
    });

    socket.current.on('offer', async ({ sdp, callerId }) => {
       // YOU are the receiver
       await pc.current?.setRemoteDescription(new RTCSessionDescription(sdp));
       const answer = await pc.current?.createAnswer();
       await pc.current?.setLocalDescription(answer);
       socket.current?.emit('answer', { sdp: answer, targetSocketId: callerId });
    });

    socket.current.on('answer', async ({ sdp }) => {
       await pc.current?.setRemoteDescription(new RTCSessionDescription(sdp));
    });

    socket.current.on('ice-candidate', async ({ candidate }) => {
       await pc.current?.addIceCandidate(new RTCIceCandidate(candidate));
    });

    return () => {
      pc.current?.close();
      socket.current?.disconnect();
    };
  }, [roomId]);

  return { localStream, remoteStream };
};
```

---

## Part 3: Group Calls (Mesh Topology) — The Hard Part

For group calls, **you cannot use a single `RTCPeerConnection`**.
You need **one PC per participant**.

If there are 4 people (A, B, C, D):
*   A needs connections to B, C, D (3 connections).
*   B needs connections to A, C, D (3 connections).

### Refactoring for Group Calls

1.  **State**: Instead of `pc.current`, use `peersRef.current = { [socketId]: RTCPeerConnection }`.
2.  **Creation**:
    *   When `user-connected` fires: Create a **new** PC for that user. Store it in map. Create Offer.
    *   When `offer` arrives: Create a **new** PC for that caller. Store it in map. Create Answer.
3.  **Streams**: You will have an array of streams, not just one `remoteStream`.

```typescript
// Managing multiple peers
const peersRef = useRef<{ [key: string]: RTCPeerConnection }>({});
const [remoteStreams, setRemoteStreams] = useState<{ [key: string]: MediaStream }>({});

// Helper to create a peer connection for a specific user
const createPeer = (targetUserId: string, initiator: boolean) => {
  const pc = new RTCPeerConnection(STUN_SERVERS);
  
  // Add local tracks
  if (localStream) {
    localStream.getTracks().forEach(track => pc.addTrack(track, localStream));
  }

  // Handle incoming stream from THIS specific user
  pc.ontrack = (event) => {
    setRemoteStreams(prev => ({ ...prev, [targetUserId]: event.streams[0] }));
  };

  // Handle ICE
  pc.onicecandidate = (event) => {
    if (event.candidate) {
      socket.current.emit('ice-candidate', {
        candidate: event.candidate,
        targetSocketId: targetUserId
      });
    }
  };

  peersRef.current[targetUserId] = pc;
  return pc;
}
```

---

## Part 4: Screen Sharing Logic

Screen sharing is just **replacing the video track**. You don't need to hang up.

```typescript
const startScreenShare = async () => {
  const screenStream = await navigator.mediaDevices.getDisplayMedia({ video: true });
  const screenTrack = screenStream.getVideoTracks()[0];

  // Replace video track in ALL active peer connections
  Object.values(peersRef.current).forEach((pc) => {
    const sender = pc.getSenders().find(s => s.track?.kind === 'video');
    if (sender) {
      sender.replaceTrack(screenTrack);
    }
  });

  // Handle user clicking "Stop Sharing" on the browser native UI
  screenTrack.onended = () => {
    stopScreenShare();
  };
  
  setLocalStream(screenStream); // Update local view
};

const stopScreenShare = async () => {
  // Get camera back
  const cameraStream = await navigator.mediaDevices.getUserMedia({ video: true, audio: true });
  const videoTrack = cameraStream.getVideoTracks()[0];

  Object.values(peersRef.current).forEach((pc) => {
     const sender = pc.getSenders().find(s => s.track?.kind === 'video');
     if (sender) sender.replaceTrack(videoTrack);
  });
  
  setLocalStream(cameraStream);
};
```

---

## Part 5: Production Checklist (The "Gotchas")

Your app will work locally but **FAIL** when users are on different Wi-Fi networks.

**Why?** Firewalls and NATs block direct connections.

**The Fix: TURN Servers.**
You absolutely need a TURN server for production.

1.  **Free Option**: None that are reliable for production.
2.  **Self-Host**: Set up `coturn` on an AWS EC2 instance (Free Tier works for small tests).
3.  **Paid Service**: Twilio Network Traversal, Xirsys, Metered.ca.

**Configuring TURN:**
```js
const STUN_SERVERS = {
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    {
      urls: 'turn:global.turn.twilio.com:3478',
      username: 'your-twilio-username',
      credential: 'your-twilio-password'
    }
  ]
};
```

---

## Summary of Files You Need

1.  **Backend**: `call.gateway.ts` (handle signaling events).
2.  **Frontend Hook**: `useWebRTC.ts` (manage logic).
3.  **Frontend Component**: `VideoRoom.tsx` (UI for rendering `<video>` tags).

### VideoRoom.tsx Component Structure
```tsx
const VideoRoom = () => {
  const { localStream, remoteStreams } = useWebRTC(roomId, userId);

  return (
    <div className="grid grid-cols-2 gap-4">
      {/* My Video */}
      <div className="relative">
         <video ref={video => video && (video.srcObject = localStream)} autoPlay muted />
         <span>Me</span>
      </div>

      {/* Remote Videos */}
      {Object.entries(remoteStreams).map(([peerId, stream]) => (
        <div key={peerId} className="relative">
           <video ref={video => video && (video.srcObject = stream)} autoPlay />
           <span>User {peerId}</span>
        </div>
      ))}
    </div>
  );
};
```

This guide now bridges the gap between "knowing WebRTC" and "building WebRTC". Start with Part 1 & 2 for a basic call, then implement Part 3 for groups.
