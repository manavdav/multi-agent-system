# Multi-Agent System
[![Ask DeepWiki](https://devin.ai/assets/askdeepwiki.png)](https://deepwiki.com/manavdav/multi-agent-system.git)

This repository contains a multi-agent AI system designed to handle complex user requests by breaking them down into smaller, manageable tasks and delegating them to specialized agents. The system is built using Python and FastAPI, providing a RESTful API for interaction.

## Architecture

The system is composed of several key components that work together to process user requests:

-   **API (`api/main.py`)**: A FastAPI application that serves as the entry point for all user interactions. It exposes endpoints for chat, as well as direct manipulation of tasks, events, and notes.
-   **Orchestrator (`core/orchestrator.py`)**: The central brain of the system. It receives incoming requests, uses a `PlannerAgent` to create a step-by-step execution plan, and then dispatches each step to the appropriate agent.
-   **Agents (`agents/`)**: Specialized components responsible for specific domains:
    -   **`PlannerAgent`**: Analyzes the user's natural language input to generate a sequence of actions for other agents to perform.
    -   **`TaskAgent`**: Manages to-do lists and tasks.
    -   **`SchedulerAgent`**: Handles calendar events and meetings.
    -   **`MemoryAgent`**: Responsible for storing and retrieving information and notes.
-   **Tools (`tools/`)**: Lower-level modules that provide concrete functionalities used by the agents, such as `CalendarTool`, `TaskTool`, and `NotesTool`.
-   **Memory Store (`db/memory_store.py`)**: A simple in-memory database to store state, including tasks, calendar events, notes, and user context. This serves as a non-persistent storage for demonstration purposes.
-   **Models (`models/schemas.py`)**: Pydantic schemas that define the data structures used throughout the application, ensuring type safety and clear data contracts.

## Features

-   **Natural Language Processing**: Understands user requests in plain English.
-   **Dynamic Planning**: Creates multi-step plans to fulfill complex requests.
-   **Task Management**: Create tasks from natural language.
-   **Scheduling**: Schedule events and meetings in a calendar.
-   **Information Storage**: Remember and recall information as notes.
-   **RESTful API**: Interact with the system via a clean and simple API.
-   **Containerized**: Includes a `Dockerfile` for easy deployment.

## Getting Started

Follow these instructions to get the application running on your local machine.

### Prerequisites

-   Python 3.9+
-   Pip

### 1. Clone the Repository

```bash
git clone https://github.com/manavdav/multi-agent-system.git
cd multi-agent-system
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Run the Application

Use Uvicorn to run the FastAPI server:

```bash
uvicorn api.main:app --host 0.0.0.0 --port 8080
```

The application will be available at `http://127.0.0.1:8080`.

### Using Docker

You can also run the application within a Docker container.

1.  **Build the Docker image:**
    ```bash
    docker build -t multi-agent-system .
    ```

2.  **Run the container:**
    ```bash
    docker run -p 8080:8080 multi-agent-system
    ```

## API Endpoints

The application exposes the following endpoints:

| Method | Endpoint         | Description                                        |
| :----- | :--------------- | :------------------------------------------------- |
| `GET`  | `/`              | Shows the service status and available agents.     |
| `POST` | `/chat`          | Main endpoint for interacting with the agent system. |
| `GET`  | `/tasks`         | Retrieves all tasks.                               |
| `POST` | `/tasks`         | Creates a new task.                                |
| `GET`  | `/schedule`      | Retrieves all calendar events.                     |
| `POST` | `/schedule`      | Creates a new calendar event.                      |
| `GET`  | `/notes`         | Retrieves all notes.                               |
| `POST` | `/notes`         | Creates a new note.                                |

### Example Usage

You can interact with the system using `curl` or any other API client.

#### Example 1: Create a task and schedule a meeting

```bash
curl -X POST "http://127.0.0.1:8080/chat" \
-H "Content-Type: application/json" \
-d '{
  "message": "Create a task to write the project summary and schedule a team meeting for tomorrow.",
  "user_id": "user123"
}'
```

**Expected Response:**

```json
{
    "response": "✓ ✅ Task 'write the project summary' created successfully\n✓ ✅ Event 'Team Meeting' scheduled for YYYY-MM-DD HH:MM",
    "actions_taken": [
        "{'agent': 'TaskAgent', 'action': 'create_task', 'result': ...}",
        "{'agent': 'SchedulerAgent', 'action': 'create_event', 'result': ...}"
    ],
    "data": {
        "plan": [
            {
                "step": 1,
                "agent": "TaskAgent",
                "action": "create_task",
                "description": "Create or manage tasks"
            },
            {
                "step": 2,
                "agent": "SchedulerAgent",
                "action": "schedule",
                "description": "Manage calendar"
            }
        ],
        "user_id": "user123"
    }
}
```

#### Example 2: Store and remember information

```bash
curl -X POST "http://127.0.0.1:8080/chat" \
-H "Content-Type: application/json" \
-d '{
  "message": "Remember that the project deadline is next Friday.",
  "user_id": "user123"
}'
```

**Expected Response:**

```json
{
    "response": "✓ Note 'Remembered Info' created",
    "actions_taken": [
        "{'agent': 'MemoryAgent', 'action': 'create_note', 'result': ...}"
    ],
    "data": {
        "plan": [
            {
                "step": 3,
                "agent": "MemoryAgent",
                "action": "store_info",
                "description": "Store information"
            }
        ],
        "user_id": "user123"
    }
}
