# APOGEE Room Scheduler — MVP

A browser-based room scheduling tool built for the **APOGEE core team at BITS Pilani, Pilani Campus**. Designed to replace the painful Excel drag-and-drop process of allocating rooms across 4 festival days.

---

## The Problem It Solves

Every year, the APOGEE scheduling team has to manually:
- Write room numbers in Excel
- Drag and drop events into time slots by hand
- Mentally track which rooms are free at what time
- Detect conflicts without any tooling

This tool makes that entire process visual, interactive, and instant.

---

## How to Use It

### Running locally

```bash
# Clone the repo
git clone https://github.com/HimnishTaneja/apogee-scheduler.git
cd apogee-scheduler

# Serve it (Python 3)
python -m http.server 3000

# Open in browser
# http://localhost:3000/scheduler.html
```

No install. No build step. One HTML file.

---

### The Three Views

#### 1. Timeline View (default)
A room × time grid — every room as a row, time (00:00–24:00) as the horizontal axis.

- All 5 days of APOGEE pre-loaded
- Events shown as colour-coded blocks per club
- **Red border + `!` badge** = conflict detected (two events in the same room at the same time)
- Filter bar to isolate NAB / FD-2 / FD-1 rooms
- **Click any room label** → side panel slides in showing events across all 5 days + capacity editor
- Drag any event block to move it to a different room or time — conflicts recalculate live

#### 2. NAB Map View
A spatial floor plan of the New Academic Block — rooms laid out exactly as they are in the building.

**Normal mode (no event selected):**
- Green = room is free on the selected day
- Yellow / Orange / Red = 1 / 2 / 3+ events booked
- Red glow = conflict in that room
- Hover any room → tooltip shows all events, times, and clubs for that day
- Click any room → full detail panel (events across all days, capacity, utilisation bar)

**Find-slot mode (event card selected):**
1. Create an event card using **"+ New Event Card"** in the left sidebar
2. Click the card → map switches mode
3. **Bright green rooms** = have a free time slot that fits this event's duration
4. **Faded red rooms** = fully booked, no slot fits
5. Click a green room → pick from available time windows → hit **"Book"**
6. Event is placed, card disappears, map updates instantly

Switch between days using the day buttons (Day 0–4) directly in the map view.

#### 3. Booking View
A vertical calendar grid — rooms as columns, time as rows (like Google Calendar but for rooms).

- All existing events shown as placed blocks
- **Left sidebar** shows pending event cards
- **Drag a card** from the sidebar onto any room/time cell
  - Green highlight = slot is free, drop allowed
  - Red highlight = conflict, drop blocked
- **Drag existing blocks** to move them to a different room or time
- Conflicts recalculate after every drop

---

### Typical Workflow (Club Head Walk-in)

1. Club head arrives and says *"I need a room for 3 hours on Day 2 afternoon"*
2. Core member opens **NAB Map** → hits **"+ New Event Card"** → fills event name, club, `3h`, Day 2
3. Clicks the card → map goes green/red showing exactly which rooms have 3-hour windows free
4. Hover rooms to see current bookings → click a green one → pick the time → **Book**
5. Done in under 30 seconds, no Excel involved

---

## What's Pre-loaded

The APOGEE 2026 schedule (Apr 10–14) is baked directly into the HTML — all 5 days, all clubs, all rooms. No file upload needed.

Rooms covered:
- **NAB (6101–6164)** — New Academic Block, upper and lower wings
- **FD-2 First Floor (22xx)** — rooms 2201, 2203, 2204, 2206, 2222, 2224
- **FD-1 (5xxx)**, **FD-3 (1xxx)**, and named venues (Main Audi, Rotunda, VK Lawns, Krishna Parking, etc.)

---

## MVP Limitations

This is a **frontend-only scaffold**. Everything lives in one HTML file, in memory. Refreshing the page resets any changes made during the session.

Current limitations:
- No persistent storage — bookings made during a session are lost on refresh
- No multi-user support — two people can't schedule simultaneously
- No authentication — anyone with the URL can edit
- Schedule data is hardcoded — updating it requires editing the HTML
- No export of newly added/moved events back to Excel (only the original data exports cleanly)
- Room capacities entered in the panel are not saved between sessions

---

## Roadmap — Polished Version

### Phase 1 — Persistence (Backend + Database)

Replace the hardcoded JSON with a **REST API + database**:

```
Frontend (HTML/React) ←→ API (Node/FastAPI) ←→ PostgreSQL
```

- Events, rooms, and capacities stored in a DB
- Schedule changes persist across sessions and users
- Excel import/export goes through the server, not the browser

**Schema sketch:**
```sql
events(id, name, club_id, room_id, day, start_min, end_min, created_by)
rooms(id, number, building, floor, capacity)
clubs(id, name, contact_email)
bookings(id, event_id, room_id, day, start_min, end_min, status)
```

### Phase 2 — Multi-user + Roles

Different access levels for different people:

| Role | Can do |
|------|--------|
| **Club Head** | Submit event request (name, duration, day preferences) |
| **Core Member** | View all requests, assign rooms, approve/reject |
| **Admin** | Full edit access, room management, export |

Club heads get a simple form — they never see the full scheduler. Core members handle all allocation.

### Phase 3 — Smart Scheduling (Constraint Solver)

Instead of manual drag-and-drop, the system can **auto-suggest** room assignments:

- Club submits: "2 hours, Day 2, prefer NAB, need 100+ capacity"
- System finds all valid slots that satisfy constraints
- Core member picks from the shortlist or auto-accepts

This is a **constraint satisfaction problem** — solvable with Google OR-Tools, OptaPlanner, or a simple backtracking algorithm for a festival scale.

### Phase 4 — Real-time Collaboration

Multiple core members working simultaneously:

- WebSocket-based live updates (Socket.io or Supabase Realtime)
- Lock a room slot while someone is booking it
- See other team members' cursors/selections on the map
- Conflict resolution when two people book the same slot at the same time

### Phase 5 — Club Head Portal

A separate interface for club heads:

- Submit event request with preferences (day, duration, room type, capacity needed)
- Track request status (pending → approved → room assigned)
- Get notified (email/WhatsApp) when their room is confirmed
- See their confirmed schedule

### Infrastructure Stack (Suggested)

```
Frontend:   React + Tailwind (keep the same visual design)
Backend:    FastAPI (Python) or Express (Node)
Database:   PostgreSQL (events, rooms, clubs, users)
Auth:       BITS SSO / Google OAuth with BITS email restriction
Hosting:    Railway / Render (backend) + Vercel (frontend)
Realtime:   Supabase or Socket.io
```

Total estimated dev time for a polished v1: **4–6 weeks** with a 3-person team.

---

## Built For

APOGEE 2026, BITS Pilani — Pilani Campus  
Built as an MVP to demonstrate the scheduling workflow concept.

---
