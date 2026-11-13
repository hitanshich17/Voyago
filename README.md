# Voyago

**A collaborative travel planner: plan a trip together with friends, live, in one place.**

Built for CS478 at Drexel University.
**Live app:** https://voyago.hitanshichhabria.com

## What it does

- **Trips together.** Create a trip, invite friends as editors or viewers, and see who's looking at it right now.
  Changes by one person show up for everyone on the trip immediately.
- **Plan the whole trip.** Itinerary, calendar view, budget, accommodation and getting there: flight search
  (Google Flights via SerpApi) with airport lookup, plus maps, places and routes from Google Maps.
- **Friends and chat.** Send and accept friend requests, block users, and message friends in real time with
  unread counts.
- **Accounts.** Sign up and log in; passwords are hashed with Argon2 and sessions use HTTP-only cookies.

## Architecture

```
React + TypeScript (Vite, MUI)  ──REST──▶  Express 5 + TypeScript  ──▶  MongoDB
            ▲                                     │
            └──────────── Socket.IO ──────────────┘
       live trip edits, presence, chat messages
```

| Part | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite, Material UI, React Router, FullCalendar |
| Maps and travel data | Google Maps (Maps, Places, Routes), SerpApi (Google Flights), OpenStreetMap (Nominatim, Overpass) |
| Backend | Node.js, Express 5, TypeScript |
| Real time | Socket.IO: a room per trip for edits and presence; direct messages between friends |
| Database | MongoDB |
| Auth | Argon2 password hashing, session tokens in HTTP-only cookies |
| Tests | Vitest (auth and friend-relation routes) |

**Real-time collaboration:** each trip is a Socket.IO room. When someone joins, the server checks their access
(owner, editor or viewer) and broadcasts who is present. Edits are saved through the REST API and broadcast over the
socket to everyone else in the room; viewers can't edit.

## Run it locally

You need Node.js, a MongoDB connection string, a Google Maps API key and a SerpApi key.

```bash
# Backend (http://localhost:5001 by default)
cd backend
npm install
# .env: MONGODB_URI, SERP_API_KEY, FRONTEND_URL
npm run dev

# Frontend (http://localhost:5173)
cd frontend
npm install
# .env: VITE_API_URL, VITE_GOOGLE_API_KEY, VITE_GOOGLE_MAP_ID
npm run dev
```

Run the backend tests with `cd backend && npm test`.

If you see "Invalid session", log out and back in: sessions are kept in server memory, so a restart signs
everyone out.

## Roadmap

- [ ] Event-driven collaborative editing: edits published to **Kafka** keyed by trip ID, so each trip's edits stay in
      order, and consumers pushing them to clients over WebSockets
- [ ] Two load-balanced WebSocket instances, with **Redis** pub/sub fanning each edit out to every client on a trip
- [ ] Persistent sessions (Redis) so restarts don't sign users out
- [ ] **GitHub Actions** for builds, pull-request checks and deployment
