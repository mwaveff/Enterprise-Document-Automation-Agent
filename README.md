# 🤖 Contract Automation Agent (Gemini-Powered)

This project demonstrates a resilient, multi-stage workflow for **end-to-end contract analysis and data integration** using the **Google Gemini API**. The agent processes a scanned contract image, performs a rule-based risk assessment, and securely transforms the results into a structured, auditable JSON record.

---

## ✨ Key Features & Concepts

This agent leverages several advanced architectural concepts to ensure reliability, structure, and enterprise readiness:

| Concept | Component | Purpose |
| :--- | :--- | :--- |
| **Sessions & State Management** | `MemorySessionService` | Provides an in-memory store to **persist context** (raw analysis text) between the sequential LLM stages (Stage 1 → Stage 2), enabling stateful processing. |
| **Tools (Custom)** | `perform_risk_assessment` | A **rule-based expert tool** that the LLM utilizes to perform an objective, deterministic scan for critical legal risk keywords. |
| **A2A Protocol** | `send_data_to_external_system` | Implements the **Application-to-Application** data transfer protocol, securely sending the final JSON record to a simulated downstream system (e.g., ERP, Archive). |
| **Structured Output** | `OUTPUT_SCHEMA` | Enforces a **strict JSON contract** on the LLM's final output, making the analysis machine-readable and ready for integration. |
| **Sequential Workflow** | Stage 1 → Stage 2 | Defines a clear, two-step pipeline where the model first performs creative analysis and then is tasked with structured formatting. |

---

## 🏗️ Workflow Architecture

The contract is processed through a two-stage pipeline, ensuring separation of concerns:

### 1. 🚀 Stage 1: Analysis and Risk Scoring
1.  **Input:** The encrypted contract image (`contract01.png`) and the main instruction (`prompt_text`) are sent to the `gemini-2.0-flash` model.
2.  **Tool Use:** The LLM extracts clauses and calls the **`perform_risk_assessment`** tool to determine the objective risk level.
3.  **Output:** The model generates a comprehensive, narrative analysis text (`response1.text`). This raw text is saved to the **`MemorySessionService`**.

### 2. 🔄 Stage 2: Structuring and Finalization
1.  **Retrieval:** The Raw Analysis Text is retrieved from memory.
2.  **Conversion:** A second LLM call uses a specific prompt to **strictly convert** the raw text into the format defined by the **`OUTPUT_SCHEMA`**.
3.  **Finalization:** The resulting validated JSON is passed to **`populate_database`**, which adds an audit timestamp and triggers the **A2A transfer** via `send_data_to_external_system`. 

---

## 🛠️ Setup and Execution

### 1. Prerequisites

* Python 3.9+
* A valid **Gemini API Key**.
* The contract file (`contract01.png`) accessible at the defined path.

### 2. Configuration

* **API Key:** The script securely retrieves the Gemini API Key from **Kaggle Secrets** under the name `GOOGLE_API_KEY`.
* **External Config:** Ensure `EXTERNAL_API_URL` and `API_KEY` are configured for the A2A simulation.

### 3. Execution

The script runs sequentially, tracking file upload, session creation, both analysis stages, and the final A2A status. The final output provides a clean, color-coded summary of the analysis results.

### 4. Cleanup

The `finally` block ensures responsible resource management:
* Temporary data is cleared using **`MemorySessionService.delete_session`**.
* The uploaded file is removed from the Gemini API using **`client.files.delete`** to save storage.

---

## ⚙️ Custom Tools Overview

| Function Name | Role | Inputs | Outputs |
| :--- | :--- | :--- | :--- |
| `perform_risk_assessment` | **Risk Tool** | `List[str]` (Extracted Clauses) | `JSON` (risk_score, rationale) |
| `populate_database` | **Audit/Trigger Tool** | `JSON` (Final Structured Analysis) | `str` (SUCCESS/ERROR report) |
| `send_data_to_external_system` | **A2A Protocol** | `Dict` (Analyzed Data) | `Dict` (Simulated SUCCESS/ERROR status) |
