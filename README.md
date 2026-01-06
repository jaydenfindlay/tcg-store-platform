# TCG White‑Label Store Platform – Consolidated Plan

## 1. Purpose

This project is a **portfolio‑grade Django + React application** designed to demonstrate:

* Multi‑tenant (white‑label) architecture
* Clean backend design and API structure
* Third‑party data integration (JustTCG)
* Thoughtful UX around inventory management and pricing transparency

Secondary goal: validate interest from real TCG stores if the project is later pushed toward a business.

---

## 2. Problem Statement

Independent TCG stores struggle to provide pricing confidence to buyers.

* Generic storefronts lack market context
* Marketplaces provide pricing, but remove brand control

Buyers want to understand **market value and price history** before purchasing singles.

---

## 3. Vision

A **white‑label, TCG‑native storefront platform** where:

* Stores manage their own inventory and prices
* Buyers see clear market pricing context and historical charts
* The platform supports multiple TCGs over time

---

## 4. MVP Positioning

This MVP is:

* **Private / portfolio‑focused** (not public launch ready)
* **No checkout or payments**
* Focused on **Pokémon singles only**

---

## 5. MVP Feature Scope (Locked)

### Inventory Management (Core Feature)

* Add single cards manually via admin UI
* Bulk upload inventory via CSV
* CSV upload must include:

  * Downloadable template
  * Import preview (first N rows)
  * Row‑level validation with clear errors
  * “Needs review” state for ambiguous rows
  * Import summary (created / updated / failed)

### Storefront

* Home page

  * Popular products (engagement‑based)
  * Fallback to newly added products
* Browse

  * Filters: Set, Rarity
* Search

  * Search by card name, number, set
* Card detail page

  * Card image and metadata
  * **Store price** (transactional)
  * **Market price** (informational)
  * Price chart (7D / 30D / 90D)
  * Clear disclaimer that market price is informational

### Engagement Tracking (No Checkout)

Used for “popular products”:

* Product view events
* Optional: add‑to‑cart events
* Optional: watchlist/favourites

---

## 6. Technical Architecture

### Tenancy Model

* **Path‑based multi‑tenancy**
* URLs scoped as: `/s/<store_slug>/...`
* Store resolved via middleware and attached to `request.store`
* Designed so domain‑based routing can be added later without rewrites

### Backend Stack

* Django + Django REST Framework
* Modular monolith structure
* Server‑side JustTCG integration
* Redis (or similar) for caching

### Frontend Stack

* React SPA
* React Router with `storeSlug` as top‑level route param
* All API calls store‑scoped

---

## 7. Django Project Structure

```
backend/
  config/
    settings/
    urls.py
    middleware.py
  apps/
    stores/        # tenancy, branding
    catalog/       # global card & variant data
    inventory/     # store listings & pricing
    pricing/       # market prices & snapshots
    events/        # views / engagement
    imports/       # CSV import pipeline
  common/
    services/
      justtcg.py   # single wrapper for JustTCG API
    permissions.py
    utils/
```

## 8. React Project Structure (Suggested)

```
frontend/
  src/
    routes/
      StoreShell.tsx          # reads storeSlug, provides store context
      Home.tsx
      Browse.tsx
      Search.tsx
      ItemDetail.tsx
      admin/
        Inventory.tsx
        Import.tsx
    api/
      client.ts               # fetch/axios wrapper
      storeApi.ts             # store-scoped API functions
    components/
      CardGrid.tsx
      PriceChart.tsx
```

---

## 8. Data Model (High Level)

### Store

* slug
* name
* branding config

### CatalogCard (Global)

* justtcg_card_id
* name, set, number, rarity, image

### CatalogVariant (Global)

* justtcg_variant_id
* condition, printing, language

### InventoryItem (Store‑scoped)

* store
* catalog_variant
* quantity
* store_price
* status (draft / live)

### MarketPriceSnapshot

* catalog_variant
* date
* market_price
* source (JustTCG)

### ProductViewEvent

* store
* inventory_item
* timestamp

---

## 9. Pricing & Charting Strategy

* Use JustTCG for current market price
* Cache upstream API responses
* Snapshot market prices **daily** for in‑stock variants
* Charts read exclusively from local DB snapshots
* Clear UI separation:

  * Store price = what buyer pays
  * Market price = informational context

---

## 10. API Design Principles

* All store‑specific endpoints are store‑scoped
* No unscoped inventory queries
* Catalog data is global and reused

Example endpoints:

* `GET /api/stores/<slug>/browse`
* `GET /api/stores/<slug>/search`
* `GET /api/stores/<slug>/items/<id>`
* `GET /api/stores/<slug>/variants/<variant_id>/history`
* `POST /api/stores/<slug>/admin/imports`

---

## 11. MVP Constraints (Intentional)

* Pokémon only
* Singles only
* One currency
* No checkout
* No auto‑repricing
* No grading support

---

## 12. Why This Is a Strong Portfolio Project

* Demonstrates multi‑tenant architecture
* Shows real‑world data integration
* Solves a real business problem
* Clean separation of concerns
* Extensible into a SaaS if desired

---

## 13. Future (Post‑MVP, Not Implemented)

* Checkout & orders
* Domain‑based tenancy
* Multi‑TCG support
* Auto‑pricing suggestions
* Analytics dashboards
* Multi‑currency

---

This document represents the **locked plan** for the project. All implementation should adhere to MVP scope unless explicitly expanded.
