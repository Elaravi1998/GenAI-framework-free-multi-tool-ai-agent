# 🤖 Framework-Free Multi-Tool AI Agent with OpenAI

A practical Python implementation of a **framework-free AI Agent** using the **OpenAI Responses API** and **multiple function/tool calling**.

This project demonstrates how an AI agent can receive a natural-language request, decide which tool to use, execute that tool, pass the result back to the LLM, and continue the agent loop until a final answer is generated.

> 🚀 **No LangChain, LangGraph, CrewAI, or other agent framework is used.**
> The agent loop is implemented directly in Python using the OpenAI SDK.

---

## 📌 Project Overview

This project demonstrates the fundamental architecture behind an AI Agent capable of using multiple tools.

The notebook starts with individual Python functions and progressively converts them into LLM-callable tools.

The implemented agent can work with tools such as:

* 👨‍💼 Employee lookup
* 🏖️ Leave balance calculation
* 🧮 Arithmetic calculation
* 📧 Simulated email preparation
* 📅 Simulated meeting booking

The project then combines these tools into a **Tool Registry** and implements an iterative **Agent Loop**.

---

## 🎯 Learning Objectives

By working through this notebook, you will learn how to:

* 🤖 Build an AI Agent without an agent framework
* 🔧 Define Python functions as LLM tools
* 🧠 Allow an LLM to decide which tool to call
* 🔄 Handle multiple tool calls
* 📦 Maintain a tool registry
* 🔁 Implement an agent execution loop
* 📨 Send tool results back to the LLM
* 🛡️ Validate requested tools before execution
* ⏱️ Limit the maximum number of agent iterations
* 🧩 Combine multiple tools to solve a single user request

---

## 🏗️ Architecture

The overall agent workflow implemented in this project is:

```text
                👤 User Input
                     │
                     ▼
              🧠 OpenAI LLM
                     │
                     ▼
             Does it need a tool?
                /          \
              No            Yes
              │              │
              ▼              ▼
        📝 Final Answer   🔧 Tool Call
                              │
                              ▼
                       🗂️ Tool Registry
                              │
                              ▼
                       ⚙️ Execute Tool
                              │
                              ▼
                       📦 Tool Result
                              │
                              ▼
                       🧠 Send Result
                         Back to LLM
                              │
                              ▼
                       🔄 Continue Loop
                              │
                              ▼
                        📝 Final Answer
```

---

# 🧰 Tools Implemented

The notebook defines five Python functions that are exposed to the LLM as tools.

## 1. 👨‍💼 Employee Lookup

```python
employee_lookup(employee_id)
```

Looks up an employee using an employee ID.

Example:

```text
Employee ID: 102
```

Example result:

```json
{
  "name": "Rajesh",
  "department": "Operations"
}
```

---

## 2. 🏖️ Leave Balance Calculator

```python
calculate_leave_balance(total_leaves, leaves_taken)
```

Calculates the remaining number of leaves.

Formula:

```text
Remaining Leaves = Total Leaves - Leaves Taken
```

Example:

```text
Total Leaves = 20
Leaves Taken = 5

Remaining Leaves = 15
```

---

## 3. 🧮 Calculator

```python
calculate(expression)
```

Performs basic arithmetic expressions.

Supported characters include:

```text
0-9
+
-
*
/
(
)
.
%
```

Example:

```python
calculate("10 + 20 * 2")
```

The function also performs validation and returns an error when an invalid expression is supplied.

---

## 4. 📧 Simulated Email

```python
send_email(to, subject, body)
```

Prepares a simulated email.

The notebook does **not actually send an email**.

Instead, it returns a simulated response such as:

```json
{
  "status": "simulated",
  "message": "Email prepared..."
}
```

---

## 5. 📅 Simulated Meeting Booking

```python
book_meeting(title, attendee, time)
```

Prepares a simulated meeting booking.

Like the email tool, this is a simulation and does not create a real calendar event.

---

# 🔧 OpenAI Tool Definitions

The Python functions are converted into the OpenAI SDK tool format.

Each tool contains information such as:

* Tool name
* Description
* Parameters
* Parameter types
* Required parameters
* Additional-property restrictions

Example structure:

```python
{
    "type": "function",
    "name": "employee_lookup",
    "description": "Look up an employee's name and department using an employee ID.",
    "parameters": {
        "type": "object",
        "properties": {
            "employee_id": {
                "type": "integer"
            }
        },
        "required": ["employee_id"],
        "additionalProperties": False
    }
}
```

This allows the LLM to understand what tools are available and what arguments each tool expects.

---

# 🧠 Tool Calling Example

A simple request such as:

```text
Who is employee 102?
```

is sent to the OpenAI Responses API along with the available tools.

The model can determine that the appropriate tool is:

```text
employee_lookup
```

with arguments similar to:

```json
{
  "employee_id": 102
}
```

The Python application then executes the corresponding function.

---

# 🗂️ Tool Registry

One of the important concepts demonstrated in this project is the **Tool Registry**.

```python
TOOL_REGISTRY = {
    "employee_lookup": employee_lookup,
    "calculate_leave_balance": calculate_leave_balance,
    "calculate": calculate,
    "send_email": send_email,
    "book_meeting": book_meeting
}
```

The registry maps the tool name returned by the LLM to the actual Python function.

For example:

```text
LLM requests:
employee_lookup

        ↓

Tool Registry

        ↓

employee_lookup(employee_id=102)
```

This provides a simple mechanism for dynamically executing different tools.

---

# 🤖 Building an AI Agent Without a Framework

A major concept demonstrated by this notebook is:

> **Can we create an AI Agent without using LangChain, LangGraph, CrewAI, or another agent framework?**

Yes.

The notebook implements the core agent architecture directly using Python and the OpenAI SDK.

The agent follows these **10 steps**:

### 1️⃣ Receive User Input

The agent receives a natural-language request.

### 2️⃣ Store Conversation

The user input is stored in the agent's input list.

### 3️⃣ Start Agent Loop

The agent begins an iterative execution loop.

### 4️⃣ Send Request to LLM

The current conversation state and available tools are sent to the OpenAI model.

### 5️⃣ Check for Tool Call

The application checks whether the model requested a function/tool call.

### 6️⃣ Save LLM Response

The model output is added to the conversation state.

### 7️⃣ Execute Selected Tool

The requested Python function is retrieved from the Tool Registry and executed.

### 8️⃣ Get Tool Result

The application receives the function result.

### 9️⃣ Send Tool Result Back to LLM

The tool output is added to the conversation as a function-call output.

### 🔟 Repeat

The agent continues until:

* The LLM produces a final answer, or
* The maximum number of agent steps is reached.

---

# 🔄 Agent Loop

The central function in the notebook is:

```python
def run_agent(user_input, max_steps=5):
```

The loop repeatedly sends the conversation to the LLM.

Conceptually:

```text
User Request
     │
     ▼
LLM
     │
     ├── Final Answer ───────────────► Return Answer
     │
     └── Tool Call
            │
            ▼
      Find Tool in Registry
            │
            ▼
       Execute Function
            │
            ▼
        Tool Result
            │
            ▼
      Send Result to LLM
            │
            └──────────────► Repeat
```

---

# 🧪 Example 1 — Employee Lookup

The notebook runs:

```python
run_agent(
    "Find employee 102 and tell me their name and department."
)
```

The agent can:

1. Understand the request.
2. Select `employee_lookup`.
3. Extract employee ID `102`.
4. Execute the Python function.
5. Receive the employee information.
6. Return the result to the LLM.
7. Produce the final response.

---

# 🧪 Example 2 — Multiple Tool Usage

The notebook also demonstrates a more complex request:

```text
Find employee 103. Then prepare a simulated email to HR that mentions
the employee's name and department and asks HR to confirm the employee's
leave balance.
```

This request requires multiple steps.

Conceptually:

```text
User Request
     │
     ▼
employee_lookup
     │
     ▼
Employee Name + Department
     │
     ▼
send_email
     │
     ▼
Simulated Email Result
     │
     ▼
Final LLM Response
```

This demonstrates how an agent can use the output of one tool as context for a subsequent tool call.

---

# 🛡️ Tool Validation

The agent checks whether the requested tool exists in the Tool Registry:

```python
if tool_call.name not in TOOL_REGISTRY:
    raise ValueError(
        f"Unknown tool requested: {tool_call.name}"
    )
```

This prevents the application from blindly attempting to execute an unknown function.

---

# ⏱️ Maximum Agent Steps

The agent uses:

```python
max_steps=5
```

This prevents an uncontrolled infinite tool-calling loop.

If the maximum number of iterations is reached, the function returns:

```text
Maximum agent steps reached.
```

This is an important basic safeguard when implementing an agent loop manually.

---

# 🔐 API Key Configuration

The notebook obtains the OpenAI API key using an environment variable.

If the key is not already configured, it prompts the user:

```python
if not os.environ.get("OPENAI_API_KEY"):
    os.environ["OPENAI_API_KEY"] = getpass(
        "Enter your OpenAI API key: "
    )
```

The OpenAI client is then initialized:

```python
client = OpenAI(
    api_key=os.environ["OPENAI_API_KEY"]
)
```

The notebook configures the generation model as:

```python
GENERATION_MODEL = "gpt-4.1-mini"
```

---

# 📦 Requirements

The notebook requires Python and the OpenAI Python SDK.

Install the OpenAI SDK:

```bash
pip install openai
```

Recommended:

```bash
python -m pip install --upgrade openai
```

---

# ⚙️ Setup

## 1️⃣ Clone the Repository

```bash
git clone https://github.com/<your-username>/framework-free-multi-tool-ai-agent.git
```

```bash
cd framework-free-multi-tool-ai-agent
```

---

## 2️⃣ Install Dependencies

```bash
pip install openai
```

---

## 3️⃣ Configure OpenAI API Key

### Windows PowerShell

```powershell
$env:OPENAI_API_KEY="your-api-key"
```

### Linux / macOS

```bash
export OPENAI_API_KEY="your-api-key"
```

Alternatively, the notebook can prompt for the API key using `getpass()` when the environment variable is not available.

⚠️ **Never commit your API key to GitHub.**

---

# ▶️ Running the Notebook

Open the notebook using Jupyter:

```bash
jupyter notebook
```

Or use JupyterLab:

```bash
jupyter lab
```

You can also open the notebook in Google Colab.

Then execute the notebook cells sequentially.

---

# 📁 Project Structure

A suggested repository structure is:

```text
framework-free-multi-tool-ai-agent/
│
├── Multiple_Tool_Calling_Vs_1_AI_Agent_Script.ipynb
├── README.md
├── requirements.txt
└── .gitignore
```

---

# 📄 requirements.txt

You can create a `requirements.txt` file containing:

```text
openai
```

Then install dependencies with:

```bash
pip install -r requirements.txt
```

---

# 🚫 .gitignore

Create a `.gitignore` file:

```gitignore
# Python
__pycache__/
*.py[cod]

# Virtual Environment
.venv/
venv/
env/

# Environment Variables
.env

# Jupyter
.ipynb_checkpoints/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
```

---

# 🧩 Technologies Used

| Technology          | Purpose                                |
| ------------------- | -------------------------------------- |
| 🐍 Python           | Agent implementation                   |
| 🧠 OpenAI API       | LLM reasoning and tool selection       |
| 🔧 Function Calling | Connecting the LLM with Python tools   |
| 📓 Jupyter Notebook | Development and experimentation        |
| 🔄 Agent Loop       | Iterative tool execution               |
| 🗂️ Tool Registry   | Mapping tool names to Python functions |

---

# 🆚 Framework-Free Agent vs Agent Framework

This project intentionally implements the agent loop directly.

Instead of using:

```text
LangChain
LangGraph
CrewAI
AutoGen
```

the notebook uses:

```text
Python
   +
OpenAI SDK
   +
Tool Definitions
   +
Tool Registry
   +
Agent Loop
```

This makes the underlying mechanics of an AI Agent easier to understand before moving to higher-level frameworks.

---

# 💡 Key Concepts Demonstrated

### 🔹 Function Calling

The LLM can request a specific Python function with structured arguments.

### 🔹 Tool Registry

A dictionary maps tool names to executable Python functions.

### 🔹 Tool Execution

The application executes the requested function.

### 🔹 Tool Result Injection

The function result is passed back to the LLM.

### 🔹 Iterative Reasoning Loop

The agent can repeatedly call tools until it reaches a final answer.

### 🔹 Multi-Tool Workflows

One user request can require multiple tools.

### 🔹 Framework-Free Architecture

The fundamental mechanics of an agent can be implemented without an external agent framework.

---

# 🎓 Interview Preparation

This project is also useful for understanding common AI Engineer / Generative AI interview questions.

### ❓ Can an AI Agent be created without LangChain?

Yes. An agent can be implemented directly using an LLM API, tool definitions, tool execution logic, conversation state, and an iterative loop.

### ❓ What is a Tool Registry?

A mapping between tool names and the actual functions that should execute when the LLM requests those tools.

### ❓ How does an agent know which function to execute?

The LLM selects a tool based on the available tool definitions and generates the required arguments.

### ❓ Why do we need an agent loop?

Because a tool result may provide information required for another tool call or for generating the final response.

### ❓ Why use `max_steps`?

To prevent an uncontrolled or infinite sequence of tool calls.

---

# 🚀 Possible Future Enhancements

This notebook provides a foundational implementation. It can be extended with:

* 🔐 Better authentication and secret management
* 🧠 Conversation memory
* 📝 Persistent conversation history
* 🛡️ More robust input validation
* 🔄 Retry mechanisms
* ⏱️ Tool execution timeouts
* 📊 Agent execution tracing
* 🧪 Unit tests for tools
* 🌐 REST API integration
* 💬 Streamlit chat interface
* 🗄️ Database-backed tool state
* 🔌 MCP-based tools
* 🧩 More advanced multi-agent workflows
* 📈 Observability and monitoring
* 🏗️ Production-grade error handling

---

# ⚠️ Important Notes

### 🔒 API Key Security

Do not place your OpenAI API key directly inside the notebook before uploading it to GitHub.

Use an environment variable or another secure secret-management mechanism.

### 📧 Email Tool

The email function in this project is **simulated**. It does not actually send an email.

### 📅 Meeting Tool

The meeting function is also **simulated**. It does not create an actual calendar event.

### 🧮 Calculator

The calculator is intentionally limited to basic arithmetic expressions and validates allowed characters before evaluation.

---

# 📚 What This Project Teaches

The main purpose of this project is not simply to demonstrate OpenAI API usage.

It demonstrates the fundamental architecture behind an AI Agent:

```text
                LLM
                 │
          ┌──────┴──────┐
          │             │
      Reasoning      Tool Selection
                        │
                        ▼
                  Tool Registry
                        │
                        ▼
                  Tool Execution
                        │
                        ▼
                   Tool Result
                        │
                        ▼
                       LLM
                        │
                        ▼
                  Final Response
```

Understanding this architecture makes it easier to understand more advanced frameworks and agent systems later.

---

# 🌟 Project Highlights

* 🤖 Framework-free AI Agent
* 🔧 Multiple Python tools
* 🧠 OpenAI tool calling
* 🗂️ Tool registry architecture
* 🔄 Iterative agent loop
* 📦 Structured tool arguments
* 🛡️ Unknown-tool validation
* ⏱️ Maximum-step protection
* 📧 Simulated email workflow
* 📅 Simulated meeting workflow
* 👨‍💼 Employee information workflow
* 🏖️ Leave calculation workflow

---

# 👨‍💻 Author

Developed as a practical learning project for understanding:

**Generative AI • LLMs • Function Calling • AI Agents • Tool Calling • Agent Architecture • Python**

---

# ⭐ If You Find This Useful

If this project helps you understand AI Agents and tool calling, consider giving the repository a ⭐ on GitHub.

---

## 📌 Repository Name

Recommended repository name:

```text
framework-free-multi-tool-ai-agent
```

### Suggested GitHub description

```text
🤖 A framework-free Python AI Agent demonstrating OpenAI tool calling, multiple tools, tool registry, and iterative agent execution.
```

### Suggested topics

```text
ai-agent
ai-agents
openai
openai-api
function-calling
tool-calling
llm
generative-ai
genai
python
jupyter-notebook
llm-agents
agentic-ai
```
