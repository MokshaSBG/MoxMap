# MoxMaps

A decentralised, user‑owned map application built on the **AT Protocol**.  
Users can publish places, attach photos and reviews, and follow other users' contributions — all without relying on a single central server.

Think of it as a **social map** where your data lives in your own repository, portable across any compatible app.

---

## What This Project Does

- **Publish places** — Add locations with coordinates, names, categories, and descriptions.
- **Attach media** — Upload photos tied to places or reviews.
- **Write reviews** — Share opinions and ratings on places, with strong references to the original record.
- **Follow contributors** — See places and reviews from users you follow.
- **Portable identity** — Your places, reviews, and photos live in your AT Protocol repository, not ours.

---

## Architecture Overview

| Layer | Technology / Approach |
|-------|----------------------|
| **Protocol** | AT Protocol (ATProto) — decentralised identity, data repos, and lexicons |
| **Backend** | Go — handles indexing, federation via relays, and geospatial queries |
| **Frontend** | Map‑based UI (Leaflet or MapLibre) talking to backend via REST/GraphQL |
| **Database** | PostGIS or SQLite+SpatiaLite for local indexing and spatial queries |
| **Identity** | User‑owned DIDs and handles via ATProto PDS (Personal Data Server) |
| **Lexicons** | Custom + community lexicons for places, reviews, and media |

---

## Lexicon Design

AT Protocol uses **lexicons** to define record schemas. This project composes multiple lexicons to build a rich map experience.

### Community Lexicons Used

| Lexicon | Purpose |
|---------|---------|
| `community.lexicon.location.geo` | WGS84 coordinates (latitude, longitude, altitude, name) |
| `community.lexicon.location.address` | Structured postal addresses (used in real apps like DropAnchor) |
| `com.atproto.repo.strongRef` | Immutable reference to another record (URI + CID) |

### Custom Lexicons (Planned / In Development)

| Lexicon | Purpose |
|---------|---------|
| `app.yourname.place` | A place record — name, location ref, category, description |
| `app.yourname.review` | A review record — subject strongRef, rating, text, photos |
| `app.yourname.photo` | A photo record — blob ref, dimensions, alt text, MIME type |

### How Lexicons Compose
```
Place Record
├── name: string
├── location: ref → community.lexicon.location.geo
├── address: ref → community.lexicon.location.address (optional)
├── category: string
├── photos: array of ref → app.yourname.photo
└── createdAt: datetime
```
```
Review Record
├── subject: ref → com.atproto.repo.strongRef (points to a Place)
├── rating: integer (1–5) or adjective‑based system
├── text: string (optional)
├── photos: array of ref → app.yourname.photo
└── createdAt: datetime
```

The `strongRef` pattern lets reviews point to **any** record — places, other reviews, or external entities — while guaranteeing the referenced record hasn't changed since the review was written.

---

## Core Features

- Create, update, and delete places (stored in user's ATProto repo)
- Upload and attach photos to places and reviews
- Write reviews with ratings and text
- Follow other users and see their contributions
- Search places by name, category, or proximity
- Portable identity — move your data between compatible apps
- Instance‑level moderation and content filtering

---

## Getting Started (Conceptual Guide)

1. **Clone this repository** (or fork it).
2. **Set up a PDS** — either use a hosted provider or run your own ATProto Personal Data Server.
3. **Set up your database** — choose a spatially‑aware DB (PostGIS recommended).
4. **Configure environment variables** — PDS endpoint, database URL, relay URLs, etc.
5. **Run the backend** — the Go server indexes records, handles geospatial queries, and serves the API.
6. **Serve the frontend** — static files or a separate build process.
7. **Test federation** — publish a place from one account and verify it appears for followers.

> **No code is provided in this repository.**

---

## Expected Project Structure (Conceptual)
```
/
├── backend/
│ ├── lexicon/ # Lexicon definitions and validation
│ ├── indexer/ # Consumes ATProto firehose, indexes records
│ ├── geo/ # Spatial queries and indexing logic
│ ├── api/ # REST/GraphQL endpoints for frontend
│ └── main.go
├── frontend/
│ ├── index.html
│ ├── styles.css
│ └── app.js # Map initialisation, API calls, UI logic
├── lexicons/
│ ├── place.json # app.yourname.place
│ ├── review.json # app.yourname.review
│ └── photo.json # app.yourname.photo
├── docs/
│ └── federation.md # Explanation of lexicon composition and indexing
└── README.md # You are here
```

---

## Security & Moderation Notes

- All records are **signed** by the user's ATProto identity.
- Incoming records are **validated** against their lexicon schema before indexing.
- Place coordinates are clamped to valid ranges (`-90..90`, `-180..180`).
- Media uploads are stored as ATProto blobs with MIME type and size checks.
- Admins can filter content by domain, DID, or keyword via a deny list.

---

## The Temporal Challenge

AT Protocol records are **immutable and time‑ordered**. A review created today remains in the user's repo even if the place changes significantly later. This project addresses that by:

- Tracking the **CID** in strongRefs to detect when a place has been updated.
- Flagging stale reviews or prompting users to re‑review.
- Offering filters for "all reviews" vs. "reviews matching current place state."

---

## Dependencies (Conceptual, Not Versioned)

- Go 1.22+ (or newer)
- A spatially‑aware database (PostGIS, SQLite+SpatiaLite)
- An ATProto PDS (self‑hosted or hosted provider)
- A map tile provider (OSM, Carto, or self‑hosted tiles)
- A relay or firehose consumer for indexing global activity

---

## Roadmap

- [ ] Define and publish custom lexicons (`place`, `review`, `photo`)
- [ ] Build lexicon validation and indexing pipeline
- [ ] Implement geospatial query API
- [ ] Build interactive map frontend
- [ ] Add follow graph and timeline
- [ ] Integrate with ATGeo Working Group standards as they mature
- [ ] Add moderation tools for instance admins

---

## Contributing

This project is intentionally a **blueprint and evolving implementation**.
If you want to contribute ideas, open an issue to discuss:

- Alternative lexicon designs for reviews and media
- Integration with the ATGeo Working Group's location standards
- Better indexing strategies for the ATProto firehose
- Spam prevention and content moderation heuristics

---

## License

[Choose your preferred license — e.g., AGPLv3, MIT, etc.]
This project is open‑source but **requires attribution** if you redistribute.

---

## Disclaimer

This software is **under active conceptual design and early development**.
It is not yet production‑ready. Use it as a learning tool or a foundation for your own federated map application.
