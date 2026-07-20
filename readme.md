# Asana Task Management Agent

A Streamlit chatbot that lets you manage your Asana workspace using natural language. It's powered by Google's Gemini model (via LangChain) with tool-calling, so the AI can create, list, update, and delete Asana tasks and projects on your behalf.

## Features

- 💬 Chat-based interface built with Streamlit
- 🤖 Gemini LLM with function/tool calling (via `langchain-google-genai`)
- ✅ Create tasks in a specific Asana project with a due date
- 📋 List all projects in your Asana workspace
- 📁 Create new Asana projects
- 🔎 Get all tasks within a given project
- ✏️ Update a task's completion status and/or due date
- 🗑️ Delete a task
- 🔁 Automatically loops the AI on tool calls (up to 5 nested calls) so it can chain multiple actions and then respond with a final answer

## How It Works

1. The user types a request in the Streamlit chat input (e.g. "Create a task called 'Write report' in the Marketing project due tomorrow").
2. The message is sent to the Gemini model along with a set of available tools (defined with LangChain's `@tool` decorator).
3. If the model decides it needs to call one or more tools (e.g. `get_asana_projects`, `create_asana_task`), the app executes the corresponding Python function against the Asana API.
4. Tool results are appended to the conversation, and the model is called again to produce a natural-language response.
5. The full conversation history is kept in Streamlit's session state so context persists across turns.

## Requirements

- Python 3.9+
- An [Asana account](https://asana.com/) with a Personal Access Token
- A [Google AI Studio / Gemini API key](https://ai.google.dev/)

### Python Dependencies

```
streamlit
python-dotenv
asana
langchain-core
langchain-google-genai
```

Install them with:

```bash
pip install streamlit python-dotenv asana langchain-core langchain-google-genai
```

## Setup

1. **Clone/download this project** and navigate into its directory.

2. **Create a `.env` file** in the project root with the following variables:

   ```env
   ASANA_ACCESS_TOKEN=your_asana_personal_access_token
   ASANA_WORKPLACE_ID=your_asana_workspace_gid
   GOOGLE_API_KEY=your_google_gemini_api_key
   LLM_MODEL=gemini-1.5-flash
   ```

   | Variable | Required | Description |
   |---|---|---|
   | `ASANA_ACCESS_TOKEN` | Yes | Personal Access Token generated from your Asana account settings (Apps → Manage Developer Apps → Create new token). |
   | `ASANA_WORKPLACE_ID` | Yes | The GID of your Asana workspace/organization. You can find this via the Asana API or URL when browsing your workspace. |
   | `GOOGLE_API_KEY` | Yes | API key for Gemini, required by `langchain-google-genai`. |
   | `LLM_MODEL` | No | Which Gemini model to use. Defaults to `gemini-1.5-flash` if not set. |

3. **Run the app:**

   ```bash
   streamlit run task-management-agent.py
   ```

4. Open the local URL Streamlit provides (usually `http://localhost:8501`) and start chatting.

## Available Tools (Functions the AI Can Call)

| Function | Purpose |
|---|---|
| `create_asana_task(task_name, project_gid, due_on)` | Creates a task in a given project. Defaults to today's date if no due date is given. |
| `get_asana_projects()` | Retrieves all non-archived projects in the configured workspace. |
| `create_asana_project(project_name, due_on)` | Creates a new project in the workspace. |
| `get_asana_tasks(project_gid)` | Retrieves all tasks in a given project (name, creation date, due date). |
| `update_asana_task(task_gid, data)` | Updates a task's `completed` status and/or `due_on` date. |
| `delete_task(task_gid)` | Deletes a task by its GID. |

The AI never exposes internal Asana IDs (GIDs) to the user — it uses them internally to track projects/tasks, and asks for clarification if it's unsure which project a task belongs to.

## Notes & Limitations

- The agent will make **at most 5 nested tool-calling round trips** per user message before raising an exception, to prevent infinite loops.
- If a project name is ambiguous or unspecified when creating a task, the AI is instructed to ask the user for clarification rather than guess.
- Error handling for Asana API calls returns the exception message as a string rather than raising, so failures show up as chat responses.
- This app stores conversation history only in Streamlit's session state — it is not persisted between app restarts.

## Project Structure

```
.
├── task-management-agent.py   # Main application (tools, AI prompting, Streamlit UI)
└── .env                        # Environment variables (not committed to version control)
```

## Security Note

Keep your `.env` file out of version control (add it to `.gitignore`). Your `ASANA_ACCESS_TOKEN` and `GOOGLE_API_KEY` grant access to your Asana workspace and Gemini API usage respectively.