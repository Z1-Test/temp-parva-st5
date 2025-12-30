# 📄 Feature Specification: Shopping Cart & Wishlist

---

## 0. Metadata

```yaml
feature_name: "Shopping Cart & Wishlist"
bounded_context: "catalog"
status: "draft"
owner: "Product + Engineering"
```

---

## 1. Overview

The **Shopping Cart & Wishlist** feature enables customers to add, update, and remove products from a persistent shopping cart and save products to a wishlist for future consideration. For authenticated users, cart and wishlist state persists across sessions and devices, creating a seamless shopping experience.

This feature bridges product discovery and checkout, enabling customers to curate their purchase intent and return to complete transactions later.

**Meaningful Change:**  
Users transition from ephemeral browsing sessions to persistent shopping intent, reducing friction and enabling thoughtful purchase decisions over time.

---

## 2. User Problem

**Who:** Beauty shoppers who want to save products for later purchase or maintain a running cart across browsing sessions.

**When:** During product discovery and purchase consideration, especially when shopping across multiple sessions or devices.

**Current Friction:**
- Cart contents are lost when users close the browser or switch devices
- No way to save products for future consideration (wishlist)
- Users must re-search and re-add products if they lose their cart
- Quantity adjustments are cumbersome or require removing and re-adding items
- No visibility into total cart value before checkout
- Anonymous users lose cart state, creating friction for casual browsers who later decide to purchase

**Why Existing Solutions Are Insufficient:**  
Many e-commerce platforms tie cart persistence to account creation, forcing users to register before they know if they want to purchase. This creates unnecessary friction and cart abandonment. Wishlist functionality is often buried or poorly integrated with the cart, making it difficult to move items between states.

---

## 3. Goals

### User Experience Goals

- **Persistent Cart State**: Cart contents persist across sessions, devices, and browser restarts for authenticated users
- **Flexible Cart Management**: Users can easily add, update quantities, and remove items without page reloads
- **Wishlist for Deferred Purchases**: Users can save products to wishlist for future consideration without cluttering the cart
- **Transparent Pricing**: Cart displays real-time subtotal and item count
- **Seamless Transitions**: Users can move items between cart and wishlist effortlessly
- **Anonymous Cart Preservation**: Anonymous users retain cart state via browser storage (with migration to account on login)

### Business / System Goals

- Reduce cart abandonment by preserving cart state across sessions (supports KPI-01: Conversion Rate target 5-6%)
- Increase repeat purchases by enabling wishlist reminders (supports KPI-02: Repeat Purchase Rate target 35%)
- Provide accurate cart data for checkout flow (dependency for Feature 4)
- Enable cart-based analytics (cart add rate, abandonment patterns)
- Support scalable cart storage (database for authenticated, localStorage for anonymous)

---

## 4. Non-Goals

- **Cart Sharing**: Sharing cart links with others deferred to Phase 2+
- **Save for Later (Separate from Wishlist)**: Separate "save for later" feature not implemented; wishlist serves this purpose
- **Cart Expiration**: Cart items do not expire or become unavailable (inventory checks happen at checkout)
- **Wishlist Notifications**: Email/push notifications for wishlist price drops or restocks deferred
- **Multiple Wishlists**: Users can only maintain one wishlist in MVP
- **Bulk Actions**: Bulk add/remove or "move all to cart" operations not included
- **Cart Recommendations**: Suggested products or "frequently bought together" deferred

---

## 5. Functional Scope

The Shopping Cart & Wishlist feature enables:

**Core Capabilities:**

**Cart:**
1. **Add to Cart**: Add products from catalog or product detail pages with quantity selection
2. **Update Quantity**: Increase or decrease item quantities directly in cart
3. **Remove from Cart**: Delete individual items from cart
4. **View Cart**: Display all cart items with product details (name, image, price, quantity)
5. **Cart Summary**: Show item count and subtotal (pre-tax, pre-shipping)
6. **Persistent Cart**: Preserve cart state for authenticated users across sessions and devices
7. **Anonymous Cart**: Preserve cart in browser localStorage for anonymous users
8. **Cart Migration**: Merge anonymous cart with account cart on login

**Wishlist:**
1. **Add to Wishlist**: Save products from catalog or product detail pages
2. **View Wishlist**: Display all saved products with details (name, image, price, stock status)
3. **Remove from Wishlist**: Delete individual items from wishlist
4. **Move to Cart**: Transfer wishlist items to cart with quantity selection
5. **Persistent Wishlist**: Preserve wishlist state for authenticated users

**System Responsibilities:**
- Store cart and wishlist data (database for authenticated, localStorage for anonymous)
- Emit `CartUpdated` and `WishlistUpdated` events for analytics
- Validate product availability and pricing at display time (fresh from catalog)
- Handle cart/wishlist conflicts on login (merge anonymous cart with account cart)
- Provide real-time cart updates (no page reload required)

---

## 6. Dependencies & Assumptions

**Dependencies:**
- **Product Catalog & Discovery (Feature 1)**: Cart/wishlist require product data (name, price, images, stock status)
- **Authentication & Account Management (Feature 2)**: Persistent cart/wishlist for authenticated users

**Assumptions:**
- Products in cart/wishlist remain available in the catalog (no automatic removal if product deleted)
- Cart subtotal does not include tax or shipping (calculated at checkout)
- Inventory checks happen at checkout, not at cart add (optimistic UX)
- Anonymous users accept browser localStorage limitations (data lost if browser data cleared)
- Most users will have <20 items in cart and <50 items in wishlist

**External Constraints:**
- Browser localStorage has ~5MB limit (sufficient for cart/wishlist JSON)
- Cart/wishlist data must sync across devices for authenticated users (requires backend storage)

---

## 7. User Stories & Experience Scenarios

### User Story 1 — Add Products to Cart

**As a** shopper browsing the catalog  
**I want** to add products to my cart with selected quantities  
**So that** I can collect items for purchase and proceed to checkout when ready

---

#### Scenario 1.1 — First-Time Add to Cart (Initial Experience)

**Given** a first-time user viewing a product detail page  
**When** the user selects a quantity (default: 1) and clicks "Add to Cart"  
**Then** the product is added to the cart successfully  
**And** a confirmation message displays: "Product added to cart!" (toast notification)  
**And** the cart icon in the header updates to show item count (e.g., "Cart (1)")  
**And** the user can continue browsing or click "View Cart" to review

---

#### Scenario 1.2 — Add Multiple Items to Cart (Returning Use)

**Given** a user already has items in their cart  
**When** the user adds another product  
**Then** the new product is appended to the cart  
**And** the cart count updates dynamically  
**And** the user can view all items in the cart summary

---

#### Scenario 1.3 — Add Same Product Twice (Interruption & State Management)

**Given** a user has already added "Rosehip Oil" to their cart (quantity: 1)  
**When** the user adds "Rosehip Oil" again from the product page  
**Then** the quantity in the cart increments (quantity: 2) rather than creating a duplicate entry  
**And** the user is informed: "Quantity updated in cart"

---

#### Scenario 1.4 — Add Out-of-Stock Product (Unexpected Outcome)

**Given** a product is currently out of stock  
**When** the user attempts to add it to cart  
**Then** the "Add to Cart" button is disabled or replaced with "Out of Stock"  
**And** the user can optionally "Add to Wishlist" instead  
**And** a message explains: "This product is currently unavailable. Add to wishlist to be reminded later."

---

#### Scenario 1.5 — Add to Cart Performance (High Load)

**Given** the cart service experiences high traffic  
**When** the user clicks "Add to Cart"  
**Then** the action completes within 500ms  
**And** a loading indicator briefly shows during the add operation  
**And** the cart count updates immediately upon success

---

#### Scenario 1.6 — Mobile Add to Cart (Context Sensitivity)

**Given** a user on a mobile device  
**When** the user taps "Add to Cart"  
**Then** a mobile-optimized toast notification confirms the add  
**And** the cart icon in the mobile header updates  
**And** the user can tap "View Cart" or continue browsing via sticky footer or toast action

---

### User Story 2 — Manage Cart Items (Update & Remove)

**As a** shopper with items in my cart  
**I want** to update quantities or remove items  
**So that** I can refine my purchase before checkout

---

#### Scenario 2.1 — Update Item Quantity (Initial)

**Given** a user views the cart page with an item (quantity: 1)  
**When** the user increases the quantity to 3 using the quantity selector  
**Then** the cart updates immediately (no page reload)  
**And** the subtotal recalculates to reflect the new quantity  
**And** the updated quantity persists in backend (authenticated) or localStorage (anonymous)

---

#### Scenario 2.2 — Remove Item from Cart (Returning Use)

**Given** a user wants to remove an item from the cart  
**When** the user clicks "Remove" next to the item  
**Then** the item is removed from the cart immediately  
**And** the cart count and subtotal update  
**And** a brief confirmation displays: "Item removed" with optional "Undo" action (if time permits)

---

#### Scenario 2.3 — Reduce Quantity to Zero (Edge Case)

**Given** a user reduces an item's quantity to 0 using the quantity selector  
**When** the quantity reaches 0  
**Then** the item is automatically removed from the cart  
**And** the behavior is the same as clicking "Remove"

---

#### Scenario 2.4 — Cart Update Failure (Unexpected Outcome)

**Given** a user updates item quantity or removes an item  
**When** the update request fails (network error, server issue)  
**Then** an error message displays: "We couldn't update your cart. Please try again."  
**And** the cart reverts to its previous state (optimistic UI rollback)  
**And** the user can retry the action

---

#### Scenario 2.5 — Cart Update Performance (Performance)

**Given** a user updates cart quantities multiple times rapidly  
**When** the user adjusts quantities  
**Then** the UI updates instantly (optimistic updates)  
**And** backend synchronization happens asynchronously (debounced)  
**And** the experience feels immediate and responsive

---

#### Scenario 2.6 — Mobile Cart Management (Context Sensitivity)

**Given** a user on a mobile device  
**When** the user opens the cart page  
**Then** items are displayed in a stacked list (mobile-optimized)  
**And** quantity selectors are touch-friendly (large tap targets)  
**And** "Remove" buttons are clearly labeled and easy to tap  
**And** the subtotal is visible at the top or sticky footer

---

### User Story 3 — Persistent Cart Across Sessions

**As a** registered customer  
**I want** my cart to persist across sessions and devices  
**So that** I can continue shopping seamlessly without losing my selections

---

#### Scenario 3.1 — Authenticated Cart Persistence (Initial)

**Given** an authenticated user adds items to cart  
**When** the user logs out or closes the browser  
**And** returns to the site and logs in again (same device or different device)  
**Then** the cart contents are restored exactly as left  
**And** all quantities and items are preserved

---

#### Scenario 3.2 — Anonymous Cart Migration on Login (Returning Use)

**Given** an anonymous user has items in cart (stored in localStorage)  
**When** the user registers or logs in  
**Then** the anonymous cart is merged with the account cart  
**And** if duplicate products exist, quantities are summed  
**And** the user sees all items in the unified cart

---

#### Scenario 3.3 — Cart Persistence Across Devices (Context Switching)

**Given** a user adds items to cart on desktop  
**When** the user opens the site on mobile (logged in with same account)  
**Then** the cart contents sync automatically  
**And** the user sees the same items and quantities on mobile  
**And** changes on mobile sync back to desktop

---

#### Scenario 3.4 — Cart Sync Conflict (Unexpected Outcome)

**Given** a user modifies cart on two devices simultaneously (rare edge case)  
**When** the updates conflict (e.g., different quantities for same item)  
**Then** the most recent update wins (last-write-wins strategy)  
**And** the user is optionally notified: "Your cart was updated from another device"

---

#### Scenario 3.5 — Cart Load Performance (Performance)

**Given** a returning user logs in  
**When** the cart loads from backend  
**Then** the cart data is fetched and displayed within 500ms  
**And** skeleton loaders indicate cart is loading  
**And** the page remains interactive during load

---

#### Scenario 3.6 — Anonymous Cart Storage Limits (Context Sensitivity)

**Given** an anonymous user adds many items to cart (approaching localStorage limit)  
**When** the cart exceeds safe storage size (~50 items)  
**Then** the user is prompted to create an account to save more items: "Your cart is getting full. Sign up to save unlimited items."  
**And** critical cart data is prioritized (oldest items may be dropped if limit exceeded)

---

### User Story 4 — Add Products to Wishlist

**As a** shopper browsing products  
**I want** to save products to a wishlist for future consideration  
**So that** I can track items I'm interested in without committing to purchase immediately

---

#### Scenario 4.1 — First-Time Add to Wishlist (Initial)

**Given** a logged-in user views a product detail page  
**When** the user clicks the "Add to Wishlist" button (heart icon)  
**Then** the product is added to the wishlist  
**And** the heart icon fills in (visual feedback)  
**And** a confirmation displays: "Added to wishlist!"  
**And** the user can view the wishlist from the account menu or header

---

#### Scenario 4.2 — Remove from Wishlist (Returning Use)

**Given** a user has items in their wishlist  
**When** the user clicks the filled heart icon on a product (or "Remove" in wishlist view)  
**Then** the product is removed from the wishlist  
**And** the heart icon empties (visual feedback)  
**And** a brief confirmation displays: "Removed from wishlist"

---

#### Scenario 4.3 — Move Wishlist Item to Cart (Interruption & Workflow)

**Given** a user views their wishlist  
**When** the user clicks "Add to Cart" next to a wishlist item  
**Then** the item is added to the cart (with quantity selector if available)  
**And** the item optionally remains in the wishlist (or is removed based on UX preference)  
**And** the user is confirmed: "Item moved to cart"

---

#### Scenario 4.4 — Wishlist Requires Authentication (Unexpected Outcome)

**Given** an anonymous user attempts to add a product to wishlist  
**When** the user clicks "Add to Wishlist"  
**Then** a prompt displays: "Please log in to save items to your wishlist"  
**And** a "Log In" or "Sign Up" button is provided  
**And** after login, the user is returned to the product page (optional: item auto-added to wishlist)

---

#### Scenario 4.5 — Wishlist Load Performance (Performance)

**Given** a user opens the wishlist page  
**When** the wishlist contains 20+ items  
**Then** all items load within 1 second  
**And** skeleton loaders indicate loading state  
**And** the page remains scrollable and responsive during load

---

#### Scenario 4.6 — Mobile Wishlist Management (Context Sensitivity)

**Given** a user on a mobile device  
**When** the user views the wishlist  
**Then** items are displayed in a mobile-optimized grid or list  
**And** "Add to Cart" and "Remove" buttons are touch-friendly  
**And** the user can easily navigate between wishlist and cart

---

## 8. Edge Cases & Constraints (Experience-Relevant)

### Hard Limits
- **Cart Size**: Maximum 100 items per cart (soft limit; prevents abuse and performance issues)
- **Wishlist Size**: Maximum 200 items per wishlist (soft limit)
- **Quantity Per Item**: Maximum 99 units per product (prevents bulk order abuse in MVP)
- **Anonymous Cart Expiration**: Anonymous carts persist until browser data is cleared (no server-side expiration)

### Irreversible Actions
- **Item Removal**: Removing items from cart/wishlist is permanent (no automatic recovery unless undo feature implemented)

### Compliance & Policy Constraints
- **Price Accuracy**: Cart displays current product prices (fetched from catalog); prices may change between adding to cart and checkout
- **Stock Availability**: Cart add is optimistic (no inventory check); inventory validated at checkout

### Experience Constraints
- **Anonymous Cart Limitations**: Anonymous users lose cart if they clear browser data or switch browsers/devices
- **Offline Cart**: Cart updates require network connectivity; offline cart updates not supported in MVP
- **Browser Compatibility**: localStorage required for anonymous cart; users with storage disabled cannot persist cart

---

## 9. Implementation Tasks (Execution Agent Checklist)

```markdown
- [ ] T01 [Scenario 1.1, 2.1] — Implement cart data model and CRUD APIs (add, update, remove items) with backend storage for authenticated users and localStorage fallback for anonymous
- [ ] T02 [Scenario 1.1, 1.2, 1.6] — Build "Add to Cart" UI with quantity selector, toast notifications, and dynamic cart count updates
- [ ] T03 [Scenario 2.1, 2.2, 2.6] — Build cart management UI (cart page with item list, quantity selectors, remove buttons, subtotal calculation)
- [ ] T04 [Scenario 3.1, 3.2] — Implement cart persistence logic (sync cart to backend for authenticated users, merge anonymous cart on login)
- [ ] T05 [Scenario 4.1, 4.2, 4.3] — Implement wishlist data model, APIs, and UI (add/remove from wishlist, wishlist page, move to cart)
- [ ] T06 [Scenario 1.4, 2.4, 4.4] — Implement error handling and edge cases (out of stock, update failures, anonymous wishlist restriction)
- [ ] T07 [Rollout] — Implement feature flag `feature.catalog.shopping_cart_wishlist` to gate cart and wishlist features
```

---

## 10. Acceptance Criteria (Verifiable Outcomes)

```markdown
- [ ] AC1 [Cart Add & Update] — User can add products to cart, update quantities, remove items, and see real-time cart count and subtotal updates
- [ ] AC2 [Cart Persistence] — Authenticated user's cart persists across sessions and devices; anonymous cart migrates to account on login
- [ ] AC3 [Wishlist Management] — Authenticated user can add products to wishlist, view wishlist, remove items, and move items to cart
- [ ] AC4 [Performance] — Cart add completes within 500ms, cart page loads within 1s, quantity updates feel instant (optimistic UI)
- [ ] AC5 [Error Handling] — Out-of-stock products show appropriate messaging, cart update failures are gracefully handled with retry options
- [ ] AC6 [Gating] — Feature flag `feature.catalog.shopping_cart_wishlist` correctly gates cart and wishlist UI; when disabled, users cannot add to cart/wishlist
```

---

## 11. Rollout & Risk

### Rollout Strategy

**Feature Flag:** `feature.catalog.shopping_cart_wishlist` (Temporary)

**Purpose:** Control progressive rollout of cart and wishlist features, allowing gradual exposure and monitoring for data sync issues or performance problems.

**Rollout Plan:**
- **Alpha (Week 1):** Internal team only (5-10 users) — Validate cart add/update/remove, wishlist management, persistence logic
  - **Success Gate:** Error rate <1%, cart persistence verified, anonymous cart migration successful
- **Beta (Weeks 2-3):** Limited external users (100-500) — Monitor cart sync across devices, localStorage behavior, cart abandonment patterns
  - **Success Gate:** Error rate <0.5%, cart sync success rate >99%, no data loss incidents
- **GA (Week 4+):** Full public launch
  - **Success Gate:** Error rate <0.3%, cart abandonment rate <70% (industry baseline), wishlist adoption >15% of users

**Removal Criteria:**  
Feature flag can be removed after >2 weeks in GA with:
- Error rate consistently <0.3%
- No cart data loss or sync issues
- Cart and wishlist performance stable under production load

**Rollback Threshold:**  
If error rate >1% OR cart data loss detected OR cart sync failure rate >5%, initiate immediate rollback and disable cart/wishlist features.

### Risk Mitigation

- **Cart Data Loss**: Implement robust error handling and logging for cart sync operations; monitor for data loss incidents
- **localStorage Limits**: Warn anonymous users approaching storage limits; prompt account creation
- **Cart/Wishlist Conflicts**: Use last-write-wins strategy with timestamps to resolve conflicts
- **Performance Degradation**: Monitor cart API latency; optimize database queries and add caching if needed

---

## 12. History & Status

- **Status:** Draft
- **Related PRD:** `docs/product/PRD.md` (v0.1)
- **Related Roadmap:** `docs/product/roadmap.md` (Feature 3)
- **Dependencies:**
  - Product Catalog & Discovery (Feature 1) — Required for product data
  - Authentication & Account Management (Feature 2) — Required for persistent cart/wishlist
- **Dependent Features:**
  - Checkout & Payment Processing (Feature 4) — Requires cart data for checkout
- **Related Epics:** TBD (post-approval)
- **Related Issues:** TBD (post-approval)

---

## Final Note

> This document defines **intent and experience**.  
> Execution details are derived from it — never the other way around.
