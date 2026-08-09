Production deployment checklist

1. Build frontend for production

- From the `frontend` directory:

```bash
npm install --legacy-peer-deps
npm run build
```

2. Production `Dockerfile` for frontend (multi-stage)

Replace `frontend/Dockerfile` with the following for production static serving via nginx:

```dockerfile
# Build stage
FROM node:18-alpine as build
WORKDIR /app
COPY package*.json ./
RUN npm install --legacy-peer-deps
COPY . .
RUN npm run build

# Production stage
FROM nginx:stable-alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

Set `REACT_APP_API_URL` at build time if needed by replacing API URL in code or using runtime configuration.

3. Backend Dockerfile

The existing `backend/Dockerfile` is suitable for production. Ensure `DATABASE_URL` and other secrets are provided via environment or Docker secrets.

4. docker-compose (production)

Create or adjust a `docker-compose.prod.yml` that references built images or uses `build:` with production Dockerfiles. Example:

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: roommate_matcher
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    environment:
      - DATABASE_URL=postgresql://user:password@postgres:5432/roommate_matcher
    ports:
      - "8000:8000"
    depends_on:
      - postgres

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

5. Build and run with Docker Compose (on a machine with Docker installed):

```bash
# build
docker-compose -f docker-compose.prod.yml up --build -d

# check logs
docker-compose -f docker-compose.prod.yml logs -f
```

6. Alternative: Deploy to Render / Heroku / DigitalOcean App Platform

- Build Docker images locally or via CI and push to a registry (Docker Hub, GitHub Container Registry).
- Create services in the cloud provider pointing to the image tags and set environment variables.

7. Notes for this environment

- I built the frontend (`npm run build`) and installed backend dependencies locally.
- I started the backend locally with `uvicorn` for verification.
- I could not run Docker here because the `docker` CLI is not available on this machine; follow the `docker-compose` steps above on a machine with Docker or in CI.

If you want, I can:
- Update the `frontend/Dockerfile` to the multi-stage production version and commit it.
- Create a `docker-compose.prod.yml` in the repo.
- Produce GitHub Actions workflow that builds images and pushes to a registry.

Tell me which of those you'd like me to do next.