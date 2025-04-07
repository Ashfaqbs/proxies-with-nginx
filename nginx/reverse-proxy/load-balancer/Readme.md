## Nginx as a Reverse Proxy Load Balancer

### 📘 Introduction
Nginx is a powerful web server that can also be used as a reverse proxy, load balancer, and HTTP cache. One of the most common uses of Nginx is to act as a reverse proxy that distributes incoming client requests across multiple backend servers, improving reliability and scalability.

In this guide, we document how to use **Nginx as a reverse proxy load balancer**, demonstrate a real-world working example using Docker and a Spring Boot REST API, and explain different load balancing algorithms with use cases.

---

### 🔧 What is a Reverse Proxy Load Balancer?
- **Reverse Proxy:** Forwards client requests to backend servers and returns the server’s response to the client. The client doesn’t know which server actually handled the request.
- **Load Balancer:** Distributes traffic across multiple servers to optimize resource usage, reduce latency, and ensure fault tolerance.

Together, Nginx acts as a gateway that forwards requests to backend services while balancing the traffic.

---

### 💡 Why Use Nginx for Load Balancing?
- Lightweight and fast
- Simple to configure
- Supports multiple algorithms
- Great for horizontal scaling
- Built-in health checks (with Nginx Plus)

---

### ⚙️ Example Setup: Load Balancing Spring Boot REST API with Nginx

#### 🧱 Project Components
- **Dockerized Spring Boot REST API image:** `darksharkash/simplerestapisb-k8s:latest`
- **Exposes:** `GET /helloworld` on port `8080`
- **Three containers running the same image**
- **Nginx container** acting as reverse proxy on port `8088`

#### 📝 Docker Compose Configuration
```yaml


services:
  spring-app1:
    image: darksharkash/simplerestapisb-k8s:latest
    ports:
      - "8081:8080"
    restart: unless-stopped

  spring-app2:
    image: darksharkash/simplerestapisb-k8s:latest
    ports:
      - "8082:8080"
    restart: unless-stopped

  spring-app3:
    image: darksharkash/simplerestapisb-k8s:latest
    ports:
      - "8083:8080"
    restart: unless-stopped

  nginx-reverse:
    image: nginx:latest
    ports:
      - "8088:8080"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    restart: unless-stopped
    depends_on:
      - spring-app1
      - spring-app2
      - spring-app3



```

#### 🔧 Nginx Config (nginx.conf)
```nginx
http {
  upstream backend {
    server spring-app1:8080;
    server spring-app2:8080;
    server spring-app3:8080;
  }

  server {
    listen 80;

    location / {
      proxy_pass http://backend;
    }
  }
}
```

Now run the system:
```bash
docker-compose up -d
```

Send requests to:
```
curl http://localhost:8088/helloworld
```

Watch round-robin balancing across app1, app2, app3.

---

### 🔁 Load Balancing Algorithms in Nginx

#### 1. Round Robin (Default)
- Requests rotate through each server sequentially.
- Best for general use.
- No special config needed.

#### 2. Least Connections
- Routes to server with the fewest active connections.
- Useful for long-running or resource-heavy requests.
```nginx
upstream backend {
  least_conn;
  server spring-app1:8080;
  server spring-app2:8080;
  server spring-app3:8080;
}
```

#### 3. IP Hash
- Routes a client IP to the same server consistently.
- Best for session stickiness.
```nginx
upstream backend {
  ip_hash;
  server spring-app1:8080;
  server spring-app2:8080;
  server spring-app3:8080;
}
```

#### 4. Hash (Nginx Plus Only)
- Advanced custom key-based hashing.

---

### 📌 When to Use Which Algorithm
| Algorithm        | Best Use Case                                 |
|------------------|-----------------------------------------------|
| Round Robin      | Equal load, stateless services                |
| Least Connections| Uneven workloads or varying response times    |
| IP Hash          | Session stickiness (e.g., user login sessions)|
| Hash (Plus)      | Custom routing logic                          |

---

### 📊 Monitoring the Load Balancing
To check which server handles a request:
- Add instance-specific info in `/helloworld`
- Tail logs:
```bash
docker logs -f spring-app1
```

---

### 📚 Resources
- [Nginx Load Balancing Docs](https://nginx.org/en/docs/http/load_balancing.html)
- [Spring Boot Docker Guide](https://spring.io/guides/gs/spring-boot-docker/)
- [Docker Compose Docs](https://docs.docker.com/compose/)
- [DigitalOcean Nginx Proxy Example](https://www.digitalocean.com/community/tutorials/how-to-set-up-nginx-load-balancing)

---