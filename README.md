# B Forward Motors Masaka

A real car-dealership website: public site, customer logins, cart, test-drive bookings, orders, live chat, new-car notifications, a chatbot that starts when a customer adds a car or books one, and an admin dashboard with a progression graph and visitor insights.

- **Server:** Node.js (no framework) with a PostgreSQL database
- **Pages:** plain HTML, CSS and JavaScript in `public/`
- **Colours:** blue, dark red, black, white, dark green (top of `public/css/style.css`)

## What is in the box

| Page | What it does |
|---|---|
| `/` | Public site: cars, payment methods, feedback form, company contacts, login / sign up |
| `/login` | Log in and sign up (admins go to `/admin`, customers go to `/shop`) |
| `/shop` | Customers: browse, add to cart, book a test drive, place an order, send payment reference, notifications, chat with the team, chatbot |
| `/admin` | Admin: dashboard and progression graph, upload / edit / delete cars, booked products, orders, users, messages, feedback, payment methods, contacts, insights |

How it behaves:

- **Chatbot:** hidden until a customer adds a car to the cart or books one. It answers price, payment, test drive, delivery and location questions from your real data and can pass the customer to the team.
- **Live chat and alerts:** messages, new-car notifications and order status changes appear instantly on open pages. Optional phone alerts reach people when the site is closed (see step 5).
- **Orders:** placing an order reserves the cars so two buyers can never get the same car. The customer pays by mobile money, bank or cash, sends the transaction ID, and you mark the order Paid. The car then shows as Sold.
- **Visitor insights:** after a visitor taps Allow on the banner, the site records views, searches and cart adds. Insights shows the buying funnel, popular searches, busiest hours and plain suggestions.
- **Reviews** from the home page stay hidden until you approve them.

## Put it online with Render

### 1. Put the code on GitHub

```
cd bforward-motors
git init
git add .
git commit -m "B Forward Motors website"
git branch -M main
git remote add origin https://github.com/YOUR-NAME/bforward-motors.git
git push -u origin main
```

### 2. Create the website and database

**Easiest:** in Render choose **New > Blueprint**, pick your repository and press apply. `render.yaml` creates the web service and the database and connects them. Render asks you for `ADMIN_EMAIL` and `ADMIN_PASSWORD`. Choose a password of 10 or more characters. Leave the VAPID boxes empty for now.

**By hand:**

1. New > PostgreSQL. After it is created, copy its **Internal Database URL**.
2. New > Web Service > your repository. Runtime Node. Build command `npm install`. Start command `npm start`. Health check path `/api/health`.
3. Add these environment variables:

| Name | Value |
|---|---|
| `NODE_ENV` | `production` |
| `DATABASE_URL` | the Internal Database URL |
| `SESSION_SECRET` | a long random text (32 or more characters) |
| `ADMIN_EMAIL` | your email |
| `ADMIN_PASSWORD` | 10 or more characters |

The tables are created automatically the first time the server starts.

### 3. Log in

Open your Render address, go to `/login` and use `ADMIN_EMAIL` and `ADMIN_PASSWORD`. Then:

1. **Payment & contacts:** add your phone, WhatsApp number (with country code, for example `256700123456`), email, address, opening hours and each payment method you accept.
2. **Products:** upload your first car with a photo. Leave "Notify all customers" ticked.
3. Change your password under Payment & contacts. The `ADMIN_PASSWORD` variable is only used the first time to create the account. Later changes made in the dashboard are kept.

If you ever forget the admin password: set `ADMIN_RESET_PASSWORD=true` and a new `ADMIN_PASSWORD`, restart, log in, then delete `ADMIN_RESET_PASSWORD`.

### 4. Your own domain

Render > your web service > Settings > Custom Domains. Add the domain and follow the DNS steps. HTTPS is automatic.

### 5. Phone alerts when the site is closed (optional)

1. On your computer run `npx web-push generate-vapid-keys`.
2. In Render add `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY` and `VAPID_SUBJECT` (for example `mailto:you@example.com`).
3. Redeploy. Customers see "Turn on phone and browser alerts" in their notification panel. You see "Turn on alerts on this device" on the dashboard so you hear about new orders, bookings and chats.

On iPhone, alerts only work after the person adds the website to their Home Screen.

## Costs and plans (check Render's pricing page, it changes)

- **Web service:** a free instance goes to sleep after a period with no visitors, so the first visit is slow and live chat is not running while it sleeps. A paid instance stays awake. Use a paid instance for a real business.
- **Database:** Render's free database is deleted after 30 days. Do not use it for real customers. Use a paid Render database (what `render.yaml` asks for), or a free database from Neon or Supabase: create it there, copy its connection string into `DATABASE_URL` and delete the database block from `render.yaml`.
- If Render rejects the plan names in `render.yaml`, change `plan:` to a name shown in your Render dashboard.

## Run it on your own computer

```
npm install
cp .env.example .env     # then fill it in
```

Create an empty Postgres database, set `DATABASE_URL`, and start with your variables loaded:

```
export $(grep -v '^#' .env | xargs)
npm start
```

Open http://localhost:3000.

Run the automated tests against a **throwaway** database (they delete all data in it):

```
TEST_DATABASE_URL=postgres://user:pass@localhost:5432/bf_test npm test
```

## What is not included (decide before you promote the site)

- **Automatic mobile money payments.** Payments are confirmed by you: the customer pays, sends the transaction ID, and you press Paid. MTN MoMo, Airtel Money, Flutterwave or Pesapal can be connected later, but each needs your own merchant account and keys.
- **Password reset by email.** The admin resets a customer's password in Users. Adding email needs an email service such as Resend or Brevo.
- **A smarter chatbot.** The assistant uses fixed rules and your real prices and payment details. It does not understand free-form questions. The team chat is always there for those.
- **More than one server.** Live chat connections live in the server's memory, so run one web instance. Scaling to several needs a shared message layer.
- **Privacy policy page.** The site asks permission before tracking, but you should publish a privacy policy that fits Uganda's Data Protection and Privacy Act, 2019.

## Project layout

```
server/index.js        web server, security headers, static files
server/routes/         API: public.js, auth.js, shop.js, admin.js
server/schema.sql      database tables (created automatically)
server/realtime.js     live chat and notifications (server-sent events)
server/push.js         phone alerts (optional)
public/                the pages and their scripts
test/e2e.js            automated test of the whole API
render.yaml            one-click Render setup
```
