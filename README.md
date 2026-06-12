# Ashu's Productivity Dashboard

A secure, private, and lightweight productivity dashboard for managing university tasks, deadlines, and schedule focus blocks.

This project is built as a **Bring Your Own Backend (BYOB)** single-page application. It stores your tasks in a private, hidden folder on your own Google Drive and communicates directly with your Google Calendar to sync deadlines and write scheduled focus blocks. 

*Zero databases to manage, zero hosting costs, and absolute data privacy.*

---

## Architecture Overview

```
                     ┌──────────────────┐
                     │   Your Browser   │
                     │  (Dashboard UI)  │
                     └─┬──────────────┬─┘
                       │              │
                       ▼              ▼
             ┌─────────────────┐    ┌─────────────────┐
             │  Google Drive   │    │ Google Calendar │
             │  AppData Folder │    │   Events API    │
             │  (tasks.json)   │    │ (Read Deadlines │
             │                 │    │  Write Blocks)  │
             └─────────────────┘    └─────────────────┘
```

* **Frontend**: Pure HTML5, CSS3, and JavaScript served locally.
* **Authentication**: Google Identity Services (OAuth 2.0 Client-side Flow).
* **Storage**: Read/write access to Google Drive's hidden `appDataFolder` to store your personal `tasks.json` data.
* **Calendar Integration**: Google Calendar API to automatically sync deadlines/exams and schedule time-blocks.

---

## Quick Start (Local Run)

Google OAuth requires the dashboard to be loaded from an HTTP server (`http://localhost:8000`) rather than directly opening the file in your browser (`file://...`).

1. Open your terminal in this directory.
2. Run a simple local HTTP server:
   ```bash
   npx -y http-server -p 8000
   ```
3. Open **[http://localhost:8000](http://localhost:8000)** in your web browser.
4. Keep the server running in the background while using the dashboard.

---

## Detailed Google Cloud Integration Setup

To connect Google Calendar & Google Drive, you need to create your own free OAuth credentials in the Google Cloud Console.

### Step 1: Create a Google Cloud Project
1. Open the [Google Cloud Console](https://console.cloud.google.com/).
2. Click the project dropdown in the top-left corner and click **New Project**.
3. Name it `Ashu Dashboard` and click **Create**.

### Step 2: Enable APIs
You need to enable access to Google Calendar and Google Drive for this project:
1. In the top search bar, search for **Google Calendar API** and click **Enable**.
2. Next, search for **Google Drive API** and click **Enable**.

### Step 3: Configure the OAuth Consent Screen
1. Go to **APIs & Services** &rarr; **OAuth consent screen** in the sidebar.
2. Select **External** as the user type and click **Create**.
3. Fill in the basic application info:
   * **App name**: `Ashu Dashboard`
   * **User support email**: *Your Gmail address*
   * **Developer contact information**: *Your Gmail address*
4. Click **Save and Continue**.
5. In the **Scopes** tab, click **Add or Remove Scopes**. Add the following scopes:
   * `https://www.googleapis.com/auth/calendar.events` (To read deadlines and write new events)
   * `https://www.googleapis.com/auth/drive.appdata` (To store `tasks.json` in your private Drive folder)
6. In the **Test users** tab, click **Add Users** and add your Gmail address. *Note: Only email addresses added here can log into your application while it's in testing mode.*
7. Click **Save and Continue**.

### Step 4: Create OAuth Client Credentials
1. Go to **APIs & Services** &rarr; **Credentials** in the sidebar.
2. Click **Create Credentials** at the top and select **OAuth client ID**.
3. Set **Application type** to **Web application**.
4. In the **Authorized JavaScript origins** section, click **Add URI** and enter:
   `http://localhost:8000`
5. Click **Create**.
6. Copy the generated **Client ID** (it looks like `xxxxxxxx-xxxxxxxx.apps.googleusercontent.com`).

### Step 5: Save & Connect the Dashboard
1. Go to your dashboard at `http://localhost:8000`.
2. Click the **Settings** gear icon in the top right.
3. Paste your copied **Client ID** into the settings field and click **Save & Connect**.
4. Log into your Google Account through the popup window and consent to the permissions.
5. The connection state will change from **Local Mode** to **Google Synced**!

---

## 🌐 Deploying as a Public Website (Plug-and-Play Login)

You can host this dashboard online so that you (and anyone else) can access it from anywhere by simply clicking **Connect Google Account**—without having to do any Google Cloud Console configuration!

Because it is a frontend-only app, user data stays 100% private. Their tasks are saved in *their own* Google Drive, and their deadlines are fetched from *their own* Google Calendar.

### Setup Steps:

1. **Deploy the Files**:
   * Upload `index.html` to a free static hosting service like GitHub Pages, Vercel, Netlify, or your own server.
   * Note the public URL (e.g., `https://your-username.github.io/dashboard`).

2. **Register a Single Google OAuth Client ID**:
   * Open the [Google Cloud Console](https://console.cloud.google.com/).
   * Select your project, go to **Credentials**, and edit or create a **Web application** Client ID.
   * Under **Authorized JavaScript origins**, add your public URL (e.g., `https://your-username.github.io`).
   * Copy the generated **Client ID**.

3. **Configure the Default Client ID in Code**:
   * Open `index.html`.
   * Find `const DEFAULT_CLIENT_ID = '';` near the top of the script block (around line 958).
   * Paste your Client ID inside the quotes:
     ```javascript
     const DEFAULT_CLIENT_ID = 'your-client-id-here.apps.googleusercontent.com';
     ```
   * Save and redeploy.

Now, anyone visiting your website can sign in directly!

---

## Features

### 📅 Sync Calendar & Fetch Deadlines
Clicking **Sync Calendar** searches your Google Calendar events for the next 30 days. It filters events matching keywords such as `exam` (Klausur/Prüfung), `deadline` (Frist/Due), and `submission` (Abgabe) to automatically build your deadlines view.

### ⏰ Write Focus Blocks (Block Time)
Click **Block time** next to any deadline. A scheduler modal will open. You can edit the focus block name, set the date (defaults to the day before), select your start/end times, and click **Schedule on Google**. The app will create a 2-hour event directly in your Google Calendar.

### 📝 Horizontal Task Boards
Manage tasks for **TU Braunschweig** and **TU Berlin** across four status horizons:
* **Today**
* **Next up**
* **Pending**
* **Backlog**

---

## Expanding for Claude (Model Context Protocol / MCP)

If you want Claude (e.g. Claude Desktop) to read your tasks and help check things off, we can create a companion **MCP (Model Context Protocol) Server** in this directory. 

Claude can load this server locally via stdin/stdout, read the `tasks.json` file synced to your local machine, and let you complete tasks directly from your chat prompt:
> *"Claude, what are my tasks for TU Berlin today?"*
> *"Claude, I finished planning work for Silas, check it off."*
