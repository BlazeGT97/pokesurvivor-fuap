# Your own free server (accounts, friends, co-op, leaderboard)

By default, your copy of PokéSurvivor+ runs **without a server**: progress is saved in the browser and there's no multiplayer.

Follow this guide to give **your** copy its **own free server** on Google Firebase, so that you and your friends get:

- accounts (name + password, or Google), with progress saved online;
- friends, invites and **co-op games**;
- your own leaderboard.

**It's free** (Firebase "Spark" plan: no credit card needed). Allow **30 to 45 minutes** the first time, even with no experience. Lost at some step? Ask ChatGPT (or any AI assistant): copy the step and describe what you see on your screen.

> Before you start, you need your own copy of the game online on GitHub Pages: fork this repo, then **Settings → Pages → Deploy from a branch → main / (root) → Save**. Your game will be at `https://YOUR-NAME.github.io/pokesurvivor-plus/`.

---

## ⚠️ The one rule

In Firebase, **NEVER click "Upgrade to Identity Platform"**, and never turn on anything that asks for it (MFA, blocking functions...). Identity Platform charges per user. You don't need it: the normal Authentication used here is free with unlimited accounts.

You also never need to enter a credit card. If Firebase asks you to "upgrade to Blaze", you clicked something you don't need: go back.

---

## 1. Create the Firebase project

1. Go to **https://console.firebase.google.com** and sign in with a Google account.
2. Click **Create a project** (or "Add project"). Give it a name, e.g. `my-pokesurvivor`.
3. Google Analytics: **turn it off** (not needed). Click **Create project** and wait.

## 2. Turn on accounts

1. Left menu: **Build → Authentication → Get started**.
2. Tab **Sign-in method**:
   - **Email/Password** → Enable → Save. (The game uses a player name + password; internally it turns it into an email that never receives anything.)
   - **Google** → Enable → pick your support email → Save. *(Optional, but nice.)*
3. Tab **Settings → Authorized domains → Add domain**: type `YOUR-NAME.github.io` (your GitHub username, no `https://`, no `/pokesurvivor-plus`). Without this, "Sign in with Google" says *unauthorized domain*.

## 3. Create the web app and get your config

1. Top left, the ⚙ gear → **Project settings**.
2. Scroll down to **Your apps** and click the web icon **`</>`**.
3. Nickname: `web`. **Don't** tick Firebase Hosting. Click **Register app**.
4. You'll see a block like `const firebaseConfig = { apiKey: "...", ... };`. Keep this page open, you'll copy it in step 6.

## 4. Create the main database (Firestore)

1. Left menu: **Build → Firestore Database → Create database**.
2. Location: pick one close to your players (e.g. `eur3` for Europe, `nam5` for the US). It can't be changed later.
3. Start in **production mode** → Create.
4. Open the **Rules** tab. Delete everything in the editor, then copy **all** of the file [`firestore.rules`](firestore.rules) from this repo (open it on GitHub, click the "Copy raw file" button) and paste it. Click **Publish**.

These rules make sure each player can only read and write **their own** save. Don't skip this step.

## 5. Create the live database (Realtime Database, for co-op and friends)

1. Left menu: **Build → Realtime Database → Create database**.
2. Pick a location (e.g. Belgium `europe-west1`) → **Start in locked mode** → Enable.
3. Open the **Rules** tab, delete everything, paste **all** of [`database.rules.json`](database.rules.json) from this repo, then click **Publish**.
4. At the top of the **Data** tab you'll see your database address, something like
   `https://my-pokesurvivor-default-rtdb.europe-west1.firebasedatabase.app`. Copy it.

## 6. Plug the server into your game

1. On GitHub, open **your** copy of the repo and click the file **`config.js`**.
2. Click the ✏️ pencil (Edit this file).
3. Replace the last line `window.PS_FIREBASE = null;` with your config from step 3, **plus** the `databaseURL` from step 5. It must look like this, with your own values:

   ```js
   window.PS_FIREBASE = {
     apiKey: 'AIza...',
     authDomain: 'my-pokesurvivor.firebaseapp.com',
     projectId: 'my-pokesurvivor',
     storageBucket: 'my-pokesurvivor.firebasestorage.app',
     messagingSenderId: '1234567890',
     appId: '1:1234567890:web:abc123',
     databaseURL: 'https://my-pokesurvivor-default-rtdb.europe-west1.firebasedatabase.app'
   };
   ```

   Keep the quotes `'...'`, the commas at the end of each line, and the `};` at the end.
4. Click **Commit changes** (green button).

These values are **not secret**: Firebase is made to put them in a web page. What protects the data are the rules from steps 4 and 5.

## 7. Done! Test it

Wait **2 to 5 minutes** (GitHub updates your site), then open `https://YOUR-NAME.github.io/pokesurvivor-plus/` and do a hard refresh (**Ctrl + F5** on PC).

On the start screen you should now be able to **create an account** or **sign in with Google**, and the menu shows **Friends** and **Leaderboard**. Send your link to your friends, add each other as friends, and start a co-op game!

---

## Updates

When I release new content: on your copy on GitHub, click **Sync fork → Update branch**. Your `config.js` is kept (I never change it), so your server keeps working.

If GitHub says there's a conflict on `config.js`, keep **your** version.

## Problems?

- **No account / Friends buttons:** `config.js` isn't filled in correctly (a missing quote or comma is enough). Open it again and compare with the example. Also wait a few minutes and do Ctrl + F5.
- **"Unauthorized domain" with Google:** step 2.3, add `YOUR-NAME.github.io` to the authorized domains.
- **Can't save / "permission denied":** the rules from step 4 or 5 weren't published. Paste them again and click Publish.
- **Co-op doesn't connect:** most of the time players connect directly to each other. On some strict networks (some schools or companies, some mobile networks) it can fail. Try another network.

## Free plan limits

Plenty for a group of friends: about 50,000 reads and 20,000 writes per day for Firestore, and 100 simultaneous connections for the Realtime Database (only players in a lobby or a co-op game count). If you ever go over, Firebase just refuses some actions until the next day. It **never charges** you unless you add a paid plan yourself.

## Privacy

With your own server, **you** host your players' data (player name, save, friends, leaderboard; the Google email if they use Google sign-in). Keep it a free, non-profit fan project: no ads, no paid access (the sprites are CC BY-NC 4.0).
