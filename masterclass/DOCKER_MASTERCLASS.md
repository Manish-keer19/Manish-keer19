# Docker Masterclass: Zero to Hero 🐳

## 1. What is Docker? (The "Why")
Imagine you cooked a perfect meal (App) in your kitchen (Laptop). You want to send it to your friend (Server).
*   **Without Docker:** You send the recipe. Your friend buys ingredients (Node.js, libraries), but buys the wrong brands. The meal tastes bad.
*   **With Docker:** You put the entire finished meal in a **Container**. Your friend just opens it. Tastes exactly the same.

**Key Concept:** Docker eliminates "It works on my machine" problems.

---

## 2. Core Concepts (Beginner)
*   **Dockerfile:** The Recipe. A text file with instructions.
*   **Image:** The Meal. The frozen, read-only package created from the recipe.
*   **Container:** The Eating. A running instance of the image.
*   **Volume:** The Plate. Where you put persistent data (like databases) so it doesn't vanish when the container stops.

---

## 3. Essential Commands (Cheat Sheet)
```bash
# Build an image
docker build -t my-app .

# Run a container (d = background, p = port)
docker run -d -p 8080:80 my-app

# List running containers
docker ps

# Check logs (Debugging)
docker logs <container_id>

# Stop/Remove
docker stop <container_id>
docker rm <container_id>

# Clean up space
docker system prune -a
```

---

## 4. Writing a Professional Dockerfile (Intermediate)
Don't just copy-paste. Understand every line.

```dockerfile
# 1. Base Image: Use Alpine (Tiny Linux, 5MB)
FROM node:18-alpine

# 2. Working Directory: Create a folder inside the container
WORKDIR /app

# 3. Cache Dependencies: Copy package.json FIRST
# Docker builds in layers. If package.json doesn't change, 
# it skips 'npm install' on re-builds. Huge speed boost!
COPY package*.json ./
RUN npm ci --only=production

# 4. Copy Code: Then copy the rest of your app
COPY . .

# 5. Security: Don't run as Root user
USER node

# 6. Start
CMD ["node", "index.js"]
```

---

## 5. Docker Compose (Orchestration)
You rarely run one container. You usually have App + Database + Redis. `docker-compose.yml` runs them all together.

```yaml
version: '3.8'
services:
  backend:
    build: .
    ports: ["3000:3000"]
    environment:
      - DB_HOST=postgres
  
  postgres:
    image: postgres:14
    environment:
      POSTGRES_PASSWORD: secretpassword
    volumes:
      - pg-data:/var/lib/postgresql/data
```
**Run:** `docker-compose up -d`

---

## 6. Advanced Industry Standards 🚀
1.  **Multi-Stage Builds:** Compile TypeScript/React in a heavy image, but ship only the JS/HTML in a tiny Nginx image. (Reduces size 500MB -> 20MB).
2.  **Distroless Images:** Use Google's distroless images (no Shell, no OS tools) for maximum security.
3.  **Health Checks:**
    ```dockerfile
    HEALTHCHECK --interval=30s CMD curl -f http://localhost/ || exit 1
    ```
    This lets Docker auto-restart your app if it freezes.
