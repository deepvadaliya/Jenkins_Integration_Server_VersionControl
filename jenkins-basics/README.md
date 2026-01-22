## Architecture (What you are building)

* Jenkins Controller (Linux EC2)
* Windows EC2 (Agent / Slave)
* Agent connects **outbound** to controller using `agent.jar`
* Communication happens over **HTTP/WebSocket (8080)**

---

## Prerequisites

You must already have:

* Jenkins running on Linux EC2
* Jenkins accessible in browser:
  `http://<jenkins-public-ip>:8080`
* Windows EC2 instance created
* RDP access to Windows EC2

---

## STEP 1 — Fix Networking (Most common failure point)

On the **Jenkins controller EC2 Security Group**, add inbound rule:

| Type       | Protocol | Port | Source    |
| ---------- | -------- | ---- | --------- |
| Custom TCP | TCP      | 8080 | 0.0.0.0/0 |

Test from Windows browser:

```
http://<jenkins-ip>:8080
```

Jenkins dashboard must open.

---

## STEP 2 — Install Java on Windows Agent

Jenkins agents require Java (same version as controller is best practice).

Controller used Java 21, so install Java 21 on Windows.

### Recommended:

* Download **Eclipse Temurin JDK 21 (x64 MSI)**
* During install select:

  * Install for all users
  * Add to PATH
  * Set JAVA_HOME

Verify:

```
java -version
```

Must show version 21.x

---

## STEP 3 — Create Agent Node in Jenkins

In Jenkins UI:

```
Manage Jenkins → Nodes → New Node
```

Fill:

* Node name: `window-slave`
* Type: Permanent Agent

Configure:

* Remote root directory:
  `C:\Jenkins`
* Labels:
  `windows`
* Usage:
  Use this node as much as possible
* Launch method:
  **Launch agent by connecting it to the controller**

Click **Save**.

---

## STEP 4 — Get Agent Command

Open your node:

```
Nodes → window-slave
```

Scroll to section:

> **Run from agent command line (Windows)**

You will see two commands like:

```
curl.exe -sO http://<jenkins-ip>:8080/jnlpJars/agent.jar

java -jar agent.jar -url http://<jenkins-ip>:8080/ \
-secret <long-secret> \
-name "window-slave" -webSocket -workDir "C:\Jenkins"
```

These are the exact commands you must run on Windows agent.

---

## STEP 5 — Prepare Windows Folder

On Windows EC2:

Open Command Prompt and run:

```
mkdir C:\Jenkins
cd C:\Jenkins
```

---

## STEP 6 — Download agent.jar

Run:

```
curl.exe -sO http://<jenkins-ip>:8080/jnlpJars/agent.jar
```

Verify:

```
dir
```

You must see:

```
agent.jar
```

If curl fails, download manually from browser:

```
http://<jenkins-ip>:8080/jnlpJars/agent.jar
```

Save file into:

```
C:\Jenkins
```

---

## STEP 7 — Start Agent

Run inside C:\Jenkins:

```
java -jar agent.jar -url http://<jenkins-ip>:8080/ -secret <your-secret> -name "window-slave" -webSocket -workDir "C:\Jenkins"
```

If successful you will see logs like:

```
INFO: WebSocket connection open
INFO: Connected
```

---

## STEP 8 — Confirm in Jenkins UI

Go to:

```
Manage Jenkins → Nodes
```

Your agent should show:

* Status: **Online (Green)**

---

## STEP 9 — Test Agent with Job

Create Freestyle job:

* Configure → Restrict where this project can run
* Label: `windows`

Add Build Step:

```
Execute Windows batch command
```

Example:

```
echo Hello from Windows Agent
java -version
```

Run Build → It should execute on Windows machine.

---

## Common Errors and Fixes

| Problem                      | Cause                  | Fix                           |
| ---------------------------- | ---------------------- | ----------------------------- |
| agent.jar not found          | Not downloaded         | Run curl or download manually |
| Browser cannot open jnlpJars | Port 8080 blocked      | Fix Security Group            |
| java not recognized          | Java not installed     | Install JDK + add PATH        |
| Agent stays offline          | Wrong secret / network | Copy new command from Jenkins |

---

## This setup is now:

* Production valid
* Industry realistic
* Interview strong
* Practically tested
