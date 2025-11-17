# travel-backend-main — quick Docker run & test

This repo contains the Travel management Spring Boot backend. Below are quick commands to run the project in Docker (no volumes) and a small test page.

1) Build the image (from project root):

```powershell
docker build -t travelmanagement:latest .
```

2) Start a MySQL container and the app (creates a Docker network and runs both containers):

```powershell
# create network if missing
if (-not (docker network ls --format '{{.Name}}' | Select-String -Pattern '^travel-net$' -Quiet)) { docker network create travel-net }

# run MySQL
docker run -d --name traveldb --network travel-net -e MYSQL_ROOT_PASSWORD=root -e MYSQL_DATABASE=travelmanagement mysql:8.0
Start-Sleep -Seconds 15

# run the app, pointing it at the DB and mapping host 8082 -> container 8081
docker run -d --name travelbackend --network travel-net -e SPRING_DATASOURCE_URL="jdbc:mysql://traveldb:3306/travelmanagement" -e SPRING_DATASOURCE_USERNAME=root -e SPRING_DATASOURCE_PASSWORD=root -p 8082:8081 travelmanagement:latest
```

3) Tail logs and test signup endpoint (PowerShell):

```powershell
docker logs --tail 200 travelbackend
Invoke-RestMethod -Uri http://localhost:8082/auth/signup -Method Post -ContentType 'application/json' -Body '{"username":"testuser","email":"test@example.com","password":"pass"}'
```

4) Quick browser test: open `post_signup.html` (saved at repo root) and click "Send signup".

Notes:
- The Dockerfile already performs a multi-stage build producing a runnable jar.
- No volumes are used by the commands above.
