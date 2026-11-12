# News App 📰

**A cross-platform news reader built with Flutter and a Node.js backend. Read top headlines, search articles, bookmark your favorites and get real-time notifications.**

The Flutter app talks to a Node.js, Express and MongoDB backend. The backend pulls headlines from [NewsAPI.org](https://newsapi.org/), stores them in MongoDB, handles user accounts with JWT, and pushes live updates over Socket.io.

> Built for the **ITT632 Mobile Cloud Computing** course. UiTM, 2025.

<!-- Add screenshots here, for example:
<table align="center">
  <tr>
    <td align="center"><img src="docs/screenshots/headlines.png" alt="Headlines" width="200"><br><em>Headlines</em></td>
    <td align="center"><img src="docs/screenshots/search.png" alt="Search" width="200"><br><em>Search</em></td>
    <td align="center"><img src="docs/screenshots/bookmarks.png" alt="Bookmarks" width="200"><br><em>Bookmarks</em></td>
  </tr>
</table>
-->

---

## Features

**News**
- Top headlines from NewsAPI.org, loaded in pages with infinite scroll.
- Keyword search.
- Filter by category (by news source).
- Article detail page with the image, summary and a link to the full story in your browser.
- Cached headlines and images, so recent articles still show when you are offline (cache lasts 24 hours).
- Light and dark themes.

**Accounts and bookmarks**
- Sign up and log in with email and password. Passwords are hashed with bcrypt, and sessions use JWT tokens that last 7 days.
- The app remembers you and logs you in automatically next time.
- Bookmark articles, view your saved list and remove items. Bookmarks belong to your account and are stored in MongoDB.

**Real-time notifications**
- A Socket.io connection delivers live notifications, such as when new articles are synced or you bookmark an article.
- Each user has a private notification channel. The connection reconnects automatically.
- Notifications slide in, stack, and dismiss themselves after 5 seconds or when you tap close.

## Architecture

```
Flutter app  ──HTTP (REST)──►  Node.js + Express  ──►  MongoDB
     ▲                               │
     └──── Socket.io (live) ◄────────┤
                                     └──►  NewsAPI.org (headline sync)
```

### Tech stack

| Area | Technology |
| :--- | :--- |
| App | Flutter and Dart: `http`, `socket_io_client`, `shared_preferences`, `cached_network_image`, `flutter_dotenv`, `url_launcher` |
| Backend | Node.js, Express, Mongoose |
| Database | MongoDB |
| Auth | JWT (`jsonwebtoken`) and bcrypt (`bcryptjs`) |
| Real time | Socket.io |
| Data source | NewsAPI.org |

### Project structure

```
newsapp/
├── lib/                      # Flutter app
│   ├── main.dart             # App start, headlines, search, bookmarks entry point
│   ├── config.dart           # Reads the backend URL and settings from .env
│   ├── news_api_service.dart # REST client for the backend
│   ├── cache_manager.dart    # Offline cache (24 hours)
│   ├── article.dart          # Article model
│   ├── article_detail_page.dart
│   ├── theme.dart            # Light and dark themes
│   ├── screens/              # Login, sign up and bookmarks screens
│   ├── services/             # Auth and notification (Socket.io) services
│   └── widgets/              # Notification overlay
├── backend/                  # Node.js API
│   ├── server.js             # Routes, models, auth and Socket.io
│   └── .env.example          # Backend settings template
├── .env                      # App settings (backend URL, timeouts)
└── pubspec.yaml
```

### API

| Method | Endpoint | Auth | Description |
| :--- | :--- | :--- | :--- |
| `POST` | `/api/auth/signup` | no | Create an account |
| `POST` | `/api/auth/login` | no | Log in and get a token |
| `GET` | `/api/news/top-headlines` | no | Paged headlines |
| `GET` | `/api/news/search?q=` | no | Search articles |
| `GET` | `/api/news/category/:category` | no | Articles for one source or category |
| `GET` | `/api/news/article/:id` | no | One article |
| `GET` | `/api/news/all` | no | All stored articles |
| `POST` | `/api/news/sync` | no | Pull the latest headlines from NewsAPI |
| `GET` | `/api/bookmarks` | yes | Your bookmarks |
| `POST` | `/api/bookmarks` | yes | Bookmark an article |
| `DELETE` | `/api/bookmarks/:bookmarkId` | yes | Remove a bookmark |
| `GET` | `/api/health` | no | Health check |

Authenticated requests send `Authorization: Bearer <token>`. The backend also has article add, update and delete routes (`/api/news/add`, `PUT` and `DELETE /api/news/:id`) for development use.

**Socket.io events:** the client sends `join-notifications` with a user ID. The server sends `article-update` to everyone and `bookmark-update` to the user's own room.

## Getting started

### Prerequisites
- [Flutter SDK](https://docs.flutter.dev/get-started/install) (Dart 3.11 or newer)
- [Node.js](https://nodejs.org/) 18+ and npm
- [MongoDB](https://www.mongodb.com/try/download/community) running locally, or a MongoDB Atlas connection string
- A free API key from [NewsAPI.org](https://newsapi.org/register)

### 1. Start the backend

```bash
cd backend
npm install
```

Copy the example settings and fill them in:

```bash
cp .env.example .env          # on Windows PowerShell: Copy-Item .env.example .env
```

| Variable | Purpose |
| :--- | :--- |
| `PORT` | Server port (default `3000`) |
| `MONGODB_URI` | MongoDB connection, for example `mongodb://localhost:27017/news_app` |
| `JWT_SECRET` | A long random secret used to sign login tokens. **Set your own.** |
| `NEWSAPI_KEY` | Your NewsAPI.org key |
| `NEWSAPI_BASE_URL` | `https://newsapi.org/v2` |
| `NEWSAPI_COUNTRY` | Country for headlines, for example `us` |
| `ENABLE_SAMPLE_DATA` | Set to `true` to fill an empty database with headlines on start |
| `CORS_ORIGIN` | Allowed web origin (defaults to any origin) |

Never commit your real `.env` file. Then start the server:

```bash
npm run dev      # restarts on changes (nodemon)
npm start        # normal start
```

If the database is empty and `ENABLE_SAMPLE_DATA=true`, the first start loads headlines automatically. You can also load them any time with `POST http://localhost:3000/api/news/sync`.

### 2. Run the Flutter app

From the project root:

```bash
flutter pub get
flutter run
```

The app reads `.env` in the project root:

```env
BACKEND_URL=http://localhost:3000/api
```

Use the address that matches where the app runs:

| Where the app runs | `BACKEND_URL` |
| :--- | :--- |
| Chrome or desktop | `http://localhost:3000/api` |
| Android emulator | `http://10.0.2.2:3000/api` |
| A physical phone | `http://<your-computer-IP>:3000/api` (same Wi-Fi network) |

## Limitations

- The article add, update, delete and sync routes are not protected by login. They are meant for local development, so protect or remove them before any public deployment.
- Headlines are copied from NewsAPI when you sync, so the app does not refresh on its own after the first load. The free NewsAPI plan is intended for development and has request limits, so check their terms.
- Category filtering uses the article's news source rather than a true topic category.
- There are no automated tests yet beyond the Flutter template test.

## License

Copyright (c) 2026 malsaaaa. All rights reserved.

This repository is public for viewing and portfolio purposes. No license is granted to copy, modify, distribute or use the code without the author's written permission.
