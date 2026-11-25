# Prooffolio

*Don't just list your skills. Prove them.*

An evidence-backed student portfolio, ATS-friendly resume builder and career-readiness app. Students log what they really do (assignments, practicals, events, roles, projects), attach proof, see which skills it proves for a target role, and share a portfolio, resume and QR profile card.

This is a **framework-free web app** (plain HTML, CSS and JavaScript, no build tooling required) with one small serverless function for the AI features. It runs as a website on Vercel and can be wrapped as an **Android app with Capacitor** for Google Play.

```
.
├── src/                  The web app (this is what users get)
│   ├── index.html        Entry point (hash routing: #home, #journey, #readiness ...)
│   ├── css/styles.css    Design system, animations, phone/tablet/laptop layouts
│   ├── js/               data, core, engine, qr, pages1-3, buddy, main, native, pwa, config
│   ├── assets/           Logo, robot buddy, illustrations (assets/el), icons
│   ├── vendor/pdfjs/     Self-hosted PDF.js (Apache-2.0) for reading uploaded PDFs
│   ├── privacy.html, terms.html   Legal pages (templates; filled in at build time)
│   ├── manifest.webmanifest, sw.js   Installable web app + offline shell
├── api/ai.js             Vercel serverless function: secure proxy to the Claude API
├── scripts/              build.mjs, dev.mjs, check.mjs (no dependencies)
├── resources/            Source icon and splash images for the Android app
├── capacitor.config.json Android/Capacitor configuration
├── vercel.json           Vercel build, headers (CSP etc.) and function settings
├── .env.example          Every environment variable, explained
└── docs/                 AUDIT.md (technical audit) and ANDROID_PLAY_STORE.md
```

## 1. Install and run locally

Requires **Node 22 or newer**.

```bash
npm install                # only needed for the Android tooling; the web app has no runtime dependencies
cp .env.example .env       # then edit .env (see below)
npm run dev                # http://localhost:3000, includes the /api/ai route
```

Without an API key everything works except the AI features (AI Career Buddy, AI polish/summary, "generate from a document"), which show a clear "AI is not connected" message instead of fake answers.

To enable AI locally, put your key in `.env`:

```
ANTHROPIC_API_KEY=sk-ant-...
```

Other checks: `npm run check` (syntax, file references, manifest/config validity).

## 2. Build

```bash
npm run build      # copies src/ to www/, writes www/js/config.js, fills privacy/terms placeholders
```

`www/` is a static site you can host anywhere. Build-time variables (see `.env.example`):

| Variable | Purpose |
|---|---|
| `PF_API_BASE` | Address of the server that hosts `/api/ai`. Leave empty for the website. **Required for the Android app** (e.g. `https://prooffolio.vercel.app`). |
| `PF_PUBLIC_URL` | Public https address of the website. Used in share links and QR codes. **Required for the Android app.** |
| `PF_CONTACT_EMAIL`, `PF_OPERATOR_NAME`, `PF_APP_NAME` | Shown in the legal pages; the email also powers the "Report a problem" link under AI replies. |

## 3. Deploy to Vercel

1. Push this folder to a GitHub repository.
2. In Vercel: **Add New → Project → Import** the repository. No framework preset is needed; `vercel.json` already sets the build command (`npm run build`), output folder (`www`), security headers and the API function.
3. Under **Settings → Environment Variables** add:
   - `ANTHROPIC_API_KEY` (secret, required for AI)
   - `PF_CONTACT_EMAIL`, `PF_OPERATOR_NAME` (for the legal pages)
   - optional: `ANTHROPIC_MODEL`, `ALLOWED_ORIGINS`
4. Deploy. Open `https://<your-project>.vercel.app/api/ai`; it should answer `{"configured":true}`.
5. After the first deploy set `PF_PUBLIC_URL` to your final address (or custom domain) and redeploy so share links use it.

Vercel CLI alternative: `npx vercel` then `npx vercel --prod`.

Notes: the AI route has a simple per-instance rate limit (15 requests per minute per IP). For strict limits across instances use a shared store such as Upstash Redis or Vercel KV.

## 4. Build the Android app (Capacitor)

Full step-by-step guide, including signing and Google Play submission, is in **docs/ANDROID_PLAY_STORE.md**. Short version:

```bash
# 0. In .env set PF_API_BASE and PF_PUBLIC_URL to your deployed https address, and set the permanent
#    appId in capacitor.config.json (reverse-domain name you control, e.g. com.yourname.prooffolio)
npm install
npm run cap:add          # creates android/ (once)
npm run cap:assets       # generates launcher icons and splash screens from resources/
npm run cap:sync         # builds the web app and copies it into the Android project
npm run cap:open         # opens Android Studio: run on an emulator or phone
npm run android:bundle   # release .aab for Google Play (after configuring signing)
```

## 5. What the app uses

* **No account server and no database.** Accounts and all user data live on the user's device (localStorage + IndexedDB). Passwords are salted PBKDF2-SHA256 hashes. Users export/import backups from Settings. A different device means a different, empty account unless a backup is imported.
* **One external service:** the Anthropic Claude API, reached only through `/api/ai` so the key never reaches the browser or the app.
* **Google Fonts** (Manrope, DM Sans and others) load from fonts.googleapis.com and fall back to system fonts offline. See docs/AUDIT.md for how to bundle them.
* **Self-hosted** PDF.js, a built-in QR encoder and a built-in PDF writer (no third-party runtime libraries).

## 6. Features

Landing, sign up, log in, recovery-code reset, onboarding · Home dashboard · **From Classroom to Career** (records, proof, resume lines) · **Career Readiness Analyzer** (roles or pasted job description) · **Portfolio** and **Theme Studio** · **Smart Share Card** (holographic and four other styles, QR, customisation) · **ATS Resume Builder** (real-text PDF, TXT, ATS checklist) · Skill Quest (quizzes, challenges, XP, badges) · **AI Career Buddy** (full page and a floating assistant on every page) · Opportunity Zone · Campus Spotlight (personal) · Profile and Settings (privacy switches, backup, delete account) · responsive layouts for phone, tablet and laptop · installable PWA · achievement celebrations and animated page transitions.

## 7. Known limits

* No cross-device sync, email-based password reset or campus-wide shared feeds, because there is no shared backend. Adding one (for example Supabase) is the natural next step; the data model is already a single JSON document per user.
* Cover images and large proof files stay on the device and are not included in share links.
* Fonts need an internet connection the first time (see docs/AUDIT.md for bundling).
