## 1️⃣ Apa itu **WebSocket**?

| **WebSocket** | **HTTP** |
|---------------|----------|
| *Protocol* yang berada di atas TCP, bukan di atas request‑response. | *Protocol* teks berbasis request‑response. |
| Setelah satu kali **handshake**, koneksi tetap terbuka (full‑duplex). | Setiap request harus membuat koneksi TCP baru → overhead. |
| Server dapat **push** data ke client kapan saja (tanpa client melakukan request). | Server hanya bisa mengirim data sebagai balasan request. |
| Frame‑frame yang dikirim berukuran kecil, bisa **text** atau **binary**. | HTTP mengirimkan seluruh header + body setiap kali. |
| Cocok untuk aplikasi yang membutuhkan *real‑time* (chat, game, monitoring, dll). | Cocok untuk aplikasi yang bersifat *request‑response* (CRUD biasa). |

### Proses “handshake”

1. Client (biasanya browser) mengirim HTTP request dengan header `Upgrade: websocket` dan `Connection: Upgrade`.
2. Server mengakui dan merespon `101 Switching Protocols`.
3. Dari titik itu, protokol berubah menjadi WebSocket. Kedua pihak dapat mengirimkan frame secara bebas sampai salah satu menutup koneksi.

> **Catatan:** WebSocket biasanya dijalankan lewat **wss://** (TLS) agar aman, persis seperti HTTPS.

---

## 2️⃣ Kapan (dan kenapa) Pakai WebSocket?

| Use‑case | Kenapa WebSocket? |
|----------|--------------------|
| **Chat / Instant Messaging** | Pesan harus sampai dalam milidetik, tanpa polling. |
| **Live Dashboard / Monitoring** | Data sensor/metrics terus mengalir ke UI. |
| **Collaborative Editing** (Google Docs‑style) | Perubahan dokumen harus disiarkan ke semua peserta secara bersamaan. |
| **Online Multiplayer Games** | Latency rendah, data binary kecil (posisi, aksi). |
| **Notification System** (push notifikasi ke web) | Server dapat mengirim notifikasi walau pengguna tidak sedang melakukan request. |
| **Stock / Crypto Ticker** | Harga berubah tiap detik – tidak efisien pakai polling. |

Jika aplikasi Anda *hanya* membutuhkan data yang berubah jarang (mis. refresh tiap 5‑10 detik), **Polling / Long‑Polling** atau **Server‑Sent Events (SSE)** sudah cukup. WebSocket lebih “berat” secara konseptual, jadi gunakan ketika benar‑benar butuh *duplex* yang terus‑menerus.

---

## 3️⃣ Framework / Library WebSocket di **Node.js**

| Library | Tingkat Abstraksi | Fitur Utama | Pro/Con |
|---------|------------------|-------------|--------|
| **`ws`** | Rendah (native WebSocket API) | Ultra‑ringan, standar RFC6455, binary support. | Tidak ada fallback (hanya WebSocket). Ideal untuk performa & belajar bagaimana WebSocket bekerja. |
| **`socket.io`** | Tinggi (wrapper) | Fallback ke long‑polling, rooms & namespaces, auto‑reconnect, ACK, broadcast, built‑in heartbeat. | Lebih berat, tidak 100 % standar (menggunakan own protokol di atas WebSocket). |
| **`uWebSockets.js`** | Rendah‑tinggi, sangat cepat (C++ core). | 5‑10× lebih cepat dari `ws`, dapat meng-handle > 5 M koneksi. | API sedikit berbeda, dukungan fallback terbatas. |
| **`@nestjs/websockets`** (module di **NestJS**) | Tinggi (dekorator, DI) | Integrasi dengan **Gateway**, **@SubscribeMessage**, **Guard**, **interceptor**, dukungan **socket.io** atau **ws**. | Memaksa penggunaan NestJS (TypeScript, opiniated). |
| **`FeathersJS`** | Tinggi (services) | CRUD otomatis via REST *dan* real‑time (socket.io). | Overhead tambahan, cocok bila sudah pakai Feathers. |
| **`SockJS`** | Tinggi (fallback) | Menyediakan WebSocket‑like API dengan fallback ke XHR streaming, iframe. | Umumnya dipakai di sisi client bersama server yang custom. |
| **`Primus`** | Tinggi (abstraction) | Dapat men‑swap engine (`ws`, `socket.io`, `engine.io`, `sockjs`). | Karena banyak lapisan, performa agak turun. |

### Rekomendasi Berdasarkan Kebutuhan

| Kebutuhan | Pilihan |
|----------|---------|
| **Cepat & Minimal** (hanya WebSocket, tidak butuh fallback) | `ws` atau `uWebSockets.js` |
| **Fitur Lengkap + Fallback** (chat, room, auto‑reconnect) | `socket.io` |
| **Aplikasi Enterprise dengan TypeScript, DI, Guard** | **NestJS** + `@nestjs/websockets` (menggunakan `socket.io` atau `ws` sebagai engine) |
| **Serverless atau tanpa server (Firebase, Deno)** | **Socket.io** di Cloud Functions atau **WebSocket API** di AWS API GW. |
| **High‑throughput real‑time game** | `uWebSockets.js` + custom binary protocol. |

---

## 4️⃣ Contoh Implementasi Praktis

### 4.1 Server dengan **`ws`** (native)

```js
// server-ws.js
const http = require('http');
const WebSocket = require('ws');

const server = http.createServer();               // HTTP server (bisa untuk static file)
const wss = new WebSocket.Server({ server });    // attach WS ke HTTP

wss.on('connection', (ws, req) => {
  console.log('🔗 New client:', req.socket.remoteAddress);

  // kirim pesan ke client setelah terkoneksi
  ws.send(JSON.stringify({ type: 'welcome', message: 'Hello from WS server!' }));

  // terima pesan dari client
  ws.on('message', data => {
    const msg = JSON.parse(data);
    console.log('📩 Received:', msg);

    // contoh broadcast ke semua client
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({ type: 'broadcast', payload: msg }));
      }
    });
  });

  ws.on('close', () => console.log('❌ Client disconnected'));
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => console.log(`🚀 WS listening on http://localhost:${PORT}`));
```

**Client (plain HTML/JS)**

```html
<!DOCTYPE html>
<html>
<head><title>WebSocket Demo</title></head>
<body>
  <h1>WebSocket Chat</h1>
  <ul id="log"></ul>
  <input id="msg" placeholder="ketik pesan"/>
  <button onclick="send()">Kirim</button>

  <script>
    const ws = new WebSocket(`ws://${location.host}`); // otomatis terhubung ke server di atas
    const log = document.getElementById('log');
    const input = document.getElementById('msg');

    ws.onopen = () => log.insertAdjacentHTML('beforeend', '<li>✅ Connected</li>');
    ws.onmessage = e => {
      const data = JSON.parse(e.data);
      log.insertAdjacentHTML('beforeend', `<li>${data.type}: ${JSON.stringify(data.payload || data.message)}</li>`);
    };
    ws.onclose = () => log.insertAdjacentHTML('beforeend', '<li>❌ Disconnected</li>');

    function send() {
      const text = input.value.trim();
      if (!text) return;
      ws.send(JSON.stringify({ text, ts: Date.now() }));
      input.value = '';
    }
  </script>
</body>
</html>
```

> **Catatan keamanan** – Pada produksi gunakan **wss://** (TLS). Anda dapat menambahkan middleware JWT di `ws` dengan mengecek token pada query‑string saat `connection` event.

---

### 4.2 Server dengan **`socket.io`** (fallback + rooms)

```js
// server-socketio.js
const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
app.use(express.static('public')); // folder berisi index.html di atas

const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: '*' },         // untuk development
  // transports: ['websocket']   // uncomment untuk men‑force hanya WebSocket
});

/* Middleware auth (contoh JWT) */
io.use((socket, next) => {
  const token = socket.handshake.auth?.token;
  if (token && token === 'secret-demo-token') return next(); // validate di sini
  return next(new Error('unauthorized'));
});

io.on('connection', socket => {
  console.log('🔗 New socket:', socket.id);

  // join default room
  socket.join('global');

  // broadcast dummy hello
  socket.emit('welcome', { msg: 'Welcome to Socket.io!' });

  // handling chat message
  socket.on('chat', data => {
    // broadcast ke semua klien di room "global"
    io.to('global').emit('chat', { from: socket.id, text: data.text });
  });

  socket.on('private', ({ to, text }) => {
    // kirim ke satu socket tertentu
    socket.to(to).emit('private', { from: socket.id, text });
  });

  socket.on('disconnect', reason => {
    console.log('❌ Socket disconnected:', socket.id, reason);
  });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => console.log(`🚀 Socket.io server listening on ${PORT}`));
```

**Client (socket.io) – `public/index.html`**

```html
<!DOCTYPE html>
<html>
<head><title>Socket.io Chat</title></head>
<body>
  <h2>Chat (Socket.io)</h2>
  <ul id="log"></ul>
  <input id="msg" placeholder="ketik pesan"/>
  <button onclick="send()">Kirim</button>

  <script src="/socket.io/socket.io.js"></script>
  <script>
    const socket = io({
      auth: { token: 'secret-demo-token' } // contoh auth sederhana
    });
    const log = document.getElementById('log');
    const input = document.getElementById('msg');

    socket.on('connect', () => log.insertAdjacentHTML('beforeend', '<li>✅ Connected</li>'));
    socket.on('welcome', data => log.insertAdjacentHTML('beforeend', `<li>${data.msg}</li>`));
    socket.on('chat', data => log.insertAdjacentHTML('beforeend', `<li>${data.from}: ${data.text}</li>`));
    socket.on('disconnect', () => log.insertAdjacentHTML('beforeend', '<li>❌ Disconnected</li>'));

    function send() {
      const text = input.value.trim();
      if (!text) return;
      socket.emit('chat', { text });
      input.value = '';
    }
  </script>
</body>
</html>
```

Kelebihan `socket.io`:
- Otomatis **reconnect** dengan back‑off.
- **ACK** untuk memastikan pesan diterima.
- **Rooms**/`namespace` untuk segmentasi (public, private, admin, dsb).
- **Fallback** ke HTTP long‑polling bila WebSocket diblokir.

---

### 4.3 Server dengan **NestJS** (WebSocket Gateway)

```ts
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ChatGateway } from './chat.gateway';

@Module({
  providers: [ChatGateway],
})
export class AppModule {}
```

```ts
// src/chat.gateway.ts
import {
  WebSocketGateway,
  SubscribeMessage,
  MessageBody,
  ConnectedSocket,
  WebSocketServer,
} from '@nestjs/websockets';
import { Server, Socket } from 'socket.io';

@WebSocketGateway({
  cors: { origin: '*' },
  // gunakan 'ws' sebagai engine jika tidak butuh fallback
})
export class ChatGateway {
  @WebSocketServer()
  server: Server;

  // ketika client mengirim event 'msg'
  @SubscribeMessage('msg')
  handleMessage(@MessageBody() payload: { text: string }, @ConnectedSocket() client: Socket) {
    // broadcast ke semua client (kecuali pengirim)
    client.broadcast.emit('msg', { from: client.id, text: payload.text });
    // optional: return value akan otomatis dikirim ke client yang bersangkutan
    return { ack: true };
  }

  // contoh otorisasi dengan guard (bisa pakai JWT Guard)
}
```

**Client** – sama seperti contoh `socket.io` di atas (karena NestJS memakai `socket.io` di belakang).

> **Tip:** Untuk skala horizontal, pasang **RedisAdapter** di `main.ts`:

```ts
import { NestFactory } from '@nestjs/core';
import { RedisAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  const redis = createClient({ url: process.env.REDIS_URL });
  await redis.connect();
  app.useWebSocketAdapter(new RedisAdapter(redis, redis));
  await app.listen(3000);
}
bootstrap();
```

---

## 5️⃣ Memilih Framework yang Tepat

| **Kriteria** | **Rekomendasi** |
|--------------|-----------------|
| **Performance kritikal (10k+ koneksi)** | `uWebSockets.js` / `ws` + load‑balancer + sticky‑session. |
| **Fallback penting (user di jaringan yang tidak mengizinkan WS)** | `socket.io` (auto fallback ke polling) atau `SockJS`. |
| **TypeScript, DI, guard, modular** | **NestJS** (gateway + Redis adapter). |
| **Mau cepat prototipe dengan CRUD + real‑time** | **FeathersJS** (service auto‑exposed lewat REST + socket.io). |
| **Serverless (AWS Lambda, Cloud Functions)** | **socket.io**‑compatible *serverless‑websocket* (e.g., `serverless-websockets-plugin` atau API‑Gateway WebSocket). |
| **Game dengan binary protocol** | `uWebSockets.js` **+** custom binary serialization (msgpack, protobuf). |
| **Integrasi dengan microservices** | **NATS JetStream** atau **Kafka** sebagai broker; gunakan `socket.io` + Redis‑adapter hanya untuk *presence*; proses bisnis di consumer lain. |

### Pertimbangan Tambahan

| Aspek | Hal yang perlu dicek |
|-------|----------------------|
| **Keamanan** | Gunakan `wss://` (TLS). Autentikasi via JWT di query/handshake atau token di `socket.handshake.auth`. Implementasikan **rate‑limit** pada koneksi masuk (mis. max 5 koneksi per IP per menit). |
| **Load Balancer** | NGINX/HAProxy dengan `proxy_set_header Upgrade $http_upgrade;` dan **sticky‑session** (`ip_hash` atau `hash $binary_remote_addr $binary_remote_port`). |
| **Horizontal Scaling** | Pakai **Redis Adapter** (pub/sub) atau **NATS** untuk sinkronisasi pesan antar instance. |
| **Observability** | Export metrik *connections*, *messages/sec*, *latency* ke **Prometheus** (`socket.io-prometheus-metrics` atau custom middleware). |
| **Graceful Shutdown** | Pada `SIGTERM`, tutup semua socket dengan `server.close()`; beri waktu client untuk reconnect. |
| **Testing** | Unit‑test handler dengan `socket.io-client` atau `ws` client; load‑test dengan **k6** (`k6 ws`), **Artillery**, atau **Tsung**. |
| **Deploy** | Dockerize (`FROM node:20-alpine`), push ke registry, jalankan di **K8s** (Deployment + Service). Gunakan **HorizontalPodAutoscaler** + **KEDA** untuk scaling berdasarkan *Redis channel length* atau *WebSocket connections*. |
| **Compatibility** | Pastikan client (browser, mobile app, desktop) dapat menggunakan WebSocket (semua modern browser dukung). Jika target termasuk **older browsers** (IE), pertimbangkan fallback (`socket.io`/`SockJS`). |

---

## 6️⃣ Ringkasan & Rekomendasi Praktis

| Goal | Stack Ringkas (Node) |
|------|----------------------|
| **MVP cepat (chat sederhana)** | `express + socket.io + Redis (optional)`. Deploy di **Render/ Fly.io** (auto‑TLS). |
| **Aplikasi enterprise dengan TypeScript** | **NestJS** + **socket.io** (gateway) + **Redis Adapter** + **PostgreSQL** (data persistent). Deploy di **K8s** (EKS/GKE). |
| **High‑throughput real‑time game** | **uWebSockets.js** (core) + custom binary (msgpack) + **NATS JetStream** untuk event broadcast. Deploy di **bare‑metal** atau **ECS** dengan **ALB** TCP. |
| **Serverless** | **AWS API Gateway (WebSocket) + Lambda (Node.js) + DynamoDB Streams**. Front end dengan **Amplify** atau **React**. |
| **Hybrid (fallback + rooms)** | **socket.io** (engine.io) + **Redis** (pub/sub) + **MongoDB Change Streams** (feed). |

---

## 7️⃣ Sumber Belajar & Dokumentasi

| Topik | Link |
|-------|------|
| **WebSocket RFC6455** | https://datatracker.ietf.org/doc/html/rfc6455 |
| **Socket.io Docs** | https://socket.io/docs/v4/ |
| **ws (Node) Docs** | https://github.com/websockets/ws |
| **uWebSockets.js** | https://github.com/uNetworking/uWebSockets.js |
| **NestJS WebSocket Gateway** | https://docs.nestjs.com/websockets/gateways |
| **Redis Adapter for Socket.io** | https://socket.io/docs/v4/redis-adapter/ |
| **NATS JetStream (Node)** | https://github.com/nats-io/nats.deno |
| **Kangaroo (Scalable WebSocket in K8s)** | https://kubernetes.io/blog/2020/09/24/websocket-load-balancing/ |
| **k6 WebSocket Load Testing** | https://k6.io/docs/using-k6/websockets/ |
| **Authenticating WebSocket Connections** | https://socket.io/docs/v4/middlewares/ |
| **WebSocket Security (WSS & JWT)** | https://developer.mozilla.org/en-US/docs/Web/API/WebSocket/Using_WebSockets#security_considerations |

---

## 8️⃣ Contoh Mini‑Project (Langkah‑dari‑Awal)

1. **Buat folder project**  
   ```bash
   mkdir chat-demo && cd chat-demo
   npm init -y
   npm i express socket.io
   npm i -D nodemon
   ```

2. **Buat `server.js`** (copy contoh `socket.io` di atas).

3. **Buat folder `public`** berisi `index.html` (client socket.io).

4. **Tambahkan skrip dev** ke `package.json`  
   ```json
   "scripts": {
     "dev": "nodemon server.js"
   }
   ```

5. **Jalankan**  
   ```bash
   npm run dev
   ```

6. **Buka** `http://localhost:3000` di 2 tab browser, coba kirim pesan – Anda melihat broadcast real‑time tanpa refresh.

7. **Upgrade** ke produksi:  
   - Tambahkan **SSL** (LetsEncrypt) → gunakan **nginx** sebagai reverse‑proxy (`proxy_set_header Upgrade $http_upgrade;`).  
   - Pasang **Redis** (`docker run -p 6379:6379 redis`) dan aktifkan `io.adapter(createAdapter(redisClient))`.  
   - Build Docker image dan push ke registry → jalankan di **Kubernetes** atau **Docker‑Compose** multi‑node.

Dengan langkah‑langkah itu Anda sudah memiliki pondasi **WebSocket** yang dapat dikembangkan menjadi chat, notifikasi, live dashboard, atau apa pun yang membutuhkan data real‑time.

--- 

**Semoga penjelasan ini membantu Anda memahami apa itu WebSocket, kapan harus menggunakannya, dan framework‑framework apa yang paling cocok di ekosistem Node.js.**  
Jika ada pertanyaan lanjutan – misalnya cara men‑setup **Redis Adapter** di Kubernetes, cara meng‑implementasikan **JWT auth** yang aman, atau contoh **binary protocol** untuk game – silakan beri tahu saya! 🚀
