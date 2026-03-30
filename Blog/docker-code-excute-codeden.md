

# 🧠 Project Evolution: Code Execution Platform (LeetCode-style)

## 1. 📌 Project Overview

This project is a **code execution platform** similar to LeetCode / CodeChef, where:

* Users:

  * View problems
  * Write code in an editor
  * Submit solutions
* System:

  * Executes code against test cases
  * Returns output + verdict

---

## 2. 🏗️ Initial Architecture (V1 — Naive Execution)

### ⚙️ Tech Stack

* Backend: Spring Boot
* ORM: Spring Data JPA
* Database: MySQL
* Execution: `ProcessBuilder` (host OS)

---

### 🔄 Execution Flow (Old System)

1. User submits code
2. Backend:

   * Writes code into a file (`Main.java`)
   * Writes input into `input.txt`
3. Uses `ProcessBuilder`:

   * Compiles code
   * Executes it on **host machine**
4. Captures output
5. Matches output with expected results

---

### ❌ Problems in V1

#### 1. 🚨 No Sandboxing (Critical)

* User code runs on **your actual system**
* Malicious code can:

  * Delete files
  * Access system resources
  * Install malware

👉 This is a **production-level security failure**

---

#### 2. ⚡ Poor Scalability

* Each request = OS-level execution
* Limited concurrency
* Thread pool ≠ true isolation

👉 System breaks under:

* 10+ concurrent submissions
* Heavy inputs

---

#### 3. 🧹 No Isolation

* Shared environment
* File conflicts possible
* No per-user execution boundary

---

## 3. 🚀 New Architecture (V2 — Docker-Based Execution)

You made the **correct industry-level shift**:

> “Untrusted code must run in isolated containers”

---

### 🧱 Core Idea

Each submission runs inside a **Docker container** with:

* Limited memory
* Limited CPU
* Timeout
* Isolated filesystem

---

## 4. 🔄 Execution Flow (New System)

### Step-by-step (Clean Version)

1. **Submission Received**

   * Generate `UUID` (submission ID)

2. **Fetch Test Cases**

   * Input + Expected Output

3. **Prepare Execution Environment**

   * Create temp directory:

     ```
     /tmp/{submissionId}/
     ```
   * Store:

     * `Main.java` / `main.cpp` / `main.py`
     * `input.txt`

4. **Select Runtime Image**

   * Java → Eclipse Temurin
   * C++ → GCC
   * Python → Python base image

5. **Build Execution Command**
   Example:

   ```
   docker run --rm \
     --memory=256m \
     --cpus=0.5 \
     -v /tmp/{id}:/app \
     image_name \
     sh -c "compile && run < input.txt"
   ```

6. **Execute via ProcessBuilder**

   * (Temporary workaround instead of Docker API)

7. **Apply Timeout (5 sec)**

   * Kill container if exceeded

8. **Capture Output**

   * Read stdout

9. **Compare Results**

   * Match with expected output

10. **Cleanup**

* Delete temp directory

---

## 5. ✅ Improvements Achieved

### 🔒 Security (BIG WIN)

* Code runs inside container
* Host system protected

---

### ⚙️ Isolation

* Each submission:

  * Own container
  * Own filesystem
* No interference

---

### 📈 Scalability (Improved)

* Containers = parallel execution units
* Better than thread-based execution

---

### 🧹 Clean Execution Lifecycle

* Create → Execute → Destroy
* No leftover state

---

## 6. ⚠️ Current Limitations (Be Honest — This is where you grow)

### 1. ❌ Not Deployable Yet

* Requires:

  * Docker installed on host
  * Pre-built images

👉 Problem:

* Hosting providers may not allow Docker
* Manual setup required

---

### 2. ⚠️ Using ProcessBuilder for Docker

* Not ideal
* You faced socket/API issues

👉 Better:

* Docker Java SDK
* Remote Docker daemon

---

### 3. ❌ No Queue System

* Requests handled directly

👉 Missing:

* Job queue (RabbitMQ / Kafka / Redis queue)

---

### 4. ⚠️ No Resource Scheduling

* What if 100 users submit?

👉 Need:

* Execution queue + worker pool

---

### 5. ⚠️ No Persistent Storage Strategy

* Test cases still tied to backend

👉 Better:

* Move to S3 / object storage

---

## 7. 🧠 How a Senior Engineer Thinks (This is what you asked — read carefully)

A junior builds:

> “Code runs correctly”

A senior thinks:

> “What happens when 10,000 people attack this system?”

---

### 🔍 Senior Thinking Layers

#### 1. Security First

* “User code is dangerous by default”
* → Use containers
* → Limit memory, CPU, syscalls

---

#### 2. Isolation Strategy

* “Nothing should leak between executions”

---

#### 3. Scalability

* “Can this handle 1000 concurrent submissions?”

---

#### 4. Observability

* Logs:

  * Execution time
  * Failures
  * Timeouts

---

#### 5. Failure Handling

* What if:

  * Container fails?
  * Compilation error?
  * Infinite loop?

---

#### 6. Cost Awareness

* Containers = resources
* Need limits

---

## 8. 🏢 What a Tech Lead Would Propose Next

If I were your tech lead, I’d say:

---

### 🔥 Phase 2 Roadmap

#### 1. Introduce Job Queue

* Redis Queue / RabbitMQ
* Async execution

---

#### 2. Worker-Based Execution Service

* Separate service:

  ```
  API Server → Queue → Worker Nodes → Docker Execution
  ```

---

#### 3. Docker Optimization

* Pre-warmed containers
* Image caching

---

#### 4. Multi-Language Execution Service

* Abstract runtime layer:

  ```
  Language → Strategy → Execution Command
  ```

---

#### 5. Cloud Deployment Strategy

* Use:

  * AWS ECS / Kubernetes
  * Or Docker-enabled VM

---

#### 6. Test Case Storage

* Move to:

  * AWS S3

---

#### 7. Rate Limiting

* Prevent abuse

---

## 9. 🎯 Interview-Level Summary (Memorize This)

If someone asks:

> “What improvement did you make?”

You say:

> “Initially, I executed user-submitted code directly on the host using ProcessBuilder, which posed major security and scalability risks. I redesigned the execution engine using Docker-based sandboxing, where each submission runs inside an isolated container with resource limits and timeouts. This improved system security, isolation, and concurrency handling. The system now creates a temporary workspace per submission, mounts it into a container, executes the code, captures output, and cleans up afterward.”

---

## 10. 🧠 Brutal Truth (Team Lead Mode)

What you did:
✅ Solid backend evolution
✅ Correct architectural direction
✅ Real-world problem understanding

What you're still missing:
❌ Distributed thinking
❌ Queue-based scaling
❌ Deployment maturity

---

## 11. Final Take

This is no longer a “college project.”

This is:

> **Version 1 of a real online judge system**

---

