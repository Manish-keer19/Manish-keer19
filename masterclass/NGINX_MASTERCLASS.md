# Nginx Masterclass: Zero to Hero 🚦

## 1. What is Nginx? (The "Why")
Imagine a generic Office Building.
*   **Without Nginx:** Every customer (User) walks directly into the CEO's office (Your Node.js App). The CEO gets overwhelmed answering the phone and door.
*   **With Nginx:** A professional Receptionist sits at the front desk. They answer phones, direct people to the right room, and kick out salespeople. The CEO focuses purely on work.

**Key Concept:** Nginx handles the "traffic". Node.js handles the "logic".

---

## 2. Core Concepts (Beginner)
*   **Reverse Proxy:** Takes a request from the internet and forwards it to an internal server.
*   **Load Balancing:** Spreading visitors across 3 servers so no single server crashes.
*   **Static Serving:** Serving Images/HTML instantly without waking up Node.js.

---

## 3. The Configuration Structure
Located at `/etc/nginx/nginx.conf`

```nginx
events { } # Connection settings

http {
    server {
        listen 80;        # Listen on HTTP
        server_name mywebsite.com;

        # The Rules
        location / {
            return 200 "Hello World";
        }
    }
}
```

---

## 4. Professional Configuration (Intermediate)

### A. Reverse Proxy (The Standard Setup)
```nginx
server {
    listen 80;
    server_name api.myapp.com;

    location / {
        # Pass request to Node.js running on localhost:8080
        proxy_pass http://localhost:8080;
        
        # Forward the real visitor's IP address
        # Otherwise Node.js thinks everyone is "localhost"
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header Host $host;
    }
}
```

### B. Static Files (React/Images)
Nginx serves files 10x faster than Node.js.
```nginx
location /images/ {
    alias /var/www/uploads/;
    expires 30d; # Browser Cache for 30 days
}
```

---

## 5. Advanced Industry Standards 🚀

### 1. SSL/HTTPS (Security)
Never run HTTP in production.
```nginx
server {
    listen 443 ssl http2;
    ssl_certificate /etc/letsencrypt/live/domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain/privkey.pem;
}
```

### 2. Rate Limiting (DDoS Protection)
Stop bots from crashing your API.
```nginx
# Define limit: 10 requests per second per IP
limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;

location /api/ {
    limit_req zone=mylimit burst=20;
}
```

### 3. Load Balancing
Distribute traffic among 3 Node.js servers.
```nginx
upstream backend_cluster {
    server 10.0.0.1;
    server 10.0.0.2;
    server 10.0.0.3;
}

server {
    location / {
        proxy_pass http://backend_cluster;
    }
}
```
