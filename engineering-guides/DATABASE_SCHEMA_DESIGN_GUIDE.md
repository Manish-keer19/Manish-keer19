# 🎓 Complete Database Schema Design Guide
## From Requirements to Production-Ready Schema

> **Learn to design any database schema by understanding the thought process behind our Social Media App**

---

## 📚 Table of Contents
1. [The Design Process Overview](#the-design-process-overview)
2. [Step 1: Gather & Analyze Requirements](#step-1-gather--analyze-requirements)
3. [Step 2: Identify Entities (Nouns)](#step-2-identify-entities-nouns)
4. [Step 3: Identify Relationships (Verbs)](#step-3-identify-relationships-verbs)
5. [Step 4: Design Basic Schema](#step-4-design-basic-schema)
6. [Step 5: Add Constraints & Validation](#step-5-add-constraints--validation)
7. [Step 6: Optimize for Performance](#step-6-optimize-for-performance)
8. [Step 7: Handle Edge Cases](#step-7-handle-edge-cases)
9. [Common Patterns & Best Practices](#common-patterns--best-practices)
10. [Practice Exercises](#practice-exercises)

---

## 🎯 The Design Process Overview

```
Requirements → Entities → Relationships → Basic Schema → Constraints → Optimization → Edge Cases
     ↓            ↓            ↓              ↓              ↓              ↓            ↓
  Questions    Nouns        Verbs         Tables         Rules         Indexes      Testing
```

**Golden Rule:** Start simple, iterate based on real usage patterns!

---

## 📋 Step 1: Gather & Analyze Requirements

### **The Social Media App Requirements**

Let's say a client comes to you with this:

> *"I want to build a social media platform where users can:*
> - *Create posts with text, images, and videos*
> - *Like and comment on posts (with nested replies)*
> - *Follow other users*
> - *Send private messages and create group chats*
> - *See who read their messages*
> - *Get notifications for new activity"*

### **🧠 How to Think:**

Ask yourself these questions:

#### **1. What are the MAIN "things" (entities)?**
Read the requirements and highlight **nouns**:
- ✅ Users
- ✅ Posts
- ✅ Images/Videos (Attachments)
- ✅ Likes
- ✅ Comments
- ✅ Follows
- ✅ Messages
- ✅ Chats (Conversations)
- ✅ Read receipts

#### **2. What ACTIONS happen?**
Highlight **verbs**:
- ✅ Create (post, message, comment)
- ✅ Like (post)
- ✅ Comment (on post)
- ✅ Follow (user)
- ✅ Send (message)
- ✅ Read (message)

#### **3. What are the RELATIONSHIPS?**
Think about connections:
- User **creates** Post (1 user → many posts)
- User **likes** Post (many users ↔ many posts)
- User **follows** User (many ↔ many)
- Post **has** Comments (1 post → many comments)
- Comment **has** Replies (1 comment → many comments)
- User **sends** Message (1 user → many messages)
- Message **belongs to** Conversation (1 conversation → many messages)

#### **4. What needs to be FAST?**
Identify performance-critical queries:
- ✅ "Show me unread message count" (queried every page load)
- ✅ "Get user's feed" (queried often)
- ✅ "Count likes on post" (displayed for every post)
- ✅ "Get comments with replies" (complex nested query)

---

## 🏗️ Step 2: Identify Entities (Nouns)

### **Entity Identification Process**

For each noun, ask:
1. **Does it have its own data?** (Yes = Entity, No = Attribute)
2. **Can it exist independently?** (Yes = Entity, No = Attribute)
3. **Will we query it separately?** (Yes = Entity, No = Attribute)

### **Example Analysis:**

| Noun | Is it an Entity? | Why? |
|------|------------------|------|
| **User** | ✅ YES | Has own data (username, email), exists independently |
| **Post** | ✅ YES | Has own data (title, content), can be queried separately |
| **Like** | ✅ YES | Needs timestamp, can query "who liked this?" |
| **Comment** | ✅ YES | Has text, timestamp, can have replies |
| **Username** | ❌ NO | Just an attribute of User |
| **Post Title** | ❌ NO | Just an attribute of Post |
| **Attachment** | ✅ YES | Has URL, type, size - complex enough to be separate |
| **Message** | ✅ YES | Has content, timestamp, read status |
| **Conversation** | ✅ YES | Groups messages, has participants |

### **Our Final Entities:**

```
1. User          - People using the app
2. Post          - Content created by users
3. Attachment    - Files (images/videos) attached to posts/messages
4. Comment       - Comments on posts (with nested replies)
5. Like          - User likes on posts
6. Follow        - User following relationships
7. Conversation  - Chat rooms (private or group)
8. Message       - Messages in conversations
9. MessageRead   - Track who read which message
10. ConversationRead - Track last read position per user
```

---

## 🔗 Step 3: Identify Relationships (Verbs)

### **Relationship Types:**

#### **One-to-Many (1:N)**
One parent → Many children

**Examples:**
- 1 User → Many Posts
- 1 Post → Many Comments
- 1 Conversation → Many Messages

**How to implement:**
```prisma
model User {
  id    String @id
  posts Post[]  // Array = "many"
}

model Post {
  id       String @id
  authorId String  // Foreign key
  author   User   @relation(fields: [authorId], references: [id])
}
```

#### **Many-to-Many (M:N)**
Many ↔ Many (needs join table)

**Examples:**
- Many Users ↔ Many Posts (Likes)
- Many Users ↔ Many Users (Follows)
- Many Users ↔ Many Conversations (Participants)

**How to implement:**

**Option 1: Simple (no extra data)**
```prisma
model User {
  id        String @id
  likedPosts Post[] @relation("PostLikes")
}

model Post {
  id        String @id
  likedBy   User[] @relation("PostLikes")
}
```

**Option 2: Explicit Join Table (with extra data like timestamp)**
```prisma
model User {
  id    String @id
  likes Like[]
}

model Post {
  id    String @id
  likes Like[]
}

model Like {
  userId    String
  postId    String
  createdAt DateTime @default(now()) // Extra data!
  
  user User @relation(fields: [userId], references: [id])
  post Post @relation(fields: [postId], references: [id])
  
  @@id([userId, postId]) // Composite primary key
}
```

**When to use explicit join table?**
- ✅ Need timestamp (when did they like?)
- ✅ Need extra metadata (like type, reaction emoji)
- ✅ Need to query the relationship itself

#### **Self-Referencing (Special Case)**
Entity relates to itself

**Example: Comments with Replies**
```prisma
model Comment {
  id       String    @id
  text     String
  parentId String?   // Points to another Comment
  
  parent   Comment?  @relation("CommentTree", fields: [parentId], references: [id])
  replies  Comment[] @relation("CommentTree")
}
```

**Example: User Follows User**
```prisma
model User {
  id         String   @id
  followers  Follow[] @relation("UserFollowers")
  following  Follow[] @relation("UserFollowing")
}

model Follow {
  followerId  String
  followingId String
  
  follower  User @relation("UserFollowers", fields: [followerId], references: [id])
  following User @relation("UserFollowing", fields: [followingId], references: [id])
  
  @@id([followerId, followingId])
}
```

### **Our Relationship Map:**

```
User ──1:N──> Post ──1:N──> Comment ──1:N──> Comment (replies)
 │             │              │
 │             │              └──M:N──> User (via Like)
 │             │
 │             └──1:N──> Attachment
 │
 ├──M:N──> User (via Follow)
 │
 ├──M:N──> Conversation (via ConversationUser)
 │
 └──1:N──> Message ──1:N──> MessageRead
                │
                └──1:N──> Attachment
```

---

## 🛠️ Step 4: Design Basic Schema

### **Start Simple, Add Complexity Later**

#### **Phase 1: Core Entities Only**

```prisma
// Minimal viable schema
model User {
  id       String @id @default(uuid())
  username String @unique
  email    String @unique
  password String
}

model Post {
  id       String @id @default(uuid())
  title    String
  content  String
  authorId String
  author   User   @relation(fields: [authorId], references: [id])
}
```

#### **Phase 2: Add Timestamps**

```prisma
model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  createdAt DateTime @default(now())  // ✅ Added
  updatedAt DateTime @updatedAt        // ✅ Added
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  authorId  String
  createdAt DateTime @default(now())  // ✅ Added
  updatedAt DateTime @updatedAt        // ✅ Added
  author    User     @relation(fields: [authorId], references: [id])
}
```

#### **Phase 3: Add Relationships**

```prisma
model User {
  id        String   @id @default(uuid())
  username  String   @unique
  email     String   @unique
  password  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  posts     Post[]   // ✅ Added relationship
  comments  Comment[] // ✅ Added relationship
  likes     Like[]    // ✅ Added relationship
}

model Post {
  id        String   @id @default(uuid())
  title     String
  content   String
  authorId  String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  author    User      @relation(fields: [authorId], references: [id])
  comments  Comment[] // ✅ Added
  likes     Like[]    // ✅ Added
}

model Comment {
  id        String   @id @default(uuid())
  text      String
  postId    String
  userId    String
  createdAt DateTime @default(now())
  
  post      Post @relation(fields: [postId], references: [id])
  user      User @relation(fields: [userId], references: [id])
}

model Like {
  userId    String
  postId    String
  createdAt DateTime @default(now())
  
  user User @relation(fields: [userId], references: [id])
  post Post @relation(fields: [postId], references: [id])
  
  @@id([userId, postId])
}
```

---

## ✅ Step 5: Add Constraints & Validation

### **Common Constraints:**

#### **1. Unique Constraints**
Prevent duplicates:
```prisma
model User {
  username String @unique  // No two users with same username
  email    String @unique  // No two users with same email
}
```

#### **2. Composite Unique Constraints**
Combination must be unique:
```prisma
model Like {
  userId String
  postId String
  
  @@id([userId, postId])  // User can only like a post once
}
```

#### **3. Optional vs Required Fields**
```prisma
model User {
  username String   // Required (no ?)
  bio      String?  // Optional (has ?)
  avatarUrl String? // Optional
}
```

#### **4. Default Values**
```prisma
model User {
  role      Role     @default(USER)  // Default to USER role
  createdAt DateTime @default(now()) // Auto-set to current time
}
```

#### **5. Enums (Fixed Values)**
```prisma
enum Role {
  USER
  ADMIN
  MODERATOR
}

model User {
  role Role @default(USER)
}
```

#### **6. Cascade Deletes**
What happens when parent is deleted?

```prisma
model Post {
  id       String    @id
  comments Comment[]
}

model Comment {
  id     String @id
  postId String
  post   Post   @relation(fields: [postId], references: [id], onDelete: Cascade)
  //                                                          ↑
  //                                    Delete comments when post is deleted
}
```

**Options:**
- `onDelete: Cascade` - Delete children too
- `onDelete: SetNull` - Set foreign key to null
- `onDelete: Restrict` - Prevent deletion if children exist

---

## ⚡ Step 6: Optimize for Performance

### **The Performance Mindset**

Ask: **"What queries will run most often?"**

#### **1. Add Indexes**

**Rule:** Index fields used in `WHERE`, `ORDER BY`, `JOIN`

```prisma
model Post {
  id        String   @id
  authorId  String
  createdAt DateTime
  
  @@index([authorId])           // Fast: "Get posts by user"
  @@index([createdAt])          // Fast: "Get recent posts"
  @@index([authorId, createdAt]) // Fast: "Get user's recent posts"
}
```

**Real Example from Our Schema:**
```prisma
model Message {
  id             String   @id
  conversationId String
  createdAt      DateTime
  
  @@index([conversationId, createdAt])
  //       ↑                ↑
  //    Filter by chat    Sort by time
  //
  // Query: "Get messages in conversation, sorted by time"
  // Without index: Scans ALL messages (slow)
  // With index: Direct lookup (fast!)
}
```

#### **2. Denormalization (Strategic Duplication)**

**Problem:** Counting is slow
```typescript
// Slow: Count every time
const likeCount = await prisma.like.count({
  where: { postId: "post-123" }
});
```

**Solution:** Store the count
```prisma
model Post {
  id        String @id
  likeCount Int    @default(0) // ✅ Store count
}

// Update count when like is added/removed
await prisma.post.update({
  where: { id: "post-123" },
  data: { likeCount: { increment: 1 } }
});
```

**Our Real Example:**
```prisma
model PostStats {
  postId       String @id
  likeCount    Int    @default(0)
  commentCount Int    @default(0)
  viewCount    Int    @default(0)
  
  post Post @relation(fields: [postId], references: [id])
}
```

#### **3. Pagination Support**

Always design for pagination:
```prisma
model Post {
  id        String   @id
  createdAt DateTime
  
  @@index([createdAt]) // Enables fast pagination
}
```

```typescript
// Cursor-based pagination (fast for large datasets)
const posts = await prisma.post.findMany({
  take: 20,
  skip: 1,
  cursor: { id: lastPostId },
  orderBy: { createdAt: 'desc' }
});
```

---

## 🎯 Step 7: Handle Edge Cases

### **Think About Real-World Scenarios**

#### **1. Soft Deletes**
Don't actually delete data (for recovery/audit):

```prisma
model Post {
  id        String    @id
  deletedAt DateTime? // null = active, has value = deleted
  
  @@index([deletedAt]) // Fast filtering
}
```

```typescript
// Get only active posts
const posts = await prisma.post.findMany({
  where: { deletedAt: null }
});

// "Delete" a post (soft delete)
await prisma.post.update({
  where: { id: "post-123" },
  data: { deletedAt: new Date() }
});
```

#### **2. Nested Comments (Unlimited Depth)**

**Problem:** Comments can have replies, which can have replies, etc.

**Solution:** Self-referencing with `parentId`
```prisma
model Comment {
  id       String    @id
  text     String
  parentId String?   // null = top-level, has value = reply
  
  parent   Comment?  @relation("CommentTree", fields: [parentId], references: [id])
  replies  Comment[] @relation("CommentTree")
  
  @@index([parentId]) // Fast: "Get replies to this comment"
}
```

#### **3. Group vs Private Chats**

**Problem:** Need to support both 1-on-1 and group chats

**Solution:** Unified `Conversation` model
```prisma
enum ConversationType {
  PRIVATE
  GROUP
}

model Conversation {
  id      String           @id
  type    ConversationType @default(PRIVATE)
  name    String?          // Only for groups
  ownerId String?          // Only for groups
}

model ConversationUser {
  conversationId String
  userId         String
  
  @@id([conversationId, userId])
}
```

#### **4. Message Read Tracking (The Optimization We Added!)**

**Problem:** Counting unread messages is slow in large group chats

**Solution:** Track last read position
```prisma
// Detailed tracking (for "Read by Alice, Bob")
model MessageRead {
  messageId String
  userId    String
  readAt    DateTime
  
  @@id([messageId, userId])
}

// Quick lookup (for "5 unread messages")
model ConversationRead {
  conversationId    String
  userId            String
  lastReadMessageId String?
  lastReadAt        DateTime
  
  @@id([conversationId, userId])
}
```

---

## 🎨 Common Patterns & Best Practices

### **Pattern 1: Polymorphic Associations**

**Problem:** Attachments can belong to Posts OR Messages

**Solution:**
```prisma
model Attachment {
  id        String  @id
  url       String
  postId    String? // Optional
  messageId String? // Optional
  
  post    Post?    @relation(fields: [postId], references: [id])
  message Message? @relation(fields: [messageId], references: [id])
  
  @@index([postId])
  @@index([messageId])
}
```

### **Pattern 2: Audit Trail**

Track who did what when:
```prisma
model Post {
  id        String   @id
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  createdBy String   // Who created it
  updatedBy String?  // Who last updated it
}
```

### **Pattern 3: Versioning**

Keep history of changes:
```prisma
model Post {
  id      String        @id
  content String
  version Int           @default(1)
  history PostHistory[]
}

model PostHistory {
  id        String   @id
  postId    String
  content   String
  version   Int
  createdAt DateTime @default(now())
  
  post Post @relation(fields: [postId], references: [id])
}
```

### **Pattern 4: Tags/Categories (Many-to-Many)**

```prisma
model Post {
  id   String     @id
  tags PostTag[]
}

model Tag {
  id    String    @id
  name  String    @unique
  posts PostTag[]
}

model PostTag {
  postId String
  tagId  String
  
  post Post @relation(fields: [postId], references: [id])
  tag  Tag  @relation(fields: [tagId], references: [id])
  
  @@id([postId, tagId])
}
```

---

## 🏆 Best Practices Checklist

### **✅ Naming Conventions**
- Models: `PascalCase` (User, Post, Comment)
- Fields: `camelCase` (userId, createdAt)
- Relations: Plural for arrays (posts, comments)
- Enums: `UPPER_CASE` (USER, ADMIN)

### **✅ Always Include**
- `id` field (use UUID for distributed systems)
- `createdAt` timestamp
- `updatedAt` timestamp (for tracking changes)

### **✅ Performance**
- Index foreign keys
- Index fields used in WHERE clauses
- Index fields used in ORDER BY
- Consider denormalization for counts

### **✅ Data Integrity**
- Use enums for fixed values
- Add unique constraints
- Use cascade deletes carefully
- Validate at database level when possible

### **✅ Scalability**
- Design for pagination from day 1
- Use soft deletes for important data
- Plan for archiving old data
- Consider sharding strategy for huge tables

---

## 💪 Practice Exercises

### **Exercise 1: E-Commerce Platform**

**Requirements:**
> "Users can browse products, add to cart, place orders. Products have categories, reviews, and ratings. Users can track order status."

**Your Task:**
1. List all entities
2. Identify relationships
3. Design the schema
4. Add indexes for common queries

<details>
<summary>Click to see solution</summary>

**Entities:**
- User
- Product
- Category
- Cart
- CartItem
- Order
- OrderItem
- Review
- Rating

**Schema:**
```prisma
model User {
  id     String  @id @default(uuid())
  email  String  @unique
  cart   Cart?
  orders Order[]
  reviews Review[]
}

model Product {
  id          String   @id @default(uuid())
  name        String
  price       Decimal
  categoryId  String
  category    Category @relation(fields: [categoryId], references: [id])
  cartItems   CartItem[]
  orderItems  OrderItem[]
  reviews     Review[]
  
  @@index([categoryId])
}

model Category {
  id       String    @id @default(uuid())
  name     String    @unique
  products Product[]
}

model Cart {
  id     String     @id @default(uuid())
  userId String     @unique
  user   User       @relation(fields: [userId], references: [id])
  items  CartItem[]
}

model CartItem {
  id        String  @id @default(uuid())
  cartId    String
  productId String
  quantity  Int
  cart      Cart    @relation(fields: [cartId], references: [id])
  product   Product @relation(fields: [productId], references: [id])
  
  @@unique([cartId, productId])
}

model Order {
  id        String      @id @default(uuid())
  userId    String
  status    OrderStatus @default(PENDING)
  total     Decimal
  createdAt DateTime    @default(now())
  user      User        @relation(fields: [userId], references: [id])
  items     OrderItem[]
  
  @@index([userId, createdAt])
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}

model OrderItem {
  id        String  @id @default(uuid())
  orderId   String
  productId String
  quantity  Int
  price     Decimal
  order     Order   @relation(fields: [orderId], references: [id])
  product   Product @relation(fields: [productId], references: [id])
}

model Review {
  id        String   @id @default(uuid())
  productId String
  userId    String
  rating    Int      // 1-5
  comment   String
  createdAt DateTime @default(now())
  product   Product  @relation(fields: [productId], references: [id])
  user      User     @relation(fields: [userId], references: [id])
  
  @@unique([productId, userId]) // One review per user per product
  @@index([productId])
}
```
</details>

### **Exercise 2: Learning Management System**

**Requirements:**
> "Teachers create courses with lessons. Students enroll in courses, watch lessons, submit assignments. Track progress and grades."

**Your Task:** Design the complete schema!

### **Exercise 3: Optimize This Schema**

Given this slow schema, add optimizations:

```prisma
model Post {
  id      String @id
  title   String
  content String
  userId  String
  likes   Like[]
}

model Like {
  id     String @id
  postId String
  userId String
}
```

**Problems:**
- Counting likes is slow
- Finding user's posts is slow
- No pagination support

**Your Task:** Fix it!

---

## 🎯 Summary: The Mental Model

When designing any schema, follow this thought process:

```
1. READ requirements carefully
   ↓
2. EXTRACT nouns (entities) and verbs (relationships)
   ↓
3. DRAW relationships on paper
   ↓
4. START with minimal schema
   ↓
5. ADD timestamps, constraints
   ↓
6. IDENTIFY slow queries
   ↓
7. ADD indexes and optimizations
   ↓
8. THINK about edge cases
   ↓
9. TEST with real data
   ↓
10. ITERATE based on usage
```

### **Key Principles:**

1. **Start Simple** - Don't over-engineer early
2. **Think Queries** - Design for how data will be accessed
3. **Normalize First** - Then denormalize strategically
4. **Index Wisely** - Index what you query, not everything
5. **Plan for Scale** - Pagination, soft deletes, archiving
6. **Document Decisions** - Why did you choose this design?

---

## 📚 Further Learning

### **Books:**
- "Database Design for Mere Mortals" by Michael J. Hernandez
- "SQL Performance Explained" by Markus Winand

### **Practice:**
- Design schemas for: Twitter, Instagram, Uber, Airbnb
- Analyze existing open-source projects
- Build small projects and measure query performance

### **Tools:**
- [dbdiagram.io](https://dbdiagram.io) - Visual schema design
- [Prisma Studio](https://www.prisma.io/studio) - Browse your data
- PostgreSQL EXPLAIN - Analyze query performance

---

## 🎓 Final Advice

**The best way to learn schema design is to:**

1. ✅ **Build real projects** - Theory only goes so far
2. ✅ **Make mistakes** - You'll learn what NOT to do
3. ✅ **Measure performance** - Use real data, not assumptions
4. ✅ **Study existing schemas** - See how others solved problems
5. ✅ **Iterate** - First version is never perfect

**Remember:** Every expert was once a beginner. Keep practicing! 🚀

---

*Created for the Social Media App project - A real-world example of schema design in action.*
