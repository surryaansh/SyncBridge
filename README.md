# Notion <--> Trello Two-Way Sync

A reliable event-driven synchronization system that keeps Notion leads and Trello tasks continuously aligned.
Built in Python using FastAPI, Redis, Celery, and real REST APIs.

# Overview

* Lead Tracker: Notion database
* Work Tracker: Trello board
* When a Notion lead is created -> a sync job creates a Trello task
* When lead status changes -> Trello card moves to correct list
* When Trello card moves -> Notion lead status updates
* Asynchronous background processing using Celery workers
* Automatic retry handling for failed sync operations
* No duplicates (idempotent)
* Uses timestamp comparison + small grace window
* Includes logging and fault-tolerant execution

# Architecture

Notion Leads
|
v
FastAPI
|
v
Redis Queue <----> Celery Workers
|
v
Trello Tasks

| Notion Status | Trello List |
| ------------- | ----------- |
| New           | To Do       |
| Contacted     | In Progress |
| Qualified     | Done        |
| Lost          | Lost        |

# Project Structure

* notion_client.py
* trello_client.py
* sync_logic.py
* tasks.py (Celery workers)
* main.py
* celery_worker.py
* .env.example
* requirements.txt

# Setup

## 1️⃣ Clone

git clone https://github.com/surryaansh/automation-two-way-sync-suryansh-singh

## 2️⃣ Notion Setup

* Create an Internal Integration at https://www.notion.so/my-integrations
* Copy token and share your database with it (from the Access Tab)
* Copy Notion database ID from the URL

## 3️⃣ Trello Setup

* Create Power-up and get API key: https://trello.com/power-ups/admin
* Generate token on same page (from the hyperlink “Token” on the right side of API Key)
* Get list IDs by opening board JSON (simply add “.json” at the end of the URL on boards webpage)

## 4️⃣ Create .env

* NOTION_TOKEN=xxx
* NOTION_DATABASE_ID=xxx
* TRELLO_KEY=xxx
* TRELLO_TOKEN=xxx
* TRELLO_BOARD_ID=xxx
* TRELLO_LIST_TODO=xxx
* TRELLO_LIST_INPROGRESS=xxx
* TRELLO_LIST_DONE=xxx
* TRELLO_LIST_LOST=xxx
* REDIS_URL=redis://localhost:6379/0

## 5️⃣ Install deps

pip install -r requirements.txt

# ▶️ Running the Sync

## Start Redis

redis-server

## Start Celery Worker

celery -A tasks worker --loglevel=info

## Start API

python3 main.py

### Example Output:

Running sync...
[Worker] Sync job received
[Decision] Trello Update is newer for 'Test_Name'
Moved card 'Test_Name' to list 'Qualified'
Sync completed successfully.

### You can test by:

* Creating a lead in Notion -> Trello card appears
* Changing status in Notion -> Trello card moves
* Dragging card in Trello -> Notion status updates

### Idempotent: running sync repeatedly does not duplicate cards.

# Error Handling & Idempotency

* Celery retry policies handle transient API failures
* Redis-backed task queues prevent request blocking
* Duplicate prevention using TrelloCardID stored in Notion
* Grace window avoids false conflicts
* Timestamp comparison decides which system is newer

# Assumptions & Limitations

* Polling-based synchronization
* Limited to main statuses
* External API rate limits may affect sync speed
* Small timestamp differences possible between tools

# AI Usage Notes

Used ChatGPT for:

* Understanding Notion and Trello API authentication flows
* Finding API keys, tokens, and board metadata
* Designing retry strategies and error handling approaches
* Reviewing timestamp conflict-resolution logic
* Drafting README structure and documentation

One suggestion I rejected: AI proposed adding additional workflow complexity and integrations; I kept the architecture focused on reliable synchronization between two systems.
