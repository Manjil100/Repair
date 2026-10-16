# RepairIQ – setup guide

A React Native (Expo) app with a Supabase backend for booking and tracking phone repairs.

## What's in this folder

| File | What it does |
|---|---|
| `App.js` | Starts the app and sets up navigation (Login/Register when logged out, everything else when logged in) |
| `src/lib/supabase.js` | Connects to your Supabase project |
| `src/lib/status.js` | The repair stages (Pending → Received → Diagnosing → In Repair → Ready → Completed / Cancelled) |
| `src/context/AuthContext.js` | Remembers who is logged in and whether they are staff |
| `src/screens/LoginScreen.js` | Log in |
| `src/screens/RegisterScreen.js` | Create an account (name, phone, email, password) |
| `src/screens/HomeScreen.js` | Main menu (staff also see "Staff Dashboard") |
| `src/screens/BookRepairScreen.js` | Book a repair: device, issue type, description, preferred date |
| `src/screens/RepairListScreen.js` | Used for Track My Repairs, Repair History and Staff Dashboard |
| `src/screens/RepairDetailsScreen.js` | One repair's details, progress steps and update timeline; staff can change the status here |
| `src/components/` | Small reusable pieces (button, status badge, repair card) |
| `src/theme.js` | Colours and shared styles |
| `supabase/schema.sql` | Database tables, automatic triggers and security rules |
| `supabase/upgrade_v2.sql` | Adds repair quotes, damage photos and live updates (run after `schema.sql`) |
| `src/lib/photos.js` | Picks, uploads and shows damage photos |

## Step 1 – Set up Supabase

1. Go to https://supabase.com, sign in and create a new project.
2. Open **SQL Editor → New query**, paste the whole of `supabase/schema.sql` and click **Run**. Then do the same with `supabase/upgrade_v2.sql`.
3. For easy testing, go to **Authentication → Sign In / Providers → Email** and turn off **Confirm email**, so new accounts can log in straight away.
4. Go to **Project Settings → API** and copy the **Project URL** and the **anon public** key.

## Step 2 – Create the Expo project

In a terminal in VS Code:

```bash
npx create-expo-app@latest RepairIQ --template blank
cd RepairIQ
npx expo install @supabase/supabase-js @react-native-async-storage/async-storage react-native-url-polyfill @react-navigation/native @react-navigation/native-stack react-native-screens react-native-safe-area-context expo-image-picker base64-arraybuffer
```

If you already have a RepairIQ Expo project, run just the `npx expo install ...` line inside it.

## Step 3 – Add the code

1. Copy `App.js` and the whole `src` folder into your project folder, replacing the existing `App.js`.
2. Copy `.env.example` to a new file called `.env` and paste in your Supabase URL and anon key.

## Step 4 – Run it

```bash
npx expo start
```

Scan the QR code with the Expo Go app on your phone. If you change `.env`, stop and restart with `npx expo start -c`.

## Step 5 – Make a staff account

1. In the app, register a second account, for example `staff@repairiq.test`.
2. In Supabase **SQL Editor**, run:

```sql
update public.profiles set role = 'staff'
where id = (select id from auth.users where email = 'staff@repairiq.test');
```

3. Log out and back in as that account. You'll now see **Staff Dashboard** on the Home screen.

## Trying the full workflow

1. Log in as a customer and book a repair.
2. Open **Track My Repairs** and check the repair is listed as *Pending*.
3. Log out, log in as staff, open **Staff Dashboard**, tap the repair, choose a new status, add a note and tap **Save Status**.
4. Log back in as the customer and pull down to refresh. The new status and note appear in the timeline.
5. When staff set the status to *Completed*, the repair moves to the customer's **Repair History**.

### Quotes, photos and live updates (v2)

- **Damage photos:** when booking, tap **+ Add** to attach up to 3 photos. They show on the Repair Details screen for both the customer and staff.
- **Quotes:** staff enter a price and number of days on Repair Details and tap **Send Quote**. The customer sees the quote with **Approve** and **Decline** buttons. Every quote event is added to the timeline.
- **Live updates:** keep the customer's Repair Details screen open on one phone (or the web version) and change the status as staff on another. The customer's screen updates by itself, with no need to refresh.

## How the security works

Supabase Row Level Security is turned on for every table:

- Customers can only see and create their own devices and repairs.
- Only staff can change a repair's status.
- Nobody can make themselves staff from the app. Only you can, in the dashboard.
- Every status change is written to `repair_status_history` automatically by a database trigger, so the timeline can't be skipped or edited from the app.
# Repair
