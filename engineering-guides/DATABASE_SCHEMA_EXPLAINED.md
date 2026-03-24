# 📊 Social Media App - Database Schema Complete Guide

> **Complete explanation of the database schema, relationships, and data flow for the Social Media Application**

---

## 📑 Table of Contents

1. [Overview](#overview)
2. [User Model - The Foundation](#1-user-model---the-foundation)
3. [Post Creation Flow](#2-post-creation-flow)
4. [Engagement System (Likes & Comments)](#3-engagement-system---likes--comments)
5. [Follow System](#4-follow-system---social-network)
6. [Messaging System](#5-messaging-system---complete-chat-flow)
7. [Complete Data Flow Diagram](#6-complete-data-flow-diagram)
8. [Real-World Example](#7-real-world-example---complete-user-journey)
9. [Key Relationships Summary](#8-key-relationships-summary)
10. [Soft Delete Pattern](#9-soft-delete-pattern)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## 🎯 Overview

### The Big Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                    SOCIAL MEDIA APP ECOSYSTEM                    │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌──────────┐    ┌──────────┐    ┌──────────────┐              │
│  │  USERS   │───▶│  POSTS   │───▶│  ENGAGEMENT  │              │
│  │          │    │          │    │ (Like/Comment)│              │
│  └──────────┘    └──────────┘    └──────────────┘              │
│       │                                                          │
│       │          ┌──────────────┐                                │
│       └─────────▶│  MESSAGING   │                                │
│                  │ (Chat System)│                                │
│                  └──────────────┘                                │
│                                                                   │
│       ┌──────────────┐                                           │
│       │   FOLLOWS    │  (Social Network)                         │
│       └──────────────┘                                           │
└─────────────────────────────────────────────────────────────────┘
```

### Core Components

The application consists of **5 main systems**:

1. **User Management** - Authentication, profiles, and roles
2. **Social Network** - Follow/unfollow relationships
3. **Content System** - Posts with media attachments
4. **Engagement** - Likes and nested comments
5. **Messaging** - Private and group chat with read receipts

---

## 1️⃣ User Model - The Foundation

### Schema Structure

```
┌─────────────────────────────────────────────────────────────┐
│                        USER                                  │
├─────────────────────────────────────────────────────────────┤
│ id            : UUID (Primary Key)                          │
│ username      : String (Unique)                             │
│ email         : String (Unique)                             │
│ password      : String (Hashed)                             │
│ avatarUrl     : String? (Optional)                          │
│ bio           : String? (Optional)                          │
│ role          : Role (USER/ADMIN)                           │
│ createdAt     : DateTime                                    │
│ updatedAt     : DateTime                                    │
│ deletedAt     : DateTime? (Soft Delete)                     │
└─────────────────────────────────────────────────────────────┘
         │
         │ Has Many Relationships:
         │
         ├──▶ posts[]          (User creates Posts)
         ├──▶ comments[]       (User writes Comments)
         ├──▶ likes[]          (User likes Posts)
         ├──▶ followers[]      (Users who follow this user)
         ├──▶ following[]      (Users this user follows)
         ├──▶ messages[]       (Messages sent by user)
         ├──▶ conversations[]  (Chat conversations)
         └──▶ ownedGroups[]    (Group chats owned by user)
```

### Key Features

- **Unique Identifiers**: Both `username` and `email` are unique
- **Security**: Password is stored as a hashed string
- **Role-Based Access**: Supports `USER` and `ADMIN` roles
- **Soft Delete**: Users can be deactivated without losing data
- **Timestamps**: Automatic tracking of creation and updates

### Prisma Schema

```prisma
model User {
  id            String             @id @default(uuid())
  username      String             @unique
  email         String             @unique
  password      String
  avatarUrl     String?
  bio           String?
  role          Role               @default(USER)
  createdAt     DateTime           @default(now())
  updatedAt     DateTime           @updatedAt
  deletedAt     DateTime?
  comments      Comment[]
  ownedGroups   Conversation[]     @relation("GroupOwner")
  conversations ConversationUser[]
  followers     Follow[]           @relation("UserFollowers")
  following     Follow[]           @relation("UserFollowing")
  likes         Like[]
  messages      Message[]
  messageReads  MessageRead[]
  posts         Post[]

  @@index([username])
  @@index([deletedAt])
}

enum Role {
  USER
  ADMIN
}
```

---

## 2️⃣ Post Creation Flow

### How Users Create Posts

#### STEP 1: User Creates a Post

```
    ┌──────────┐
    │   USER   │
    │ (Author) │
    └─────┬────┘
          │
          │ Creates
          ▼
    ┌─────────────────────────────────────────┐
    │              POST                        │
    ├─────────────────────────────────────────┤
    │ id        : UUID                        │
    │ title     : "My First Post"             │
    │ content   : "Hello World!"              │
    │ authorId  : <User.id> ◀─────────────┐  │
    │ createdAt : 2025-12-21 12:00:00     │  │
    │ updatedAt : 2025-12-21 12:00:00     │  │
    │ deletedAt : null                     │  │
    └─────────────────────────────────────────┘
                                           │
                                    Foreign Key
                                    Relationship
```

#### STEP 2: User Can Add Media (Optional)

```
    ┌─────────────────────┐
    │       POST          │
    └──────────┬──────────┘
               │
               │ Can have multiple
               ▼
    ┌─────────────────────────────────────────┐
    │          ATTACHMENT                      │
    ├─────────────────────────────────────────┤
    │ id       : UUID                         │
    │ url      : "https://s3.../image.jpg"    │
    │ key      : "uploads/abc123.jpg"         │
    │ type     : IMAGE/VIDEO/FILE/AUDIO       │
    │ mimeType : "image/jpeg"                 │
    │ size     : 1024000 (bytes)              │
    │ postId   : <Post.id> ◀──────────────┐  │
    │ messageId: null                      │  │
    └─────────────────────────────────────────┘
                                          │
                                   Foreign Key
                                   (Links to Post)
```

#### STEP 3: Post Gets Stats Tracking

```
    ┌─────────────────────┐
    │       POST          │
    └──────────┬──────────┘
               │
               │ Has one
               ▼
    ┌─────────────────────────────────────────┐
    │          POST STATS                      │
    ├─────────────────────────────────────────┤
    │ postId       : <Post.id> (PK)           │
    │ likeCount    : 0                        │
    │ commentCount : 0                        │
    │ viewCount    : 0                        │
    │ score        : 0.0                      │
    │ lastUpdatedAt: DateTime                 │
    └─────────────────────────────────────────┘
```

### Prisma Schema

```prisma
model Post {
  id          String       @id @default(uuid())
  title       String
  content     String
  authorId    String
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
  deletedAt   DateTime?
  attachments Attachment[]
  comments    Comment[]
  likes       Like[]
  author      User         @relation(fields: [authorId], references: [id])
  stats       PostStats?

  @@index([authorId, createdAt])
}

model Attachment {
  id        String    @id @default(uuid())
  url       String
  key       String
  type      MediaType
  mimeType  String
  size      Int
  postId    String?
  messageId String?
  createdAt DateTime  @default(now())
  message   Message?  @relation(fields: [messageId], references: [id])
  post      Post?     @relation(fields: [postId], references: [id])

  @@index([postId])
  @@index([messageId])
}

model PostStats {
  postId        String   @id
  likeCount     Int      @default(0)
  commentCount  Int      @default(0)
  viewCount     Int      @default(0)
  score         Float    @default(0)
  lastUpdatedAt DateTime @updatedAt
  post          Post     @relation(fields: [postId], references: [id])
}

enum MediaType {
  IMAGE
  VIDEO
  FILE
  AUDIO
}
```

---

## 3️⃣ Engagement System - Likes & Comments

### 🔥 Like System

#### How Likes Work

```
User Likes a Post:
═══════════════════════════════════════════════════════════════

    ┌──────────┐                    ┌─────────────┐
    │   USER   │                    │    POST     │
    │  (Alice) │                    │ (Bob's Post)│
    └─────┬────┘                    └──────┬──────┘
          │                                │
          │         Clicks "Like"          │
          └────────────────┬───────────────┘
                           │
                           ▼
              ┌────────────────────────────┐
              │          LIKE               │
              ├────────────────────────────┤
              │ postId  : <Post.id>  ◀─────┼─── FK to Post
              │ userId  : <User.id>  ◀─────┼─── FK to User
              │ createdAt: DateTime         │
              └────────────────────────────┘
                    Composite Primary Key
                    (postId + userId)
                    
                    ▼ Triggers Update
                    
              ┌────────────────────────────┐
              │       POST STATS            │
              ├────────────────────────────┤
              │ likeCount: 1 (incremented) │
              └────────────────────────────┘
```

#### Key Features

- **Composite Primary Key**: `(postId, userId)` ensures one like per user per post
- **Automatic Stats Update**: Incrementing `PostStats.likeCount`
- **Timestamp Tracking**: Know when each like was created
- **Efficient Queries**: Indexed for fast lookups

#### Prisma Schema

```prisma
model Like {
  postId    String
  userId    String
  createdAt DateTime @default(now())
  post      Post     @relation(fields: [postId], references: [id])
  user      User     @relation(fields: [userId], references: [id])

  @@id([postId, userId])
}
```

---

### 💬 Comment System (With Nested Replies!)

#### Basic Comment Structure

```
User Comments on a Post:
═══════════════════════════════════════════════════════════════

    ┌──────────┐                    ┌─────────────┐
    │   USER   │                    │    POST     │
    │  (Alice) │                    │ (Bob's Post)│
    └─────┬────┘                    └──────┬──────┘
          │                                │
          │      Writes Comment            │
          └────────────────┬───────────────┘
                           │
                           ▼
              ┌────────────────────────────────┐
              │         COMMENT                 │
              ├────────────────────────────────┤
              │ id       : UUID                │
              │ text     : "Great post!"       │
              │ postId   : <Post.id>  ◀────────┼─── FK to Post
              │ userId   : <User.id>  ◀────────┼─── FK to User
              │ parentId : null (root comment) │
              │ createdAt: DateTime             │
              │ deletedAt: null                 │
              └────────────────────────────────┘
```

#### Nested Replies (Comment Tree)

```
NESTED REPLIES (Comment Tree):
═══════════════════════════════════════════════════════════════

    Comment #1 (Root)
    ├─ id: "abc-123"
    ├─ text: "Great post!"
    ├─ parentId: null
    └─ postId: "post-xyz"
         │
         │ Someone replies to Comment #1
         ▼
    Comment #2 (Reply)
    ├─ id: "def-456"
    ├─ text: "I agree!"
    ├─ parentId: "abc-123" ◀──── Points to Comment #1
    └─ postId: "post-xyz"
         │
         │ Another reply to Comment #2
         ▼
    Comment #3 (Nested Reply)
    ├─ id: "ghi-789"
    ├─ text: "Me too!"
    ├─ parentId: "def-456" ◀──── Points to Comment #2
    └─ postId: "post-xyz"
```

#### Visual Tree Structure

```
    POST: "My First Post"
    │
    ├── COMMENT #1: "Great post!" (parentId: null)
    │   │
    │   ├── COMMENT #2: "I agree!" (parentId: abc-123)
    │   │   │
    │   │   └── COMMENT #3: "Me too!" (parentId: def-456)
    │   │
    │   └── COMMENT #4: "Thanks!" (parentId: abc-123)
    │
    └── COMMENT #5: "Nice work!" (parentId: null)
        │
        └── COMMENT #6: "Indeed!" (parentId: comment-5-id)
```

#### Key Features

- **Self-Referencing**: `parentId` points to another comment's `id`
- **Unlimited Nesting**: Comments can be nested infinitely
- **Root Comments**: `parentId = null` for top-level comments
- **Soft Delete**: Deleted comments maintain tree structure

#### Prisma Schema

```prisma
model Comment {
  id        String    @id @default(uuid())
  text      String
  postId    String
  userId    String
  parentId  String?
  createdAt DateTime  @default(now())
  deletedAt DateTime?
  parent    Comment?  @relation("CommentTree", fields: [parentId], references: [id])
  replies   Comment[] @relation("CommentTree")
  post      Post      @relation(fields: [postId], references: [id])
  user      User      @relation(fields: [userId], references: [id])

  @@index([postId, createdAt])
  @@index([parentId])
}
```

---

## 4️⃣ Follow System - Social Network

### How Following Works

```
User Follows Another User:
═══════════════════════════════════════════════════════════════

    ┌──────────────┐                    ┌──────────────┐
    │  USER (Alice)│                    │  USER (Bob)  │
    │  id: user-1  │                    │  id: user-2  │
    └──────┬───────┘                    └──────┬───────┘
           │                                   │
           │      Alice follows Bob            │
           └────────────┬──────────────────────┘
                        │
                        ▼
              ┌─────────────────────────────┐
              │         FOLLOW               │
              ├─────────────────────────────┤
              │ followerId : user-1 ◀───────┼─── Alice (Follower)
              │ followingId: user-2 ◀───────┼─── Bob (Following)
              │ createdAt  : DateTime        │
              └─────────────────────────────┘
                   Composite Primary Key
                   (followerId + followingId)
```

### Relationship Visualization

```
    USER (Alice)
    ├─ followers[]   : [Charlie, David]    ← People who follow Alice
    └─ following[]   : [Bob, Eve]          ← People Alice follows

    USER (Bob)
    ├─ followers[]   : [Alice, Frank]      ← People who follow Bob
    └─ following[]   : [Alice, Grace]      ← People Bob follows
```

### Follow Table Example

```
    ┌─────────────┬──────────────┬─────────────────────┐
    │ followerId  │ followingId  │ Meaning             │
    ├─────────────┼──────────────┼─────────────────────┤
    │ Alice       │ Bob          │ Alice follows Bob   │
    │ Bob         │ Alice        │ Bob follows Alice   │
    │ Charlie     │ Alice        │ Charlie follows Alice│
    └─────────────┴──────────────┴─────────────────────┘
```

### Key Features

- **Bidirectional Relationships**: Users can follow each other
- **Composite Primary Key**: Prevents duplicate follows
- **Efficient Queries**: Indexed on both `followerId` and `followingId`
- **Timestamp Tracking**: Know when follow relationships started

### Prisma Schema

```prisma
model Follow {
  followerId  String
  followingId String
  createdAt   DateTime @default(now())
  follower    User     @relation("UserFollowers", fields: [followerId], references: [id])
  following   User     @relation("UserFollowing", fields: [followingId], references: [id])

  @@id([followerId, followingId])
  @@index([followerId])
  @@index([followingId])
}
```

---

## 5️⃣ Messaging System - Complete Chat Flow

### 📱 Conversation Types

```
Two Types of Conversations:
═══════════════════════════════════════════════════════════════

1. PRIVATE (One-to-One Chat)
   ┌─────────────────────────────────────┐
   │       CONVERSATION                   │
   ├─────────────────────────────────────┤
   │ id      : "conv-123"                │
   │ name    : null (auto-generated)     │
   │ type    : PRIVATE                   │
   │ isGroup : false                     │
   │ ownerId : null                      │
   └─────────────────────────────────────┘
            │
            ├──▶ User 1 (Alice)
            └──▶ User 2 (Bob)


2. GROUP (Multi-User Chat)
   ┌─────────────────────────────────────┐
   │       CONVERSATION                   │
   ├─────────────────────────────────────┤
   │ id      : "conv-456"                │
   │ name    : "Project Team"            │
   │ type    : GROUP                     │
   │ isGroup : true                      │
   │ ownerId : <User.id> (Creator)       │
   └─────────────────────────────────────┘
            │
            ├──▶ User 1 (Alice) - Owner
            ├──▶ User 2 (Bob)
            ├──▶ User 3 (Charlie)
            └──▶ User 4 (David)
```

### 💬 Complete Messaging Flow

#### STEP 1: Create Conversation

```
    Alice wants to chat with Bob
    
    ┌─────────────────────────────────────┐
    │       CONVERSATION                   │
    ├─────────────────────────────────────┤
    │ id              : "conv-abc"        │
    │ type            : PRIVATE           │
    │ lastMessageId   : null              │
    │ createdAt       : DateTime          │
    └─────────────────────────────────────┘
```

#### STEP 2: Add Users to Conversation

```
    ┌──────────────────────────────────────┐
    │    CONVERSATION_USER (Junction)      │
    ├──────────────────────────────────────┤
    │ conversationId: "conv-abc"           │
    │ userId        : "alice-id"           │
    │ joinedAt      : DateTime             │
    └──────────────────────────────────────┘

    ┌──────────────────────────────────────┐
    │    CONVERSATION_USER (Junction)      │
    ├──────────────────────────────────────┤
    │ conversationId: "conv-abc"           │
    │ userId        : "bob-id"             │
    │ joinedAt      : DateTime             │
    └──────────────────────────────────────┘
```

#### STEP 3: Send Messages

```
    Alice sends: "Hey Bob!"
    
    ┌──────────────────────────────────────┐
    │           MESSAGE #1                  │
    ├──────────────────────────────────────┤
    │ id            : "msg-001"            │
    │ conversationId: "conv-abc" ◀─────────┼─── FK to Conversation
    │ senderId      : "alice-id" ◀─────────┼─── FK to User (Alice)
    │ content       : "Hey Bob!"           │
    │ createdAt     : 12:00:00             │
    │ deletedAt     : null                 │
    └──────────────────────────────────────┘
    
    Bob replies: "Hi Alice!"
    
    ┌──────────────────────────────────────┐
    │           MESSAGE #2                  │
    ├──────────────────────────────────────┤
    │ id            : "msg-002"            │
    │ conversationId: "conv-abc"           │
    │ senderId      : "bob-id" ◀───────────┼─── FK to User (Bob)
    │ content       : "Hi Alice!"          │
    │ createdAt     : 12:01:00             │
    └──────────────────────────────────────┘
```

#### STEP 4: Message with Attachments

```
    Alice sends an image
    
    ┌──────────────────────────────────────┐
    │           MESSAGE #3                  │
    ├──────────────────────────────────────┤
    │ id            : "msg-003"            │
    │ conversationId: "conv-abc"           │
    │ senderId      : "alice-id"           │
    │ content       : "Check this out!"    │
    │ createdAt     : 12:02:00             │
    └──────────────────────────────────────┘
                    │
                    │ Has attachment
                    ▼
    ┌──────────────────────────────────────┐
    │          ATTACHMENT                   │
    ├──────────────────────────────────────┤
    │ id       : "att-001"                 │
    │ url      : "https://s3.../photo.jpg" │
    │ type     : IMAGE                     │
    │ mimeType : "image/jpeg"              │
    │ size     : 2048000                   │
    │ messageId: "msg-003" ◀───────────────┼─── FK to Message
    │ postId   : null                      │
    └──────────────────────────────────────┘
```

#### STEP 5: Track Read Status

```
    Bob reads Alice's message
    
    ┌──────────────────────────────────────┐
    │        MESSAGE_READ                   │
    ├──────────────────────────────────────┤
    │ messageId: "msg-001" ◀───────────────┼─── FK to Message
    │ userId   : "bob-id"  ◀───────────────┼─── FK to User (Bob)
    │ readAt   : 12:05:00                  │
    └──────────────────────────────────────┘
           Composite PK (messageId + userId)
           
    This allows tracking:
    - Who read which message
    - When they read it
    - "Seen by" feature
    - "Read receipts" (✓✓)
```

### Prisma Schema

```prisma
model Conversation {
  id            String             @id @default(uuid())
  name          String?
  type          ConversationType   @default(PRIVATE)
  ownerId       String?
  isGroup       Boolean?
  lastMessageId String?            @unique
  createdAt     DateTime           @default(now())
  updatedAt     DateTime           @updatedAt
  deletedAt     DateTime?
  owner         User?              @relation("GroupOwner", fields: [ownerId], references: [id])
  users         ConversationUser[]
  messages      Message[]          @relation("ChatMessages")

  @@index([ownerId])
}

model ConversationUser {
  conversationId String
  userId         String
  joinedAt       DateTime     @default(now())
  conversation   Conversation @relation(fields: [conversationId], references: [id])
  user           User         @relation(fields: [userId], references: [id])

  @@id([conversationId, userId])
}

model Message {
  id             String        @id @default(uuid())
  conversationId String
  senderId       String
  content        String?
  createdAt      DateTime      @default(now())
  deletedAt      DateTime?
  attachments    Attachment[]
  conversation   Conversation  @relation("ChatMessages", fields: [conversationId], references: [id])
  sender         User          @relation(fields: [senderId], references: [id])
  reads          MessageRead[]

  @@index([conversationId, createdAt])
}

model MessageRead {
  messageId String
  userId    String
  readAt    DateTime @default(now())
  message   Message  @relation(fields: [messageId], references: [id])
  user      User     @relation(fields: [userId], references: [id])

  @@id([messageId, userId])
}

enum ConversationType {
  PRIVATE
  GROUP
}
```

---

## 6️⃣ Complete Data Flow Diagram

```
═══════════════════════════════════════════════════════════════
                    SOCIAL MEDIA APP - FULL FLOW
═══════════════════════════════════════════════════════════════

                        ┌──────────────┐
                        │     USER     │
                        └───────┬──────┘
                                │
                ┌───────────────┼───────────────┐
                │               │               │
                ▼               ▼               ▼
         ┌──────────┐    ┌──────────┐   ┌──────────┐
         │  POSTS   │    │ MESSAGES │   │ FOLLOWS  │
         └─────┬────┘    └─────┬────┘   └──────────┘
               │               │
        ┌──────┼──────┐        │
        │      │      │        │
        ▼      ▼      ▼        ▼
    ┌─────┐ ┌────┐ ┌────┐  ┌──────────────┐
    │LIKES│ │CMTS│ │ATCH│  │CONVERSATION  │
    └─────┘ └────┘ └────┘  └──────┬───────┘
       │      │      │            │
       │      │      │            ▼
       │      │      │     ┌──────────────┐
       │      │      │     │CONVERSATION  │
       │      │      │     │    USER      │
       │      │      │     └──────────────┘
       │      │      │
       └──────┴──────┴────────────┐
                                  ▼
                          ┌──────────────┐
                          │  POST STATS  │
                          └──────────────┘
```

---

## 7️⃣ Real-World Example - Complete User Journey

### Alice's Journey Through the App

```
═══════════════════════════════════════════════════════════════
                    ALICE'S JOURNEY
═══════════════════════════════════════════════════════════════

1. ALICE SIGNS UP
   ┌─────────────────────────────────────┐
   │ USER                                 │
   ├─────────────────────────────────────┤
   │ id      : "alice-123"               │
   │ username: "alice_wonder"            │
   │ email   : "alice@example.com"       │
   │ role    : USER                      │
   └─────────────────────────────────────┘


2. ALICE FOLLOWS BOB
   ┌─────────────────────────────────────┐
   │ FOLLOW                               │
   ├─────────────────────────────────────┤
   │ followerId : "alice-123"            │
   │ followingId: "bob-456"              │
   └─────────────────────────────────────┘


3. ALICE CREATES A POST
   ┌─────────────────────────────────────┐
   │ POST                                 │
   ├─────────────────────────────────────┤
   │ id      : "post-789"                │
   │ title   : "My Day at the Beach"     │
   │ content : "Had an amazing time!"    │
   │ authorId: "alice-123"               │
   └─────────────────────────────────────┘
            │
            ├──▶ ATTACHMENT (beach.jpg)
            └──▶ POST_STATS (0 likes, 0 comments)


4. BOB LIKES ALICE'S POST
   ┌─────────────────────────────────────┐
   │ LIKE                                 │
   ├─────────────────────────────────────┤
   │ postId: "post-789"                  │
   │ userId: "bob-456"                   │
   └─────────────────────────────────────┘
            │
            └──▶ POST_STATS (likeCount: 1)


5. BOB COMMENTS ON ALICE'S POST
   ┌─────────────────────────────────────┐
   │ COMMENT                              │
   ├─────────────────────────────────────┤
   │ id      : "cmt-001"                 │
   │ text    : "Looks fun!"              │
   │ postId  : "post-789"                │
   │ userId  : "bob-456"                 │
   │ parentId: null                      │
   └─────────────────────────────────────┘
            │
            └──▶ POST_STATS (commentCount: 1)


6. ALICE REPLIES TO BOB'S COMMENT
   ┌─────────────────────────────────────┐
   │ COMMENT                              │
   ├─────────────────────────────────────┤
   │ id      : "cmt-002"                 │
   │ text    : "Thanks Bob!"             │
   │ postId  : "post-789"                │
   │ userId  : "alice-123"               │
   │ parentId: "cmt-001" ◀─────────────┐ │
   └─────────────────────────────────────┘
                                        │
                            Nested reply to Bob's comment


7. ALICE MESSAGES BOB
   
   a) Create Conversation
   ┌─────────────────────────────────────┐
   │ CONVERSATION                         │
   ├─────────────────────────────────────┤
   │ id  : "conv-999"                    │
   │ type: PRIVATE                       │
   └─────────────────────────────────────┘
   
   b) Add participants
   ┌─────────────────────────────────────┐
   │ CONVERSATION_USER                    │
   │ conversationId: "conv-999"          │
   │ userId: "alice-123"                 │
   └─────────────────────────────────────┘
   
   ┌─────────────────────────────────────┐
   │ CONVERSATION_USER                    │
   │ conversationId: "conv-999"          │
   │ userId: "bob-456"                   │
   └─────────────────────────────────────┘
   
   c) Send message
   ┌─────────────────────────────────────┐
   │ MESSAGE                              │
   ├─────────────────────────────────────┤
   │ id            : "msg-111"           │
   │ conversationId: "conv-999"          │
   │ senderId      : "alice-123"         │
   │ content       : "Thanks for the     │
   │                  comment!"          │
   └─────────────────────────────────────┘
   
   d) Bob reads it
   ┌─────────────────────────────────────┐
   │ MESSAGE_READ                         │
   ├─────────────────────────────────────┤
   │ messageId: "msg-111"                │
   │ userId   : "bob-456"                │
   │ readAt   : 13:30:00                 │
   └─────────────────────────────────────┘
```

---

## 8️⃣ Key Relationships Summary

### Relationship Types

```
═══════════════════════════════════════════════════════════════
                    RELATIONSHIP TYPES
═══════════════════════════════════════════════════════════════

ONE-TO-MANY (1:N)
─────────────────
User ──────▶ Posts         (One user has many posts)
User ──────▶ Comments      (One user has many comments)
User ──────▶ Messages      (One user sends many messages)
Post ──────▶ Comments      (One post has many comments)
Post ──────▶ Likes         (One post has many likes)
Post ──────▶ Attachments   (One post has many attachments)
Conversation ──▶ Messages  (One conversation has many messages)
Comment ────▶ Replies      (One comment has many replies)


ONE-TO-ONE (1:1)
────────────────
Post ──────▶ PostStats     (One post has one stats record)


MANY-TO-MANY (M:N) - Using Junction Tables
───────────────────────────────────────────
User ◀────▶ User          (via FOLLOW table)
  - followerId + followingId

User ◀────▶ Post          (via LIKE table)
  - userId + postId

User ◀────▶ Conversation  (via CONVERSATION_USER table)
  - userId + conversationId

User ◀────▶ Message       (via MESSAGE_READ table)
  - userId + messageId


SELF-REFERENCING
────────────────
Comment ──▶ Comment       (parentId → id)
  - Allows nested comment threads
```

### Foreign Key Relationships

| Child Table | Foreign Key | References | Description |
|------------|-------------|------------|-------------|
| Post | authorId | User.id | Post author |
| Comment | userId | User.id | Comment author |
| Comment | postId | Post.id | Parent post |
| Comment | parentId | Comment.id | Parent comment (for replies) |
| Like | userId | User.id | User who liked |
| Like | postId | Post.id | Liked post |
| Follow | followerId | User.id | User who follows |
| Follow | followingId | User.id | User being followed |
| Message | senderId | User.id | Message sender |
| Message | conversationId | Conversation.id | Parent conversation |
| MessageRead | messageId | Message.id | Read message |
| MessageRead | userId | User.id | User who read |
| Attachment | postId | Post.id | Attached to post |
| Attachment | messageId | Message.id | Attached to message |
| PostStats | postId | Post.id | Stats for post |
| ConversationUser | conversationId | Conversation.id | Conversation membership |
| ConversationUser | userId | User.id | User in conversation |
| Conversation | ownerId | User.id | Group owner |

---

## 9️⃣ Soft Delete Pattern

### What is Soft Delete?

Instead of permanently deleting data from the database, we mark it as deleted using a `deletedAt` timestamp.

```
═══════════════════════════════════════════════════════════════
                    SOFT DELETE EXPLAINED
═══════════════════════════════════════════════════════════════

ACTIVE RECORD:
┌─────────────────────────────────────┐
│ POST                                 │
├─────────────────────────────────────┤
│ id       : "post-123"               │
│ title    : "My Post"                │
│ deletedAt: null ◀──────────────────┼─── NULL = Active
└─────────────────────────────────────┘


DELETED RECORD:
┌─────────────────────────────────────┐
│ POST                                 │
├─────────────────────────────────────┤
│ id       : "post-456"               │
│ title    : "Deleted Post"           │
│ deletedAt: 2025-12-21 10:00:00 ◀───┼─── Has timestamp = Deleted
└─────────────────────────────────────┘
```

### Benefits

✅ **Data Recovery**: Can restore deleted content  
✅ **Data Integrity**: Foreign keys remain valid  
✅ **Audit Trail**: Know when something was deleted  
✅ **Analytics**: Track deletion patterns  
✅ **Compliance**: Meet data retention requirements

### Models with Soft Delete

- **User** (`deletedAt`)
- **Post** (`deletedAt`)
- **Comment** (`deletedAt`)
- **Message** (`deletedAt`)
- **Conversation** (`deletedAt`)

### Querying with Soft Delete

```typescript
// Get only active posts
const activePosts = await prisma.post.findMany({
  where: {
    deletedAt: null
  }
});

// Get deleted posts
const deletedPosts = await prisma.post.findMany({
  where: {
    deletedAt: { not: null }
  }
});

// Soft delete a post
await prisma.post.update({
  where: { id: postId },
  data: { deletedAt: new Date() }
});

// Restore a post
await prisma.post.update({
  where: { id: postId },
  data: { deletedAt: null }
});
```

---

## 🎯 Quick Reference Cheat Sheet

### Common Operations

```
═══════════════════════════════════════════════════════════════
                    QUICK REFERENCE
═══════════════════════════════════════════════════════════════

CREATE POST:
  User → Post → Attachment (optional) → PostStats

LIKE POST:
  User + Post → Like → Update PostStats.likeCount

COMMENT ON POST:
  User + Post → Comment → Update PostStats.commentCount
  
REPLY TO COMMENT:
  User + Post + ParentComment → Comment (with parentId)

FOLLOW USER:
  User (follower) + User (following) → Follow

START CHAT:
  Create Conversation → Add ConversationUser records

SEND MESSAGE:
  User + Conversation → Message → Attachment (optional)

READ MESSAGE:
  User + Message → MessageRead

DELETE CONTENT:
  Set deletedAt = current timestamp (soft delete)
```

### Database Indexes

Indexes are created for optimal query performance:

```
User:
  - username (unique)
  - email (unique)
  - deletedAt

Post:
  - (authorId, createdAt) composite

Comment:
  - (postId, createdAt) composite
  - parentId

Follow:
  - followerId
  - followingId

Message:
  - (conversationId, createdAt) composite

Attachment:
  - postId
  - messageId
```

---

## 📚 Additional Resources

### Prisma Commands

```bash
# Generate Prisma Client
npx prisma generate

# Create migration
npx prisma migrate dev --name migration_name

# Apply migrations
npx prisma migrate deploy

# Open Prisma Studio (GUI)
npx prisma studio

# Reset database
npx prisma migrate reset

# Format schema
npx prisma format
```

### Best Practices

1. **Always use transactions** for operations that modify multiple tables
2. **Implement pagination** for list queries (posts, comments, messages)
3. **Use select/include wisely** to avoid over-fetching data
4. **Index frequently queried fields** for better performance
5. **Validate data** before database operations
6. **Handle cascading deletes** carefully with soft delete
7. **Use connection pooling** in production
8. **Monitor query performance** with Prisma query logs

---

## 🎉 Conclusion

This social media application has a well-structured database schema that supports:

✅ **User Management** - Authentication, profiles, roles  
✅ **Social Features** - Follow/unfollow system  
✅ **Content Creation** - Posts with media attachments  
✅ **Engagement** - Likes and nested comments  
✅ **Messaging** - Private & group chats with read receipts  
✅ **Analytics** - Post statistics tracking  
✅ **Data Safety** - Soft delete pattern  

All relationships are properly connected with foreign keys, composite primary keys for many-to-many relationships, and efficient indexing for optimal performance! 🚀

---

**Generated on**: 2025-12-21  
**Schema Version**: 1.0  
**Database**: PostgreSQL with Prisma ORM
