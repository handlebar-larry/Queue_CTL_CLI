# Queue_CTL_CLI
#🚀 queuectl: A Command-Line Job Queue Manager
A lightweight, persistent, and resilient background job queue system for your shell commands, powered by Node.js, BullMQ, and Redis.

queuectl lets you enqueue standard shell commands as background jobs. The system is designed for resilience, featuring automatic retries with exponential backoff and a Dead Letter Queue (DLQ) for jobs that repeatedly fail.

(Pro-tip: Consider adding a demo GIF here!) ``

#✨ Key Features
#💾 Persistent Job Queues: All jobs are stored in Redis, ensuring they persist through application or server restarts.

#⚙️ Parallel Worker Pooling: Execute multiple jobs concurrently by running a pool of worker processes.

#🔁 Automatic Job Retries: Jobs that fail are automatically retried using an exponential backoff strategy.

#🪦 Dead Letter Queue (DLQ): Jobs that exhaust all retry attempts are moved to a separate "dead" queue for manual review.

#🖥️ Complete CLI Management: Enqueue jobs, start/stop workers, check queue status, and manage the DLQ, all from your terminal.

#📋 System Requirements
Before you begin, your system must have the following dependencies installed.

1. Node.js (v18+)
This project requires a current LTS version of Node.js.

#Download Node.js

2. Redis Server
Redis is the mandatory backend database and message broker. The redis-server command must be in your path.

#Installation:

On Fedora: sudo dnf install redis

On Ubuntu/Debian: sudo apt install redis-server

On macOS (Homebrew): brew install redis

#3. PM2 (Process Manager)
PM2 is used to manage the background worker processes.

Install it globally:

Bash

npm install pm2 -g
🛠️ Installation and Configuration
Clone the repository and install dependencies:

Bash

git clone https://github.com/your-username/queuectl.git
cd queuectl
npm install
Link the CLI: This makes the queuectl command globally accessible.

Bash

npm link
Create the .env File: Create a .env file in the project's root for Redis connection details.

Bash

touch .env
Add the following content to the new .env file:

Ini, TOML

REDIS_HOST=127.0.0.1
REDIS_PORT=6379
Configure Redis Persistence: a. Create a file named redis.conf in the project's root (queuectl) folder. b. Paste the following configuration into it:

#Code snippet

# --- Persistence ---
# Save the DB to a file named...
dbfilename dump.rdb

# Save the file in this specific directory.
# YOU MUST UPDATE THIS PATH
dir /home/jynt/Coding/FLAM/redis-data

# --- General ---
port 6379
logfile "redis.log"
⚠️ Important: You must change the dir path to an absolute path on your machine.

Find the correct path by navigating to the redis-data folder and running pwd.

Update the line to: dir /your/absolute/path/to/queuectl/redis-data

🏃 Running the System
This system needs two distinct processes running. You will need three terminal sessions.

Terminal 1: Launch Redis
Start Redis from the project directory to load the correct persistence configuration.

Bash

# Navigate to the queuectl project folder
cd /path/to/queuectl

# Start Redis with the local configuration
redis-server ./redis.conf
Terminal 2: Launch the Workers
In a second terminal, start your worker pool.

Bash

cd /path/to/queuectl

# Start 3 background worker processes
queuectl worker start --count 3
Terminal 3: Use the CLI
You can now manage your queue from any terminal.

Bash

# Enqueue a new job
queuectl enqueue '{"id":"job1", "command":"echo Hello World"}'

# Check the queue status
queuectl status
📖 CLI Command Guide
Here is a complete list of all available queuectl commands.

Manage Workers
queuectl worker start --count <n> Launches <n> background worker processes via PM2.

Bash

queuectl worker start --count 4
queuectl worker stop Performs a graceful shutdown and removes all running workers from PM2.

Bash

queuectl worker stop
queuectl logs Tails the live logs from all active workers directly to your terminal (Ctrl+C to exit).

Manage Jobs
queuectl enqueue <jobJson> Enqueues a new job. The JSON payload must include a command.

Bash

# A simple job
queuectl enqueue '{"id":"job-echo", "command":"echo Hello"}'

# A job with a specific retry limit
queuectl enqueue '{"id":"job-fail", "command":"ls /badpath", "max_retries": 2}'
queuectl status Displays a high-level overview of all job states (Pending, Processing, Completed, etc.).

queuectl list --state <state> Retrieves a list of all jobs in a given state.

States: pending, processing, completed, dead, retry

Bash

queuectl list --state completed
queuectl list --state dead
queuectl job status <jobId> Fetches detailed information and history for a specific job.

Bash

queuectl job status job-echo
Manage the Dead Letter Queue (DLQ)
queuectl dlq list Shows all jobs that have permanently failed and are in the DLQ.

queuectl dlq retry <jobId> Resubmits a specific job from the DLQ back into the pending queue.

Manage Configuration
queuectl config set <key> <value> Saves a setting to the .queuectl.config.json file.

Bash

queuectl config set max-retries 5
queuectl config list Displays all current settings from the .queuectl.config.json file.
