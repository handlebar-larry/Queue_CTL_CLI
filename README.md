# Queue_CTL_CLI

queuectl: A Command-Line Job Queue Manager queuectl is a CLI tool for handling a persistent, background job queue. Built with Node.js and powered by BullMQ and Redis, it lets you enqueue standard shell commands to be processed as background jobs.

The system is designed for resilience, featuring automatic retries with exponential backoff and a Dead Letter Queue (DLQ) for jobs that repeatedly fail.

Key Features Persistent Job Queues: All jobs are stored in Redis, ensuring they persist through application or server restarts.

Parallel Worker Pooling: Execute multiple jobs concurrently by running a pool of worker processes.

Automatic Job Retries: Jobs that fail are automatically retried using an exponential backoff strategy.

Dead Letter Queue (DLQ): Jobs that exhaust all retry attempts are moved to a separate "dead" queue for manual review.

Complete CLI Management: Enqueue jobs, start/stop workers, check queue status, and manage the DLQ, all from the command line.

System Requirements Before you begin, your system must have the following dependencies installed.

Node.js This project requires Node.js. A current LTS version (v18 or newer) is recommended.

Download Node.js

Redis Server Redis serves as the backend database and message broker for BullMQ. It is a mandatory component. The redis-server command must be available in your path.

On Fedora: sudo dnf install redis

On Ubuntu/Debian: sudo apt install redis-server

On macOS (Homebrew): brew install redis

PM2 (Process Manager) PM2 is used by the CLI to manage the worker processes in the background, allowing them to run persistently.

Install it globally: npm install pm2 -g

🚀 Installation and Configuration Clone the repository:

Bash

git clone https://github.com/your-username/queuectl.git cd queuectl Install NPM dependencies:

Bash

npm install Link the CLI: This command leverages the bin field in package.json to make queuectl globally accessible from any directory.

Bash

npm link Create the .env File: The CLI uses environment variables to find your Redis instance. Create a .env file in the project's root directory.

Bash

touch .env Add the following content to the new .env file. These are the default values, matching the provided redis.conf.

Ini, TOML

REDIS_HOST=127.0.0.1 REDIS_PORT=6379 Configure Redis Persistence: queuectl is designed to run Redis with a custom configuration file for data persistence. a. Create a file named redis.conf in the project's root (queuectl) folder. b. Paste the following configuration into it:

Code snippet

--- Persistence ---
Save the DB to a file named...
dbfilename dump.rdb

Save the file in this specific directory.
YOU MUST UPDATE THIS PATH
dir /home/jynt/Coding/FLAM/redis-data

--- General ---
port 6379 logfile "redis.log" c. ⚠️ Important: You must change the dir path to an absolute path on your machine.

Find the correct path by navigating to the redis-data folder and running pwd.

Update the line to: dir /your/absolute/path/to/queuectl/redis-data

🏃 Running the System This system needs two distinct processes running in the background. You will need three terminal sessions.

Terminal 1: Launch Redis Always start Redis from the project directory to ensure it loads the correct redis.conf file, which enables persistence.

Bash

Navigate to the queuectl project folder
cd /path/to/queuectl

Start Redis with the local configuration
redis-server ./redis.conf Terminal 2: Launch the Workers In a second terminal (also in the project folder), begin the worker pool.

Bash

cd /path/to/queuectl

Start 3 background worker processes
queuectl worker start --count 3 Terminal 3: Use the CLI You can now manage your queue from any terminal.

Bash

Enqueue a new job
queuectl enqueue '{"id":"job1", "command":"echo Hello World"}'

Check the queue status
queuectl status 📋 CLI Command Guide The following is a complete list of available queuectl commands.

Manage Workers queuectl worker start --count <n> Launches <n> background worker processes via PM2. If workers are active, this command restarts them with the specified count.

Bash

queuectl worker start --count 4 queuectl worker stop Performs a graceful shutdown and removes all running workers from PM2.

Bash

queuectl worker stop queuectl logs Tails the live logs from all active workers directly to your terminal. (Use Ctrl+C to exit).

Manage Jobs queuectl enqueue <jobJson> Enqueues a new job. The JSON payload must include a command. You can also optionally specify an id and max_retries. If max_retries is omitted, the default from queuectl config is used.

Bash

A simple job
queuectl enqueue '{"id":"job-echo", "command":"echo Hello"}'

A job with a specific retry limit
queuequeuectl enqueue '{"id":"job-fail", "command":"ls /badpath", "max_retries": 2}' queuectl status Displays an overview of all job states (Pending, Processing, Completed, Failed, etc.).

queuectl list --state <state> Retrieves a list of all jobs in a given state.

States: pending, processing (active), completed, dead (failed), retry (delayed)

Bash: queuectl list --state completed queuectl list --state dead queuectl job status <jobId> Fetches the current state and detailed information for a specific job, including its timestamps and any results.

Bash :queuectl job status job-echo

Manage the Dead Letter Queue (DLQ) The DLQ (Dead Letter Queue) stores jobs that have permanently failed.

queuectl dlq list Shows all jobs that have exhausted their retries and are now in the DLQ.

queuectl dlq retry <jobId> Resubmits a specific job from the DLQ back into the pending queue, allowing workers to attempt it again.

Manage Configuration queuectl config set <key> <value> Writes a setting to the .queuectl.config.json file. This is currently used for setting the default maxRetries for new jobs.

Bash :queuectl config set max-retries 5 queuectl config list Displays all current settings from the .queuectl.config.json file.
Bash :queuectl config set max-retries 5
queuectl config list Displays all current settings from the .queuectl.config.json file.
