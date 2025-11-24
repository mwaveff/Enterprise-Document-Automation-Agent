# 🤖 Contract Automation and Data Integration Agent

**Project:** Enterprise-Document-Automation-Agent

This project demonstrates a **multi-stage workflow** for automating contract analysis. Using **Google Gemini API**, the agent converts an unstructured document (contract image) into a verified, structured JSON record ready for integration into enterprise systems.

---

## ✨ Key Architectural Concepts (Advanced Concepts)

This agent is built on modern architectural patterns, clearly demonstrating the fulfillment of the following requirements:

| Concept | Component | Purpose |
| :--- | :--- | :--- |
| **Sessions & State Management** | `MemorySessionService` | **In-Memory Store** for storing context (`response1.text`) between sequential LLM calls (Sequential Agents), ensuring workflow continuity. |
| **A2A Protocol** | `send_data_to_external_system` | Implementation of the **Application-to-Application** protocol for secure transmission of the final JSON to external systems (using Bearer Token). |
| **Tools (Custom)** | `perform_risk_assessment` | **Rule-Based Expert System** (Expert tool) — performs an objective risk assessment based on fixed legal rules (keyword search). |
| **LLM Agent Tools** | `populate_database` | Used as a **“Commit”** tool that validates the final output and triggers A2A transfer. |
| **Sequential Agents** | Stage 1 → Stage 2 Workflow | The process is structured as a clear **conveyor belt**, where the output of one stage is strictly controlled input for the next. |

---

## 🚀 Workflow Breakdown

The agent performs a two-step, controlled process:

### Stage 1: Analysis, Tool Execution, and Context Preservation

1.  **Initialization:** A connection is established (`genai.Client`), authentication is performed, the contract file is uploaded (`client.files.upload()`), and **`MemorySessionService`** (`SESSION_ID`) is created.
2.  **LLM Call (Analysis):** The `gemini-2.0-flash` model reads the file and accesses the **`perform_risk_assessment`** tool.
3.  **Tool Execution:** LLM calls the tool, obtains a deterministic `risk_score`, and includes it in its raw narrative report.
4.  **State Saving:** The complete narrative text (`response1.text`) is saved in the session (`MemorySessionService.save_data()`).

### Stage 2: Structuring, Validation, and Integration

1.  **Retrieving State:** The narrative text is retrieved from memory.
2.  **LLM Call (Structuring):** A second LLM call is tasked with strictly converting the raw text to JSON, adhering to **`response_schema=OUTPUT_SCHEMA`**.
3.  **A2A Trigger:** The resulting, validated JSON is passed to the **`populate_database`** function, which adds an audit timestamp and triggers the final **A2A transfer** to the external system.

---

## 💻 Step-by-Step Instructions

### 1. 📦 Installing Dependencies and Configuration

* **Installation:** Install the necessary libraries: `pip install google-genai requests kaggle-secrets`.
* **Authentication:** Make sure your `GOOGLE_API_KEY` is added to the environment secrets.

### 2. 🚀 Workflow Execution

| Phase | Action | Expected Output/Result |
| :--- | :--- | :--- |
| **Setup** | Initial cell execution (`client = genai.Client()`). | `✅ File successfully uploaded: files/...` <br> `✅ Session created with ID: ...` |
| **Stage 1 Execution** | Launch the first `generate_content` (Analysis). | The model uses `perform_risk_assessment` and generates raw text. |
| **Stage 2 Execution** | Launch the second `generate_content` (Structuring). | The model returns structured JSON. <br> **Key Action:** Triggers `populate_database` and A2A transfer. |

### 3. ✅ Final Report and Cleanup

* **Final Report:** The script outputs a structured, color-coded report on risk, metadata, and A2A transfer status.
* **Cleanup (`finally` block):**
* **`MemorySessionService.delete_session`** (Memory cleanup).
    * **`client.files.delete`** (Deleting a file from the Gemini File API) – mandatory procedure.

---

## ⚙️ Detailed Overview of Tools

| Function Name | Role | Key Actions |
| :--- | :--- | :--- |
| `MemorySessionService` | **State Manager** | Stores and retrieves context between sequential calls. |
| `perform_risk_assessment` | **Expert Tool** | Performs a static search for critical legal terms and returns a rating. |
| `populate_database` | **A2A Trigger Tool** | Validates the final JSON and adds a timestamp for auditing. |
| `send_data_to_external_system` | **A2A Protocol** | Formats the data and makes an authenticated call to an external service. |
