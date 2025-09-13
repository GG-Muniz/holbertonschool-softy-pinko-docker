## Softy Pinko Docker – Reverse Proxy + Load Balancing Lab

End-to-end Docker exercise building a small web app with:
- static front-end served by Nginx
- Python/Flask API
- Nginx reverse proxy in front
- Docker Compose orchestration
- Horizontal scaling of the API via Round Robin load balancing

### Architecture (high-level)
- A single proxy server (Nginx) is the public entry point.
- Requests to `/` are routed to the front-end static server (Nginx).
- Requests to `/api/*` are routed to the Flask API service.
- With multiple API replicas, Nginx distributes requests using Round Robin.

### What is Round Robin load balancing?
Round Robin sends each incoming request to the next server in a rotating order. With servers A, B, C: 1st → A, 2nd → B, 3rd → C, 4th → A, and so on. This balances traffic evenly and is sufficient for this learning project.

## Prerequisites
- Install Docker Desktop
- Git installed locally
- Recommended: modern terminal (Windows PowerShell or cmd)

If you see Python “EXTERNALLY-MANAGED” errors when installing packages in images, the Dockerfiles already remove that marker before pip installs. No action needed.

## Repository layout
- `task0/` – First Docker image (Ubuntu) that echoes “Hello, World!”
- `task1/` – Single Flask API container on port 5252
- `task2/` – Split into `back-end/` (Flask) and `front-end/` (Nginx static site)
- `task3/` – Front-end consumes API; CORS enabled on Flask; dynamic content in HTML
- `task4/` – Docker Compose for front-end and back-end
- `task5/` – Adds Nginx proxy in front; front-end and back-end are internal-only; proxy maps `80:80`
- `task6/` – Scale API horizontally; includes `2-api-servers.txt` with the exact scaling command

## Task-by-task

### Task 0 – First Docker image
Build and run:
```bash
docker build -f ./task0/Dockerfile -t softy-pinko:task0 ./task0
docker run -it --rm --name softy-pinko-task0 softy-pinko:task0
```
Expected output: `Hello, World!`

### Task 1 – Back-end (Flask)
Build and run:
```bash
docker build -f ./task1/Dockerfile -t softy-pinko:task1 ./task1
docker run -p 5252:5252 -it --rm --name softy-pinko-task1 softy-pinko:task1
```
Visit: `http://localhost:5252/api/hello` → `Hello, World!`

### Task 2 – Front-end (Nginx) + Back-end structure
Back-end build/run:
```bash
docker build -f ./task2/back-end/Dockerfile -t softy-pinko-back-end:task2 ./task2/back-end
docker run -p 5252:5252 -it --rm --name softy-pinko-back-end-task2 softy-pinko-back-end:task2
```
Front-end build/run:
```bash
docker build -f ./task2/front-end/Dockerfile -t softy-pinko-front-end:task2 ./task2/front-end
docker run -p 9000:9000 -it --rm --name softy-pinko-front-end-task2 softy-pinko-front-end:task2
```
Visit front-end: `http://localhost:9000`

### Task 3 – Connect front-end and back-end
- Front-end HTML includes a new `<h1 id="dynamic-content"></h1>` before the main headline.
- jQuery AJAX script near the end of `index.html` queries `http://localhost:5252/api/hello` and inserts the response text.
- Back-end installs `flask-cors` and enables `CORS(app)` so cross-origin requests succeed.

Build and run (two terminals):
```bash
# Terminal A (back-end)
docker build -f ./task3/back-end/Dockerfile -t softy-pinko-back-end:task3 ./task3/back-end
docker run -p 5252:5252 -it --rm --name softy-pinko-back-end-task3 softy-pinko-back-end:task3

# Terminal B (front-end)
docker build -f ./task3/front-end/Dockerfile -t softy-pinko-front-end:task3 ./task3/front-end
docker run -p 9000:9000 -it --rm --name softy-pinko-front-end-task3 softy-pinko-front-end:task3
```
Visit front-end: `http://localhost:9000` (dynamic header should show `Hello, World!`)

### Task 4 – Docker Compose (front-end + back-end)
In `task4/docker-compose.yml`, two services (`front-end`, `back-end`). Ports mapped to host for both.
```bash
cd task4
docker-compose build
docker-compose up
```
Visit: `http://localhost:9000` (front-end), API at `http://localhost:5252/api/hello`

### Task 5 – Proxy server (Nginx reverse proxy)
- New `proxy/` service listens on port 80 and routes:
  - `/` → `front-end:9000`
  - `/api` → `back-end:5252`
- Front-end AJAX changed to call `"/api/hello"` (through the proxy) instead of direct API port.
- In `task5/docker-compose.yml`, only the proxy maps to the host (`80:80`). Front-end and back-end expose ports internally (no host mapping).

Run:
```bash
cd task5
docker-compose build
docker-compose up
```
Visit: `http://localhost` (all traffic flows through proxy)

### Task 6 – Horizontal scaling
Scale the back-end to 2 replicas (Round Robin balancing by Nginx):
```bash
cd task6
docker-compose build
docker-compose up --scale back-end=2
```
The exact command is also saved in `task6/2-api-servers.txt` (ends with a newline as required).

Reload the front-end multiple times and observe alternating requests across API containers in the logs.

## Useful commands
```bash
# Stop and remove containers
docker-compose down

# Rebuild without cache
docker-compose build --no-cache

# Show logs
docker-compose logs -f

# Remove dangling images/containers/networks (be careful!)
docker system prune -f
```

## Troubleshooting
- Port already in use: stop other apps bound to `80`, `9000`, or `5252`.
- Cannot reach API directly when using proxy (task5+): this is expected; access API through `http://localhost/api/hello`.
- CORS errors on task3: ensure you’re running the CORS-enabled back-end and the front-end AJAX URL matches the API port.
- Windows PowerShell glitches: if interactive issues occur, try `cmd`-based commands (e.g., `cmd /c "…"`).

## Credits
- Front-end starter from `atlas-school/softy-pinko-front-end`
- Based on project directives for `holbertonschool-softy-pinko-docker`


