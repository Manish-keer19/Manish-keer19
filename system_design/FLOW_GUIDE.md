# GetPet Backend — Complete Developer Guide

> **Purpose**: This is the single source of truth for backend developers to understand every database table, every API endpoint, how features interconnect, and how data flows through the system.  
> **Stack**: Node.js + Express.js + Prisma ORM + PostgreSQL  
> **Auth**: JWT Bearer Token (`Authorization: Bearer <token>`)  
> **Base URL**: `http://localhost:5000/api/v1`

---

## Table of Contents

1. [Database Schema — Why Each Table Exists](#1-database-schema--why-each-table-exists)
2. [Complete API Reference — All Endpoints](#2-complete-api-reference--all-endpoints)
3. [Feature Flow 1: Auth & Onboarding](#3-feature-flow-1-auth--onboarding)
4. [Feature Flow 2: Breeder Verification & Admin Approval](#4-feature-flow-2-breeder-verification--admin-approval)
5. [Feature Flow 3: Contact Unlock Decision Matrix](#5-feature-flow-3-contact-unlock-decision-matrix)
6. [Feature Flow 4: Plan Purchase, Payment & Subscription](#6-feature-flow-4-plan-purchase-payment--subscription)
7. [Feature Flow 5: Listings, Favorites, Reviews, Reports & Messaging](#7-feature-flow-5-listings-favorites-reviews-reports--messaging)
8. [Module Architecture & File Map](#8-module-architecture--file-map)

---

## 1. Database Schema — Why Each Table Exists

### Enums

| Enum | Values | Purpose |
|------|--------|---------|
| `UserRole` | `buyer`, `breeder`, `admin` | Controls which dashboard the user sees (mobile: buyer/breeder, web: admin). |
| `VerificationStatus` | `pending`, `approved`, `rejected` | Tracks breeder KYC document review lifecycle. |
| `ListingStatus` | `available`, `reserved`, `sold`, `archived` | Allows breeders to manage listing visibility. |
| `SubscriptionStatus` | `active`, `expired`, `cancelled`, `pending` | Tracks subscription lifecycle after purchase. |
| `PaymentStatus` | `created`, `processing`, `success`, `failed`, `refunded` | Tracks Razorpay payment state machine. |
| `ReportStatus` | `pending`, `under_review`, `resolved`, `dismissed` | Admin moderation workflow for reported content. |
| `PlanTarget` | `buyer`, `breeder` | Determines which plans appear for which user role. |
| `UnlockSource` | `free_credit`, `subscription`, `direct_purchase` | Records how a buyer unlocked a breeder contact. |
| `DocumentType` | `selfie`, `aadhaar`, `pan`, `pet_shop_license`, `kennel_club_cert`, `health_cert`, `other` | Typed identity/business document categories. |

### Tables

| Table | Purpose | Key Relationships |
|-------|---------|-------------------|
| **`users`** | Every person in the system (buyer, breeder, or admin). Stores auth phone, role, city, and the `free_unlock_credits` counter (default: 1). | Has many: `breeders`, `subscriptions`, `payments`, `favorites`, `contacts_unlocked`, `messages`, `reviews`, `reports`, `push_tokens`. |
| **`otps`** | Stores temporary OTP codes for phone verification. Deleted after successful verification. | Standalone — matched by `mobile_no`. |
| **`breeders`** | A breeder store profile linked to a user. Stores store name, city, selfie, doc references, `avg_rating`, `review_count`, and verification `status` (`pending`/`approved`/`rejected`). A user can technically have multiple breeder profiles but typically has one. | Belongs to `users`. Has many: `listings`, `reviews`, `contacts_unlocked`, `verification_requests`, `documents`, `reports`. |
| **`breeder_documents`** | **Relational document storage** — each row is one uploaded identity/business document (typed via `DocumentType` enum). Linked to both a `breeder` and a `verification_request`. Admin approves/rejects documents in batch. | Belongs to `breeders` and optionally to `verification_requests`. |
| **`verification_requests`** | Created when a breeder submits onboarding. Admin reviews the request and all attached `breeder_documents`. Tracks `reviewer_id`, `admin_notes`, `rejection_reason`. | Belongs to `breeders`. Has many: `documents`. Reviewed by: `users` (admin role). |
| **`listings`** | A pet for sale. Contains name, breed, type, price, photos array, health cert URL, video URL. Status tracks availability. | Belongs to `breeders`. Has many: `favorites`, `contacts_unlocked`, `messages`, `reports`. |
| **`favorites`** | Buyer bookmarks/saves a listing. Unique constraint on `(user_id, listing_id)` prevents duplicates. | Belongs to `users` and `listings`. |
| **`contacts_unlocked`** | **Core monetization table** — records that buyer X unlocked breeder Y's contact details. Stores `unlocked_via` (free_credit / subscription / direct_purchase). Unique constraint on `(buyer_id, breeder_id)` means once unlocked, it's permanent. | Belongs to `users` (buyer), `breeders`, optionally `listings`. |
| **`reviews`** | Buyer writes a rating + comment for a breeder. Unique constraint on `(buyer_id, breeder_id)` means one review per buyer per breeder. | Belongs to `users` (buyer) and `breeders`. |
| **`messages`** | Chat messages between buyer and breeder. Optionally linked to a listing for context. | Belongs to `users` (sender & recipient), optionally `listings`. |
| **`plans`** | Subscription plan definitions (seeded by admin). Contains `price`, `contact_unlocks`, `max_listings`, `validity_days`, `features[]`. `target_role` determines if it's a buyer or breeder plan. `code` is unique identifier (e.g. `BUYER_STARTER`). | Has many: `subscriptions`, `payments`. |
| **`payments`** | Razorpay payment records. Created with status `created` when order is initiated. Updated to `success` after signature verification. Stores `razorpay_order_id`, `razorpay_payment_id`, `razorpay_signature`. | Belongs to `users` and `plans`. Has many: `subscriptions`. |
| **`subscriptions`** | Active subscription created after successful payment. Contains `remaining_unlocks` (decremented on each contact unlock), `start_date`, `end_date`. Status transitions: `active` → `expired` (via cron) or `cancelled`. | Belongs to `users`, `plans`, and `payments`. |
| **`push_tokens`** | Expo/FCM push notification tokens. Composite primary key `(user_id, token)`. | Belongs to `users`. |
| **`reports`** | Content moderation — buyer reports a listing or breeder. Admin reviews on web panel. | Belongs to `users` (reporter & reviewer), optionally `listings` and `breeders`. |

---

## 2. Complete API Reference — All Endpoints

### Auth Module (`/api/v1/auth`)

| Method | Endpoint | Auth | Description | Request Body |
|--------|----------|------|-------------|-------------|
| `POST` | `/send-otp` | ❌ | Send SMS OTP to phone number | `{ "phone": "+919876543210" }` |
| `POST` | `/verify-otp` | ❌ | Verify OTP, return JWT token | `{ "phone": "+919876543210", "otp": "1234" }` |

### User Module (`/api/v1/user`)

| Method | Endpoint | Auth | Description | Request Body |
|--------|----------|------|-------------|-------------|
| `GET` | `/me` | ✅ | Get current user profile + active subscription + counts | — |
| `POST` | `/onboard/buyer` | ✅ | Complete buyer profile (sets `role=buyer`, `is_onboarded=true`) | `{ "name": "Amit", "city": "Mumbai", "city_lat": 19.07, "city_lng": 72.87 }` |
| `PUT` | `/profile` | ✅ | Update user name, email, city | `{ "name": "...", "email": "...", "city": "..." }` |
| `POST` | `/unlock-contact` | ✅ | **Contact Unlock Decision Matrix** — checks free credits → subscription → paywall | `{ "breeder_id": "uuid", "listing_id": "uuid" }` |
| `GET` | `/unlocked-contacts` | ✅ | List all contacts unlocked by this buyer | — |

### Breeder Module (`/api/v1/breeder`)

| Method | Endpoint | Auth | Description | Request Body |
|--------|----------|------|-------------|-------------|
| `POST` | `/onboard` | ✅ | Breeder onboarding: creates Breeder + VerificationRequest + BreederDocuments | `{ "name": "...", "city": "...", "selfie_url": "...", "id_doc_type": "aadhaar", "id_doc_url": "...", "has_pet_shop": true, "pet_documents": [{"doc_type": "pet_shop_license", "doc_url": "..."}] }` |
| `GET` | `/me` | ✅ | Get breeder profile with documents, verification requests, listing/review counts | — |
| `PUT` | `/me` | ✅ | Update breeder profile fields | `{ "name": "...", "city": "..." }` |
| `GET` | `/status` | ✅ | Get verification status (`pending`/`approved`/`rejected`) + rejection reason | — |
| `GET` | `/public/:id` | ❌ | Public breeder profile for buyers (safe fields + active listings) | — |
| `PATCH` | `/:id/approve` | ✅ | Legacy direct approve endpoint | — |

### Payment Module (`/api/v1/payment`)

| Method | Endpoint | Auth | Description | Request Body |
|--------|----------|------|-------------|-------------|
| `GET` | `/plans` | ❌ | List active subscription plans (filterable by `?target_role=buyer`) | — |
| `POST` | `/create-order` | ✅ | Create Razorpay order + Payment record (`status: created`) | `{ "plan_id": "uuid" }` |
| `POST` | `/verify` | ✅ | Verify Razorpay signature, set `Payment.status=success`, create active `Subscription` | `{ "razorpay_order_id": "...", "razorpay_payment_id": "...", "razorpay_signature": "..." }` |
| `GET` | `/history` | ✅ | User's payment history | — |

### Subscription Module (`/api/v1/subscription`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/me` | ✅ | All subscriptions for this user (past + active) |
| `GET` | `/active` | ✅ | Current active subscription only (if any) |

### Admin Module (`/api/v1/admin`) — All routes require `isAuthenticated` + `isAdmin`

| Method | Endpoint | Description | Request Body |
|--------|----------|-------------|-------------|
| `GET` | `/verifications/pending` | List all pending breeder verification requests with documents | — |
| `GET` | `/verifications/:id` | Get specific verification request detail | — |
| `POST` | `/verifications/:id/approve` | **Approve** breeder — updates VerificationRequest, BreederDocuments, and Breeder in one transaction | `{ "admin_notes": "..." }` |
| `POST` | `/verifications/:id/reject` | **Reject** breeder — requires `rejection_reason` | `{ "rejection_reason": "...", "admin_notes": "..." }` |
| `GET` | `/users` | List all users (filterable: `?role=buyer&is_onboarded=true`) | — |
| `GET` | `/breeders` | List all breeders (filterable: `?status=pending`) | — |
| `GET` | `/stats` | Platform overview (total users, breeders, listings, revenue, etc.) | — |
| `POST` | `/seed-plans` | Upsert default subscription plans | — |

### Media Module (`/api/v1/media`)

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `POST` | `/upload` | ✅ | Upload file (multipart/form-data, field: `file`). Returns public URL. |

---

## 3. Feature Flow 1: Auth & Onboarding

### How Phone OTP Auth Works

```
┌──────────────┐    POST /auth/send-otp     ┌──────────────────┐
│  Mobile App  │ ────────────────────────►  │  Backend Server   │
│  enters phone│                            │                   │
└──────────────┘                            │  1. Generate 4-digit OTP
                                            │  2. Upsert into `otps` table
                                            │  3. Send SMS via Twilio
                                            │  4. Return { otp } (dev mode only)
                                            └────────┬─────────┘
                                                     │
┌──────────────┐    POST /auth/verify-otp   ┌────────▼─────────┐
│  Mobile App  │ ────────────────────────►  │  Backend Server   │
│  enters OTP  │                            │                   │
└──────────────┘                            │  1. Lookup `otps` by phone
                                            │  2. Validate OTP + expiry
                                            │  3. Find or Create user in `users`
                                            │  4. Delete OTP row (consumed)
                                            │  5. Generate JWT token
                                            │  6. Return { token, user, needsOnboarding }
                                            └──────────────────┘
```

### Why Two Onboarding Paths?

- **Buyer onboarding** (`POST /user/onboard/buyer`): Simple — just name + city. Sets `role = buyer`, `is_onboarded = true`.
- **Breeder onboarding** (`POST /breeder/onboard`): Complex — creates **4 table rows** in a single database transaction:
  1. Updates `users` row (`role = breeder`, `is_onboarded = true`)
  2. Creates `breeders` row (store profile with `status = pending`)
  3. Creates `verification_requests` row (for admin to review)
  4. Creates multiple `breeder_documents` rows (Aadhaar, Selfie, Pet Shop License)

---

## 4. Feature Flow 2: Breeder Verification & Admin Approval

### Why We Need `breeder_documents` + `verification_requests`

Instead of storing document URLs as unindexed string arrays on the `breeders` table, we use a **relational document model**:

- Each document is a typed row (`doc_type: aadhaar | pan | selfie | pet_shop_license | ...`).
- Each document has its own `status` (`pending` → `approved` or `rejected`).
- Documents are grouped under a `verification_request`, allowing admin to review a batch.
- This enables future features like re-submission of individual documents, document expiry, etc.

### Database State: Before & After Admin Approval

**BEFORE (Breeder just submitted onboarding):**
```
users:                  { role: "breeder", is_onboarded: true }
breeders:               { status: "pending", is_approved: false, verified: false }
verification_requests:  { status: "pending", reviewer_id: null }
breeder_documents:      [{ doc_type: "aadhaar", status: "pending" }, { doc_type: "selfie", status: "pending" }]
```

**AFTER Admin Approves (single atomic transaction):**
```
verification_requests:  { status: "approved", reviewer_id: "admin-uuid", reviewed_at: now() }
breeder_documents:      [{ status: "approved" }, { status: "approved" }]   ← batch updateMany
breeders:               { status: "approved", is_approved: true, verified: true }
```

**AFTER Admin Rejects:**
```
verification_requests:  { status: "rejected", rejection_reason: "Blurry aadhaar photo", reviewed_at: now() }
breeder_documents:      [{ status: "rejected" }, { status: "rejected" }]
breeders:               { status: "rejected", is_approved: false, verified: false }
```

### Prisma Transaction Code (Admin Approval)
```typescript
await prisma.$transaction([
  prisma.verificationRequest.update({
    where: { id: verificationRequestId },
    data: { status: "approved", reviewer_id: adminId, admin_notes: "Docs verified.", reviewed_at: new Date() }
  }),
  prisma.breederDocument.updateMany({
    where: { verification_request_id: verificationRequestId },
    data: { status: "approved" }
  }),
  prisma.breeder.update({
    where: { id: breederId },
    data: { status: "approved", is_approved: true, verified: true }
  }),
]);
```

---

## 5. Feature Flow 3: Contact Unlock Decision Matrix

This is the **core monetization logic** of GetPet. When a buyer wants to see a breeder's phone number, the system checks credits in priority order.

### Decision Tree

```
          ┌─────────────────────────────────────────┐
          │  Buyer taps "Unlock Breeder Contact"     │
          │  POST /user/unlock-contact               │
          │  Body: { breeder_id, listing_id }        │
          └──────────────────┬──────────────────────┘
                             │
                             ▼
               ┌─────────────────────────────┐
               │  SELECT FROM contacts_unlocked │
               │  WHERE buyer_id = X           │
               │    AND breeder_id = Y         │
               └──────────┬──────────────────┘
                          │
                   ┌──────┴──────┐
              Found │            │ Not Found
                   ▼            ▼
          ┌──────────────┐   ┌──────────────────────┐
          │ HTTP 200     │   │ Check free_unlock_    │
          │ Already      │   │ credits on users table│
          │ unlocked.    │   └──────────┬───────────┘
          │ Return       │             │
          │ contact info │      ┌──────┴──────┐
          └──────────────┘    > 0 │          │ = 0
                                  ▼          ▼
                          ┌──────────┐   ┌──────────────────────┐
                          │ TXNCTION │   │ SELECT FROM           │
                          │ Deduct 1 │   │ subscriptions         │
                          │ credit   │   │ WHERE user_id = X     │
                          │ Insert   │   │   AND status = active  │
                          │ unlock   │   │   AND end_date >= now  │
                          │ via      │   │   AND remaining > 0   │
                          │ free_    │   └──────────┬───────────┘
                          │ credit   │             │
                          └──────────┘      ┌──────┴──────┐
                                          Found │        │ Not Found
                                               ▼        ▼
                                       ┌──────────┐  ┌──────────────┐
                                       │ TXNCTION │  │ HTTP 402     │
                                       │ Deduct 1 │  │ Paywall      │
                                       │ sub      │  │ Required.    │
                                       │ credit   │  │ Return       │
                                       │ Insert   │  │ available    │
                                       │ unlock   │  │ plans[]      │
                                       │ via sub  │  └──────────────┘
                                       └──────────┘
```

### Why `contacts_unlocked` Has a Unique Constraint

```prisma
@@unique([buyer_id, breeder_id], map: "buyer_breeder_unlock_unique")
```

This means: **once a buyer unlocks a breeder, the unlock is permanent**. The buyer can always view that breeder's contact info again without spending credits. The system checks this first (Case 1 above) so re-views are free.

### Why `unlocked_via` Exists

The `unlocked_via` enum (`free_credit` / `subscription` / `direct_purchase`) lets you analyze revenue:
- How many unlocks come from free credits vs paid subscriptions?
- Which plans drive the most unlock activity?

---

## 6. Feature Flow 4: Plan Purchase, Payment & Subscription

### How Plans, Payments & Subscriptions Relate

```
┌──────────────┐         ┌──────────────┐         ┌──────────────────┐
│    plans     │◄────────│   payments   │────────►│  subscriptions   │
│ (seed data)  │         │ (per txn)    │         │  (active sub)    │
├──────────────┤         ├──────────────┤         ├──────────────────┤
│ name         │         │ user_id      │         │ user_id          │
│ code (unique)│         │ plan_id ─────┼────┐    │ plan_id ─────────┤
│ price        │         │ amount       │    │    │ payment_id ──────┤
│ contact_     │         │ razorpay_    │    │    │ status: active   │
│   unlocks: 5 │         │   order_id   │    │    │ remaining_       │
│ validity_    │         │ status:      │    │    │   unlocks: 5     │
│   days: 30   │         │   created →  │    │    │ start_date       │
│ target_role  │         │   success    │    │    │ end_date (+ 30d) │
│ is_active    │         │ paid_at      │    │    │                  │
└──────────────┘         └──────────────┘    │    └──────────────────┘
                                              │
                              plan.contact_unlocks is copied to
                              subscription.remaining_unlocks on creation
```

### Step-by-Step Payment Flow

**Step 1: User selects plan → App calls `POST /payment/create-order`**

- Backend creates a `payments` row with `status: "created"` and a unique `razorpay_order_id`.
- Returns the order ID + amount in paise (₹99 = 9900 paise) for the Razorpay SDK.

**Step 2: Razorpay SDK opens on mobile → User completes payment → SDK returns callback**

- App receives `razorpay_payment_id` and `razorpay_signature` from SDK callback.

**Step 3: App calls `POST /payment/verify`**

- Backend verifies HMAC-SHA256 signature: `sha256(order_id|payment_id, secret) === signature`.
- If valid, executes a **single database transaction**:
  1. Updates `payments` row: `status = "success"`, `razorpay_payment_id`, `paid_at = now()`
  2. Creates `subscriptions` row: `status = "active"`, `remaining_unlocks = plan.contact_unlocks`, `end_date = now + plan.validity_days`

### Database State: Before & After

```
BEFORE (User buys Buyer Starter Plan — ₹99, 5 unlocks, 30 days):

  plans:          { id: "p-01", code: "BUYER_STARTER", price: 99, contact_unlocks: 5, validity_days: 30 }
  payments:       [] (empty)
  subscriptions:  [] (empty)

AFTER create-order:

  payments:       [{ razorpay_order_id: "order_K123", status: "created", amount: 99 }]

AFTER verify (success):

  payments:       [{ status: "success", razorpay_payment_id: "pay_P456", paid_at: "2026-08-20T..." }]
  subscriptions:  [{ status: "active", remaining_unlocks: 5, end_date: "2026-09-19T..." }]
```

---

## 7. Feature Flow 5: Listings, Favorites, Reviews, Reports & Messaging

### Listings

- **Created by**: Breeder (only approved breeders should create listings in production).
- **Photos**: Array of URL strings. Upload each photo via `POST /media/upload` first, then include URLs in listing creation payload.
- **Status lifecycle**: `available` → `reserved` → `sold` (or `archived` by breeder).

### Favorites

- Simple bookmark. Unique constraint `(user_id, listing_id)` prevents duplicates.
- Toggle endpoint: if favorite exists, delete it; if not, create it.

### Reviews

- Buyer rates a breeder (1.0 to 5.0, half-star precision via `Decimal(2,1)`).
- Unique constraint `(buyer_id, breeder_id)` → one review per breeder per buyer.
- Breeder's `avg_rating` and `review_count` should be updated after each review (application logic or database trigger).

### Reports

- Buyer reports a listing or breeder for moderation.
- Admin reviews on web panel, sets `status`: `pending` → `under_review` → `resolved` / `dismissed`.

### Messaging

- Direct chat between buyer and breeder.
- Each message has `sender_id`, `recipient_id`, `body`, `created_at`, `read_at`.
- Optionally linked to a `listing_id` for context.
- Indexed by `(sender_id, recipient_id, created_at)` for efficient conversation loading.

---

## 8. Module Architecture & File Map

```
backend/src/
├── app.ts                          # Express app setup, route mounting, middleware
├── index.ts                        # Server startup (port binding)
├── prisma.ts                       # Prisma client singleton
│
├── middleware/
│   ├── auth.middleware.ts           # isAuthenticated (JWT verify), isAdmin (role check)
│   ├── error.middleware.ts          # Global error handler
│   └── multer.middleware.ts         # File upload middleware
│
├── modules/
│   ├── auth/
│   │   ├── auth.controller.ts      # sendOtp, verifyOtp
│   │   └── auth.route.ts           # POST /send-otp, POST /verify-otp
│   │
│   ├── user/
│   │   ├── user.controller.ts      # getMe, completeBuyerProfile, updateUserProfile,
│   │   │                           # unlockBreederContact (Decision Matrix), getUnlockedContacts
│   │   └── user.route.ts           # GET /me, POST /onboard/buyer, PUT /profile,
│   │                               # POST /unlock-contact, GET /unlocked-contacts
│   │
│   ├── breeder/
│   │   ├── breeder.controller.ts   # createBreeder (onboarding + docs + verification),
│   │   │                           # getBreederProfile, updateBreederProfile, getBreederStatus,
│   │   │                           # getPublicBreeder, approveBreeder
│   │   └── breeder.route.ts        # POST /onboard, GET /me, PUT /me, GET /status,
│   │                               # GET /public/:id, PATCH /:id/approve
│   │
│   ├── payment/
│   │   ├── payment.controller.ts   # getPlans, createPaymentOrder, verifyPayment, getPaymentHistory
│   │   └── payment.route.ts        # GET /plans, POST /create-order, POST /verify, GET /history
│   │
│   ├── subscription/
│   │   ├── subscription.controller.ts  # getMySubscriptions, getActiveSubscription
│   │   └── subscription.route.ts       # GET /me, GET /active
│   │
│   ├── admin/
│   │   ├── Admin.controller.ts     # getPendingVerifications, getVerificationDetails,
│   │   │                           # approveBreederVerification, rejectBreederVerification,
│   │   │                           # getAllUsers, getAllBreeders, getAdminStats, seedPlans
│   │   └── Admin.route.ts          # All routes use isAuthenticated + isAdmin middleware
│   │
│   ├── media/
│   │   ├── media.controller.ts     # uploadMedia (Cloudinary/S3)
│   │   └── media.route.ts          # POST /upload (multipart/form-data)
│   │
│   └── query/                      # (Future: shared query helpers)
│
├── utils/
│   ├── AppError.ts                 # Custom error class with statusCode
│   ├── asyncHandler.ts             # Wraps async route handlers to catch errors
│   ├── response.ts                 # sendResponse() standard format
│   └── token.ts                    # generateAccessToken, generateRefreshToken, verifyAccessToken
│
├── config/
│   ├── swagger.ts                  # Swagger/OpenAPI spec
│   └── twilio.ts                   # Twilio SMS client
│
└── cron/                           # (Future: subscription expiry cron jobs)
```

### Standard API Response Format

Every endpoint returns this shape:
```json
{
  "success": true,
  "message": "Human-readable message",
  "data": { ... }
}
```

Error responses:
```json
{
  "success": false,
  "message": "Error description",
  "statusCode": 400
}
```

---

## Quick Reference: Environment Variables

| Variable | Purpose |
|----------|---------|
| `DATABASE_URL` | PostgreSQL connection string |
| `JWT_SECRET` | Secret for signing access tokens |
| `JWT_REFRESH_SECRET` | Secret for refresh tokens |
| `TWILIO_ACCOUNT_SID` | Twilio SMS account |
| `TWILIO_AUTH_TOKEN` | Twilio SMS auth |
| `TWILIO_PHONE_NUMBER` | Twilio sender number |
| `TWILIO_SMS_BODY` | SMS template with `{{otp}}` placeholder |
| `RAZORPAY_KEY_ID` | Razorpay public key (for frontend) |
| `RAZORPAY_KEY_SECRET` | Razorpay secret (for signature verification) |
| `CLOUDINARY_*` / `AWS_*` | Media upload service credentials |
| `NODE_ENV` | `development` / `production` |
