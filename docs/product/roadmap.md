# Feature Roadmap (Draft)

**Version:** 0.1 | **Status:** Draft | **Based on PRD v0.1**

---

## Core Features (MVP)

### 1. Product Catalog & Discovery

- **Description:** Browse, search, and filter products by category (skin care, hair care, cosmetics) with product detail pages showing descriptions, ingredients, images, and ethical badges.
- **Depends on:** None (foundational)
- **Bounded Contexts:** `catalog`

### 2. Authentication & Account Management

- **Description:** User registration, email/password login, and session management via Firebase Auth with user profile and saved preferences.
- **Depends on:** None (foundational)
- **Bounded Contexts:** `auth`

### 3. Shopping Cart & Wishlist

- **Description:** Add/update/remove items from a persistent shopping cart and save items to a wishlist for authenticated users.
- **Depends on:** Product Catalog & Discovery, Authentication & Account Management
- **Bounded Contexts:** `catalog`, `auth`

### 4. Checkout & Payment Processing

- **Description:** Multi-step checkout flow with address management, tax calculation, payment orchestration via Cashfree with automatic retry on failure and order confirmation.
- **Depends on:** Shopping Cart & Wishlist, Authentication & Account Management
- **Bounded Contexts:** `checkout`

### 5. Order Management & Fulfillment

- **Description:** Order lifecycle management (placed → processing → shipped) with real-time status tracking and integration with Shiprocket for shipment generation and carrier selection.
- **Depends on:** Checkout & Payment Processing
- **Bounded Contexts:** `orders`

### 6. Order History & Tracking

- **Description:** Display customer's order history with shipment details, tracking links, and real-time status updates from the fulfillment partner.
- **Depends on:** Order Management & Fulfillment
- **Bounded Contexts:** `orders`, `auth`

### 7. Admin Dashboard (Core Operations)

- **Description:** Product catalog management (CRUD), inventory visibility, order dashboard with status updates and filtering, and basic analytics.
- **Depends on:** Product Catalog & Discovery, Order Management & Fulfillment, Authentication & Account Management
- **Bounded Contexts:** `admin`, `catalog`, `orders`

---

## Dependencies Summary

```
Product Catalog & Discovery (foundational)
├── Shopping Cart & Wishlist
├── Checkout & Payment Processing
│   ├── Order Management & Fulfillment
│   │   ├── Order History & Tracking
│   │   └── Admin Dashboard (Core Operations)
└── Admin Dashboard (Core Operations)

Authentication & Account Management (foundational)
├── Shopping Cart & Wishlist
├── Checkout & Payment Processing
│   └── Order Management & Fulfillment
└── Admin Dashboard (Core Operations)
```

---

## Roadmap Timeline (High-Level, Not Scheduled)

**Phase 1 (Alpha):** Foundational features (Product Catalog, Auth) + minimal checkout flow  
**Phase 2 (Beta):** Full checkout + payment + order management + tracking  
**Phase 3 (GA):** Admin dashboard + full analytics integration + carrier fallback optimization  

*(Timeline and phase assignments are illustrative and will be finalized during planning.)*

---

## Feature-to-Bounded-Context Mapping

| Feature | Primary Context | Secondary Contexts |
|---------|-----------------|-------------------|
| Product Catalog & Discovery | catalog | — |
| Authentication & Account Management | auth | — |
| Shopping Cart & Wishlist | catalog | auth |
| Checkout & Payment Processing | checkout | auth |
| Order Management & Fulfillment | orders | — |
| Order History & Tracking | orders | auth |
| Admin Dashboard (Core Operations) | admin | catalog, orders, auth |

---

## Feature-to-KPI Alignment

| Feature | KPI(s) |
|---------|--------|
| Product Catalog & Discovery | KPI-01 (Conversion Rate) |
| Authentication & Account Management | KPI-02 (Repeat Purchase Rate) |
| Shopping Cart & Wishlist | KPI-01 (Conversion Rate), KPI-02 (Repeat Purchase Rate) |
| Checkout & Payment Processing | KPI-01 (Conversion Rate), KPI-03 (Checkout Success Rate) |
| Order Management & Fulfillment | Operational (time-to-fulfill, SLA compliance) |
| Order History & Tracking | KPI-02 (Repeat Purchase Rate) |
| Admin Dashboard (Core Operations) | Operational (time-to-fulfill, inventory accuracy) |

---

## Next Steps

1. Each feature will be decomposed into **Epic** + **User Stories** with acceptance criteria (Gherkin scenarios).
2. Each feature will have a **Feature Blueprint** under `docs/features/feat-{ID}-{name}.md`.
3. Feature flags will be assigned per feature using the naming convention: `feature.{domain}.{feature_name}`.
4. Implementation sequencing will be determined during technical planning (respecting dependencies).

---

## Notes

- Out of scope: Marketplace / third-party seller onboarding (deferred to Phase 2+).
- All features require feature flags for progressive rollout (per PRD).
- Event taxonomy and integration points will be defined in feature blueprints.
