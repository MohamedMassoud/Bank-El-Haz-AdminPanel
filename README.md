# 🎲 Bank El Haz — Admin Panel

A fancy, game-themed admin dashboard for the **Bank El Haz** Realtime Database.
It gives you a live, user-friendly view and control over every game room without touching the game code.

![theme](https://img.shields.io/badge/theme-yellow%20%2B%20red-c8102e) ![firebase](https://img.shields.io/badge/data-Firebase%20RTDB-f4a300)

---

## ✨ Features

**Live monitoring**
- Auto-refreshes every 15s (and instantly after any change) with a `● LIVE` indicator. It only re-draws when something actually changed, so an open room keeps its scroll position instead of jumping back to the top.
- Stat cards: total rooms, live games, players online, finished games.
- Room cards show: code, status badge (Live / Finished / Lobby / Empty), player tokens, host 👑, online count, version `vN`, current turn ▶, winner 🏆, and "last activity".
- Search by room code **or** player name, filter by status, and sort by activity / code / players / status.

**Room detail (click any room)**
- 🎮 **Game control** — set the current player, phase (`roll/buy/end/ro`), declare/clear a winner, or advance to the next turn.
- 👥 **Player editor** (per player): rename, set cash, set position (0–47), set jail turns, ±100 cash, eliminate/revive, toggle bot/human, and kick. Live HOST/BOT/OUT/JAIL/TURN tags.
- 🏠 **Properties** owned (with mortgaged 🔒 and house/hotel counts).
- 📜 **Activity log** (rendered readable, with square names from the room's config).

**Room actions**
- 🔄 **Restart** — resets board, cash & positions and returns everyone to the lobby (players stay).
- 🗑️ **Delete room** (also clears its presence data).
- 🔗 **Copy game link** (a clickable invite link to that room).
- ⬇️ **Export JSON** (download the raw room state).
- 🤖 **Add bot**.

**Quality-of-life**
- 🧹 **Clean empty** — one click to delete all rooms with no players.
- ☠️ **Clear all** — wipe **every** room (live games, finished, lobbies and empty) in one go, with a two-step confirmation so it can't be hit by accident. The button auto-disables when there are no rooms.
- Confirm dialogs on every destructive action, plus toast feedback.
- Fully responsive + dark-mode aware, same yellow/red "Monopoly" look as the game.

---

## ▶️ How to run

The panel is a single file: **`index.html`** (no build step, no dependencies).

Because it talks to Firebase, open it through a small web server (recommended) rather than `file://`:

```bash
# from this AdminPanel folder
python -m http.server 8000
# then open http://localhost:8000
```

Or host the folder on any static host (Firebase Hosting, Netlify, GitHub Pages, etc.).

---

## 🔥 Firebase

It's already wired to the same project as the game:

```
databaseURL: https://bank-el-haz-da25a-default-rtdb.europe-west1.firebasedatabase.app
apiKey:      AIzaSyDO51e5_E3fxP69QRGsI5cyylvugeV25Xw
```

To change it, edit the two constants at the top of the `<script>` in `index.html`:

```js
const API_KEY="...";
const DB_URL ="https://<project>-default-rtdb.<region>.firebasedatabase.app";
```

It signs in with **Firebase Email/Password authentication** and reads/writes via the REST API — so it works even from `file://` and needs no npm install. The session is cached in `localStorage` and auto-restores on reload.

### 🔐 Set up the admin login (one-time — no credentials in the code)

1. **Enable Email/Password:** Firebase Console → **Authentication → Sign-in method → Email/Password → Enable**.
2. **Create the admin user:** Authentication → **Users → Add user** → enter the admin **email + password**. (These live in Firebase, never in the code.)
3. **Copy the admin's UID** (shown next to the user you just created).
4. **Publish the rules below**, replacing the placeholder with that UID — so only the admin account can use the panel.

Then open the panel → enter that email/password → you're in. Use **🚪 Sign out** (top-right) to lock it again.

### Database rules (admin-only)
The game needs anonymous read/write on individual rooms/presence; the admin additionally needs to **list all rooms**. This scopes listing to the admin UID while leaving the game untouched:

```json
{
  "rules": {
    "rooms": {
      ".read": "auth != null && auth.uid === \"PASTE_ADMIN_UID\"",
      "$room": {
        ".read": "auth != null",
        ".write": "auth != null",
        ".validate": "$room.matches(/^[a-zA-Z0-9_-]{1,20}$/)"
      }
    },
    "presence": {
      ".read": "auth != null && auth.uid === \"PASTE_ADMIN_UID\"",
      "$room": {
        ".read": "auth != null",
        "$uid": { ".write": "auth != null && auth.uid === $uid" }
      }
    }
  }
}
```

Replace `PASTE_ADMIN_UID` with the admin's UID from step 3. Now only the admin can list/manage every room; everyone else (the game) can still read/write their own room.

If listing fails with a permission error, either the UID doesn't match the logged-in user or the rules weren't published.

---

## ⚠️ Safety

This tool has **full read/write** over every room. Keep it private (don't deploy it publicly).
Every change is applied to players **instantly** — restart/delete/kick are confirmed first, but always double-check the room code.

---

## 🗂️ Data model (for reference)

```
rooms/<code>          → full game state (P[] players, st, ph, t, w, rc, ro, cfg, v, log, O/H/M/V …)
presence/<code>/<uid> → true while a client is connected
```

`v` is a monotonically increasing version number; the panel increments it on every write so live game clients stay in sync.