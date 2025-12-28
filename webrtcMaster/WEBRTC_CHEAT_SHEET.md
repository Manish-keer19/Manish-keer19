# 🎯 WebRTC Quick Reference Cheat Sheet

## 📚 Essential Concepts

### **WebRTC = Peer-to-Peer Communication**
```
Traditional:  User A → Server → User B (expensive!)
WebRTC:       User A ←─────────→ User B (direct!)
```

### **WebSocket = Signaling Only**
```
WebSocket:  Helps users find each other
WebRTC:     Actual video/audio transfer
```

---

## 🧩 Core Components

### **1. MediaStream**
```typescript
// Get camera + microphone
const stream = await navigator.mediaDevices.getUserMedia({
    video: true,
    audio: true
});

// Access tracks
stream.getVideoTracks()[0]  // Camera
stream.getAudioTracks()[0]  // Microphone

// Mute/unmute
track.enabled = false;  // Mute
track.enabled = true;   // Unmute

// Stop completely
stream.getTracks().forEach(t => t.stop());
```

### **2. RTCPeerConnection**
```typescript
// Create connection
const pc = new RTCPeerConnection({
    iceServers: [
        { urls: 'stun:stun.l.google.com:19302' }
    ]
});

// Add your media
localStream.getTracks().forEach(track => {
    pc.addTrack(track, localStream);
});

// Receive partner's media
pc.ontrack = (event) => {
    const [remoteStream] = event.streams;
    // Display remoteStream
};

// Handle ICE candidates
pc.onicecandidate = (event) => {
    if (event.candidate) {
        // Send to partner via WebSocket
        socket.emit('ice-candidate', event.candidate);
    }
};
```

### **3. SDP (Offer/Answer)**
```typescript
// INITIATOR creates offer
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
socket.emit('offer', { offer });

// RECEIVER handles offer
socket.on('offer', async ({ offer }) => {
    await pc.setRemoteDescription(offer);
    const answer = await pc.createAnswer();
    await pc.setLocalDescription(answer);
    socket.emit('answer', { answer });
});

// INITIATOR handles answer
socket.on('answer', async ({ answer }) => {
    await pc.setRemoteDescription(answer);
});
```

### **4. ICE Candidates**
```typescript
// Send your candidates
pc.onicecandidate = (event) => {
    if (event.candidate) {
        socket.emit('ice-candidate', event.candidate);
    }
};

// Receive partner's candidates
socket.on('ice-candidate', async (candidate) => {
    await pc.addIceCandidate(new RTCIceCandidate(candidate));
});
```

---

## 🔄 Complete Connection Flow

```typescript
// STEP 1: Get media
const stream = await getUserMedia({ video: true, audio: true });

// STEP 2: Create peer connection
const pc = new RTCPeerConnection(config);

// STEP 3: Add tracks
stream.getTracks().forEach(t => pc.addTrack(t, stream));

// STEP 4: Handle remote tracks
pc.ontrack = (e) => setRemoteStream(e.streams[0]);

// STEP 5: Handle ICE candidates
pc.onicecandidate = (e) => {
    if (e.candidate) socket.emit('ice', e.candidate);
};

// STEP 6: Create offer (if initiator)
const offer = await pc.createOffer();
await pc.setLocalDescription(offer);
socket.emit('offer', offer);

// STEP 7: Handle offer (if receiver)
socket.on('offer', async (offer) => {
    await pc.setRemoteDescription(offer);
    const answer = await pc.createAnswer();
    await pc.setLocalDescription(answer);
    socket.emit('answer', answer);
});

// STEP 8: Handle answer (if initiator)
socket.on('answer', async (answer) => {
    await pc.setRemoteDescription(answer);
});

// STEP 9: Handle ICE candidates
socket.on('ice', async (candidate) => {
    await pc.addIceCandidate(candidate);
});

// STEP 10: Connection established! 🎉
```

---

## 🏢 NestJS Gateway (Backend)

### **Basic Setup**
```typescript
@WebSocketGateway({
    namespace: 'video-chat',
    cors: { origin: '*' }
})
export class VideoChatGateway {
    @WebSocketServer()
    server: Server;
    
    private matches = new Map<string, string>();
    
    @SubscribeMessage('offer')
    handleOffer(@MessageBody() data, @ConnectedSocket() client: Socket) {
        const partnerId = this.matches.get(client.id);
        if (partnerId) {
            this.server.to(partnerId).emit('offer', data);
        }
    }
    
    @SubscribeMessage('answer')
    handleAnswer(@MessageBody() data, @ConnectedSocket() client: Socket) {
        const partnerId = this.matches.get(client.id);
        if (partnerId) {
            this.server.to(partnerId).emit('answer', data);
        }
    }
    
    @SubscribeMessage('ice-candidate')
    handleIce(@MessageBody() data, @ConnectedSocket() client: Socket) {
        const partnerId = this.matches.get(client.id);
        if (partnerId) {
            this.server.to(partnerId).emit('ice-candidate', data);
        }
    }
}
```

---

## 🎛️ Connection States

### **RTCPeerConnection States**
```typescript
pc.onconnectionstatechange = () => {
    switch (pc.connectionState) {
        case 'new':          // Just created
        case 'connecting':   // Attempting connection
        case 'connected':    // ✅ Connected!
        case 'disconnected': // ⚠️ Temporarily lost
        case 'failed':       // ❌ Connection failed
        case 'closed':       // 🔒 Connection closed
    }
};
```

### **ICE Connection States**
```typescript
pc.oniceconnectionstatechange = () => {
    switch (pc.iceConnectionState) {
        case 'new':          // Starting
        case 'checking':     // Testing candidates
        case 'connected':    // ✅ At least one path works
        case 'completed':    // ✅ All checks done
        case 'failed':       // ❌ No path works
        case 'disconnected': // ⚠️ Lost connection
        case 'closed':       // 🔒 Closed
    }
};
```

---

## 🔧 Common Operations

### **Mute/Unmute Audio**
```typescript
const toggleMute = () => {
    localStream.getAudioTracks().forEach(track => {
        track.enabled = !track.enabled;
    });
};
```

### **Turn Video On/Off**
```typescript
const toggleVideo = () => {
    localStream.getVideoTracks().forEach(track => {
        track.enabled = !track.enabled;
    });
};
```

### **Switch Camera (Front/Back)**
```typescript
const switchCamera = async () => {
    const newStream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: facingMode === 'user' ? 'environment' : 'user' }
    });
    
    const newTrack = newStream.getVideoTracks()[0];
    const sender = pc.getSenders().find(s => s.track?.kind === 'video');
    
    await sender.replaceTrack(newTrack);
};
```

### **Screen Sharing**
```typescript
const shareScreen = async () => {
    const screenStream = await navigator.mediaDevices.getDisplayMedia({
        video: true
    });
    
    const screenTrack = screenStream.getVideoTracks()[0];
    const sender = pc.getSenders().find(s => s.track?.kind === 'video');
    
    await sender.replaceTrack(screenTrack);
};
```

### **End Call**
```typescript
const endCall = () => {
    // Close peer connection
    pc.close();
    
    // Stop all tracks
    localStream.getTracks().forEach(track => track.stop());
    
    // Notify partner
    socket.emit('call-ended');
};
```

---

## 📊 Get Connection Stats

```typescript
const getStats = async () => {
    const stats = await pc.getStats();
    
    stats.forEach(report => {
        if (report.type === 'inbound-rtp' && report.kind === 'video') {
            console.log('Packets lost:', report.packetsLost);
            console.log('Packets received:', report.packetsReceived);
            console.log('Bytes received:', report.bytesReceived);
            console.log('Jitter:', report.jitter);
        }
    });
};
```

---

## 🐛 Common Errors & Solutions

### **Error: "Permission denied"**
```typescript
// User denied camera/microphone access
// Solution: Show instructions to enable in browser settings
```

### **Error: "Cannot set remote description"**
```typescript
// Cause: Setting remote description before local
// Solution: Always set local description first!

// ✅ CORRECT ORDER:
await pc.setLocalDescription(offer);
socket.emit('offer', offer);
```

### **Error: "ICE connection failed"**
```typescript
// Cause: Both users behind strict NAT
// Solution: Add TURN server

const config = {
    iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        {
            urls: 'turn:your-turn-server.com:3478',
            username: 'user',
            credential: 'pass'
        }
    ]
};
```

### **Error: "No remote video"**
```typescript
// Cause: Forgot to add tracks
// Solution: Add tracks before creating offer

localStream.getTracks().forEach(track => {
    pc.addTrack(track, localStream);
});
```

---

## 🎯 Best Practices

### **1. Always Clean Up**
```typescript
useEffect(() => {
    // Setup
    const stream = await getUserMedia(...);
    
    return () => {
        // Cleanup
        stream.getTracks().forEach(t => t.stop());
        pc?.close();
        socket?.disconnect();
    };
}, []);
```

### **2. Handle Errors**
```typescript
try {
    const stream = await getUserMedia({ video: true, audio: true });
} catch (err) {
    if (err.name === 'NotAllowedError') {
        // User denied permission
    } else if (err.name === 'NotFoundError') {
        // No camera/microphone
    } else if (err.name === 'NotReadableError') {
        // Device in use
    }
}
```

### **3. Monitor Connection Quality**
```typescript
setInterval(async () => {
    const stats = await pc.getStats();
    // Check packet loss, latency, etc.
}, 5000);
```

### **4. Implement Reconnection**
```typescript
pc.oniceconnectionstatechange = () => {
    if (pc.iceConnectionState === 'failed') {
        // Attempt ICE restart
        pc.restartIce();
    }
};
```

---

## 📚 Key Takeaways

✅ **WebRTC = P2P**, WebSocket = Signaling  
✅ **STUN** discovers public IP, **TURN** relays if needed  
✅ **Offer/Answer** negotiate capabilities  
✅ **ICE candidates** find network paths  
✅ **Always clean up** resources  
✅ **Handle errors** gracefully  
✅ **Monitor quality** for best UX  

---

## 🔗 Useful Resources

- [WebRTC API Docs](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API)
- [Socket.IO Docs](https://socket.io/docs/)
- [NestJS WebSockets](https://docs.nestjs.com/websockets/gateways)
- [STUN/TURN Servers](https://gist.github.com/sagivo/3a4b2f2c7ac6e1b5267c2f1f59ac6c6b)

---

**Print this cheat sheet and keep it handy!** 📄
