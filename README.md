# 🤖 AI-Powered Database Profiler

This project is a real-time, AI-driven observability tool that automatically detects slow SQL queries from a PostgreSQL database, analyzes them using the Gemini generative AI model, and provides actionable recommendations for optimization in a clean web interface.

It actively monitors database logs, captures queries that exceed a time threshold, and presents them one at a time to a database administrator (DBA) for review and resolution.

## 🎥 Live Demo

Below is a brief demonstration of the AI Database Profiler in action, from detecting a slow query to displaying the AI-powered recommendation.

[![Project Demo](https://github.com/Ronitb26/Database-Profiler/blob/main/Demo.gif)

---

## ✨ Core Features

-   **Real-Time Monitoring**: Actively watches PostgreSQL log files to detect slow queries as they happen.
-   **Intelligent Analysis**: Sends slow queries, execution metrics, and table schemas to a Python-based AI service for in-depth analysis.
-   **Actionable Recommendations**: The AI provides a simple summary, a clear recommendation (like creating an index), and an optimized version of the query.
-   **Stateful Issue Tracking**: Uses a PostgreSQL database to track all identified issues and their status (unsolved, solved), preventing duplicate alerts.
-   **Interactive Queue UI**: A clean, web-based frontend that displays one unsolved issue at a time, allowing an administrator to focus and resolve problems sequentially.
-   **Microservice Architecture**: Built with a Node.js server that acts as a manager/orchestrator and a separate Python (FastAPI) server that serves as the AI brain.

---

## ⚙️ How It Works

1.  **Detection (Node.js)**: The Node.js server monitors the PostgreSQL log directory. When a query exceeds a predefined time threshold, it's identified as an issue.
2.  **Enrichment (Node.js)**: The server parses the query to identify the table involved and fetches its schema from the database.
3.  **Delegation (Node.js → Python)**: The query, execution time, and schema are sent via an API call to the Python FastAPI server.
4.  **Analysis (Python)**: The Python server formats the data into a prompt and sends it to the Gemini AI model, requesting a JSON response containing a summary, recommendation, and optimized query.
5.  **Storage (Node.js → PostgreSQL)**: The Node.js server receives the AI's analysis and stores the complete issue in a dedicated `issues` table in its own database.
6.  **Notification (WebSocket)**: The Node.js server broadcasts a WebSocket signal to all connected web clients, alerting them that a new issue is available.
7.  **Display (Frontend)**: The frontend receives the signal, fetches the most recent unsolved issue, and displays it.
8.  **Resolution**: When the user clicks "Mark as Solved," the backend updates the issue's status and triggers the frontend to fetch the next unsolved issue.

---

## 🛠️ Tech Stack

-   **Backend (Manager)**: Node.js, Express.js, WebSocket (`ws`), Axios, `node-sql-parser`, `pg`
-   **Backend (AI Brain)**: Python, FastAPI, Google Generative AI (`gemini`)
-   **Database**: PostgreSQL
-   **Frontend**: HTML, CSS, JavaScript (Fetch API & WebSocket API)

---

## 🚀 Getting Started

### Prerequisites

-   Node.js and npm
-   Python and pip
-   A running PostgreSQL instance
-   A Google AI (Gemini) API Key

### 1. Configure Your Target PostgreSQL Database

For the profiler to detect slow queries, you must configure your target database instance to log them. Connect to your PostgreSQL server and run the following commands. This tells PostgreSQL to log any statement that takes longer than 1 second (1000ms).

```sql
ALTER SYSTEM SET log_min_duration_statement = '1000';
SELECT pg_reload_conf();
```

You will also need to know the path to your PostgreSQL log directory.

### 2. Setup the Project

Clone this repository to your local machine:

```bash
git clone <your-repo-url>
cd <your-repo-name>
```

**A. AI Brain Setup (Python Service)**

1.  Navigate to the directory containing `main.py`.
2.  Install the required Python packages:
    ```bash
    pip install "fastapi[all]" google-generativeai python-dotenv
    ```
3.  Create a file named `.env` in the same directory and add your Google API key:
    ```
    GOOGLE_API_KEY="YOUR_GEMINI_API_KEY_HERE"
    ```

**B. Manager Setup (Node.js Service)**

1.  Navigate to the directory containing `server.js`.
2.  Install the required Node.js packages:
    ```bash
    npm install express cors ws pg axios node-sql-parser dotenv
    ```
3.  **Create the Issues Database**:
    -   Create a new PostgreSQL database (e.g., `db_profiler`).
    -   Connect to this new database and run the following SQL to create the table for tracking issues:
        ```sql
        CREATE TABLE issues (
            id SERIAL PRIMARY KEY,
            query_hash VARCHAR(32) UNIQUE NOT NULL,
            query_text TEXT NOT NULL,
            status VARCHAR(10) NOT NULL DEFAULT 'unsolved',
            execution_time_sec NUMERIC,
            summary TEXT,
            recommendation TEXT,
            optimized_query TEXT,
            first_seen TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
            last_seen TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP
        );
        ```
4.  **Configure Environment Variables**:
    -   Create a file named `.env` in the Node.js directory and add your database password.
        ```
        PASSWORD="YOUR_POSTGRES_PASSWORD"
        ```
    -   Inside `server.js`, update the `dbConfig` object with your database credentials and update `logDirectoryPath` with the correct path to your PostgreSQL logs.

### 3. Running the System

You must start both backend services in separate terminals.

**Terminal 1: Start the AI Brain**

```bash
# In your Python directory
uvicorn main:app --reload
```
This will start the FastAPI server, typically on `http://localhost:8000`.

**Terminal 2: Start the Node.js Manager**

```bash
# In your Node.js directory
node server.js
```
This will start the orchestrator, which begins watching the log files.

**Finally, open the `index.html` file in your web browser.**

Now, run slow queries on your target PostgreSQL database and watch them appear in the dashboard for analysis!

