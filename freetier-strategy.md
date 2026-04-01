## ✅ Ringkasan singkat  
**Goal:** Buat aplikasi *multi‑user* yang **realtime** (chat, notifikasi, feed, dsb.) dengan **layanan “free‑tier”** saja, sehingga tidak mengeluarkan biaya apa‑pun di awal.  

Berikut saya susun menjadi 5 bagian:  

1. **Arsitektur & layanan free‑tier yang cocok**  
2. **Stack (Node JS + frontend) + kode contoh**  
3. **Pemberitahuan (Web‑Push + FCM) yang “nyata”**  
4. **Deploy, CI/CD & monitoring**  
5. **Tips skalabilitas & batasan free‑tier**  

Setiap bagian dilengkapi dengan **link sumber** (jumlah baris < 10) yang membuktikan kuota free‑tier.

---

## 1️⃣ Arsitektur “free‑tier” yang dapat dipakai sekarang

| Komponen | Layanan free‑tier yang direkomendasikan | Kenapa dipilih? | Kuota / batas utama* |
|----------|----------------------------------------|----------------|----------------------|
| **VPS / runtime** | **Fly.io** – 3 shared VMs, 3 GB storage, 160 GB outbound bandwidth. <br> *Dukungan native WebSocket (tanpa proxy)*. | Gratis, **global (35 region)**, tidak ada “sleep after 15 min” seperti Render. | 3 VM, 10 000 concurrent WS per instance (konfigurasi `type = "connections"`). Lihat contoh config di bawah. |
| **Message broker (pub/sub)** | **Upstash Redis** – serverless, 10 000 concurrent connections, 500 k perintah/bulan, 256 MB data. <br> *HTTP‑only API, jadi mudah dipasang ke Fly atau server lain.*【1†L33-L49】 | Gratis, sangat cocok untuk **socket.io‑redis‑adapter** sehingga banyak instance Fly dapat berbagi channel. |
| **Database (persistensi)** | **Supabase** (Postgres + Auth + Realtime) **Free plan** – 500 MB storage, 5 000 row‑read‑capacity, 200 concurrent WS (Realtime)【4†L15-L19】 | Semua‑in‑one (SQL, auth, realtime). Jika butuh lebih banyak koneksi, gunakan **PostgreSQL** di **Render** (free DB) atau **Neon Serverless** (free tier). |
| **Notifikasi push** | **Firebase Cloud Messaging (FCM)** – unlimited push ke Android, iOS, Web (pakai VAPID). <br> **Web‑Push (library `web-push`)** – gratis, gunakan VAPID keys. | Gratis, API HTTP, tidak butuh server tambahan. |
| **Static assets** | **Render** (static‑site free) atau **Cloudflare Pages** – CDN global, TLS otomatis. | Hanya static, jadi file HTML/JS/CSS dapat di‑serve cepat. |
| **CI/CD** | **GitHub Actions** → Docker → `flyctl deploy`. | Gratis untuk repo publik / private (batas 2 000 menit/bulan). |
| **Observability** | **Prometheus** (self‑hosted di Fly) + **Grafana Cloud** (free tier 50 k samples). | Gratis, dapat memantau *ws‑connections, latency, redis‑lag*. |

\*Catatan: limit‑limit di atas bersifat *circa*; Anda tetap perlu memantau penggunaan lewat dashboard masing‑masing layanan.

### Diagram alur (teks)

```
[Browser] ──WSS──► Fly.io (Node + Socket.io) ◄─► Upstash Redis (pub/sub)
      │                              │
      │                              └─► Supabase Postgres (data & auth)
      │
      └─► Service‑Worker (Web‑Push) ◄─► FCM / web‑push endpoint (HTTPS)
```

- **Frontend** meng‑hubungkan ke **WebSocket** (`wss://api.myapp.com/ws`).  
- **Node** menerima event, menyimpannya ke **Supabase**, lalu **publish** ke **Redis** (agar semua instance Fly menerima).  
- **Redis** men‑broadcast ke semua socket.io client.  
- **Notifikasi**: ketika ada event baru, selain `socket.emit`, server juga mengirim **push** ke semua *subscription* yang tersimpan di Postgres.  

---

## 2️⃣ Stack & contoh kode

### 2.1 Backend – Node JS (Express + Socket.io)

**File `src/server.js`**

```js
// src/server.js
import express from "express";
import http from "http";
import { Server } from "socket.io";
import { createAdapter } from "@socket.io/redis-adapter";
import { Redis } from "@upstash/redis";
import cors from "cors";
import { json } from "body-parser";
import webpush from "web-push";
import pg from "pg";

// ---- ENV ----
const {
  PORT = 8080,
  REDIS_URL,           // upstash: "rediss://:<token>@...."
  JWT_SECRET,
  VAPID_PUBLIC_KEY,
  VAPID_PRIVATE_KEY,
  SUPABASE_URL,
  SUPABASE_SERVICE_ROLE_KEY,
} = process.env;

// ---- Express & HTTP ----
const app = express();
app.use(cors({ origin: "*" }));
app.use(json());

const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: "*" },
  path: "/ws",               // optional
  transports: ["websocket"], // force WS (optional)
});

// ---- Redis adapter (pub/sub) ----
const redis = new Redis({ url: REDIS_URL });
io.adapter(createAdapter(redis, redis));

// ---- Web Push config ----
webpush.setVapidDetails(
  "mailto:admin@myapp.com",
  VAPID_PUBLIC_KEY,
  VAPID_PRIVATE_KEY
);

// ---- Supabase client (plain pg) ----
const pgPool = new pg.Pool({
  connectionString: `${SUPABASE_URL}?apikey=${SUPABASE_SERVICE_ROLE_KEY}`,
});

// ----- Helper: JWT verification (simplified) -----
import jwt from "jsonwebtoken";
function authMiddleware(socket, next) {
  const token = socket.handshake.auth?.token;
  if (!token) return next(new Error("missing token"));
  try {
    const payload = jwt.verify(token, JWT_SECRET);
    socket.data.user = payload;   // attach ke socket
    return next();
  } catch (e) {
    return next(new Error("invalid token"));
  }
}
io.use(authMiddleware);

// ----- Socket.io events -----
io.on("connection", (socket) => {
  console.log(`🔗 ${socket.id} ⇢ user ${socket.data.user.id}`);

  // contoh “join room” (misal channel per grup)
  socket.on("join", (room) => {
    socket.join(room);
    socket.emit("joined", room);
  });

  // contoh chat message
  socket.on("msg", async ({ room, text }) => {
    const msg = {
      from: socket.data.user.id,
      room,
      text,
      ts: Date.now(),
    };

    // 1️⃣ Simpan ke Postgres (Supabase)
    await pgPool.query(
      `INSERT INTO messages (user_id, room, content, created_at) VALUES ($1,$2,$3,now())`,
      [msg.from, room, text]
    );

    // 2️⃣ Broadcast via Redis → semua instance
    io.to(room).emit("msg", msg);

    // 3️⃣ Push‑notification ke device (Web‑Push)
    const subs = await pgPool.query(`SELECT endpoint, keys FROM push_subs`);
    for (const row of subs.rows) {
      const sub = {
        endpoint: row.endpoint,
        keys: {
          p256dh: row.keys->>'p256dh',
          auth: row.keys->>'auth',
        },
      };
      const payload = {
        title: `💬 Pesan baru di ${room}`,
        body: `${msg.text.substring(0, 60)}…`,
        url: `${process.env.FRONTEND_URL}/room/${room}`,
      };
      webpush.sendNotification(sub, JSON.stringify(payload)).catch(() => {});
    }
  });

  socket.on("disconnect", () => console.log(`❌ ${socket.id} left`));
});

// ----- HTTP API untuk Web‑Push subscription -----
app.post("/api/subscribe", async (req, res) => {
  const { endpoint, keys } = req.body; // from client
  // Simpan di Supabase (atau Postgres biasa)
  await pgPool.query(
    `INSERT INTO push_subs (endpoint, keys) VALUES ($1,$2) ON CONFLICT DO NOTHING`,
    [endpoint, JSON.stringify(keys)]
  );
  res.sendStatus(201);
});

server.listen(PORT, () => {
  console.log(`🚀 Server listen on ${PORT}`);
});
```

**Catatan penting**

- **Redis adapter**: memakai *Upstash* (HTTP‑Redis). Karena hanya `GET/SET` plus pub/sub, latency tetap < 10 ms untuk beban kecil.  
- **JWT** – gunakan Supabase Auth atau Firebase Auth untuk menghasilkan token di frontend.  
- **Web‑Push** – library `web-push` meng‑irimkan VAPID‑signed request ke browser.  
- **Persistensi** – semua pesan disimpan di Supabase `messages` table.  

---

### 2.2 Front‑end (React + Vite)

**Struktur folder**  

```
src/
 ├─ index.jsx
 ├─ socket.js      // socket.io client wrapper
 ├─ push.js        // Web‑Push registration
 └─ components/
      ├─ ChatRoom.jsx
      └─ NotificationProvider.jsx
```

**`src/socket.js`**

```js
import { io } from "socket.io-client";

const token = localStorage.getItem("access_token"); // JWT dari Supabase/Auth
export const socket = io(import.meta.env.VITE_API_URL, {
  path: "/ws",
  transports: ["websocket"],
  auth: { token },
});

// optional: auto‑reconnect exponential back‑off (default built‑in)
socket.on("connect", () => console.log("✅ WS connected"));
socket.on("disconnect", (reason) => console.warn("❌ WS disconnected:", reason));
export default socket;
```

**`src/push.js`** – registrasi Service Worker + subscription

```js
export async function registerPush() {
  if (!("serviceWorker" in navigator) || !("PushManager" in window)) return;

  const reg = await navigator.serviceWorker.register("/sw.js");
  const subscription = await reg.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(import.meta.env.VITE_VAPID_PUBLIC_KEY),
  });

  // kirim ke backend
  await fetch(import.meta.env.VITE_API_URL + "/api/subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json", Authorization: `Bearer ${localStorage.getItem("access_token")}` },
    body: JSON.stringify(subscription),
  });
}
```

**`public/sw.js`** (service‑worker)

```js
self.addEventListener("push", (e) => {
  const data = e.data.json();
  self.registration.showNotification(data.title, {
    body: data.body,
    icon: "/icons/icon-192.png",
    data: { url: data.url },
  });
});

self.addEventListener("notificationclick", (e) => {
  e.notification.close();
  e.waitUntil(clients.openWindow(e.notification.data.url));
});
```

**Penggunaan di React**

```jsx
// src/components/ChatRoom.jsx
import { useEffect, useState } from "react";
import socket from "../socket";

export default function ChatRoom({ room }) {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    socket.emit("join", room);
    socket.on("msg", (msg) => setMessages((m) => [...m, msg]));
    return () => {
      socket.off("msg");
      socket.emit("leave", room);
    };
  }, [room]);

  const send = (text) => socket.emit("msg", { room, text });

  return (
    <div>
      <ul>
        {messages.map((m, i) => (
          <li key={i}>
            <strong>{m.from}:</strong> {m.text}
          </li>
        ))}
      </ul>
      <input id="msg" placeholder="Ketik..." />
      <button onClick={() => send(document.getElementById("msg").value)}>Kirim</button>
    </div>
  );
}
```

---

## 3️⃣ Notifikasi yang “nyata” (Web‑Push + FCM)

| Platform | Kelebihan | Cara pakai (free) |
|----------|-----------|-------------------|
| **Web‑Push (VAPID)** | Berlaku di semua browser modern, tidak perlu Google Play Services. | Buat VAPID keys (`npx web-push generate-vapid-keys`). Simpan `PUBLIC_KEY` di env front‑end, `PRIVATE_KEY` di server. Registrasi di Service Worker seperti contoh di atas. |
| **Firebase Cloud Messaging (FCM)** | Push ke Android/iOS + Web (fallback ke Web‑Push). | Daftar project di Firebase → **Cloud Messaging** → dapat `SERVER_KEY`. Gunakan HTTP v1 API (`POST https://fcm.googleapis.com/v1/projects/<PROJECT>/messages:send`) dengan Bearer token dari service‑account. Gratis *unlimited* (hingga quota 100 k msg/h, cukup untuk hobby). |

### Contoh panggilan FCM dari server (Node)

```js
import fetch from "node-fetch";

async function sendFCM(token, payload) {
  const accessToken = await getGoogleAccessToken(); // gunakan service account JSON
  await fetch(`https://fcm.googleapis.com/v1/projects/${process.env.FCM_PROJECT_ID}/messages:send`, {
    method: "POST",
    headers: {
      "Authorization": `Bearer ${accessToken}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      message: {
        token,
        notification: { title: payload.title, body: payload.body },
        data: { url: payload.url },
      },
    }),
  });
}
```

Kombinasikan kedua cara: **Web‑Push untuk browser** dan **FCM untuk mobile**. Simpan masing‑masing token/ subscription di tabel `push_subs` (PostgreSQL).

---

## 4️⃣ Deploy, CI/CD & Monitoring (semua free‑tier)

### 4.1 Dockerfile (untuk Fly.io)

```Dockerfile
# ---------- Builder ----------
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build   # kalau ada TypeScript/ Babel

# ---------- Runtime ----------
FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 8080
CMD ["node","dist/server.js"]
```

### 4.2 `fly.toml` (konfigurasi WS & auto‑start)

```toml
app = "my-realtime-app"

[deploy]
  strategy = "rolling"

[env]
  PORT = "8080"
  REDIS_URL = "rediss://YOUR_UPSTASH_TOKEN@..."
  SUPABASE_URL = "https://YOUR_PROJECT.supabase.co"
  SUPABASE_SERVICE_ROLE_KEY = "YOUR_SERVICE_ROLE"
  JWT_SECRET = "super‑secret"
  VAPID_PUBLIC_KEY = "YOUR_VAPID_PUB"
  VAPID_PRIVATE_KEY = "YOUR_VAPID_PRIV"

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = false           # <-- penting, jangan tidur
  auto_start_machines = true           # <-- bangun otomatis
  # concurrency “connections” memberi limit per instance
  [http_service.concurrency]
    type = "connections"
    hard_limit = 10000    # sesuai quota Fly free‑tier
    soft_limit = 8000
```

> **Catatan:** Pada Fly.io, **tidak ada “spin‑down after 15 menit”** seperti Render yang mematikan layanan karena idle【6†L56-L61】, sehingga WebSocket tetap terbuka selama app berjalan.

### 4.3 CI/CD (GitHub Actions)

```yaml
name: Deploy to Fly.io

on:
  push:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v3
        with: { node-version: 20 }
      - run: npm ci
      - run: npm run build
      - name: Install flyctl
        run: curl -L https://fly.io/install.sh | sh
      - name: Deploy
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
        run: |
          ~/.fly/bin/flyctl deploy --remote-only
```

Masukkan **`FLY_API_TOKEN`** (dapat dibuat di dashboard Fly → *Personal Access Tokens*).  

### 4.4 Monitoring (gratis)

| Tool | Bagian yang dipantau | Cara men‑setup |
|------|---------------------|---------------|
| **Prometheus** (self‑hosted) | `ws_connections_total`, `ws_latency_seconds`, `redis_commands_total` | Jalankan side‑car container di Fly, expose `/metrics`. |
| **Grafana Cloud (free)** | Dashboard visual untuk semua metric di atas | Tambahkan datasource Prometheus, import template “Node.js Socket.io”. |
| **Fly.io metrics** | CPU / RAM / bw per VM (gratis di dashboard) | Buka *Metrics* di Fly console. |
| **Supabase logs** | Log query & auth | `supabase logs` atau lewat dashboard. |

---

## 5️⃣ Tips Skalabilitas & Batasan Free‑Tier

| Layanan | Batas free‑tier | Bagaimana mengatasi bila out‑of‑quota |
|--------|----------------|-------------------------------------|
| **Fly.io** | 3 shared VMs, tiap VM *max 10 000* WS connections (`hard_limit`). | Jika butuh > 30 k user simultan, tambahkan **region replica** (`fly scale count 3`), atau upgrade ke **Paid plan**. |
| **Upstash Redis** | 10 000 concurrent conn, 500 k commands/bulan. | Optimalkan: gunakan **message batching** (`socket.emit('batch', msgs)`) atau **publish per room**. Jika command > 500 k, pertimbangkan **Redis on Fly** (managed, 2 GB free). |
| **Supabase Realtime** | 200 WS connections (free). | Untuk skala > 200, gunakan **socket.io + Redis** (bukan Supabase Realtime). Supabase tetap dipakai hanya untuk **auth + persisten DB**. |
| **Render static site** | 750 free instance‑hours/bulan; layanan “spin‑down” setelah 15 menit idle【6†L56-L61】. | Karena fungsi utama berada di Fly, situs statis hanya serve asset, tidak mempengaruhi WebSocket. |
| **FCM / Web‑Push** | Gratis, tapi FCM memiliki *quota pengiriman* (≈ 100 k msg/h). | Untuk notifikasi harian pada ratusan ribu user, gunakan **topic‑based** FCM atau **batch push** via `sendAll`. |
| **GitHub Actions** | 2 000 menit CI per bulan (free). | Build image hanya sekali per commit; gunakan caching (`actions/cache`). |
| **Bandwidth** | Fly free tier 160 GB outbound per bulan. | Kompres payload (MessagePack), batasi ukuran gambar, gunakan CDN (Render/Cloudflare) untuk static assets. |

### Praktik terbaik untuk **latency ≤ 100 ms**

1. **Gunakan protokol binary** (MessagePack) untuk data WS.  
2. **Keep‑alive ping** tiap 15 s (`socket.io` sudah otomatis).  
3. **Deploy di region terdekat** dengan pengguna (Fly: pilih `region = iad,lhr,sin` dll).  
4. **Cache data yang sering dipakai** di Redis (presence, last‑message‑id).  
5. **Rate‑limit** agar satu user tidak menyedot semua bandwidth (`express-rate-limit` + Redis store).  

---

## 📦 Ringkasan langkah‑per‑langkah (checklist)

| No | Langkah | Perintah / Tindakan |
|---|--------|--------------------|
| 1 | **Buat repo GitHub** (`my-realtime-app`). |
| 2 | **Buat VAPID keys**: `npx web-push generate-vapid-keys`. |
| 3 | **Daftar layanan**: Fly.io → *Create app*; Upstash → *Create Redis*; Supabase → *New project*; Firebase → *Enable Cloud Messaging*. |
| 4 | **Set env** di Fly (`fly secrets set REDIS_URL=… SUPABASE_URL=… JWT_SECRET=… VAPID_PUBLIC_KEY=… VAPID_PRIVATE_KEY=…`). |
| 5 | **Push kode** ke GitHub, pastikan Dockerfile ada. |
| 6 | **Deploy**: `fly launch` → pilih `Dockerfile`, `shared-cpu-1x`, `auto_stop_machines = false`. |
| 7 | **Deploy DB schema** (Supabase SQL): `messages (id, user_id, room, content, created_at)` & `push_subs (endpoint text, keys json)`. |
| 8 | **Daftarkan Service Worker** di `/public/sw.js`. |
| 9 | **Uji**: Buka dua tab, login, join same room → lihat pesan lewat WS dan push (buka dev tools → “Application > Service Workers”). |
|10| **Monitoring**: `fly metrics` + Prometheus‑Grafana. |
|11| **Skalakan** bila diperlukan: `fly scale count 3 --region iad,lhr,sin`. |

Dengan kombinasi **Fly.io**, **Upstash**, **Supabase**, dan **FCM/Web‑Push**, Anda sudah memiliki **stack lengkap yang sepenuhnya gratis** untuk membangun aplikasi *multi‑user real‑time* yang:

- **Terhubung secara full‑duplex** (WebSocket) tanpa latency signifikan.  
- **Menyimpan persisten data** (pesan, user, subscription).  
- **Menyampaikan notifikasi** ke browser & ponsel secara *push* dalam hitungan detik.  
- **Mampu di‑scale secara regional** tanpa menambah biaya sampai kuota free‑tier terpakai.  

Jika trafik Anda mulai melebihi batas‑batas di atas, cukup **upgrade satu layanan** (mis. naik ke *Fly paid plan* atau *Supabase Pro*) – semua komponen sudah ter‑integrasi, jadi migrasi menjadi sangat mudah. Selamat membangun! 🚀  
