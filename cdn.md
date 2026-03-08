The **GitHub + Cloudflare R2 + Cloudflare CDN** setup is a popular **zero-cost or near-zero cost architecture** for hosting static assets (images, PDFs, videos) used by many frontend apps like Next.js. It keeps your Docker images small and delivers files globally via CDN. 🚀

---

# 🧠 Architecture Overview

```text
Next.js App
    ↓
cdn.yoursite.com
    ↓
Cloudflare CDN
    ↓
Cloudflare R2 (object storage)
    ↑
GitHub (assets source / version control)
```

**Roles**

* GitHub → version control for assets
* Cloudflare R2 → object storage
* Cloudflare CDN → global caching

Result:

```text
Docker image: ~80MB
Assets: CDN delivered globally
```

---

# 1️⃣ Create R2 Bucket

1. Log into Cloudflare dashboard
2. Go to **R2 Object Storage**
3. Create bucket

Example:

```text
bucket name: assets
```

Structure:

```text
assets/
 ├ images/
 │   └ banner.webp
 ├ pdf/
 │   └ brochure.pdf
 └ videos/
     └ intro.mp4
```

---

# 2️⃣ Enable Public Access

Create a public URL.

Example:

```text
https://pub-abc123.r2.dev
```

Now files become accessible like:

```text
https://pub-abc123.r2.dev/images/banner.webp
```

---

# 3️⃣ Add Custom CDN Domain

In Cloudflare:

**R2 → Settings → Custom Domain**

Example:

```text
cdn.yoursite.com
```

Now assets load via CDN:

```text
https://cdn.yoursite.com/images/banner.webp
```

Cloudflare automatically:

* caches globally
* enables HTTP/3
* compresses files

---

# 4️⃣ Upload Assets Automatically from GitHub

You can sync GitHub → R2 using CI.

Example **GitHub Action**:

```yaml
name: Upload Assets

on:
  push:
    paths:
      - assets/**

jobs:
  upload:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Upload to R2
        run: |
          npx wrangler r2 object put assets/images/logo.png \
          --file=assets/images/logo.png
```

This means:

```text
Git push → assets uploaded to R2
```

---

# 5️⃣ Use Assets in Next.js

Example:

```jsx
<Image
  src="https://cdn.yoursite.com/images/banner.webp"
  width={1200}
  height={600}
/>
```

Or:

```html
<img src="https://cdn.yoursite.com/pdf/brochure.pdf" />
```

---

# 📊 Cost

Cloudflare R2 free tier:

* **10GB storage free**
* **1M requests/month free**
* **no egress bandwidth cost**

For most small/medium apps:

```text
Cost = $0/month
```

---

# 🚀 Benefits

✅ Global CDN
✅ Zero egress cost
✅ Very fast delivery
✅ Smaller Docker images
✅ Git-based asset workflow

---

# ⚠️ Important Tip for Next.js

Add CDN domain to config:

```js
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    domains: ["cdn.yoursite.com"]
  }
}

module.exports = nextConfig
```

---

# ⭐ Real Production Pattern

Many modern apps follow:

```text
Next.js container
       ↓
Cloudflare CDN
       ↓
R2 storage
```

This keeps deployments **fast and lightweight**.

---

✅ If you'd like, I can also show:

* **A full working GitHub repo structure for this setup**
* **Automatic GitHub → R2 sync pipeline**
* **How to make Next.js upload assets to R2 automatically**.
