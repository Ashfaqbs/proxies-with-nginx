## Nginx as a Caching Proxy for Static Content

### 📘 Introduction
In high-traffic systems like news sites, e-commerce portals, or social media platforms, serving static content (HTML, JS, CSS, images) quickly is crucial. To achieve this, Content Delivery Networks (CDNs) are often used — but at a smaller scale, Nginx can act as a **caching proxy** to serve static content efficiently. This guide walks through how to use Nginx to cache static content with a Dockerized setup and ties the example to real-world scenarios.

---

### ❓ What is a Caching Proxy?
A **Caching Proxy** stores a copy of the server response (like HTML or images) and serves it to future requests without contacting the backend server again — until the cache expires. This reduces latency, saves bandwidth, and lightens backend load.

---

### 💡 Why Use a Caching Proxy?
- 🚀 Faster load times
- 🔄 Reduce repeated backend hits
- 📉 Decrease bandwidth usage
- 🔒 Improve reliability during backend outages
- 🌐 Acts like a mini-CDN

---

### 🕰️ Brief History / Real-World Connection
- CDNs like Cloudflare, Akamai, and CloudFront are large-scale caching proxies.
- Even modern browsers cache static content locally (browser cache).
- Nginx, originally built as a high-performance HTTP server, became popular in part due to its robust caching capabilities.

Example real-world uses:
- Caching blog/static pages
- Serving assets from edge servers
- Local dev testing of cache behaviors

---

### ⚙️ Our Example: Caching Static Content Using Nginx

#### 🧱 Project Structure
```
nginx-static-cache/
├── docker-compose.yml
├── nginx.conf
└── static-site/
    └── index.html
```

#### 📝 `static-site/index.html`
```html
<!DOCTYPE html>
<html>
<head>
  <title>Hello from Cached Static Site</title>
</head>
<body>
  <h1>This is a cached static page!</h1>
  <p>Timestamp: <span id="timestamp"></span></p>
  <script>
    document.getElementById('timestamp').innerText = new Date().toString();
  </script>
</body>
</html>
```

---

### 🐳 Docker Compose Breakdown
```yaml
version: '3'

services:
  static-site:
    image: nginx:alpine
    volumes:
      - ./static-site:/usr/share/nginx/html:ro

  nginx-cache:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - nginx-cache:/var/cache/nginx

volumes:
  nginx-cache:
```

- `static-site` serves the static HTML via basic Nginx
- `nginx-cache` is the main caching proxy in front of it
- Shared volume `nginx-cache` stores cached responses

---

### 🔧 Nginx Config Breakdown
```nginx
http {
  proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=STATICCACHE:10m inactive=1m max_size=50m;

  server {
    listen 80;

    location / {
      proxy_pass http://static_backend;
      proxy_cache STATICCACHE;
      proxy_cache_valid 200 302 10m;
      proxy_cache_valid 404 1m;
      add_header X-Cache-Status $upstream_cache_status;
    }
  }

  upstream static_backend {
    server static-site:80;
  }
}
```

- `proxy_cache_path` defines cache store location, limits, and keys
- `proxy_cache` enables caching
- `proxy_cache_valid` sets TTL (how long content stays cached)
- `X-Cache-Status` header tells us if it was a `HIT` or `MISS`

---

### ▶️ Testing It
```bash
docker-compose up -d
```

Visit:
```
http://localhost:8080/index.html
```

Check network response headers — look for:
```
X-Cache-Status:

MISS on first load

HIT on subsequent loads


```

---

### 📚 Resources
- [Nginx Caching Docs](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache)
- [Cloudflare CDN Caching Guide](https://developers.cloudflare.com/cache/)

---

This is a working and realistic CDN-like setup using Nginx as a caching proxy. It helps mimic production behavior locally and understand how static caching can speed up our services drastically.

