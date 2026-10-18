# Hotel Ananga: Hotel Booking System

## Project Overview

The official website for **Hotel Ananga**, a newly opening hotel, with an online booking system for guests and a staff page for managing reservations and payments. The whole site is available in **English and Portuguese**. This is a multi-week project, built with AI as a collaborative tool throughout design and development.

The site is treated as a real public website: welcoming, trustworthy and professional, since guests will judge the hotel by it.

## Pages

- **`index.html`**: Hotel introduction: about the hotel, facilities, room overview, hotel tour video, gallery, location and contact.
- **`booking.html`**: Rooms and booking. Guests choose dates and number of guests, see live availability and prices, select a room, enter their details, choose how they will pay, and receive a booking reference with payment instructions.
- **`admin.html`**: Staff page (not linked prominently; staff log in). Reception sees all bookings, confirms, cancels, checks guests in and out, and records payments and refunds.

All pages link to each other, with no broken links. Each page has an **EN / PT** switch; the choice is remembered across pages. Visitors whose browser is set to Portuguese see Portuguese first.

## File Structure

```
/
├── index.html          ← homepage
├── booking.html        ← rooms & booking
├── admin.html          ← staff page
├── graph/              ← all images and videos
│   └── README.md       ← list of expected photo/video filenames
├── supabase/
│   ├── schema.sql                        ← database setup, part 1
│   ├── update-2-payments-language.sql    ← database setup, part 2
│   └── fix-create-booking.sql            ← old one-off fix (already included in the files above)
└── README.md
```

- All images and videos live in `graph/` and are referenced with relative paths, e.g. `src="graph/room-deluxe.jpg"`.
- CSS and JavaScript are kept inside each HTML file (no separate files).
- Until a photo is added, each slot shows a neutral placeholder with the expected filename.

## Design

- Calm, upscale look: deep forest green, warm sand and brass accents; Cormorant Garamond headings with Inter body text.
- Responsive layout for desktop and mobile. On phones the staff page shows bookings as cards instead of a table.
- Reserved slots for room photos, facility photos, gallery images and a hotel tour video.

## Payments

No money is taken online. Guests choose how they **prefer** to pay, and staff record what is actually received.

| Method | When | What the guest sees after booking |
|---|---|---|
| Cash | At reception | "Pay at reception on arrival" |
| Card (Visa / Mastercard) | At reception | "Pay at reception on arrival" |
| M-Pesa (Vodacom) | Before arrival | Hotel's M-Pesa number + send confirmation code and booking reference |
| e-Mola (Movitel) | Before arrival | Hotel's e-Mola number + same instructions |
| mKesh (Tmcel) | Before arrival | Hotel's mKesh number + same instructions |
| Bank transfer | Before arrival | Bank, account, NIB, use booking reference as description |

Staff record each payment (amount, method, transaction ID, notes) on the staff page. Part payments (deposits) and refunds are supported. The system blocks recording more than the booking total, or refunding more than was paid.

## Backend (Booking Database)

Bookings are stored in **Supabase** (a hosted PostgreSQL database). The website stays a static site on GitHub Pages and talks to Supabase directly from the browser.

**Security rules**

- Row Level Security (RLS) is enabled on every table.
- The website only contains the Supabase **project URL** and the public **anon / publishable key**. The secret / `service_role` key is never put in the website.
- The public can **read active rooms only**. Guests cannot read, change or delete any booking or payment.
- Staff are Supabase Auth users listed in the `staff` table. Logged-in users who are not in `staff` see nothing.
- All writes go through database functions (RPCs) that validate input on the server:

| Function | Who can call it | What it does |
|---|---|---|
| `get_availability(check_in, check_out)` | Anyone | Rooms (English + Portuguese text) with units free for every night of the stay, and the total price |
| `create_booking(...)` | Anyone | Validates dates, guest count, payment method and availability; locks the room to prevent double booking; saves the booking; returns a reference like `HA-7K3Q9P` |
| `staff_bookings(from, to)` | Staff | All bookings with room name and amount paid |
| `staff_payments(reference)` | Staff | Payment history for one booking |
| `record_payment(reference, amount, method, transaction_id, notes)` | Staff | Records a payment (negative amount = refund) |
| `update_booking_status(reference, status)` | Staff | `pending` → `confirmed` → `checked_in` → `checked_out`, or `cancelled` |

**Tables**

- `rooms`: room types, English and Portuguese name/description, capacity, nightly price, number of units, photo path.
- `bookings`: guest details, dates, room, total price, preferred payment method, status.
- `payments`: every payment or refund received, with method, transaction ID and which staff member recorded it.
- `staff`: hotel staff accounts allowed to manage bookings.

**Booking rules**

- Check-in cannot be in the past; check-out must be after check-in; maximum 30 nights online.
- Guest count cannot exceed the room's capacity.
- A room type is unavailable when every unit is taken on any night of the requested stay. Cancelled bookings free the room.
- New bookings start as `pending` until staff confirm them.

## Setup

### 1. Create the database

1. Create a free project at [supabase.com](https://supabase.com).
2. Open **SQL Editor → New query**, paste the contents of `supabase/schema.sql`, and click **Run**.
3. Open another **New query**, paste `supabase/update-2-payments-language.sql`, and click **Run**.
   - If your database already existed before this update, run **only this file**. It is safe to run more than once.
4. Check **Table Editor → rooms**: four rooms, each with a Portuguese name. Edit names, prices (MZN) and number of units to match the hotel.

### 2. Connect the website

1. In Supabase, open **Project Settings → API Keys**.
2. Copy the **Project URL** (just `https://xxxx.supabase.co`) and the **anon public** or **publishable** key.
3. Paste them into the `CONFIGURATION` section near the bottom of **both** `booking.html` and `admin.html`:
   ```js
   const SUPABASE_URL = 'https://your-project.supabase.co';
   const SUPABASE_ANON_KEY = 'your-public-key';
   ```
   Without these, the pages run in **demo mode** (sample data, nothing saved).
4. In `booking.html`, fill in `PAYMENT_INFO` (just below the configuration) with the hotel's real M-Pesa, e-Mola and mKesh numbers, bank details and WhatsApp number.

### 3. Add staff accounts

1. In Supabase, open **Authentication → Users → Add user → Create new user**, enter the email and password, tick **Auto Confirm User**, and create.
2. In **SQL Editor**, run (with that exact email):
   ```sql
   insert into public.staff (user_id, full_name, role)
   select id, 'Reception', 'reception' from auth.users where email = 'reception@example.com';
   ```
   The editor shows "Success. No rows returned" even when it worked. Check under **Table Editor → staff**.
3. **Recommended:** in **Authentication → Sign In / Providers**, turn off **Allow new users to sign up**, so only accounts you create can exist.

### 4. Publish on GitHub Pages

1. Create a **public** repository on GitHub and upload all files (keep the folder structure).
2. Go to **Settings → Pages**, set the source to the `main` branch (root folder) and save.
3. After a minute, the site is live at `https://<username>.github.io/<repository>/`. The staff page is at `.../admin.html`.

## AI Collaboration Log

A record of every prompt given to the AI, with notes on why it was used and what it improved.

### 1. Rewriting the project requirements

**Prompt:** Rewrite these requirements for a hotel system project. *(pasted the original web design brief)*

**Notes:** Adapted a department-website brief into a hotel brief: renamed the second page to `booking.html` and rewrote the content for a hotel.

### 2. Removing the assessment criteria

**Prompt:** No need for assessment criteria, it's my personal project.

**Notes:** Turned the brief into a personal project document without grading or classroom framing.

### 3. Adding the backend and starting the build

**Prompt:** Yes [add a backend section], and we can start.

**Notes:** The AI designed the Supabase backend (RLS, RPC functions, staff role), wrote `schema.sql`, tested it on a local PostgreSQL database, and built the first versions of `index.html` and `booking.html`.

### 4. Asking for setup steps

**Prompt:** Please, don't forget to provide the steps.

**Notes:** Got step-by-step instructions for Supabase setup, connecting the site, and publishing on GitHub Pages.

### 5. Fixing a connection error

**Prompt:** *(screenshot of "Invalid path specified in request URL")*

**Notes:** The Supabase URL had been pasted with `/rest/v1/` at the end. The AI explained the fix and made the page clean up the URL automatically.

### 6. Correcting the hotel name

**Prompt:** It is not Hotel Ananda, it is Hotel Ananga.

**Notes:** Renamed the hotel everywhere and removed a sentence about the meaning of "Ananda".

### 7. Adding staff accounts

**Prompts:** "I can't find this line" / "It just says success, no rows returned" / "Done"

**Notes:** Clarified that the SQL is typed into Supabase rather than found in a file, and how to verify the staff account.

### 8. Fixing a booking error

**Prompt:** *(screenshot of "Something went wrong")*

**Notes:** The booking-reference generator used an add-on that Supabase stores in a different schema. The AI switched to a built-in function, provided a one-off fix script, and re-tested with Supabase's layout.

### 9. Staff page, payment methods and Portuguese

**Prompt:** Yes, also include payment methods, switch between English and Portuguese.

**Notes:** Added `admin.html` (login, booking list, status changes, payments and refunds), payment method choice with instructions for M-Pesa, e-Mola, mKesh and bank transfer, and an EN/PT switch on every page. Database changes are in `update-2-payments-language.sql`, tested for fresh installs, upgrades and permission rules.
