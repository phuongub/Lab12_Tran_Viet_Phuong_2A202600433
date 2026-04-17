# Day 12 Lab - Mission Answers

## Part 1: Localhost vs Production

### Exercise 1.1: Anti-patterns found in `01-localhost-vs-production/develop/app.py`
1. API keys and sensitive parameters are hardcoded in the source code.
2. The port number is fixed (defaulting directly to 8000), not driven by an environment variable.
3. The server runs in debug mode implicitly.
4. No health check endpoint is defined for cloud platforms/load balancers to monitor the application.
5. No graceful shutdown logic is implemented to safely finish active requests when the server stops.

### Exercise 1.3: Comparison table
| Feature        | Develop                | Production                            | Why Important?                                                                                       |
|----------------|------------------------|---------------------------------------|------------------------------------------------------------------------------------------------------|
| **Config**     | Hardcoded              | Environment Variables (`.env`)        | Keeps secrets secure, prevents accidental leaks, and allows switching environments easily.           |
| **Health Check**| None                   | `/health` & `/ready` endpoints      | Orchestrators need this to determine application availability, distribute traffic, and handle restarts. |
| **Logging**    | Quick `print()`        | Structured JSON Log                   | JSON logs can be parsed and indexed efficiently by centralized monitoring tools (DataDog, ELK).      |
| **Shutdown**   | Immediate (Ctrl+C)     | Graceful Shutdown (SIGTERM handling)  | Ensures in-flight database actions & client requests complete properly without data corruption.      |

## Part 2: Docker

### Exercise 2.1: Dockerfile questions
1. **Base image:** `python:3.11`
2. **Working directory:** `/app`
3. **Why COPY requirements.txt first?:** To take advantage of Docker's layer caching mechanism. Dependencies are usually large and take a while to download. By isolating this step, the cache is only invalidated when dependencies change, radically speeding up sequential builds when only source code changes.
4. **CMD vs ENTRYPOINT:** `CMD` provides default options/commands that can easily be overridden from the docker run CLI, while `ENTRYPOINT` configures a stable executable environment where `CMD` arguments are simply appended.

### Exercise 2.3: Image size comparison
- **Develop (`python:3.11` base):** ~1.66 GB
- **Production (`python:3.11-slim` multi-stage):** ~236 MB
- **Difference:** ~86% reduction in size.

## Part 3: Cloud Deployment

### Exercise 3.1: Railway deployment
- **URL:** https://lab11-part3-production.up.railway.app
- **Screenshot:** 

## Part 4: API Security

### Exercise 4.1-4.3: Test results
LEGION@DESKTOP-VEVODKL MINGW64 ~/day12_ha-tang-cloud_va_deployment (main)
$ curl http://localhost:8000/ask -X POST \
  -H "Content-Type: application/json" \
  -d '{"question": "Hello"}'
{"detail":"Missing API key. Include header: X-API-Key: <your-key>"}

LEGION@DESKTOP-VEVODKL MINGW64 ~/day12_ha-tang-cloud_va_deployment (main)
$ curl http://localhost:8000/ask -X POST   -H "X-API-Key: demo-key-change-in-production"   -H "Content-Type: application/json"   -d '{"question": "Hello"}'
{"detail":[{"type":"missing","loc":["query","question"],"msg":"Field required","input":null}]}






LEGION@DESKTOP-VEVODKL MINGW64 ~/day12_ha-tang-cloud_va_deployment/04-api-gateway/production (main)
$     curl -X POST http://localhost:8000/auth/token \ 
         -H "Content-Type: application/json" \ 
         -d '{"username": "student", "password": "demo123"}'
{"access_token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdHVkZW50Iiwicm9sZSI6InVzZXIiLCJpYXQiOjE3NzY0MjIwNzMsImV4cCI6MTc3NjQyNTY3M30.cGH0RaGKffTGfcnOOwcxuV4X3N21VyD_1TwgWCgGTzk","token_type":"bearer","expires_in_minutes":60,"hint":"Include in header: Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."}

LEGION@DESKTOP-VEVODKL MINGW64 ~/day12_ha-tang-cloud_va_deployment/04-api-gateway/production (main)
$ TOKEN="eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdHVkZW50Iiwicm9sZSI6InVzZXIiLCJpYXQiOjE3NzY0MjIwNzMsImV4cCI6MTc3NjQyNTY3M30.cGH0RaGKffTGfcnOOwcxuV4X3N21VyD_1TwgWCgGTzk"
curl http://localhost:8000/ask -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"question": "Explain JWT"}'
{"question":"Explain JWT","answer":"Tôi là AI agent được deploy lên cloud. Câu hỏi của bạn đã được nhận.","usage":{"requests_remaining":9,"budget_remaining_usd":1.9e-05}}





LEGION@DESKTOP-VEVODKL MINGW64 ~/day12_ha-tang-cloud_va_deployment (main)
$ for i in {1..20}; do
  curl http://localhost:8000/ask -X POST \
    -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJzdHVkZW50Iiwicm9sZSI6InVzZXIiLCJpYXQiOjE3NzY0MjI5NTgsImV4cCI6MTc3NjQyNjU1OH0.ndx4pbIugyeUtxPz9uLLQaFsFSIINBggAPmeksmpkR8" \
    -H "Content-Type: application/json" \
    -d '{"question": "Test '$i'"}'
  echo ""
done
{"question":"Test 1","answer":"Đây là câu trả lời từ AI agent (mock). Trong production, đây sẽ là response từ OpenAI/Anthropic.","usage":{"requests_remaining":9,"budget_remaining_usd":2.1e-05}}
{"question":"Test 2","answer":"Tôi là AI agent được deploy lên cloud. Câu hỏi của bạn đã được nhận.","usage":{"requests_remaining":8,"budget_remaining_usd":4e-05}}
{"question":"Test 3","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":7,"budget_remaining_usd":5.6e-05}}
{"question":"Test 4","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":6,"budget_remaining_usd":7.2e-05}}
{"question":"Test 5","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":5,"budget_remaining_usd":8.8e-05}}
{"question":"Test 6","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":4,"budget_remaining_usd":0.000104}}
{"question":"Test 7","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":3,"budget_remaining_usd":0.000121}}
{"question":"Test 8","answer":"Đây là câu trả lời từ AI agent (mock). Trong production, đây sẽ là response từ OpenAI/Anthropic.","usage":{"requests_remaining":2,"budget_remaining_usd":0.000142}}
{"question":"Test 9","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":1,"budget_remaining_usd":0.000158}}
{"question":"Test 10","answer":"Agent đang hoạt động tốt! (mock response) Hỏi thêm câu hỏi đi nhé.","usage":{"requests_remaining":0,"budget_remaining_usd":0.000174}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":57}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":56}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":56}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":56}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":56}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":55}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":55}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":55}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":55}}
{"detail":{"error":"Rate limit exceeded","limit":10,"window_seconds":60,"retry_after_seconds":54}}

### Exercise 4.4: Cost guard implementation
**Explanation:** 
To prevent abuse, we implemented budget logic. Since the app needs to be strictly stateless for scaling, we use Redis as the central data store to track user usage. We maintain a specific Redis key mapped per user per month (`budget:{user_id}:{month_key}`). For each incoming request, the function fetches their current spending, adds the estimated token cost of the requested LLM execution, and checks if it exceeds the limit (e.g. $10). If so, we reject it. If not, the application executes the prompt, increments the Redis counter value, and allows the request. Setting an expiration timer (TTL) on the Redis key ensures previous months' datasets are automatically cleaned up.

## Part 5: Scaling & Reliability

### Exercise 5.1-5.5: Implementation notes
**Explanation:**
- **5.1 Health Checks:** Added `/health` (Liveness) and `/ready` (Readiness) endpoints. Liveness tests prove the process hasn't hanged. Readiness actively checks critical backing dependencies (like Redis/Databases) to inform load balancers exactly when traffic validation can safely pass through.
- **5.2 Graceful Shutdown:** Using `signal.signal(signal.SIGTERM)` allows the app to intercept the platform environment's kill signal. It gives the application lifecycle time to process current active inflight requests and cleanly sever network integrations and DB connections before safely exiting the container.
- **5.3 Stateless Design:** Migrating shared configurations and memory objects (like conversation history variables) from local machine mapping (e.g., `conversation_history = {}`) into an external database caching layer like Redis is mandatory. Without it, replication on separate cloned containers will not share memory variables, breaking session continuity.
- **5.4 Load Balancing:** Wrapping Nginx directly in front of `app:8000` nodes natively enables routing and distributing client requests. Modifying orchestration logic via Docker Compose's `--scale agent=3` lets us create multiple clone containers behind Nginx to handle unexpected connection bursts and remove single points of failure.
- **5.5 Testing Statelessness:** Simulating random failure testing by firing consecutive requests while intentionally killing individual node instances verifies that requests seamlessly re-route to surviving instances, maintaining the exact conversation state and demonstrating solid reliability properties.
