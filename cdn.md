A **600 MB static Next.js build** usually means the Docker image is copying **everything** (source files, build cache, node modules, etc.).
The key trick used in production is:

> **Use a multi-stage Docker build so only the final static files are included in the runtime image.**

This can reduce images from **500-700 MB → ~50-100 MB**.

I'll show you the **exact technique used in production pipelines.**

---

# 1️⃣ The Core Idea (Multi-Stage Build)

Instead of this:

```dockerfile
COPY . .
```

which copies:

```
node_modules
.git
.next
src
configs
logs
etc
```

We do:

```
Stage 1 → Build Next.js
Stage 2 → Copy ONLY static output
```

Architecture:

```
Build Stage
   │
   ├─ install node modules
   ├─ run next build
   ├─ run next export
   │
   ▼
Runtime Stage (nginx)
   │
   └─ copy only /out
```

---

# 2️⃣ Production Multi-Stage Dockerfile

```dockerfile
# ---------- Stage 1: Build ----------
FROM node:20-alpine AS builder

WORKDIR /app

# Install dependencies
COPY package.json package-lock.json ./
RUN npm ci

# Copy project files
COPY . .

# Build Next.js
RUN npm run build

# Export static site
RUN npm run export


# ---------- Stage 2: Runtime ----------
FROM nginx:1.27-alpine

# Remove default nginx files
RUN rm -rf /usr/share/nginx/html/*

# Copy only the static export
COPY --from=builder /app/out /usr/share/nginx/html

# Copy nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

---

# 3️⃣ Why This Reduces Size

Without multi-stage:

```
node_modules        300MB
.next cache         200MB
source files        50MB
build output        50MB
-------------------------
Total               ~600MB
```

With multi-stage:

```
nginx image         ~25MB
static site         ~40MB
-------------------------
Total               ~65MB
```

Huge improvement.

---

# 4️⃣ Add `.dockerignore` (VERY Important)

Another major optimization.

Create `.dockerignore`.

```
node_modules
.git
.gitignore
Dockerfile
docker-compose.yml
.next
npm-debug.log
README.md
.env
logs
```

This prevents Docker from sending unnecessary files to the build context.

Without `.dockerignore` your build context may be **hundreds of MB**.

---

# 5️⃣ Optimize Images (Huge Win)

Next.js projects often contain large images.

Install:

```bash
npm install next-optimized-images
```

Or compress images using:

```
imagemin
sharp
squoosh
```

Example reduction:

```
hero.png 3.5MB → 400KB
banner.jpg 2MB → 200KB
```

Across 100 images this saves **200–300MB**.

---

# 6️⃣ Compress Static Assets in Nginx

Inside `nginx.conf`:

```
gzip on;
gzip_types text/plain text/css application/javascript application/json;
gzip_min_length 1000;
```

Reduces network transfer size by **60–80%**.

---

# 7️⃣ Final Production Build Flow

Your workflow becomes:

```
Local Machine
   │
   ▼
Docker build
   │
Multi-stage build
   │
   ▼
Small image (~80MB)
   │
   ▼
Push → DockerHub
   │
   ▼
Hostinger VPS
   │
docker compose pull
docker compose up -d
```

---

# 8️⃣ Expected Image Size

Typical results:

| Type                    | Image Size |
| ----------------------- | ---------- |
| Bad Dockerfile          | 500-700MB  |
| Multi-stage             | 70-120MB   |
| With asset optimization | 40-80MB    |

---

# 9️⃣ DevOps Tip (Very Useful)

Before pushing to DockerHub always check:

```bash
docker image ls
```

or

```bash
docker history image-name
```

This shows which layer is large.

Example:

```
node_modules layer → 300MB
```

Then you know what to optimize.

---

# 🔟 Bonus Trick (Used by Big Companies)

Use **Alpine base images** everywhere.

Example:

```
node:20-alpine
nginx:alpine
```

Alpine images are **5-10x smaller** than Debian ones.

---

✅ If you'd like, I can also show you a **Next.js + Nginx + Docker production architecture that companies use to deploy sites serving millions of users**, including:

* CDN caching
* image optimization
* edge caching
* container security

It will help you **design production-grade systems**, not just Docker builds.
