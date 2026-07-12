# ❓ Frequently Asked Questions (FAQ)

This document answers common questions from new contributors and users of **Aarogya**. If your question isn't answered here, please open an issue.

---

## 📖 General

### What is Aarogya?
Aarogya is a web-based healthcare platform built with Node.js/Express and EJS. It lets patients browse doctors and book appointments, gives doctors a dashboard to manage bookings, and includes an AI health chatbot / report-analysis prototype. See the [README](./README.md) for the full feature list.

### What tech stack does it use?
- **Frontend:** HTML, CSS, JavaScript, EJS templates (`views/`)
- **Backend:** Node.js + Express (`server.js`)
- **Database:** MongoDB (via Mongoose), typically hosted on MongoDB Atlas
- **Auth:** JWT stored in an HTTP cookie, passwords hashed with `bcryptjs`
- **AI:** the `openai` npm package (`/chat` route, `gpt-3.5-turbo`) and an `axios` call to a Hugging Face inference endpoint (`/analyze-health` route)

### Is the AI functionality "real" or simulated?
It's a mix, and this trips people up:
- `POST /chat` genuinely calls the OpenAI Chat Completions API using `OPENAI_API_KEY`.
- `POST /analyze-health` (used by the health-report-analysis feature) calls a Hugging Face inference endpoint (`mistralai/Mistral-7B-Instruct-v0.1`), **but it authenticates with your `OPENAI_API_KEY`**, not a Hugging Face token. Because of this mismatch, that endpoint will typically return an API error even with a valid OpenAI key — this is a known rough edge in the current code, not something wrong with your setup.

### Is there a live demo?
The README references hosting on Render.com, but no public demo URL is documented in this repo at the time of writing. Run the project locally to try it out (see below).

---

## 🛠️ Setup & Installation

### What do I need installed before I start?
- Node.js v14+ and npm
- A MongoDB instance — either a free [MongoDB Atlas](https://www.mongodb.com/atlas) cluster or a local `mongod`
- (Optional, only if you want AI features to work) an OpenAI API key

### How do I run the project locally?
```bash
git clone https://github.com/<your-fork>/Aarogya_ECSoC.git
cd Aarogya_ECSoC/Aarogya
npm install
```
Create a `.env` file inside the `Aarogya/` folder (see next question for the variables), then:
```bash
npm start        # runs `node server.js`
# or, for auto-restart on file changes:
npm run dev       # runs `nodemon server.js`
```
Visit `http://localhost:5000` (or whatever `PORT` you set).

> Note the project lives in the `Aarogya/` subfolder of this repository, not the repo root — run `npm install`/`npm start` from inside `Aarogya/`, not from the top level.

### What environment variables does the app actually need?
The `.env` example in the top-level README lists `MONGODB_URI`, but the code (`server.js`) actually reads `MONGO_URL`. Use the variable names below, which match what `server.js` reads via `process.env`:

```env
PORT=5000
MONGO_URL=your_mongodb_connection_string
JWT_SECRET=your_jwt_secret
OPENAI_API_KEY=your_openai_api_key
```
- `PORT` is optional and defaults to `5000` if omitted.
- `MONGO_URL` is required — the server will fail to connect to the database without it.
- `JWT_SECRET` is required for login sessions to work.
- `OPENAI_API_KEY` is only needed if you want the `/chat` (and, best-effort, `/analyze-health`) AI routes to respond; the rest of the app runs fine without it.

### `npm run dev` fails / `nodemon: command not found`
`nodemon` isn't listed in `package.json`'s dependencies. Install it globally or as a dev dependency:
```bash
npm install -g nodemon
# or
npm install --save-dev nodemon
```
Alternatively, just use `npm start` (plain `node server.js`), which doesn't require `nodemon`.

### MongoDB won't connect
- Double-check the variable is named `MONGO_URL` in your `.env` (see above), not `MONGODB_URI`.
- If you're using Atlas, confirm your current IP is added to the cluster's network access / IP allow-list.
- Confirm the connection string includes your database name and correct credentials.

### Login/signup isn't working / "Cannot read properties of undefined"
- Make sure `JWT_SECRET` is set in `.env` — without it, token signing/verification will throw.
- There is no seed data or default test account in this repo. Create an account yourself via the `/signup` page, choosing a role of **patient** or **doctor** (see `models/User.js`) — you can't log in with an account that hasn't been created first.

### Are there existing test accounts I can log in with?
No. The app has no seed script and no demo accounts baked into the database. Register through `/signup` to create your own patient or doctor account.

### Why do I see unused-looking files like `routes/authRoutes.js` and `routes/dashboardRoutes.js`?
They exist in the repo but currently aren't `require()`'d or mounted in `server.js` — all active routes (`/login`, `/signup`, `/dashboard`, `/doctors`, `/chat`, etc.) are defined directly inside `server.js`. If you're looking for where a route's logic lives, check `server.js` first.

### Is there a test suite I can run?
Not yet — `npm test` currently just prints an error and exits (`"Error: no test specified"`). There's no automated test suite in this repo at the moment.

---

## 🤝 Contributing

### How do I contribute?
1. Fork the repository.
2. Create a feature/fix branch off `main` (e.g. `git checkout -b feature/my-change`).
3. Make your changes inside the `Aarogya/` folder.
4. Commit with a clear message and push to your fork.
5. Open a Pull Request against `Princess-2002/Aarogya_ECSoC`'s `main` branch, describing what you changed and why.

### Should I comment on an issue before submitting a PR?
There's no formal "must be assigned first" rule documented in this repo, but it's good practice to comment on the issue you're picking up so effort isn't duplicated with other contributors working through the same issue list.

### What should a good PR include?
- A focused, single-purpose change (avoid bundling unrelated fixes together).
- A description of what changed and, for bug fixes, how to reproduce the original problem.
- For UI changes, a screenshot or short clip if practical.

### Where do I ask questions that aren't covered here?
Open a new issue in this repository, or see the Support section in the [README](./README.md).

---

## 📦 Project Structure Quick Reference

```
Aarogya/
├── public/       # static assets: css, js, images, standalone html
├── views/        # EJS templates rendered by server.js
├── routes/       # auth/dashboard route modules (currently unused, see FAQ above)
├── models/       # Mongoose schemas (User, Appointment)
├── server.js     # app entry point — most routes are defined here
├── package.json
└── README.md
```
