# Video Wave

Video Wave is a lightweight TikTok-like web application backend with a minimal frontend for testing. It provides user authentication, video upload and publishing, an owner moderation dashboard, follow system, and simple feeds.

Features:
- User registration & login (JWT)
- Seeded owner account: username `videowave` password `owner2024`
- Owner has special endpoints to approve/reject videos and verify users (blue badge)
- Video upload (MP4/WEBM/MOV/MKV) with basic anti-bot heuristics
- Uploaded videos go into a moderation queue (pending) unless the uploader is verified or owner
- No "free followers": followers must actively follow users
- FYP feed (random approved videos) and Following feed (videos from people you follow)
- Simple like and view tracking
- Media served from local uploads directory under `/media`

Prerequisites:
- Node.js 18+ (recommended)
- npm (included with Node)
- MongoDB (local or Atlas). Example local connection string: `mongodb://localhost:27017/videowave`
- Optional: ffmpeg (not required; thumbnail uses picsum placeholder)

Files & structure (key):
- server.js - main entry
- models/ - Mongoose models (User, Video, Follow)
- routes/ - API routes for auth, videos, owner moderation
- middleware/auth.js - JWT verification
- utils/initOwner.js - seeds owner account
- uploads/ - directory where uploaded files are stored (auto-created)
- .env.example - environment variables template

Installation & setup:

1. Clone or download this repository.
2. Copy environment template:
    cp .env.example .env
3. Edit `.env` to set your MongoDB URI and JWT secret. Example .env:
    MONGODB_URI=mongodb://localhost:27017/videowave
    JWT_SECRET=your_strong_secret_here_at_least_32_chars
    PORT=5000
    CLIENT_URL=http://localhost:5173
    UPLOAD_DIR=uploads
    MAX_VIDEO_MB=200

4. Install dependencies:
    npm install

5. Start MongoDB if running locally. For example (on macOS with brew):
    brew services start mongodb-community

6. Seed owner account (optional; server also seeds on start):
    npm run seed

7. Run the server:
    npm run dev
   or for production:
    npm start

The API will be available at `http://localhost:5000` by default.

Usage examples:

- Register a new user:
    POST /api/auth/register
    Content-Type: application/json
    Body:
        {
            "username": "alice",
            "password": "s3cureP@ss",
            "displayName": "Alice"
        }

- Login:
    POST /api/auth/login
    Body:
        {
            "username": "alice",
            "password": "s3cureP@ss"
        }
    Response contains `token` (JWT). Use `Authorization: Bearer <token>` header for protected endpoints.

- Upload a video (multipart form):
    POST /api/videos/upload
    Headers:
        Authorization: Bearer <token>
    Form-data:
        video: <file.mp4> (field name `video`)
        title: "My first clip"
        description: "Fun moment"
    Note: Unverified users' uploads go to `pending`. Verified users (and owner) get auto-approved.

- Owner moderation (owner must login as `videowave`):
    GET /api/owner/pending-videos
    POST /api/owner/approve/:videoId
    POST /api/owner/reject/:videoId  (JSON body: { "reason": "..." })
    POST /api/owner/verify-user/:userId

- Feeds:
    GET /api/videos/fyp?limit=12
    GET /api/videos/following?limit=20  (requires auth)
    GET /api/videos/:id
    POST /api/videos/:id/like  (requires auth)

- Follow:
    POST /api/auth/follow/:targetId  (requires auth)
    POST /api/auth/unfollow/:targetId (requires auth)

Media access:
- Uploaded videos are available at `/media/<filename>`. Thumbnails are generated as placeholder images.

Security & validation:
- Passwords are hashed with bcrypt.
- JWT tokens are used for auth; set `JWT_SECRET` to a strong secret.
- Multer enforces file size limit (configured by `MAX_VIDEO_MB`) and allowed mime types.
- Basic heuristics detect suspicious uploads (very small files, filenames containing "bot/auto") and reject them automatically.
- Owner privileges enforced via JWT role claim.

Troubleshooting:

- "Missing token" or "invalid token": ensure you include Authorization header: `Authorization: Bearer <token>`.
- "Video file is required": ensure you send the file under form field name `video`.
- "Uploads folder permission error": ensure the process can create the `uploads` directory or set `UPLOAD_DIR` to a writeable path.
- MongoDB connection failure: verify `MONGODB_URI` in `.env` and that MongoDB is running.
- JWT_SECRET not set: the app will run with a dev secret but you MUST set a strong secret for production.

Notes & limitations:
- This project provides a backend and minimal API centered approach. It includes a minimal redirect at `/` and serves media under `/media`.
- No full SPA client is bundled; you can build a client that consumes the API endpoints (React/Vue/Svelte).
- Thumbnail generation uses picsum placeholders to avoid adding ffmpeg runtime dependency. If you have ffmpeg and want real thumbnails, you can integrate fluent-ffmpeg and extract frames.

Project is intentionally compact and focused on essential functionality: authentication, video upload with moderation, owner controls, and follow system. Enjoy building Video Wave!
