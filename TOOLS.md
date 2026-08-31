# 🧰 Tool Mapping (Nodes)



In the **N8N** framework, each node acts as a specific tool for the agent's architecture.

Scope Defender uses an explicit separation between **semantic processing, mathematical calculation, decision-making, and language generation**.

---

## 🔹 On Manual Click — Trigger



**Function:** initiates the controlled execution of the workflow.

Used for:

* Demonstrations.


* Testing.


* Validation of different branches.



---

## 🔹 Edit Fields — Set



**Function:** text injection tool.

It simulates the inbox by intercepting the manager's request.

It can provide:

* Requested demand.


* Deadline.


* Request context.


* Effort estimation.



---

## 🔹 Read/Write Files from Disk — Reading



**Function:** extraction tool.

Loads the team's capacity through a local JSON file.

The data can represent:

* Available capacity.


* Current allocation.


* Other variables required for the calculation.



---

## 🔹 Extract from File



**Function:** parses the file into properties that can be manipulated by the workflow.

Transforms structured content into data that can be used by subsequent nodes.

---

## 🔹 Code — JavaScript



**Function:** deterministic arithmetic engine.

Executes the exact calculation of the hours balance:

```text
saldo = capacidade - alocação

```

The calculation was deliberately removed from the LLM to prevent **mathematical hallucinations**, subtraction errors, or incorrect sign interpretations.

### Principle



> **LLM interprets. Code calculates.**
> 

The result produced by the Code Node serves as a reliable numerical source for the remainder of the workflow.

---

## 🔹 Switch — Deterministic Router



**Function:** routing based on explicit rules.

The Switch evaluates the calculated balance and divides the flow into three scenarios:

### 🔴 Unfeasible



```text
saldo < 0

```

Routes to the defensive communicator.

### 🟡 Partial



```text
saldo >= 0 && saldo < 8

```

Routes to the risk alert communicator.

### 🟢 Feasible



```text
saldo >= 8

```

Routes to the clean acceptance communicator.

The decision does not depend on the LLM.

---

## 🔹 Ollama — AI Communicators (3x)



**Function:** natural language generation with **Prompt Isolation**.

There are three independent Ollama nodes, each with a specific prompt.

### 1. Communicator — Unfeasible



**Objective:** scope defense.

Characteristics:

* Corporate tone.


* Reasoning based on numbers.


* Justified rejection of the deadline.


* Proposal for scope division when applicable.



---

### 2. Communicator — Partial



**Objective:** risk alert.

Characteristics:

* Acknowledges limited capacity exists.


* Communicates the risk associated with the deadline.


* Avoids an absolute refusal when partial delivery is possible.


* May propose feature prioritization.



---

### 3. Communicator — Feasible



**Objective:** clean acceptance.

Characteristics:

* Confirms feasibility.


* Maintains objective communication.


* Does not introduce non-existent risks.


* Does not create unnecessary justifications.



---

### Why three communicators?



A single generic prompt showed inconsistent behavior across scenarios.

**Prompt Isolation** solves this issue by restricting each agent to a single communicational role.

```text
                    SWITCH
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      UNFEASIBLE      PARTIAL       FEASIBLE
          │            │            │
          ▼            ▼            ▼
      Ollama #1      Ollama #2     Ollama #3
      Defensive        Alert       Acceptance

```

---

## 🔹 Write to File — Writing



**Function:** auditing tool.

Saves the generated email directly into `.txt` files, allowing the agent's trajectory to be recorded.

Expected files:

```text
/data/trajectory_01_inviavel.txt
/data/trajectory_02_parcial.txt
/data/trajectory_03_viavel.txt

```

These files allow subsequent verification of which response was produced for each scenario.

---

# 🧩 Architecture Summary



| Node | Function | Deterministic? |
| --- | --- | --- |
| **On Manual Click** | Trigger

 | Yes

 |
| **Edit Fields** | Input

 | Yes

 |
| **Read/Write Files** | Data reading

 | Yes

 |
| **Extract from File** | Parse

 | Yes

 |
| **Code** | Mathematical calculation

 | **Yes**<br> |
| **Switch** | Routing

 | **Yes**<br> |
| **Ollama #1** | Unfeasible communication

 | No

 |
| **Ollama #2** | Partial communication

 | No

 |
| **Ollama #3** | Feasible communication

 | No

 |
| **Write to File** | Auditing

 | Yes

 |

---

# 🎯 Architecture Principle



Scope Defender follows an explicit separation of responsibilities:

```text
┌─────────────────────────────────────┐
│            SEMANTIC LAYER           │
│                                     │
│                Ollama               │
│         Interpretation / Text       │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│            LOGICAL LAYER            │
│                                     │
│          Code Node + Switch         │
│        Calculation + Decision       │
└──────────────────┬──────────────────┘
                   │
                   ▼
┌─────────────────────────────────────┐
│             OUTPUT LAYER            │
│                                     │
│            Write to File            │
│               Auditing              │
└─────────────────────────────────────┘

```

The result is a more predictable, auditable architecture that is resistant to mathematical reasoning failures or mixed instructions between different scenarios.
