# Bridges – Student Profile, Mentor Connections and Messaging

This is an **integration into the uploaded Bridges repository**, not a separate blank Expo application. Existing Clerk authentication, Expo Router tabs, mentor browsing, bookings and mentor profile editing remain in place.

## What was added

- `app/src/app/student-profile.tsx` – student profile setup/editing styled after the provided screenshots, including photo upload through existing Clerk component, name, bio, academic details, selected skills/interests and social links.
- `app/src/app/connect-mentors.tsx` – searchable mentor directory with **Message** and **View Profile** actions.
- `app/src/app/mentor-chat/[id].tsx` – opening message form. Students can send **one** introductory message per mentor.
- `app/src/app/conversation/[id].tsx` – two-way chat once the mentor has replied. While waiting, the student message input is disabled.
- `app/src/app/(tabs)/messages.tsx` – replaces the placeholder/mock chat list with authenticated backend conversations. Mentors see pending student requests here.
- `app/src/services/chat/api.ts` – typed requests to the Bridges backend.
- `backend/src/modules/chat/*` – Prisma-backed chat logic and API validation.
- `backend/src/app/api/conversations/*` – authenticated message/list APIs.
- `backend/prisma/schema.prisma` and migration – durable conversations, messages, statuses and student social links.
- Existing role selection, sign-up, home avatar, mentor detail, routing and profile edit links have been updated to reach the new screens.

## Required setup

You need **Node.js**, a **PostgreSQL database**, and the existing Bridges **Clerk** application keys. The included backend is necessary for actual messaging between two accounts; the frontend does not fake mentor replies.

**Terminal 1 – backend**

```bash
cd backend
npm install
# Copy .env.example to .env and configure DATABASE_URL, CLERK_SECRET_KEY,
# and NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY.
npx prisma generate
npx prisma migrate deploy
npm run dev
```

By default this backend listens on port **3001**.

**Terminal 2 – Expo mobile app**

```bash
cd app
npm install
# Copy .env.example to .env and configure EXPO_PUBLIC_CLERK_PUBLISHABLE_KEY.
# For a real phone, set EXPO_PUBLIC_API_URL to the computer's LAN IP, for example:
# EXPO_PUBLIC_API_URL=http://192.168.1.100:3001
npx expo start -c
```

The phone and computer should be on the same network. Ensure port 3001 is accessible. If using an Android emulator, `http://10.0.2.2:3001` is the standard host gateway. In production, use a reachable HTTPS backend.

In PowerShell, copy the environment examples using `Copy-Item .env.example .env` instead of `cp`.

**Important:** Never put `CLERK_SECRET_KEY` or database credentials in the Expo `.env` file. Expo `EXPO_PUBLIC_` variables are public.

## Exercise the full chat rule

1. Create two accounts in the same Clerk + Bridges backend: a **student** and a **mentor**. Complete their profiles.
2. Sign in as the student. Go to **Messages → Find and connect with mentors** (or Explore → Mentor Profile → Message Mentor).
3. Select the mentor and send an opening message. This creates a **PENDING** conversation. The chat composer is disabled for the student.
4. Sign in as the mentor on a second phone or emulator. Open **Messages**, tap the pending student request and send a reply.
5. The backend switches the conversation to **ACTIVE**. The student's open chat refreshes approximately every 5 seconds and now permits sending new messages.

### Server-side rules

- Authenticated **students** can start mentor conversations.
- Only one conversation is allowed per student/mentor pair. Unique database constraint prevents duplicate invitations.
- The student sends one introductory message; while **PENDING**, an additional student message receives **HTTP 403**, even if someone bypasses the disabled UI.
- Only the **mentor's actual reply**, authenticated against the database, changes the conversation from **PENDING** to **ACTIVE**.
- Both participants can chat normally after the first mentor reply. No third party may retrieve or post to their chat.
- Messages and status are stored in PostgreSQL; they persist across devices and restarts.

### API endpoints

| Method | URL | Purpose |
| --- | --- | --- |
| GET | `/api/conversations` | List my conversations |
| POST | `/api/conversations` | Start a conversation; `{mentorId, text}` |
| GET | `/api/conversations/with-mentor/:mentorId` | Find my conversation with a given mentor |
| GET | `/api/conversations/:id` | Get the conversation and messages |
| POST | `/api/conversations/:id/messages` | Send a message; `{text}` |

These use the existing Clerk Bearer token and response envelope `{status, data, message}`.

## Validation status

All original and newly added project TypeScript and TSX files were parsed with TypeScript without syntax errors. Imported local paths in the new Expo files were checked. **A full app build, live PostgreSQL migration, and two-account test were not possible in this environment** because the package registry was unreachable and no configured database/Clerk keys were supplied. Run the steps above to complete runtime verification.
