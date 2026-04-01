# Panduan Lengkap  
**Integrasi API GPT‑OSS‑120B (Ollama) → Node.js (npm) → CRUD + Database**  

> **Target:**  
> 1️⃣ Menjalankan model **120 B** secara **lokal** (atau lewat *Ollama Cloud* bila hardware tidak cukup).  
> 2️⃣ Membuat **client npm** yang memanggil endpoint Ollama.  
> 3️⃣ Membungkusnya dalam sebuah **REST API** (Express) dengan **CRUD** untuk menyimpan / mengelola riwayat percakapan di **database**.  
> 4️⃣ Menyiapkan **Docker** dan contoh **deploy ke cloud** (Railway / Render / Vercel).

---

## 1. Prasyarat (Hardware & Software)

| Kebutuhan | Keterangan |
|-----------|------------|
| **OS** | Linux (Ubuntu 22.04 atau lebih baru), macOS (Apple Silicon atau Intel) atau Windows 10/11 (WSL2). |
| **CPU / RAM** | Model 120 B sangat besar – **minimal 64 GB RAM** dan CPU dengan **AVX‑512**; **GPU NVIDIA dengan VRAM ≥ 48 GB (A100, H100, RTX 4090 x2, dll.)** sangat disarankan. |
| **Docker** | Versi ≥ 24 (untuk container lokal dan cloud). |
| **Node.js** | v18 atau lebih baru (gunakan `nvm`). |
| **npm / yarn** | Paket manager Node.js. |
| **Database** | PostgreSQL ≥ 13 (rekomendasi), atau MongoDB / SQLite jika ingin yang lebih ringan. |
| **Ollama** | Versi ≥ 0.2.9 (menyediakan endpoint OpenAI‑compatible). |
| **Akun Ollama Cloud** (opsional) | Jika hardware tidak cukup, pakai **Ollama Cloud** (paid plan) yang sudah menyediakan model 120 B. |

> **Tip:** Jika Anda tidak memiliki mesin dengan RAM ≥ 64 GB, lewati *“run locally”* dan langsung gunakan **Ollama Cloud** (endpoint `https://<your‑workspace>.ollama.ai`). Panduan ini tetap berlaku – cukup ubah URL di `.env`.

---

## 2. Instalasi Ollama (lokal)

### 2.1. Instalasi di Linux / macOS
```bash
# Tambahkan repository resmi Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Cek versi
ollama --version   # contoh: ollama version 0.2.9
```

### 2.2. Instalasi via Docker (lebih aman)
```bash
docker run -d \
  -p 11434:11434 \
  -v ollama-data:/root/.ollama \
  --name ollama \
  ghcr.io/ollama/ollama:latest
```

> **Catatan:** Container di atas menyimpan model di volume `ollama-data`.

### 2.3. Menjalankan server
```bash
# Jika instalasi native
ollama serve &   # service berjalan di http://127.0.0.1:11434

# Jika Docker, sudah otomatis listening pada port 11434
```

### 2.4. Pull model **120 B**
Ollama menyediakan model `llama3.1:120b` (atau `gemma2:120b` – tergantung repository). Pastikan Anda sudah login ke Ollama Cloud bila ingin meng‑pull.

```bash
# Jika memakai registry Ollama (free trial) – contoh:
ollama login                     # masukkan email / token
ollama pull llama3.1:120b        # ~300 GB download!
```

> **Jika tidak ada model 120 B di public registry** Anda dapat meng‑upload file `gguf` sendiri:
> ```bash
> ollama create my-120b -f /path/to/120b.gguf
> ```

#### Memeriksa model
```bash
curl http://127.0.0.1:11434/api/tags
# Response JSON berisi `"name":"llama3.1:120b"`
```

---

## 3. Membuat Project Node.js (npm)

### 3.1. Inisialisasi Project
```bash
mkdir gpt-oss-120b-crud
cd gpt-oss-120b-crud
npm init -y
```

### 3.2. Dependensi
```bash
# Core
npm i express cors helmet morgan dotenv

# HTTP client ke Ollama
npm i axios

# Database (PostgreSQL contoh)
npm i pg sequelize sequelize-cli

# Optional: validation
npm i joi

# Development
npm i -D nodemon
```

### 3.3. Struktur Direktori
```
gpt-oss-120b-crud/
│
├─ src/
│   ├─ config/
│   │   └─ database.js            # init Sequelize
│   ├─ models/
│   │   └─ conversation.js        # tabel riwayat
│   ├─ routes/
│   │   └─ conversation.routes.js
│   ├─ services/
│   │   └─ ollama.service.js       # wrapper panggil Ollama
│   ├─ app.js                     # express app
│   └─ server.js                  # start server
│
├─ .env                           # konfigurasi
├─ .gitignore
├─ package.json
└─ README.md
```

### 3.4. File `.env`
```dotenv
# -------------------------------------------------
# Ollama (lokal atau cloud)
# -------------------------------------------------
OLLAMA_BASE_URL=http://127.0.0.1:11434   # ubah ke https://<workspace>.ollama.ai jika cloud
OLLAMA_MODEL=llama3.1:120b

# -------------------------------------------------
# PostgreSQL (Docker contoh)
# -------------------------------------------------
DB_HOST=localhost
DB_PORT=5432
DB_NAME=gpt_chat
DB_USER=postgres
DB_PASS=postgres

# -------------------------------------------------
# Server
# -------------------------------------------------
PORT=3000
# -------------------------------------------------
```

> **`.gitignore`** – tambahkan baris: `.env`, `node_modules/`, `dist/`, `logs/`.

### 3.5. Database – Sequelize Setup

**src/config/database.js**
```js
require('dotenv').config();
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize(
  process.env.DB_NAME,
  process.env.DB_USER,
  process.env.DB_PASS,
  {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: 'postgres',
    logging: false,               // set true untuk debug SQL
  }
);

module.exports = sequelize;
```

**src/models/conversation.js**
```js
const { DataTypes, Model } = require('sequelize');
const sequelize = require('../config/database');

class Conversation extends Model {}

Conversation.init(
  {
    // UUID primary key
    id: {
      type: DataTypes.UUID,
      defaultValue: DataTypes.UUIDV4,
      primaryKey: true,
    },
    // Prompt yang diberikan user
    prompt: {
      type: DataTypes.TEXT,
      allowNull: false,
    },
    // Jawaban model
    answer: {
      type: DataTypes.TEXT,
      allowNull: false,
    },
    // Timestamp
    createdAt: {
      type: DataTypes.DATE,
      defaultValue: DataTypes.NOW,
    },
  },
  {
    sequelize,
    modelName: 'Conversation',
    tableName: 'conversations',
    timestamps: false,
  }
);

module.exports = Conversation;
```

**Run migration (quick‑and‑dirty)**
```bash
node -e "require('./src/config/database')
  .sync({ force: false })
  .then(()=>console.log('✅ DB ready'))
  .catch(e=>console.error(e))"
```

---

## 4. Service Wrapper ke Ollama

**src/services/ollama.service.js**
```js
const axios = require('axios');
require('dotenv').config();

const BASE_URL = process.env.OLLAMA_BASE_URL;
const MODEL = process.env.OLLAMA_MODEL;

/**
 * Mengirim request chat ke Ollama (OpenAI‑compatible endpoint)
 * @param {string} userMessage
 * @returns {Promise<string>} jawaban model
 */
async function chatCompletion(userMessage) {
  const payload = {
    model: MODEL,
    messages: [{ role: 'user', content: userMessage }],
    stream: false, // true bila ingin streaming response
    temperature: 0.7,
  };

  try {
    const response = await axios.post(
      `${BASE_URL}/v1/chat/completions`,
      payload,
      { headers: { 'Content-Type': 'application/json' } }
    );

    // Response format: { choices: [{ message: { content: "..." } }] }
    const answer = response.data.choices?.[0]?.message?.content?.trim();
    return answer || '';
  } catch (err) {
    console.error('❌ Ollama error:', err?.response?.data || err.message);
    throw err;
  }
}

module.exports = { chatCompletion };
```

> **Jika memakai Ollama Cloud** yang memerlukan **API‑Key**, tambahkan header:
> ```js
> headers: {
>   'Authorization': `Bearer ${process.env.OLLAMA_API_KEY}`,
>   'Content-Type': 'application/json'
> }
> ```

---

## 5. CRUD API (Express)

### 5.1. `src/routes/conversation.routes.js`
```js
const express = require('express');
const router = express.Router();
const Conversation = require('../models/conversation');
const { chatCompletion } = require('../services/ollama.service');
const Joi = require('joi');

// Validation schema
const promptSchema = Joi.object({
  prompt: Joi.string().min(1).required(),
});

/**
 * CREATE – Kirim prompt ke GPT‑120B, simpan result.
 */
router.post('/', async (req, res) => {
  const { error, value } = promptSchema.validate(req.body);
  if (error) return res.status(400).json({ error: error.details[0].message });

  try {
    const answer = await chatCompletion(value.prompt);
    const conversation = await Conversation.create({
      prompt: value.prompt,
      answer,
    });
    res.status(201).json(conversation);
  } catch (err) {
    res.status(500).json({ error: 'Failed to generate answer.' });
  }
});

/**
 * READ – Semua riwayat (paginated)
 */
router.get('/', async (req, res) => {
  const limit = parseInt(req.query.limit) || 20;
  const offset = parseInt(req.query.offset) || 0;
  const rows = await Conversation.findAll({ limit, offset, order: [['createdAt', 'DESC']] });
  res.json(rows);
});

/**
 * READ – Detail per ID
 */
router.get('/:id', async (req, res) => {
  const conv = await Conversation.findByPk(req.params.id);
  if (!conv) return res.status(404).json({ error: 'Not found' });
  res.json(conv);
});

/**
 * UPDATE – Update prompt **atau** answer (biasa tidak perlu, tapi contoh)
 */
router.put('/:id', async (req, res) => {
  const conv = await Conversation.findByPk(req.params.id);
  if (!conv) return res.status(404).json({ error: 'Not found' });

  const { prompt, answer } = req.body;
  if (prompt) conv.prompt = prompt;
  if (answer) conv.answer = answer;
  await conv.save();
  res.json(conv);
});

/**
 * DELETE – Hapus riwayat
 */
router.delete('/:id', async (req, res) => {
  const rowsDeleted = await Conversation.destroy({ where: { id: req.params.id } });
  if (!rowsDeleted) return res.status(404).json({ error: 'Not found' });
  res.status(204).send();
});

module.exports = router;
```

### 5.2. `src/app.js`
```js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const morgan = require('morgan');
require('dotenv').config();

const conversationRoutes = require('./routes/conversation.routes');

const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json({ limit: '2mb' }));   // body size limit
app.use(morgan('combined'));

app.get('/', (req, res) => res.send('GPT‑OSS‑120B CRUD API is alive 🚀'));

app.use('/api/conversations', conversationRoutes);

// 404 handler
app.use((req, res) => res.status(404).json({ error: 'Endpoint not found' }));

module.exports = app;
```

### 5.3. `src/server.js`
```js
const app = require('./app');
const sequelize = require('./config/database');

const PORT = process.env.PORT || 3000;

(async () => {
  try {
    await sequelize.authenticate();
    console.log('✅ Database connected');
    // sync jika belum ada tabel (production biasanya gunakan migration)
    await sequelize.sync();   // { force: false }
    app.listen(PORT, () => console.log(`🚀 Server listening on http://localhost:${PORT}`));
  } catch (err) {
    console.error('❌ Unable to start server:', err);
    process.exit(1);
  }
})();
```

### 5.4. `package.json` (scripts)
```json
{
  "name": "gpt-oss-120b-crud",
  "version": "1.0.0",
  "description": "REST API dengan GPT‑OSS‑120B (Ollama) + CRUD + PostgreSQL",
  "main": "src/server.js",
  "type": "module",
  "scripts": {
    "dev": "nodemon src/server.js",
    "start": "node src/server.js",
    "db:reset": "node -e \"require('./src/config/database').drop(); require('./src/config/database').sync({ force:true })\""
  },
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "axios": "^1.7.2",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.19.2",
    "helmet": "^7.1.0",
    "joi": "^17.13.0",
    "morgan": "^1.10.0",
    "pg": "^8.12.0",
    "sequelize": "^6.37.3",
    "sequelize-cli": "^6.6.2"
  },
  "devDependencies": {
    "nodemon": "^3.1.4"
  }
}
```

---

## 6. Menjalankan Secara Lokal

### 6.1. Pastikan PostgreSQL berjalan
```bash
# Debian/Ubuntu contoh
sudo service postgresql start

# Buat database & user (jika belum)
sudo -u postgres psql -c "CREATE DATABASE gpt_chat;"
sudo -u postgres psql -c "CREATE USER postgres WITH PASSWORD 'postgres';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE gpt_chat TO postgres;"
```

### 6.2. Start Ollama (jika belum)
```bash
ollama serve &
# atau docker run -d -p 11434:11434 ghcr.io/ollama/ollama
```

### 6.3. Test panggilan Ollama langsung (opsional)
```bash
curl -X POST http://127.0.0.1:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
        "model": "llama3.1:120b",
        "messages": [{"role":"user","content":"Hello, world!"}]
      }' | jq .
```

### 6.4. Jalankan API Node
```bash
npm run dev
# akan muncul: 🚀 Server listening on http://localhost:3000
```

### 6.5. Coba endpoint CRUD dengan *cURL* atau *Postman*

**Create (POST)**
```bash
curl -X POST http://localhost:3000/api/conversations \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Tuliskan puisi tentang laut dalam bahasa Indonesia"}'
```

**Read (GET)**
```bash
curl http://localhost:3000/api/conversations
```

**Detail (GET by ID)**
```bash
curl http://localhost:3000/api/conversations/<uuid>
```

**Update (PUT)**
```bash
curl -X PUT http://localhost:3000/api/conversations/<uuid> \
  -H "Content-Type: application/json" \
  -d '{"prompt":"Update prompt"}'
```

**Delete (DELETE)**
```bash
curl -X DELETE http://localhost:3000/api/conversations/<uuid>
```

Jika semuanya berhasil, Anda sudah memiliki **CRUD API** yang menyimpan tiap prompt & jawaban di PostgreSQL dan memanfaatkan model **120 B** melalui Ollama.

---

## 7. Dockerisasi (untuk produksi)

### 7.1. `Dockerfile`

```Dockerfile
# ---- Build stage ----
FROM node:20-alpine AS builder
WORKDIR /app

# copy package files & install
COPY package*.json ./
RUN npm ci --only=production

# copy src
COPY src ./src
COPY .env ./   # (untuk demo) – di production gunakan secret manager

# ---- Run stage ----
FROM node:20-alpine
WORKDIR /app

# copy from builder
COPY --from=builder /app .

# expose port
EXPOSE 3000

# start
CMD ["node", "src/server.js"]
```

### 7.2. `docker-compose.yml` (meliputi PostgreSQL & Ollama)

```yaml
version: "3.9"

services:
  db:
    image: postgres:15-alpine
    container_name: pg_gpt
    environment:
      POSTGRES_DB: gpt_chat
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - pg_data:/var/lib/postgresql/data

  ollama:
    image: ghcr.io/ollama/ollama:latest
    container_name: ollama
    ports:
      - "11434:11434"
    volumes:
      - ollama_data:/root/.ollama
    restart: unless-stopped

  api:
    build: .
    container_name: gpt_api
    env_file:
      - .env                # pastikan .env berisi DB_HOST=db
    depends_on:
      - db
      - ollama
    ports:
      - "3000:3000"
    restart: unless-stopped

volumes:
  pg_data:
  ollama_data:
```

**Catatan penting:**
- Pada `.env` ubah `DB_HOST=db` (service name) sehingga container `api` dapat terhubung ke PostgreSQL.
- Model `120b` **harus** di‑pull *sebelum* jalankan container, atau gunakan `ollama pull` pada **init script**. Contoh `docker-compose` dengan **entrypoint**:

```yaml
  ollama:
    image: ghcr.io/ollama/ollama:latest
    command: ["sh", "-c", "ollama pull llama3.1:120b && ollama serve"]
```

> **Perhatian:** Pull model 120 B lewat internet memakan bandwidth tinggi (≈300 GB). Pastikan jaringan Anda cukup cepat.

### 7.3. Jalankan semua layanan
```bash
docker compose up -d
# Cek log:
docker compose logs -f
```

---

## 8. Deploy ke Cloud

Berikut contoh **Deploy ke Railway** (pilihan gratis/berbayar). Proses serupa untuk Render, Fly.io, atau Vercel (gunakan Serverless + External Ollama Cloud).

### 8.1. Persiapan repo Git
```bash
git init
git add .
git commit -m "Initial commit"
git remote add origin <your‑git‑provider‑url>
git push -u origin master
```

### 8.2. Buat Project di Railway
1. Masuk ke https://railway.app → **New Project** → **Deploy from GitHub**.  
2. Pilih repository Anda.  
3. Railway akan otomatis mendeteksi `Dockerfile` → **Build & Deploy**.

### 8.3. Tambahkan *Variables* di Railway
| Nama | Value |
|------|-------|
| `OLLAMA_BASE_URL` | `https://<workspace>.ollama.ai` (jika memakai Ollama Cloud) |
| `OLLAMA_MODEL`    | `llama3.1:120b` |
| `DB_HOST`        | auto‑generated (Railway PostgreSQL plugin) |
| `DB_PORT`        | `5432` |
| `DB_NAME`        | (dari plugin) |
| `DB_USER`        | (dari plugin) |
| `DB_PASS`        | (dari plugin) |
| `PORT`           | `3000` |

### 8.4. (Opsional) Menggunakan **Ollama Cloud**  
Jika Anda tidak dapat men‑run model 120 B secara lokal, daftarkan ke **Ollama Cloud** → dapatkan **API Key** → set variabel:

- `OLLAMA_API_KEY=your_key_here`

Kemudian ubah `src/services/ollama.service.js`:

```js
const headers = {
  'Content-Type': 'application/json',
  Authorization: `Bearer ${process.env.OLLAMA_API_KEY}`,
};
```

### 8.5. Verifikasi

Setelah deploy selesai, Railway memberi URL, misalnya `https://gpt-oss-120b-production.up.railway.app`.  

Coba:

```bash
curl -X POST https://.../api/conversations \
  -H "Content-Type: application/json" \
  -d '{"prompt":"What is the capital of Indonesia?"}'
```

Jika Anda mendapatkan `answer: "Jakarta"` maka semuanya sudah ter‑integrasi.

---

## 9. Monitoring & Scaling

| Komponen | Cara Monitoring |
|----------|-----------------|
| **Ollama** (CPU/GPU) | `docker stats ollama` atau **NVIDIA‑SMI** (`nvidia-smi -l 5`). |
| **Node API** | Gunakan **PM2** (`pm2 start src/server.js --name gpt-api`) + **pm2‑logs**, atau integrasi ke **Grafana Loki** bila di Kubernetes. |
| **Database** | `pg_top`, atau gunakan layanan managed (Railway/Supabase) yang menyediakan metrics. |
| **Log Request** | `morgan` sudah ter‑pasang, atau kirim ke **Logflare / Papertrail** via `winston`. |
| **Error Tracking** | **Sentry** (npm `@sentry/node`). Tambahkan di `src/app.js`: `Sentry.init({ dsn: process.env.SENTRY_DSN })`. |

### 9.1. Auto‑Scaling (Docker Swarm / Kubernetes)

Jika beban tinggi, jalankan **multiple replicas** untuk API (tidak untuk Ollama karena model stateful). Contoh `docker compose` dengan `scale`:

```bash
docker compose up -d --scale api=3
```

Atau di **K8s**: buat `Deployment` untuk `api` dengan `replicas: 3` dan *Service* `LoadBalancer`.

---

## 10. Troubleshooting Umum

| Masalah | Penyebab Umum | Solusi |
|---------|---------------|--------|
| **Ollama mengembalikan error 500 / timeout** | Model belum dimuat (CPU/GPU belum cukup) atau RAM tidak cukup. | Pastikan `ollama serve` sudah selesai loading. Periksa log `docker logs ollama`. Tambah swap atau gunakan GPU dengan driver‑CUDA yang tepat. |
| **Database connection refused** | `DB_HOST` tidak cocok (container vs host). | Di Docker Compose gunakan `DB_HOST=db`. Di local set `localhost`. |
| **Response terlalu lambat (>30 s)** | Model 120 B memang berat; gunakan `temperature` kecil, atau *chunk* prompt. Pertimbangkan **batching** atau **token limit** (`max_tokens`). | Tambah `max_tokens` di payload, atau gunakan GPU dengan lebih banyak memori. |
| **Error `ECONNREFUSED` ke Ollama Cloud** | API Key tidak ter‑set atau endpoint salah. | Pastikan `.env` memiliki `OLLAMA_API_KEY` dan `OLLAMA_BASE_URL` yang tepat. |
| **Docker container OOM killed** | Model melebihi limit RAM. | Tambahkan `mem_limit` di `docker-compose.yml` atau jalankan di host dengan hardware lebih besar. |

---

## 11. Ringkasan Langkah (Cheat‑Sheet)

```bash
# 1. Install Ollama
curl -fsSL https://ollama.com/install.sh | sh
ollama serve &

# 2. Pull 120B model
ollama login
ollama pull llama3.1:120b

# 3. Setup project
git clone <repo>
cd gpt-oss-120b-crud
npm i
cp .env.example .env   # sesuaikan
docker compose up -d   # (opsional) Postgres + Ollama + API

# 4. Jalankan API
npm run dev            # atau docker compose up api

# 5. Test endpoint
curl -X POST http://localhost:3000/api/conversations \
 -H "Content-Type: application/json" \
 -d '{"prompt":"Berikan contoh kode Node.js"}'

# 6. Deploy ke Railway / Render
#    - push repo
#    - set env vars (OLLAMA_BASE_URL, OLLAMA_API_KEY, DB_*)
#    - jadi!
```

---

## 12. Referensi & Link Penting

| Topik | Link |
|-------|------|
| **Ollama Docs** | https://ollama.com/docs |
| **Ollama Cloud** | https://cloud.ollama.com |
| **OpenAI‑compatible API** | https://github.com/ollama/ollama/blob/main/docs/api.md |
| **Sequelize Docs** | https://sequelize.org/master/ |
| **Docker Hub – Ollama** | https://hub.docker.com/r/ollama/ollama |
| **Railway – Deploy Docker** | https://railway.app/docs/deploy/docker |
| **Sentry for Node** | https://docs.sentry.io/platforms/node/ |
| **PostgreSQL Docker Image** | https://hub.docker.com/_/postgres |

---

### 🎉 Selamat!  
Anda kini memiliki **REST API** yang terhubung ke model **GPT‑OSS‑120B** melalui **Ollama**, CRUD full dengan **PostgreSQL**, serta cara meng‑docker‑kan dan **deploy ke cloud**.  

Jika ada bagian yang kurang jelas atau Anda ingin men‑extend (mis. otentikasi JWT, pagination lanjutan, atau streaming response), beri tahu saya—saya siap membantu men‑tambah fitur lebih lanjut! 🚀
