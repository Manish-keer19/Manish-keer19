# GetPet Android Mobile App — Ultimate Screen, API & Mock API Blueprint

> **Purpose**: This is the single source of truth for any AI agent or developer to build every mobile screen, wire up every API call, and test with mock data — without needing to read any other file.  
> **Framework**: React Native (Expo) — JavaScript  
> **Mobile Roles**: **Buyer** & **Breeder** ONLY (Admin is on the Web Panel)  
> **Backend**: Express.js REST API at `http://<IP>:5000/api/v1`  
> **Auth**: JWT Bearer Token in `Authorization` header for all protected routes.

---

## Table of Contents

1. [Navigation Architecture](#1-navigation-architecture)
2. [Auth Token & State Management](#2-auth-token--state-management)
3. [Config & API Toggle](#3-config--api-toggle)
4. [Complete Mock API Engine](#4-complete-mock-api-engine)
5. [API Service Layer](#5-api-service-layer)
6. [AUTH SCREENS](#6-auth-screens)
7. [ONBOARDING SCREENS](#7-onboarding-screens)
8. [BUYER SCREENS](#8-buyer-screens)
9. [BREEDER SCREENS](#9-breeder-screens)
10. [SHARED SCREENS & MODALS](#10-shared-screens--modals)
11. [Buyer Feature Summary](#11-buyer-feature-summary)
12. [Breeder Feature Summary](#12-breeder-feature-summary)
13. [Implementation Checklist](#13-implementation-checklist)

---

## 1. Navigation Architecture

```
RootNavigator (conditional based on auth state)
│
│── IF no token in AsyncStorage:
│   └── AuthStack
│       ├── PhoneAuthScreen
│       └── OTPVerifyScreen
│
│── IF token exists but user.is_onboarded === false:
│   └── OnboardingStack
│       └── OnboardingScreen (role selection + profile form)
│
│── IF token exists, is_onboarded === true, role === "buyer":
│   └── BuyerTabNavigator (Bottom Tabs)
│       ├── 🏠 HomeTab ──────────> HomeScreen
│       ├── 🔍 SearchTab ────────> SearchScreen
│       ├── ❤️ SavedTab ─────────> SavedScreen
│       ├── 💬 MessagesTab ──────> MessagesScreen
│       └── 👤 ProfileTab ───────> ProfileScreen
│
│── IF token exists, is_onboarded === true, role === "breeder":
│   └── BreederTabNavigator (Bottom Tabs)
│       ├── 📊 DashboardTab ─────> BreederHomeScreen
│       ├── ➕ AddListingTab ────> AddListingScreen
│       ├── 💬 MessagesTab ──────> MessagesScreen
│       └── 👤 ProfileTab ───────> BreederProfile
│
└── SharedStack (push screens accessible from any tab)
    ├── ListingDetail
    ├── ContactRevealed
    ├── SubscriptionScreen (Paywall)
    ├── GetVerifiedScreen
    ├── BreederPendingScreen
    ├── BreederPublicProfile
    ├── ChatScreen
    ├── AIChatScreen
    ├── EditProfileScreen
    ├── UnlockedContactsScreen
    ├── ReviewModal
    ├── ReportModal
    ├── PaymentHistoryScreen
    └── LegalScreen
```

---

## 2. Auth Token & State Management

Store these keys in `AsyncStorage`:

| Key | Type | Description |
|-----|------|-------------|
| `userToken` | `string` | JWT access token from `/auth/verify-otp` response |
| `refreshToken` | `string` | JWT refresh token |
| `userProfile` | `JSON string` | Full user object (id, name, role, city, is_onboarded, free_unlock_credits) |
| `breederProfile` | `JSON string` | Breeder object if role === "breeder" (id, status, is_approved, verified) |

**On every app launch:**
1. Read `userToken` from AsyncStorage.
2. If token exists, call `GET /api/v1/user/me` to refresh user state.
3. Route based on `is_onboarded` and `role`.

---

## 3. Config & API Toggle

**File: `src/lib/config.js`**
```javascript
export const CONFIG = {
  // Set true to use mock data (no backend needed), false for live server
  USE_MOCK_API: true,

  // Your machine's local network IP (not localhost — Android can't reach it)
  API_BASE_URL: "http://192.168.1.100:5000/api/v1",
};
```

---

## 4. Complete Mock API Engine

**File: `src/lib/mockApi.js`**

This is a **fully self-contained mock backend** that simulates every API endpoint. It mutates in-memory state so screens behave realistically during development.

```javascript
// ─────────────────────────────────────────────────────────
// MOCK DATABASE — In-Memory State
// ─────────────────────────────────────────────────────────
const delay = (ms = 600) => new Promise(r => setTimeout(r, ms));

let MOCK_STATE = {
  // ── User ──
  currentUser: {
    id: "u-101",
    phone: "+919876543210",
    name: "Amit Kumar",
    email: null,
    role: "buyer",         // "buyer" | "breeder"
    city: "Mumbai",
    city_lat: 19.076,
    city_lng: 72.8777,
    is_onboarded: true,
    free_unlock_credits: 1,
    created_at: "2026-08-01T10:00:00.000Z",
  },

  // ── Breeder (only if currentUser.role === "breeder") ──
  currentBreeder: {
    id: "b-201",
    user_id: "u-101",
    name: "Paws & Tails Kennel",
    status: "approved",    // "pending" | "approved" | "rejected"
    is_approved: true,
    verified: true,
    city: "Mumbai",
    city_lat: 19.076,
    city_lng: 72.8777,
    selfie_url: "https://picsum.photos/200/200?random=100",
    has_pet_shop: true,
    avg_rating: 4.8,
    review_count: 12,
    member_since: "Aug 2026",
    documents: [
      { id: "doc-1", doc_type: "aadhaar", doc_url: "https://picsum.photos/300/200?random=10", status: "approved" },
      { id: "doc-2", doc_type: "selfie", doc_url: "https://picsum.photos/300/200?random=11", status: "approved" },
      { id: "doc-3", doc_type: "pet_shop_license", doc_url: "https://picsum.photos/300/200?random=12", status: "approved" },
    ],
    verificationRequest: { id: "vr-301", status: "approved", rejection_reason: null, admin_notes: "All docs verified" },
  },

  // ── Listings ──
  listings: [
    {
      id: "l-901",
      breeder_id: "b-201",
      name: "Rocky",
      breed: "Golden Retriever",
      type: "dog",
      gender: "Male",
      price: 18000,
      description: "Playful, vaccinated, KCI registered. Parents on premises.",
      photos: [
        "https://picsum.photos/600/400?random=1",
        "https://picsum.photos/600/400?random=2",
        "https://picsum.photos/600/400?random=3",
      ],
      dob: "2026-06-15",
      status: "available",
      video_url: null,
      health_cert_url: "https://picsum.photos/300/200?random=20",
      is_new: true,
      created_at: "2026-08-10T12:00:00.000Z",
      breeder: {
        id: "b-201", name: "Paws & Tails Kennel", is_approved: true, verified: true,
        avg_rating: 4.8, review_count: 12, selfie_url: "https://picsum.photos/200/200?random=100",
        city: "Mumbai",
      },
    },
    {
      id: "l-902",
      breeder_id: "b-202",
      name: "Luna",
      breed: "Persian",
      type: "cat",
      gender: "Female",
      price: 12000,
      description: "Beautiful blue-eyed Persian kitten. Dewormed & litter trained.",
      photos: [
        "https://picsum.photos/600/400?random=4",
        "https://picsum.photos/600/400?random=5",
      ],
      dob: "2026-07-01",
      status: "available",
      video_url: null,
      health_cert_url: null,
      is_new: true,
      created_at: "2026-08-12T09:00:00.000Z",
      breeder: {
        id: "b-202", name: "Royal Cats Mumbai", is_approved: true, verified: true,
        avg_rating: 4.9, review_count: 8, selfie_url: "https://picsum.photos/200/200?random=101",
        city: "Mumbai",
      },
    },
    {
      id: "l-903",
      breeder_id: "b-203",
      name: "Max",
      breed: "German Shepherd",
      type: "dog",
      gender: "Male",
      price: 25000,
      description: "Working line GSD. KCI certified parents. Champion bloodline.",
      photos: ["https://picsum.photos/600/400?random=6"],
      dob: "2026-05-20",
      status: "available",
      video_url: null,
      health_cert_url: "https://picsum.photos/300/200?random=21",
      is_new: false,
      created_at: "2026-08-05T15:00:00.000Z",
      breeder: {
        id: "b-203", name: "Elite K9 Breeders", is_approved: true, verified: true,
        avg_rating: 4.7, review_count: 22, selfie_url: "https://picsum.photos/200/200?random=102",
        city: "Pune",
      },
    },
  ],

  // ── Favorites ──
  favorites: ["l-901"],

  // ── Unlocked Contacts ──
  unlockedContacts: [],

  // ── Plans ──
  plans: [
    {
      id: "p-01", name: "Buyer Starter Plan", code: "BUYER_STARTER", target_role: "buyer",
      price: 99.0, contact_unlocks: 5, validity_days: 30, max_listings: 0,
      description: "Unlock 5 breeder contacts", features: ["5 Contact Unlocks", "30 Days Validity", "Chat Support"],
      is_active: true,
    },
    {
      id: "p-02", name: "Buyer Pro Plan", code: "BUYER_PRO", target_role: "buyer",
      price: 249.0, contact_unlocks: 15, validity_days: 60, max_listings: 0,
      description: "Unlock 15 breeder contacts", features: ["15 Contact Unlocks", "60 Days Validity", "Priority Support"],
      is_active: true,
    },
    {
      id: "p-03", name: "Breeder Starter Plan", code: "BREEDER_STARTER", target_role: "breeder",
      price: 499.0, contact_unlocks: 0, validity_days: 30, max_listings: 10,
      description: "List up to 10 pets", features: ["10 Active Listings", "Verified Badge", "30 Days Validity"],
      is_active: true,
    },
  ],

  // ── Subscription (active or null) ──
  activeSubscription: null,

  // ── Payment History ──
  payments: [],

  // ── Messages ──
  conversations: [
    {
      recipientId: "b-201",
      recipientName: "Paws & Tails Kennel",
      recipientAvatar: "https://picsum.photos/200/200?random=100",
      lastMessage: "Is Rocky still available?",
      lastMessageAt: "2026-08-20T14:30:00.000Z",
      unreadCount: 0,
    },
  ],
  messages: {
    "b-201": [
      { id: "m-1", sender_id: "u-101", recipient_id: "b-201", body: "Hi, is Rocky still available?", created_at: "2026-08-20T14:30:00.000Z", read_at: null },
      { id: "m-2", sender_id: "b-201", recipient_id: "u-101", body: "Yes! Rocky is available. Would you like to visit?", created_at: "2026-08-20T14:32:00.000Z", read_at: null },
    ],
  },

  // ── Reviews ──
  reviews: {
    "b-201": [
      { id: "r-1", buyer_id: "u-201", rating: 5.0, comment: "Excellent breeder. Very transparent and caring.", created_at: "2026-08-15T10:00:00.000Z", buyer_name: "Priya S." },
      { id: "r-2", buyer_id: "u-202", rating: 4.5, comment: "Good experience. Healthy puppy.", created_at: "2026-08-10T12:00:00.000Z", buyer_name: "Vikram R." },
    ],
  },

  // ── Reports ──
  reports: [],

  // ── Push Tokens ──
  pushTokenRegistered: false,
};

// ─────────────────────────────────────────────────────────
// MOCK API FUNCTIONS
// ─────────────────────────────────────────────────────────

export const mockApi = {

  // ═══════════════════════════════════════════════════════
  // AUTH
  // ═══════════════════════════════════════════════════════

  sendOtp: async (phone) => {
    await delay();
    return {
      success: true,
      message: "OTP sent successfully via SMS",
      data: {
        phone: phone.replace("+91", ""),
        formattedPhone: phone.startsWith("+91") ? phone : `+91${phone}`,
        smsSent: true,
        otp: "1234",  // Always 1234 in mock mode
      },
    };
  },

  verifyOtp: async (phone, otp) => {
    await delay();
    if (otp !== "1234") {
      throw new Error("Invalid OTP");
    }
    return {
      success: true,
      message: MOCK_STATE.currentUser.is_onboarded ? "Logged in successfully" : "Account created successfully",
      data: {
        token: "mock-jwt-token-" + Date.now(),
        refreshToken: "mock-refresh-token-" + Date.now(),
        isNewUser: !MOCK_STATE.currentUser.is_onboarded,
        needsOnboarding: !MOCK_STATE.currentUser.is_onboarded,
        user: { ...MOCK_STATE.currentUser, breeders: MOCK_STATE.currentUser.role === "breeder" ? [MOCK_STATE.currentBreeder] : [] },
      },
    };
  },

  // ═══════════════════════════════════════════════════════
  // USER PROFILE
  // ═══════════════════════════════════════════════════════

  getMe: async () => {
    await delay(300);
    return {
      success: true,
      message: "User fetched successfully",
      data: {
        user: {
          ...MOCK_STATE.currentUser,
          breeders: MOCK_STATE.currentUser.role === "breeder" ? [MOCK_STATE.currentBreeder] : [],
          activeSubscription: MOCK_STATE.activeSubscription,
          _count: {
            contacts_unlocked: MOCK_STATE.unlockedContacts.length,
            favorites: MOCK_STATE.favorites.length,
          },
        },
      },
    };
  },

  onboardBuyer: async ({ name, city, city_lat, city_lng }) => {
    await delay();
    MOCK_STATE.currentUser = {
      ...MOCK_STATE.currentUser,
      name, city, city_lat, city_lng,
      role: "buyer",
      is_onboarded: true,
    };
    return { success: true, message: "Profile completed successfully", data: { user: MOCK_STATE.currentUser } };
  },

  updateUserProfile: async (data) => {
    await delay();
    MOCK_STATE.currentUser = { ...MOCK_STATE.currentUser, ...data };
    return { success: true, message: "User profile updated successfully", data: { user: MOCK_STATE.currentUser } };
  },

  // ═══════════════════════════════════════════════════════
  // BREEDER ONBOARDING & PROFILE
  // ═══════════════════════════════════════════════════════

  onboardBreeder: async ({ name, city, city_lat, city_lng, selfie_url, id_doc_type, id_doc_url, has_pet_shop, pet_documents }) => {
    await delay(1000);
    MOCK_STATE.currentUser = { ...MOCK_STATE.currentUser, name, city, role: "breeder", is_onboarded: true };
    MOCK_STATE.currentBreeder = {
      ...MOCK_STATE.currentBreeder,
      name, city, city_lat, city_lng, selfie_url,
      id_doc_type, id_doc_url, has_pet_shop,
      status: "pending", is_approved: false, verified: false,
    };
    const docs = [
      { id: "doc-new-1", doc_type: id_doc_type || "aadhaar", doc_url: id_doc_url, status: "pending" },
    ];
    if (selfie_url) docs.push({ id: "doc-new-2", doc_type: "selfie", doc_url: selfie_url, status: "pending" });
    if (pet_documents) {
      pet_documents.forEach((d, i) => {
        docs.push({ id: `doc-new-${3 + i}`, doc_type: d.doc_type || "pet_shop_license", doc_url: d.doc_url, status: "pending" });
      });
    }
    MOCK_STATE.currentBreeder.documents = docs;
    return {
      success: true,
      message: "Breeder onboarding completed and verification request submitted",
      data: {
        user: MOCK_STATE.currentUser,
        breeder: MOCK_STATE.currentBreeder,
        verificationRequest: { id: "vr-mock-" + Date.now(), status: "pending" },
        documents: docs,
      },
    };
  },

  getBreederProfile: async () => {
    await delay(300);
    return {
      success: true,
      message: "Breeder profile fetched successfully",
      data: { breeder: { ...MOCK_STATE.currentBreeder, _count: { listings: MOCK_STATE.listings.filter(l => l.breeder_id === MOCK_STATE.currentBreeder.id).length, reviews: (MOCK_STATE.reviews[MOCK_STATE.currentBreeder.id] || []).length } } },
    };
  },

  getBreederStatus: async () => {
    await delay(300);
    return {
      success: true,
      message: "Breeder status fetched successfully",
      data: {
        status: MOCK_STATE.currentBreeder.status,
        is_approved: MOCK_STATE.currentBreeder.is_approved,
        verified: MOCK_STATE.currentBreeder.verified,
        rejection_reason: MOCK_STATE.currentBreeder.verificationRequest?.rejection_reason || null,
        admin_notes: MOCK_STATE.currentBreeder.verificationRequest?.admin_notes || null,
        breeder: MOCK_STATE.currentBreeder,
      },
    };
  },

  getPublicBreeder: async (breederId) => {
    await delay(300);
    const listing = MOCK_STATE.listings.find(l => l.breeder_id === breederId);
    const breederInfo = listing?.breeder || MOCK_STATE.currentBreeder;
    return {
      success: true,
      data: {
        breeder: {
          ...breederInfo,
          listings: MOCK_STATE.listings.filter(l => l.breeder_id === breederId),
          _count: { listings: MOCK_STATE.listings.filter(l => l.breeder_id === breederId).length, reviews: (MOCK_STATE.reviews[breederId] || []).length },
        },
      },
    };
  },

  updateBreederProfile: async (data) => {
    await delay();
    MOCK_STATE.currentBreeder = { ...MOCK_STATE.currentBreeder, ...data };
    return { success: true, message: "Breeder profile updated successfully", data: { breeder: MOCK_STATE.currentBreeder } };
  },

  // ═══════════════════════════════════════════════════════
  // LISTINGS
  // ═══════════════════════════════════════════════════════

  getListings: async (filters = {}) => {
    await delay(400);
    let result = [...MOCK_STATE.listings];
    if (filters.type) result = result.filter(l => l.type === filters.type);
    if (filters.breed) result = result.filter(l => l.breed.toLowerCase().includes(filters.breed.toLowerCase()));
    if (filters.city) result = result.filter(l => l.breeder?.city?.toLowerCase().includes(filters.city.toLowerCase()));
    if (filters.price_min) result = result.filter(l => l.price >= Number(filters.price_min));
    if (filters.price_max) result = result.filter(l => l.price <= Number(filters.price_max));
    if (filters.status) result = result.filter(l => l.status === filters.status);
    return { success: true, data: { listings: result } };
  },

  getListingDetail: async (id) => {
    await delay(300);
    const listing = MOCK_STATE.listings.find(l => l.id === id);
    if (!listing) throw new Error("Listing not found");
    return { success: true, data: { listing } };
  },

  createListing: async (data) => {
    await delay(800);
    const newListing = {
      id: "l-" + Date.now(),
      breeder_id: MOCK_STATE.currentBreeder.id,
      ...data,
      status: "available",
      is_new: true,
      created_at: new Date().toISOString(),
      breeder: {
        id: MOCK_STATE.currentBreeder.id,
        name: MOCK_STATE.currentBreeder.name,
        is_approved: MOCK_STATE.currentBreeder.is_approved,
        verified: MOCK_STATE.currentBreeder.verified,
        avg_rating: MOCK_STATE.currentBreeder.avg_rating,
        review_count: MOCK_STATE.currentBreeder.review_count,
        selfie_url: MOCK_STATE.currentBreeder.selfie_url,
        city: MOCK_STATE.currentBreeder.city,
      },
    };
    MOCK_STATE.listings.unshift(newListing);
    return { success: true, message: "Listing created successfully", data: { listing: newListing } };
  },

  updateListing: async (id, data) => {
    await delay();
    const idx = MOCK_STATE.listings.findIndex(l => l.id === id);
    if (idx === -1) throw new Error("Listing not found");
    MOCK_STATE.listings[idx] = { ...MOCK_STATE.listings[idx], ...data };
    return { success: true, message: "Listing updated", data: { listing: MOCK_STATE.listings[idx] } };
  },

  deleteListing: async (id) => {
    await delay();
    MOCK_STATE.listings = MOCK_STATE.listings.filter(l => l.id !== id);
    return { success: true, message: "Listing deleted" };
  },

  // ═══════════════════════════════════════════════════════
  // FAVORITES
  // ═══════════════════════════════════════════════════════

  getFavorites: async () => {
    await delay(300);
    const favListings = MOCK_STATE.listings.filter(l => MOCK_STATE.favorites.includes(l.id));
    return { success: true, data: { favorites: favListings } };
  },

  toggleFavorite: async (listing_id) => {
    await delay(200);
    const idx = MOCK_STATE.favorites.indexOf(listing_id);
    if (idx !== -1) {
      MOCK_STATE.favorites.splice(idx, 1);
      return { success: true, message: "Removed from favorites", data: { isFavorited: false } };
    } else {
      MOCK_STATE.favorites.push(listing_id);
      return { success: true, message: "Added to favorites", data: { isFavorited: true } };
    }
  },

  isFavorited: (listing_id) => MOCK_STATE.favorites.includes(listing_id),

  // ═══════════════════════════════════════════════════════
  // CONTACT UNLOCK (Decision Matrix)
  // ═══════════════════════════════════════════════════════

  unlockContact: async (breeder_id, listing_id = null) => {
    await delay(500);

    // Case 1: Already Unlocked
    const existing = MOCK_STATE.unlockedContacts.find(c => c.breeder_id === breeder_id);
    if (existing) {
      return {
        success: true,
        message: "Contact details already unlocked",
        data: { unlocked: true, source: "already_unlocked", contact: existing, unlocked_at: existing.unlocked_at },
      };
    }

    // Case 2: Free Credit Available
    if (MOCK_STATE.currentUser.free_unlock_credits > 0) {
      MOCK_STATE.currentUser.free_unlock_credits -= 1;
      const contact = {
        breeder_id,
        breeder_name: MOCK_STATE.listings.find(l => l.breeder_id === breeder_id)?.breeder?.name || "Unknown Breeder",
        phone: "+919876543210",
        email: "breeder@getpet.app",
        city: "Mumbai",
        selfie_url: "https://picsum.photos/200/200?random=100",
        unlocked_at: new Date().toISOString(),
      };
      MOCK_STATE.unlockedContacts.push(contact);
      return {
        success: true,
        message: "Breeder contact unlocked using 1 free credit!",
        data: { unlocked: true, source: "free_credit", remaining_free_credits: MOCK_STATE.currentUser.free_unlock_credits, contact, unlocked_at: contact.unlocked_at },
      };
    }

    // Case 3: Subscription Credit Available
    if (MOCK_STATE.activeSubscription && MOCK_STATE.activeSubscription.remaining_unlocks > 0) {
      MOCK_STATE.activeSubscription.remaining_unlocks -= 1;
      const contact = {
        breeder_id,
        breeder_name: MOCK_STATE.listings.find(l => l.breeder_id === breeder_id)?.breeder?.name || "Unknown Breeder",
        phone: "+919876543210",
        email: "breeder@getpet.app",
        city: "Mumbai",
        selfie_url: "https://picsum.photos/200/200?random=101",
        unlocked_at: new Date().toISOString(),
      };
      MOCK_STATE.unlockedContacts.push(contact);
      return {
        success: true,
        message: "Breeder contact unlocked using subscription plan!",
        data: { unlocked: true, source: "subscription", remaining_subscription_unlocks: MOCK_STATE.activeSubscription.remaining_unlocks, contact, unlocked_at: contact.unlocked_at },
      };
    }

    // Case 4: Paywall
    return {
      success: false,
      message: "No contact unlock credits available. Please purchase a plan to unlock this contact.",
      data: { paywall_required: true, plans: MOCK_STATE.plans.filter(p => p.target_role === "buyer") },
    };
  },

  getUnlockedContacts: async () => {
    await delay(300);
    return { success: true, data: { unlockedContacts: MOCK_STATE.unlockedContacts } };
  },

  // ═══════════════════════════════════════════════════════
  // PLANS, PAYMENT & SUBSCRIPTION
  // ═══════════════════════════════════════════════════════

  getPlans: async (target_role = null) => {
    await delay(300);
    let plans = MOCK_STATE.plans;
    if (target_role) plans = plans.filter(p => p.target_role === target_role);
    return { success: true, data: { plans } };
  },

  createPaymentOrder: async (plan_id) => {
    await delay(500);
    const plan = MOCK_STATE.plans.find(p => p.id === plan_id);
    if (!plan) throw new Error("Plan not found");
    const orderId = "order_mock_" + Date.now();
    return {
      success: true,
      message: "Payment order created",
      data: {
        payment: { id: "pay-mock-" + Date.now(), status: "created", amount: plan.price, razorpay_order_id: orderId },
        razorpay_order_id: orderId,
        amount: plan.price * 100,
        currency: "INR",
        key_id: "rzp_test_mock",
      },
    };
  },

  verifyPayment: async ({ razorpay_order_id, razorpay_payment_id, razorpay_signature }) => {
    await delay(800);
    const plan = MOCK_STATE.plans[0]; // Default to first plan for mock
    const now = new Date();
    const endDate = new Date(now.getTime() + plan.validity_days * 24 * 60 * 60 * 1000);
    const sub = {
      id: "sub-mock-" + Date.now(),
      status: "active",
      remaining_unlocks: plan.contact_unlocks,
      start_date: now.toISOString(),
      end_date: endDate.toISOString(),
      plan: plan,
    };
    MOCK_STATE.activeSubscription = sub;
    const payment = { id: "pay-mock-" + Date.now(), status: "success", amount: plan.price, paid_at: now.toISOString() };
    MOCK_STATE.payments.push(payment);
    return {
      success: true,
      message: "Payment verified and subscription activated!",
      data: { payment, subscription: sub },
    };
  },

  getActiveSubscription: async () => {
    await delay(300);
    return {
      success: true,
      data: {
        activeSubscription: MOCK_STATE.activeSubscription,
        hasActiveSubscription: !!(MOCK_STATE.activeSubscription && MOCK_STATE.activeSubscription.remaining_unlocks > 0),
      },
    };
  },

  getMySubscriptions: async () => {
    await delay(300);
    return { success: true, data: { subscriptions: MOCK_STATE.activeSubscription ? [MOCK_STATE.activeSubscription] : [] } };
  },

  getPaymentHistory: async () => {
    await delay(300);
    return { success: true, data: { payments: MOCK_STATE.payments } };
  },

  // ═══════════════════════════════════════════════════════
  // MESSAGES
  // ═══════════════════════════════════════════════════════

  getConversations: async () => {
    await delay(300);
    return { success: true, data: { conversations: MOCK_STATE.conversations } };
  },

  getMessages: async (recipientId) => {
    await delay(300);
    return { success: true, data: { messages: MOCK_STATE.messages[recipientId] || [] } };
  },

  sendMessage: async (recipientId, body, listingId = null) => {
    await delay(300);
    const msg = {
      id: "m-" + Date.now(),
      sender_id: MOCK_STATE.currentUser.id,
      recipient_id: recipientId,
      listing_id: listingId,
      body,
      created_at: new Date().toISOString(),
      read_at: null,
    };
    if (!MOCK_STATE.messages[recipientId]) MOCK_STATE.messages[recipientId] = [];
    MOCK_STATE.messages[recipientId].push(msg);
    return { success: true, data: { message: msg } };
  },

  // ═══════════════════════════════════════════════════════
  // REVIEWS
  // ═══════════════════════════════════════════════════════

  getReviews: async (breederId) => {
    await delay(300);
    return { success: true, data: { reviews: MOCK_STATE.reviews[breederId] || [] } };
  },

  addReview: async (breederId, rating, comment) => {
    await delay();
    const review = {
      id: "r-" + Date.now(),
      buyer_id: MOCK_STATE.currentUser.id,
      rating,
      comment,
      created_at: new Date().toISOString(),
      buyer_name: MOCK_STATE.currentUser.name,
    };
    if (!MOCK_STATE.reviews[breederId]) MOCK_STATE.reviews[breederId] = [];
    MOCK_STATE.reviews[breederId].push(review);
    return { success: true, message: "Review submitted", data: { review } };
  },

  // ═══════════════════════════════════════════════════════
  // REPORTS
  // ═══════════════════════════════════════════════════════

  submitReport: async ({ listing_id, breeder_id, reason, details }) => {
    await delay();
    MOCK_STATE.reports.push({ id: "rpt-" + Date.now(), listing_id, breeder_id, reason, details, status: "pending" });
    return { success: true, message: "Report submitted for review" };
  },

  // ═══════════════════════════════════════════════════════
  // PUSH TOKENS
  // ═══════════════════════════════════════════════════════

  registerPushToken: async (token, platform) => {
    await delay(200);
    MOCK_STATE.pushTokenRegistered = true;
    return { success: true, message: "Push token registered" };
  },

  // ═══════════════════════════════════════════════════════
  // MEDIA UPLOAD
  // ═══════════════════════════════════════════════════════

  uploadMedia: async (fileUri) => {
    await delay(1500);
    return {
      success: true,
      data: { url: `https://picsum.photos/600/400?random=${Date.now()}` },
    };
  },
};
```

---

## 5. API Service Layer

**File: `src/lib/api.js`**

```javascript
import AsyncStorage from "@react-native-async-storage/async-storage";
import { CONFIG } from "./config";
import { mockApi } from "./mockApi";

async function getToken() {
  return await AsyncStorage.getItem("userToken");
}

async function request(endpoint, method = "GET", body = null) {
  const token = await getToken();
  const headers = { "Content-Type": "application/json" };
  if (token) headers["Authorization"] = `Bearer ${token}`;

  const res = await fetch(`${CONFIG.API_BASE_URL}${endpoint}`, {
    method,
    headers,
    body: body ? JSON.stringify(body) : null,
  });
  const data = await res.json();
  if (!res.ok && res.status !== 402) throw new Error(data.message || "Request failed");
  return data;
}

// Exported API — automatically switches between mock and live
export const api = {
  // Auth
  sendOtp: (phone) =>
    CONFIG.USE_MOCK_API ? mockApi.sendOtp(phone) : request("/auth/send-otp", "POST", { phone }),
  verifyOtp: (phone, otp) =>
    CONFIG.USE_MOCK_API ? mockApi.verifyOtp(phone, otp) : request("/auth/verify-otp", "POST", { phone, otp }),

  // User
  getMe: () =>
    CONFIG.USE_MOCK_API ? mockApi.getMe() : request("/user/me"),
  onboardBuyer: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.onboardBuyer(data) : request("/user/onboard/buyer", "POST", data),
  updateUserProfile: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.updateUserProfile(data) : request("/user/profile", "PUT", data),
  unlockContact: (breeder_id, listing_id) =>
    CONFIG.USE_MOCK_API ? mockApi.unlockContact(breeder_id, listing_id) : request("/user/unlock-contact", "POST", { breeder_id, listing_id }),
  getUnlockedContacts: () =>
    CONFIG.USE_MOCK_API ? mockApi.getUnlockedContacts() : request("/user/unlocked-contacts"),

  // Breeder
  onboardBreeder: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.onboardBreeder(data) : request("/breeder/onboard", "POST", data),
  getBreederProfile: () =>
    CONFIG.USE_MOCK_API ? mockApi.getBreederProfile() : request("/breeder/me"),
  updateBreederProfile: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.updateBreederProfile(data) : request("/breeder/me", "PUT", data),
  getBreederStatus: () =>
    CONFIG.USE_MOCK_API ? mockApi.getBreederStatus() : request("/breeder/status"),
  getPublicBreeder: (id) =>
    CONFIG.USE_MOCK_API ? mockApi.getPublicBreeder(id) : request(`/breeder/public/${id}`),

  // Listings
  getListings: (filters) =>
    CONFIG.USE_MOCK_API ? mockApi.getListings(filters) : request(`/listings?${new URLSearchParams(filters)}`),
  getListingDetail: (id) =>
    CONFIG.USE_MOCK_API ? mockApi.getListingDetail(id) : request(`/listings/${id}`),
  createListing: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.createListing(data) : request("/listings", "POST", data),
  updateListing: (id, data) =>
    CONFIG.USE_MOCK_API ? mockApi.updateListing(id, data) : request(`/listings/${id}`, "PUT", data),
  deleteListing: (id) =>
    CONFIG.USE_MOCK_API ? mockApi.deleteListing(id) : request(`/listings/${id}`, "DELETE"),

  // Favorites
  getFavorites: () =>
    CONFIG.USE_MOCK_API ? mockApi.getFavorites() : request("/favorites"),
  toggleFavorite: (listing_id) =>
    CONFIG.USE_MOCK_API ? mockApi.toggleFavorite(listing_id) : request("/favorites/toggle", "POST", { listing_id }),

  // Plans & Payment
  getPlans: (target_role) =>
    CONFIG.USE_MOCK_API ? mockApi.getPlans(target_role) : request(`/payment/plans${target_role ? `?target_role=${target_role}` : ""}`),
  createPaymentOrder: (plan_id) =>
    CONFIG.USE_MOCK_API ? mockApi.createPaymentOrder(plan_id) : request("/payment/create-order", "POST", { plan_id }),
  verifyPayment: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.verifyPayment(data) : request("/payment/verify", "POST", data),
  getPaymentHistory: () =>
    CONFIG.USE_MOCK_API ? mockApi.getPaymentHistory() : request("/payment/history"),

  // Subscription
  getActiveSubscription: () =>
    CONFIG.USE_MOCK_API ? mockApi.getActiveSubscription() : request("/subscription/active"),
  getMySubscriptions: () =>
    CONFIG.USE_MOCK_API ? mockApi.getMySubscriptions() : request("/subscription/me"),

  // Messages
  getConversations: () =>
    CONFIG.USE_MOCK_API ? mockApi.getConversations() : request("/messages/conversations"),
  getMessages: (recipientId) =>
    CONFIG.USE_MOCK_API ? mockApi.getMessages(recipientId) : request(`/messages/${recipientId}`),
  sendMessage: (recipientId, body, listingId) =>
    CONFIG.USE_MOCK_API ? mockApi.sendMessage(recipientId, body, listingId) : request("/messages", "POST", { recipient_id: recipientId, body, listing_id: listingId }),

  // Reviews
  getReviews: (breederId) =>
    CONFIG.USE_MOCK_API ? mockApi.getReviews(breederId) : request(`/reviews/${breederId}`),
  addReview: (breederId, rating, comment) =>
    CONFIG.USE_MOCK_API ? mockApi.addReview(breederId, rating, comment) : request("/reviews", "POST", { breeder_id: breederId, rating, comment }),

  // Reports
  submitReport: (data) =>
    CONFIG.USE_MOCK_API ? mockApi.submitReport(data) : request("/reports", "POST", data),

  // Push Tokens
  registerPushToken: (token, platform) =>
    CONFIG.USE_MOCK_API ? mockApi.registerPushToken(token, platform) : request("/push-tokens", "POST", { token, platform }),

  // Media
  uploadMedia: (fileUri) =>
    CONFIG.USE_MOCK_API ? mockApi.uploadMedia(fileUri) : request("/media/upload", "POST", { file: fileUri }),
};
```

---

## 6. AUTH SCREENS

### PhoneAuthScreen

| Property | Value |
|---|---|
| **Route** | `AuthStack > PhoneAuthScreen` |
| **Purpose** | User enters 10-digit mobile number to login/register |
| **UI Elements** | App logo, Phone input (numeric keyboard), Country code label (+91), "Send OTP" button, Loading spinner |
| **Validation** | Must be exactly 10 digits, strip spaces/dashes |
| **API Call** | `api.sendOtp("+91" + phone)` |
| **On Success** | Navigate to `OTPVerifyScreen` passing `{ phone }` |
| **On Error** | Show error toast |

### OTPVerifyScreen

| Property | Value |
|---|---|
| **Route** | `AuthStack > OTPVerifyScreen` |
| **Purpose** | Verify 4-digit OTP code |
| **UI Elements** | 4-cell OTP input, 60s resend countdown timer, "Verify" button |
| **API Call** | `api.verifyOtp(phone, otp)` |
| **On Success** | Store `token` and `user` in AsyncStorage, then navigate based on: |
| | `needsOnboarding === true` → `OnboardingScreen` |
| | `user.role === "buyer"` → `BuyerTabNavigator` |
| | `user.role === "breeder"` → `BreederTabNavigator` |

---

## 7. ONBOARDING SCREENS

### OnboardingScreen

| Property | Value |
|---|---|
| **Route** | `OnboardingStack > OnboardingScreen` |
| **Purpose** | New user selects Buyer or Breeder role and completes profile |

**Step 1 — Role Selection:**
- Two large card buttons: 🐶 "I want to Buy a Pet" and 🐾 "I am a Pet Breeder".

**Step 2A — Buyer Form:**

| Field | Type | Required |
|---|---|---|
| Full Name | TextInput | ✅ (min 2 chars) |
| City | TextInput + location picker | ✅ |
| city_lat, city_lng | Auto from location picker | Optional |

- **API Call**: `api.onboardBuyer({ name, city, city_lat, city_lng })`
- **On Success**: Navigate to `BuyerTabNavigator`

**Step 2B — Breeder KYS Form (Multi-Step):**

| Step | Fields | Notes |
|---|---|---|
| 1. Store Info | Store Name, City, has_pet_shop toggle | |
| 2. Identity Doc | id_doc_type picker (Aadhaar/PAN), Camera/Gallery upload for doc photo | Upload via `api.uploadMedia()` → get URL |
| 3. Selfie | Camera capture for live selfie | Upload via `api.uploadMedia()` → get URL |
| 4. Pet Shop Docs | If has_pet_shop = true, upload pet_shop_license / kennel_club_cert | Each uploads via `api.uploadMedia()` |

- **API Call**: `api.onboardBreeder({ name, city, selfie_url, id_doc_type, id_doc_url, has_pet_shop, pet_documents: [{ doc_type, doc_url }] })`
- **On Success**: Navigate to `BreederPendingScreen`

---

## 8. BUYER SCREENS

### HomeScreen

| Property | Value |
|---|---|
| **Tab** | 🏠 Home |
| **UI Elements** | Location header ("Mumbai 📍"), Search bar, Category pills (Dog, Cat, Bird, Fish, All), Free credit banner if `free_unlock_credits > 0`, Pull-to-refresh FlatList of listing cards |
| **Each Listing Card** | Thumbnail photo, Pet name & breed, Price (₹), Verified badge if breeder is verified, Favorite heart toggle, City label |
| **API Calls** | `api.getMe()` (for credit count), `api.getListings({ type, city })` |
| **Tap Card** | Navigate to `ListingDetail` passing `{ listingId }` |
| **Tap Heart** | `api.toggleFavorite(listing_id)` |

### SearchScreen

| Property | Value |
|---|---|
| **Tab** | 🔍 Search |
| **UI Elements** | Search bar (by breed name), Filter chips: Type (Dog/Cat/Bird), Price range slider, City input, Sort by (Price Low-High, Newest), Results FlatList |
| **API Call** | `api.getListings({ type, breed, city, price_min, price_max })` |

### SavedScreen

| Property | Value |
|---|---|
| **Tab** | ❤️ Saved |
| **UI Elements** | FlatList of favorited listing cards, Empty state illustration if no favorites |
| **API Call** | `api.getFavorites()` |
| **Remove Favorite** | Swipe left or tap heart → `api.toggleFavorite(listing_id)` |

### MessagesScreen

| Property | Value |
|---|---|
| **Tab** | 💬 Messages |
| **UI Elements** | List of conversation threads (avatar, name, last message, timestamp, unread badge) |
| **API Call** | `api.getConversations()` |
| **Tap Conversation** | Navigate to `ChatScreen` passing `{ recipientId, recipientName }` |

### ProfileScreen

| Property | Value |
|---|---|
| **Tab** | 👤 Profile |
| **UI Elements** | Name, Phone, City, Role badge, Free credits remaining, Active subscription info card (plan name, remaining unlocks, expiry), Menu items: Edit Profile, My Unlocked Contacts, Payment History, Subscription Plans, Legal, Logout |
| **API Call** | `api.getMe()` |
| **Edit Profile** | Navigate to `EditProfileScreen` |
| **My Unlocked Contacts** | Navigate to `UnlockedContactsScreen` |
| **Subscription Plans** | Navigate to `SubscriptionScreen` |
| **Payment History** | Navigate to `PaymentHistoryScreen` |
| **Logout** | Clear AsyncStorage, navigate to `AuthStack` |

---

## 9. BREEDER SCREENS

### BreederHomeScreen

| Property | Value |
|---|---|
| **Tab** | 📊 Dashboard |
| **UI Elements** | Verification status banner (color-coded), Stats row (Active Listings, Total Unlocks/Leads, Average Rating), My Listings section (FlatList of own listings with status toggle) |
| **Status Banner Logic** | |
| | `status === "pending"` → 🟡 Yellow: "Your verification documents are being reviewed" |
| | `status === "rejected"` → 🔴 Red: "Verification rejected: {rejection_reason}. Tap to re-submit" (navigate to `GetVerifiedScreen`) |
| | `status === "approved"` → 🟢 Green: "✅ Verified Breeder" |
| **API Calls** | `api.getBreederStatus()`, `api.getListings({ breeder_id: currentBreeder.id })` |
| **Tap Listing** | Open edit modal or navigate to `ListingDetail` |
| **Change Status** | `api.updateListing(id, { status: "sold" })` |

### AddListingScreen

| Property | Value |
|---|---|
| **Tab** | ➕ Add Listing |
| **UI Elements** | Photo picker (up to 5 photos), Form fields table below |

| Field | Type | Required | Notes |
|---|---|---|---|
| Pet Name | TextInput | ✅ | e.g. "Rocky" |
| Breed | TextInput | ✅ | e.g. "Golden Retriever" |
| Type | Picker | ✅ | dog / cat / bird / fish / other |
| Gender | Picker | Optional | Male / Female |
| Price (₹) | NumericInput | ✅ | |
| Date of Birth | DatePicker | Optional | |
| Description | TextArea | Optional | |
| Health Certificate | Camera/Gallery | Optional | Upload via `api.uploadMedia()` |
| Video URL | TextInput | Optional | YouTube/direct link |

- **Photo Upload**: Each photo → `api.uploadMedia(fileUri)` → returns `url` string. Collect all URLs into `photos[]` array.
- **Submit**: `api.createListing({ name, breed, type, gender, price, dob, description, photos, health_cert_url, video_url })`
- **On Success**: Show success toast, navigate to `BreederHomeScreen`

### BreederProfile

| Property | Value |
|---|---|
| **Tab** | 👤 Profile |
| **UI Elements** | Store name, Verified badge, Selfie/avatar, City, Member since, Average rating, Review count, Menu items: Edit Store Profile, Get Verified / Re-submit Docs, My Subscription, Payment History, Legal, Logout |
| **API Call** | `api.getBreederProfile()` |

---

## 10. SHARED SCREENS & MODALS

### ListingDetail

| Property | Value |
|---|---|
| **UI Elements** | Swipeable photo carousel, Pet name, breed, type, gender, age (calculated from dob), Price, Description, Health cert download button, Video play button, Breeder card (avatar, name, verified badge, rating, city → tap to open `BreederPublicProfile`), Report flag icon, **Bottom CTA: "🔓 Unlock Breeder Contact"** |
| **API Call** | `api.getListingDetail(listingId)` |
| **Tap Unlock** | `api.unlockContact(breeder_id, listing_id)` |
| | If response `data.unlocked === true` → Navigate to `ContactRevealed` with contact data |
| | If response `data.paywall_required === true` → Navigate to `SubscriptionScreen` passing plans |
| **Tap Report** | Open `ReportModal` |

### ContactRevealed

| Property | Value |
|---|---|
| **UI Elements** | Success animation (confetti/checkmark), Breeder selfie, Breeder store name, Verified badge, Phone number (large, tappable), Email, City, Unlock source label ("Free Credit" / "Subscription"), Three action buttons |
| **Action Buttons** | |
| | 📞 **Call Now** → `Linking.openURL("tel:+919876543210")` |
| | 💬 **WhatsApp** → `Linking.openURL("whatsapp://send?phone=919876543210")` |
| | ✉️ **In-App Chat** → Navigate to `ChatScreen` |

### SubscriptionScreen (Paywall)

| Property | Value |
|---|---|
| **UI Elements** | Plan cards (name, price, unlock count, validity, features list), "Proceed to Pay ₹X" button, Current subscription info if active |
| **API Calls** | `api.getPlans("buyer")` or `api.getPlans("breeder")` based on role |
| **Payment Flow** | |
| | 1. User selects plan → `api.createPaymentOrder(plan_id)` |
| | 2. Open Razorpay SDK: `RazorpayCheckout.open({ key: key_id, amount, order_id, ... })` |
| | 3. On Razorpay success callback → `api.verifyPayment({ razorpay_order_id, razorpay_payment_id, razorpay_signature })` |
| | 4. On verify success → Show "🎉 Subscription Activated!" toast, pop back or retry unlock |
| **Mock Mode** | Skip Razorpay SDK, directly call `api.verifyPayment(mockData)` |

### GetVerifiedScreen

| Property | Value |
|---|---|
| **UI Elements** | Document upload form (same as breeder onboarding step 2-4), Status of current documents, Re-upload buttons for rejected docs |
| **API Call** | `api.onboardBreeder(updatedData)` (re-submit) |

### BreederPendingScreen

| Property | Value |
|---|---|
| **UI Elements** | Illustration (clock/hourglass), "Your verification is being reviewed by our team", "This usually takes 24-48 hours", Submitted document thumbnails, Refresh button |
| **API Call** | `api.getBreederStatus()` to poll |

### BreederPublicProfile

| Property | Value |
|---|---|
| **UI Elements** | Breeder avatar, name, verified badge, city, member since, avg_rating, review_count, Active listings grid, Reviews list, "Unlock Contact" button |
| **API Calls** | `api.getPublicBreeder(breederId)`, `api.getReviews(breederId)` |

### ChatScreen

| Property | Value |
|---|---|
| **UI Elements** | Header (recipient name, avatar), Message bubbles (sender right, recipient left), Text input + Send button, Timestamp labels |
| **API Calls** | `api.getMessages(recipientId)`, `api.sendMessage(recipientId, body)` |
| **Poll/Refresh** | Pull-to-refresh or periodic poll every 10s |

### UnlockedContactsScreen

| Property | Value |
|---|---|
| **UI Elements** | FlatList of unlocked breeder cards (name, phone, city, unlock date, source badge), Tap card → open `ContactRevealed` |
| **API Call** | `api.getUnlockedContacts()` |

### ReviewModal

| Property | Value |
|---|---|
| **UI Elements** | Star rating picker (1-5, half stars), Comment TextArea, Submit button |
| **Validation** | Rating required. One review per breeder per buyer (enforced by backend unique constraint). |
| **API Call** | `api.addReview(breederId, rating, comment)` |

### ReportModal

| Property | Value |
|---|---|
| **UI Elements** | Reason picker (Fake listing, Wrong price, Scam, Other), Details TextArea, Submit button |
| **API Call** | `api.submitReport({ listing_id, breeder_id, reason, details })` |

### EditProfileScreen

| Property | Value |
|---|---|
| **UI Elements** | Name input, Email input, City input with location picker |
| **API Call** | `api.updateUserProfile({ name, email, city, city_lat, city_lng })` |

### PaymentHistoryScreen

| Property | Value |
|---|---|
| **UI Elements** | FlatList of payment records (amount, plan name, status badge, date) |
| **API Call** | `api.getPaymentHistory()` |

---

## 11. Buyer Feature Summary

| Feature | Screen | API Endpoint | Mock Function |
|---|---|---|---|
| Phone Login | PhoneAuthScreen | `POST /auth/send-otp` | `mockApi.sendOtp` |
| OTP Verify | OTPVerifyScreen | `POST /auth/verify-otp` | `mockApi.verifyOtp` |
| Buyer Onboarding | OnboardingScreen | `POST /user/onboard/buyer` | `mockApi.onboardBuyer` |
| View Profile | ProfileScreen | `GET /user/me` | `mockApi.getMe` |
| Edit Profile | EditProfileScreen | `PUT /user/profile` | `mockApi.updateUserProfile` |
| Browse Listings | HomeScreen | `GET /listings` | `mockApi.getListings` |
| Search & Filter | SearchScreen | `GET /listings?type=&breed=` | `mockApi.getListings` |
| View Listing | ListingDetail | `GET /listings/:id` | `mockApi.getListingDetail` |
| Save Favorite | HomeScreen / ListingDetail | `POST /favorites/toggle` | `mockApi.toggleFavorite` |
| View Favorites | SavedScreen | `GET /favorites` | `mockApi.getFavorites` |
| Unlock Contact | ListingDetail | `POST /user/unlock-contact` | `mockApi.unlockContact` |
| View Unlocked | UnlockedContactsScreen | `GET /user/unlocked-contacts` | `mockApi.getUnlockedContacts` |
| Call / WhatsApp | ContactRevealed | Native Linking | N/A |
| View Plans | SubscriptionScreen | `GET /payment/plans` | `mockApi.getPlans` |
| Buy Plan | SubscriptionScreen | `POST /payment/create-order` | `mockApi.createPaymentOrder` |
| Verify Payment | SubscriptionScreen | `POST /payment/verify` | `mockApi.verifyPayment` |
| Payment History | PaymentHistoryScreen | `GET /payment/history` | `mockApi.getPaymentHistory` |
| Active Subscription | ProfileScreen | `GET /subscription/active` | `mockApi.getActiveSubscription` |
| Chat with Breeder | ChatScreen | `GET/POST /messages` | `mockApi.getMessages/sendMessage` |
| Write Review | ReviewModal | `POST /reviews` | `mockApi.addReview` |
| Report Listing | ReportModal | `POST /reports` | `mockApi.submitReport` |
| View Breeder Profile | BreederPublicProfile | `GET /breeder/public/:id` | `mockApi.getPublicBreeder` |

---

## 12. Breeder Feature Summary

| Feature | Screen | API Endpoint | Mock Function |
|---|---|---|---|
| Phone Login | PhoneAuthScreen | `POST /auth/send-otp` | `mockApi.sendOtp` |
| OTP Verify | OTPVerifyScreen | `POST /auth/verify-otp` | `mockApi.verifyOtp` |
| Breeder Onboarding | OnboardingScreen | `POST /breeder/onboard` | `mockApi.onboardBreeder` |
| Verification Status | BreederHomeScreen | `GET /breeder/status` | `mockApi.getBreederStatus` |
| Re-submit Docs | GetVerifiedScreen | `POST /breeder/onboard` | `mockApi.onboardBreeder` |
| Pending Status | BreederPendingScreen | `GET /breeder/status` | `mockApi.getBreederStatus` |
| View Store Profile | BreederProfile | `GET /breeder/me` | `mockApi.getBreederProfile` |
| Edit Store Profile | EditProfileScreen | `PUT /breeder/me` | `mockApi.updateBreederProfile` |
| Dashboard Stats | BreederHomeScreen | `GET /breeder/me` | `mockApi.getBreederProfile` |
| Add Listing | AddListingScreen | `POST /listings` | `mockApi.createListing` |
| Edit Listing | AddListingScreen | `PUT /listings/:id` | `mockApi.updateListing` |
| Delete Listing | BreederHomeScreen | `DELETE /listings/:id` | `mockApi.deleteListing` |
| Upload Photos/Docs | AddListingScreen / Onboarding | `POST /media/upload` | `mockApi.uploadMedia` |
| Chat with Buyer | ChatScreen | `GET/POST /messages` | `mockApi.getMessages/sendMessage` |
| View Reviews | BreederProfile | `GET /reviews/:breederId` | `mockApi.getReviews` |
| Buy Breeder Plan | SubscriptionScreen | `POST /payment/create-order` | `mockApi.createPaymentOrder` |

---

## 13. Implementation Checklist

- [ ] Set up React Navigation with conditional AuthStack / OnboardingStack / BuyerTabs / BreederTabs
- [ ] Create `src/lib/config.js`, `src/lib/mockApi.js`, `src/lib/api.js` from this guide
- [ ] Build all Auth screens (PhoneAuth + OTPVerify)
- [ ] Build OnboardingScreen with role selection and buyer/breeder forms
- [ ] Build all 5 Buyer tab screens (Home, Search, Saved, Messages, Profile)
- [ ] Build all 4 Breeder tab screens (Dashboard, AddListing, Messages, Profile)
- [ ] Build ListingDetail with Contact Unlock Decision Matrix (4 states)
- [ ] Build ContactRevealed with Call/WhatsApp/Chat buttons
- [ ] Build SubscriptionScreen with Razorpay integration (mock mode bypasses SDK)
- [ ] Build ChatScreen with message sending
- [ ] Build ReviewModal and ReportModal
- [ ] Build UnlockedContactsScreen and PaymentHistoryScreen
- [ ] Build BreederPublicProfile and BreederPendingScreen
- [ ] Register Expo push notification token on app startup
- [ ] Test all screens with `USE_MOCK_API = true`
- [ ] Switch to `USE_MOCK_API = false` and test with live backend
