# Hotel Ananga: Hotel Booking System

## Project Overview

The official website for **Hotel Ananga**, a newly opening hotel, with an online booking system for guests. This is a multi-week project, built with AI as a collaborative tool throughout design and development.

The site is treated as a real public website: welcoming, trustworthy and professional, since guests will judge the hotel by it.

## Pages

- **`index.html`**: Hotel introduction: about the hotel, facilities, room overview, hotel tour video, gallery, location and contact.
- **`booking.html`**: Rooms and booking: guests choose dates and number of guests, see live availability and prices, select a room, enter their details and receive a booking reference.

Both pages link to each other ("Book a Stay" / "Back to Home"), with no broken links.

## File Structure

```
/
├── index.html          ← homepage
├── booking.html        ← rooms & booking
├── graph/              ← all images and videos
│   └── README.md       ← list of expected photo/video filenames
├── supabase/
│   └── schema.sql      ← database setup (run once in Supabase)
└── README.md
```

- All images and videos live in `graph/` and are referenced with relative paths, e.g. `src="graph/room-deluxe.jpg"`.
- CSS and JavaScript are kept inside each HTML file (no separate files).
- Until a photo is added, each slot shows a neutral placeholder with the expected filename.

## Design

- Calm, upscale look: deep forest green, warm sand and brass accents; Cormorant Garamond headings with Inter body text.
- Responsive layout for desktop and mobile.
- Reserved slots for room photos, facility photos, gallery images and a hotel tour video.

## Backend (Booking Database)

Bookings are stored in **Supabase** (a hosted PostgreSQL database). The website stays a static site on GitHub Pages and talks to Supabase directly from the browser.

**Security rules**

- Row Level Security (RLS) is enabled on every table.
- The website only contains the Supabase **project URL** and the public **anon key**. The `service_role` key is never put in the website.
- The public can **read active rooms only**. Guests cannot read, change or delete any booking, including their own.
- All writes go through database functions (RPCs) that validate input on the server:

| Function | Who can call it | What it does |
|---|---|---|
| `get_availability(check_in, check_out)` | Anyone | Lists rooms with units free for every night of the stay and the total price |
| `create_booking(...)` | Anyone | Validates dates, guest count and availability, locks the room to prevent double booking, saves the booking and returns a reference like `HA-7K3Q9P` |
| `update_booking_status(reference, status)` | Logged-in staff only | Changes a booking to `confirmed`, `cancelled`, `checked_in` or `checked_out` |

**Tables**

- `rooms`: room types, capacity, nightly price, number of units, photo path.
- `bookings`: guest details, dates, room, total price, status.
- `staff`: hotel staff accounts allowed to view and manage bookings.

**Booking rules**

- Check-in cannot be in the past; check-out must be after check-in; maximum 30 nights online.
- Guest count cannot exceed the room's capacity.
- A room type is unavailable when every unit is taken on any night of the requested stay.
- Payment is made at the hotel; new bookings start as `pending` until staff confirm them.

**Planned next:** a staff page (`admin.html`) where reception logs in to see and confirm bookings, and email notifications for new bookings.

## Setup

### 1. Create the database

1. Create a free project at [supabase.com](https://supabase.com).
2. Open **SQL Editor → New query**, paste the contents of `supabase/schema.sql`, and click **Run**.
3. Check **Table Editor → rooms**: four sample rooms should be there. Edit names, prices (MZN) and number of units to match the hotel.

### 2. Connect the website

1. In Supabase, open **Project Settings → API**.
2. Copy the **Project URL** and the **anon public** key.
3. Open `booking.html`, find the `CONFIGURATION` section near the bottom, and paste them in:
   ```js
   const SUPABASE_URL = 'https://your-project.supabase.co';
   const SUPABASE_ANON_KEY = 'your-anon-public-key';
   ```
   Without these, the booking page runs in **demo mode** (sample rooms, nothing saved).

### 3. Add staff (for the upcoming admin page)

1. In Supabase, open **Authentication → Users → Add user** and create a reception account.
2. In **SQL Editor**, run:
   ```sql
   insert into public.staff (user_id, full_name, role)
   select id, 'Reception', 'reception' from auth.users where email = 'reception@example.com';
   ```

### 4. Publish on GitHub Pages

1. Create a **public** repository on GitHub and upload all files (keep the folder structure).
2. Go to **Settings → Pages**, set the source to the `main` branch (root folder) and save.
3. After a minute, the site is live at `https://<username>.github.io/<repository>/`.

## AI Collaboration Log

A record of every prompt given to the AI, with notes on why it was used and what it improved.

### 1. Rewriting the project requirements

**Prompt:**
Rewrite these requirements for a hotel system project. *(pasted the original web design brief)*

**Notes:**
Adapted a department-website brief into a hotel brief: renamed the second page to `booking.html` and rewrote the content for a hotel.

### 2. Removing the assessment criteria

**Prompt:**
No need for assessment criteria, it's my personal project.

**Notes:**
Turned the brief into a personal project document without grading or classroom framing.

### 3. Adding the backend and starting the build

**Prompt:**
Yes [add a backend section], and we can start.

**Notes:**
The AI added the Supabase backend design (RLS, RPC functions, staff role), wrote `schema.sql`, tested it on a local PostgreSQL database (availability, double-booking prevention, guest limits, permissions), and built the first versions of `index.html` and `booking.html`.
