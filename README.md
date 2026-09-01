# 🛡️ Scope Defender

**Scope Defender** is an AI orchestrator developed to assist engineering professionals and teams in negotiating deadlines and scopes for urgent demands.

**[🎥 Click here to watch the Demo Video](https://youtu.be/LuyZAzXFsKc)**

## 🎯 The Problem

Engineering teams frequently accept unrealistic deadlines under pressure because they lack sufficient time to calculate their technical capacity at the moment of the request.

This turns a decision that should be data-driven into a negotiation based on pressure, mental estimates, and subjective perception.

## 💡 The Solution

**Scope Defender** uses **N8N + local LLMs** to intercept an urgent demand, cross-reference it with the team's current capacity, and generate a corporate response grounded in concrete data.

The architecture clearly separates responsibilities:

* **LLM:** Semantic interpretation and language generation.
* **N8N:** Process orchestration.
* **Code Node:** Deterministic mathematical calculations.
* **Switch:** Rule-based decision making.
* **Ollama:** Local inference.
* **Write to File:** Trajectory logging for auditing.

---

## 🏗️ Workflow Architecture

<img width="803" height="503" alt="image" src="https://github.com/user-attachments/assets/08963cc1-1081-42f4-802b-26ca07e66970" />


The system operates on a deterministic architecture, separating **semantic reasoning** from **logical execution**:

### 1. Input

Receives the manager's request email and extracts structured team data.

Data used by the workflow includes information such as:

* Available capacity.
* Current allocation.
* Estimated effort for the new demand.
* Requested deadline.

### 2. Native Calculation — Code Node

A simple JavaScript script processes the actual math:

```text
capacidade - alocação = saldo

```

The calculation is executed directly by N8N, eliminating the risk of numerical hallucination by the AI.

The AI is not responsible for performing mathematical operations.

### 3. Deterministic Switch — N8N

The Switch evaluates the calculated balance and determines which scenario should be executed.

The three conditions are:

```text
saldo < 0

```

**INFEASIBLE**

```text
saldo >= 0 && saldo < 8

```

**PARTIAL**

```text
saldo >= 8

```

**FEASIBLE**

This way, routing is handled by deterministic rules, without relying on the subjective interpretation of the LLM.

### 4. Isolated Communicator Agents — Ollama

After the Switch, the workflow has **three independent Ollama nodes**.

Each agent has a specific prompt restricted to its scenario:

| Scenario | Agent | Behavior |
| --- | --- | --- |
| 🔴 Infeasible | Defensive Communicator | Scope defense and justification based on the deficit |
| 🟡 Partial | Alert Communicator | Conditional acceptance and explicit statement of risks |
| 🟢 Feasible | Acceptance Communicator | Clean and objective acceptance |

This architecture implements **Prompt Isolation**, preventing a single generic prompt from attempting to simultaneously interpret different business scenarios.

Variables calculated by the workflow are injected directly into the prompts, keeping numbers outside the model's responsibility.

### 5. Trajectory Logging

Each branch logs its output to a separate `.txt` file to allow for auditing and subsequent execution analysis.

Files are stored in:

```text
/data/

```

---

## 🧠 Engineering Principle

One of the main architectural principles of Scope Defender is:

> **The AI interprets and communicates. The code calculates. The workflow decides.**

The flow can be represented as:

```text
                    ┌─────────────────┐
                    │      INPUT      │
                    │ Demand + Data   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   CODE NODE     │
                    │ Exact math      │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │     SWITCH      │
                    │ Balance rules   │
                    └─────┬───┬───┬───┘
                          │   │   │
                 < 0 ─────┘   │   └───── >= 8
                          0–<8│
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
       ┌────────────┐  ┌────────────┐  ┌────────────┐
       │  OLLAMA #1 │  │  OLLAMA #2 │  │  OLLAMA #3 │
       │ INFEASIBLE │  │   PARTIAL  │  │  FEASIBLE  │
       └──────┬─────┘  └──────┬─────┘  └──────┬─────┘
              │               │               │
              └───────────────┼───────────────┘
                              ▼
                    ┌─────────────────┐
                    │  WRITE TO FILE  │
                    │   Trajectory    │
                    └─────────────────┘

```

---

## ⚙️ Prerequisites

To run the project locally:

* Docker
* Docker Compose
* N8N
* Ollama
* NVIDIA Container Toolkit
* NVIDIA GPU compatible with CUDA acceleration

### Used Ports

| Service | Port |
| --- | --- |
| N8N | `5678` |
| Ollama | `11434` |

---

## 🖥️ Reference Hardware

The project was developed, tested, and locally optimized using:

| Component | Specification |
| --- | --- |
| CPU | AMD Ryzen 7 5700X |
| GPU | NVIDIA RTX 3070 8 GB |
| RAM | 16 GB |
| Inference | Ollama + NVIDIA GPU |
| Model | Llama 3 8B |

The GPU accelerates local model inference, allowing workflow processing in a few seconds under adequate execution conditions.

---

# 🚀 Reproduction Guide

## 1. Clone the repository

```bash
git clone https://github.com/kanashinho/ScopeDefender.git
cd ScopeDefender
cp .env.example .env

```

Configure the `.env` file as needed.

---

## 2. Spin up the infrastructure

```bash
docker-compose up -d

```

Check the containers:

```bash
docker ps

```

---

## 3. Set up the AI

On the first run, the Ollama server will have no models.

Download the model:

```bash
docker exec -it ollama_scope_defender ollama run llama3

```

Wait for the download to finish.

When it displays:

```text
>>>

```

execute:

```text
/bye

```

and press `Enter`.

---

## 4. Import the Workflows

Access:

```text
http://localhost:5678

```

Then:

1. Go to **Workflows**.
2. Click on **Add Workflow**.
3. Open the options menu.
4. Select **Import from File**.
5. Import:

```text
baseline_workflow.json
scope_defender_workflow.json

```

---

## 5. Test the System

Open the **Scope Defender** flow, click the manual trigger node, and execute the workflow.

The flow will automatically log the outputs in the `/data/` folder as:

```text
trajectory_01_inviavel.txt
trajectory_02_parcial.txt
trajectory_03_viavel.txt

```

Each file represents the execution trajectory corresponding to the processed scenario.

---

# 📥 Input and 📤 Output

## Input

Example of data processed by the workflow:

```json
{
  "disponivel": -40,
  "exigido": 20
}

```

The **Code Node** performs the calculation:

```text
saldo = disponível - exigido
saldo = -40 - 20
saldo = -60

```

Result:

```json
{
  "disponivel": -40,
  "exigido": 20,
  "saldo": -60
}

```

---

## Output

Since the balance is less than zero, the Switch routes the execution to the **INFEASIBLE** communicator.

Example:

> Hello, Manager,
> I regret to inform you that I cannot meet today's 5 PM deadline for the new demand. The team is already working with a -40h deficit and now needs to absorb an additional 20h. This would result in a total deficit of -60h, significantly compromising the quality and security of the delivery.
> To resolve this realistically, I suggest dividing the scope into two stages: Phase 1, covering the essential features, and Phase 2, covering the remaining integrations and complementary features.

---

# 📊 Improvement Metrics

| Metric | Manual Baseline | Scope Defender |
| --- | --- | --- |
| **Analysis time** | ~25 min | **< 10 seconds** |
| **Calculation accuracy** | Mental estimate under pressure | **100% via Code Node** |
| **Argumentation used** | No concrete numbers | **Automatically calculated balance** |
| **Proposed alternative** | Rarely elaborated under stress | **Generated based on scenario** |
| **Tone control** | Generic prompt | **3 isolated prompts** |
| **Routing** | Dependent on AI interpretation | **Deterministic via Switch** |
| **Auditing** | Non-existent/manual | **`.txt` trajectories** |

> The metrics represent the reference scenario used during the project's development and testing.

---

# 🔐 Privacy and Local Processing

Scope Defender uses **Ollama** to run models locally.

The architecture reduces the need to send internal team data to external language model APIs.

Processing can remain entirely within the local environment:

* Team data.
* LLM inference.
* N8N workflows.
* Generated results.
* Execution trajectories.

---

# 🛠️ AI Disclosure

This project uses:

* **Orchestration:** N8N via Docker
* **Inference engine:** Ollama
* **Acceleration:** NVIDIA Container Toolkit
* **Model:** Llama 3 8B
* **Architecture:** Communicator agents + deterministic rules
* **Calculations:** JavaScript via N8N Code Node
* **Auditing:** `.txt` files

---

# 📁 Project Structure

```text
ScopeDefender/
│
├── README.md
├── TOOLS.md
├── CHANGELOG.md
├── docker-compose.yml
├── .env
├── .gitignore
├──  baseline_workflow.json
├──  scope_defender_workflow.json
│
├── data/
│   ├── tasks.json
│   ├── tasks.csv
|
├── trajectories/
│   ├── .gitkeep

```

